# Vì Sao Doanh Nghiệp Dùng LLM Để Phân Tích

## 1. LLM Không Thay Thế Phân Tích Truyền Thống

LLM không nên được xem là công cụ thay thế SQL, BI dashboard hoặc data mining
truyền thống.

Vai trò hợp lý hơn:

```text
BI/SQL/Data Mining: tính toán, thống kê, đo lường, khai phá cấu trúc
LLM: đọc hiểu, tổng hợp, giải thích, đối thoại và suy luận trên context
```

Do đó, LLM có giá trị khi nó đứng trên một lớp dữ liệu đã được tổ chức tốt:

```text
Lakehouse -> Knowledge Graph -> Retrieval Context -> LLM Analysis
```

## 2. Khi Nào Phân Tích Bình Thường Là Đủ

Phân tích truyền thống phù hợp với câu hỏi có cấu trúc rõ:

- số ticket theo tháng;
- số issue theo project;
- thời gian xử lý trung bình;
- số comment theo issue;
- top component có nhiều lỗi;
- reopen rate theo project/component.

Các câu hỏi này có thể giải quyết tốt bằng SQL, dashboard hoặc graph query.

## 3. Khi Nào Cần LLM

LLM hữu ích khi dữ liệu có nhiều ngôn ngữ tự nhiên, ngữ cảnh rời rạc và câu hỏi
khó định nghĩa trước.

Ví dụ:

- Issue này giống những issue cũ nào dù label khác nhau?
- Trong các comment cũ, có dấu hiệu nào nói về nguyên nhân gốc không?
- Tóm tắt diễn biến xử lý issue này cho developer mới.
- Vì sao nhóm issue này thường bị reopen?
- Các hướng xử lý cũ có mâu thuẫn với nhau không?
- Nếu gặp lỗi tương tự, nên đọc issue/comment/resolution nào trước?

Các câu hỏi này cần kết hợp:

- text understanding;
- retrieval;
- quan hệ graph;
- timeline sự kiện;
- tổng hợp câu trả lời bằng ngôn ngữ tự nhiên.

## 4. Giá Trị Kinh Doanh

Doanh nghiệp có thể trả tiền cho LLM nếu nó tạo ra lợi ích đo được:

| Giá trị | Cách đo gợi ý |
| --- | --- |
| Giảm thời gian đọc ticket/log/comment | phút tiết kiệm trên mỗi case |
| Tìm issue tương tự nhanh hơn | thời gian truy xuất trước/sau |
| Giảm lặp lại xử lý thủ công | số case dùng lại knowledge cũ |
| Phát hiện bottleneck nhanh hơn | số bottleneck được phát hiện |
| Hỗ trợ onboarding | thời gian nhân sự mới hiểu issue/context |
| Tăng chất lượng quyết định | tỷ lệ câu trả lời có evidence đúng |

Điểm mấu chốt:

```text
LLM có ROI khi chi phí dùng LLM nhỏ hơn chi phí con người đọc, tổng hợp và
phân tích dữ liệu vận hành thủ công.
```

## 5. Vì Sao Memory Backend Là Điều Kiện Quan Trọng

Nếu chỉ đưa prompt trực tiếp cho LLM, hệ thống dễ gặp các vấn đề:

- câu trả lời phụ thuộc vào context người dùng paste vào;
- LLM không biết lịch sử xử lý;
- không có nguồn kiểm chứng;
- dễ hallucinate khi thiếu dữ liệu;
- khó đánh giá đúng/sai;
- không tận dụng được dữ liệu vận hành đã có.

Memory backend giúp khắc phục bằng cách:

- chọn dữ liệu liên quan trước khi gọi LLM;
- đưa evidence/source vào prompt;
- giảm context thừa;
- liên kết fact với episode và issue;
- tạo trace để kiểm tra LLM đã dùng gì;
- tạo nền cho evaluation.

## 6. Độ Tin Cậy Về Dữ Liệu

Không nên để LLM tự quyết định mọi thứ. Hệ thống cần grounding:

```text
LLM answer
  -> dựa trên retrieved context
  -> context có source id
  -> source id truy về lakehouse/graph
  -> trace ghi lại quá trình retrieval
```

Cơ chế tin cậy cần có:

- source/citation cho mỗi kết luận quan trọng;
- confidence hoặc cảnh báo khi thiếu dữ liệu;
- không trả lời chắc chắn nếu context không đủ;
- so sánh câu trả lời với dữ liệu gốc;
- bộ câu hỏi đánh giá định kỳ;
- human review cho quyết định ảnh hưởng lớn.

## 7. Độ Tin Cậy Về Bảo Mật

Với doanh nghiệp, rủi ro lớn không chỉ là sai kết quả mà còn là lộ dữ liệu.

Thiết kế an toàn nên gồm:

- chạy local/on-prem nếu dữ liệu nhạy cảm;
- chỉ retrieve phần dữ liệu cần thiết, không đưa toàn bộ database vào prompt;
- mask/anonymize thông tin cá nhân;
- phân quyền theo user, project hoặc workspace;
- audit log cho mọi lượt truy xuất;
- cấu hình rõ dữ liệu nào được phép gửi vào LLM;
- tách runtime secret khỏi code và không commit `.env`.

## 8. Cách Diễn Đạt Trong Báo Cáo

Có thể viết:

> Phân tích truyền thống phù hợp với dữ liệu có cấu trúc, nhưng chưa đủ cho dữ
> liệu vận hành giàu ngữ cảnh như ticket, comment, changelog và tài liệu kỹ
> thuật. Đề tài không sử dụng LLM như một nguồn sự thật độc lập, mà dùng LLM như
> lớp đọc hiểu và tổng hợp trên dữ liệu đã được chuẩn hóa, có truy vết nguồn và
> được liên kết bằng Knowledge Graph.

Và:

> Giá trị của memory backend nằm ở việc biến dữ liệu vận hành rời rạc thành ngữ
> cảnh có thể truy xuất, giúp agent phân tích nhanh hơn, có căn cứ hơn và giảm
> phụ thuộc vào việc con người phải đọc lại toàn bộ lịch sử thủ công.

