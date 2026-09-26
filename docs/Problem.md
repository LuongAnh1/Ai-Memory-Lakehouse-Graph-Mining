# Vấn Đề Cần Giải Quyết

## 1. Bối Cảnh

Trong các hệ thống vận hành như IT Helpdesk, developer support, issue tracking hoặc AI Support Agent nội bộ, dữ liệu thường tồn tại dưới hai nhóm chính:

* **Dữ liệu tri thức tĩnh**: issue title/body, comment, resolution note, hướng dẫn xử lý lỗi, tài liệu kỹ thuật, FAQ, SOP.
* **Dữ liệu sự kiện theo thời gian**: issue lifecycle, status change, comment event, label change, assignee change, linked issue, lịch sử tương tác.

Hai nhóm dữ liệu này tương ứng với hai dạng bộ nhớ quan trọng trong hệ thống AI:

| Nhóm dữ liệu | Vai trò |
| --- | --- |
| Tài liệu tri thức và nội dung văn bản | Semantic Memory: lưu kiến thức nghiệp vụ và kinh nghiệm xử lý |
| Nhật ký sự kiện và thay đổi theo thời gian | Episodic Memory: lưu diễn biến, hành động, trạng thái và kết quả |

Đồ án chọn **Public Jira Dataset v7** làm dữ liệu lõi để mô phỏng bài toán này. Dataset có đủ dữ liệu văn bản, changelog, comment, project và issue link để xây dựng pipeline dữ liệu, Knowledge Graph và các phân tích graph mining.

## 2. Vấn Đề Thực Tiễn

### 2.1. Dữ liệu vận hành bị phân mảnh

Trong thực tế, tri thức xử lý sự cố thường nằm trong mô tả issue, comment, tài liệu kỹ thuật hoặc Q&A. Trong khi đó, diễn biến xử lý lại nằm trong changelog, status transition, assignment history và linked issue.

Nếu các dữ liệu này chỉ được lưu rời rạc, hệ thống khó trả lời các câu hỏi như:

* lỗi nào thường xảy ra trong project/component nào?
* status nào khiến issue bị kẹt lâu nhất?
* issue nào liên quan hoặc chặn issue nào?
* ai hoặc nhóm nào thường xử lý một loại vấn đề nhất định?
* comment hoặc resolution nào có thể dùng làm tri thức tham khảo cho issue mới?

### 2.2. Truy xuất ngữ cảnh cho LLM còn thiếu cấu trúc

Vector RAG có thể tìm văn bản tương đồng, nhưng chưa đủ tốt khi câu hỏi cần đi qua nhiều quan hệ như issue -> component -> linked issue -> comment -> status history.

Với dữ liệu issue tracking, hệ thống cần truy xuất không chỉ đoạn text gần nghĩa mà còn cả:

* issue cùng project/component;
* issue có quan hệ duplicate/blocks/relates;
* chuỗi status theo thời gian;
* actor hoặc assignee liên quan;
* comment hoặc resolution từng giúp xử lý vấn đề tương tự.

Đây là lý do cần kết hợp lakehouse-style pipeline với Knowledge Graph.

### 2.3. Thiếu nền tảng đo lường và khai phá quan hệ

Nếu không có tầng dữ liệu trung gian được chuẩn hóa, rất khó đo lường:

* số bản ghi ingest thành công/lỗi;
* dung lượng raw và Parquet;
* số issue/comment/change event/link xử lý được;
* số node/edge tạo trong graph;
* project/component/status có độ trung tâm cao;
* cụm issue liên quan;
* đường đi từ issue tới tri thức xử lý.

Do đó, đề tài cần một kiến trúc dữ liệu có thể vừa lưu trữ dữ liệu thô, vừa chuẩn hóa dữ liệu phân tích, vừa mô hình hóa quan hệ phục vụ graph mining.

## 3. Phát Biểu Bài Toán

Đề tài cần thiết kế và triển khai một nền tảng dữ liệu ở mức Proof of Concept để:

1. Thu thập và lưu trữ Public Jira Dataset v7 theo kiến trúc Bronze - Silver - Gold.
2. Chuẩn hóa dữ liệu issue, comment, changelog, project, actor, label/component và issue link thành dữ liệu phân tích được.
3. Xây dựng Semantic Memory từ issue title/body/comment/resolution.
4. Xây dựng Episodic Memory từ changelog, status transition, assignment event, comment timeline và issue link.
5. Mô hình hóa dữ liệu dưới dạng Knowledge Graph trong Neo4j.
6. Áp dụng graph mining để phát hiện bottleneck, node quan trọng, cụm issue liên quan và đường liên kết giữa issue với tri thức xử lý.
7. Cung cấp dashboard/demo truy vấn ngữ cảnh để chứng minh dữ liệu đã xử lý có thể hỗ trợ BI/Analytics và GraphRAG.

## 4. Phạm Vi Đề Tài

### 4.1. Trong phạm vi

* Thiết kế mini-lakehouse / lakehouse-style architecture theo mô hình Bronze - Silver - Gold.
* Dùng MinIO để lưu dữ liệu thô và dữ liệu trung gian dạng object storage.
* Dùng Parquet để lưu dữ liệu sạch ở tầng Silver.
* Xây dựng pipeline xử lý Public Jira Dataset v7.
* Validate dữ liệu bằng schema rõ ràng sau EDA.
* Thiết kế Knowledge Graph trên Neo4j dựa trên cấu trúc thật của dataset.
* Chạy một số phân tích graph mining cơ bản như centrality, community detection hoặc path analysis.
* Xây dựng dashboard/demo truy vấn ở mức Proof of Concept.

### 4.2. Ngoài phạm vi

* Không xây dựng AI Agent hoàn chỉnh.
* Không triển khai cơ chế tự động sinh skill/procedural memory ở mức production.
* Không đặt mục tiêu xử lý big data quy mô rất lớn.
* Không bắt buộc triển khai Delta Lake hoặc Apache Iceberg trong phiên bản đầu.
* Không chốt cứng graph schema trước khi tải dữ liệu và phân tích EDA.
* Không cam kết một tỷ lệ cố định về giảm token hoặc giảm hallucination; các lợi ích này sẽ được đánh giá bằng thực nghiệm.
* Không bắt buộc tích hợp Stack Exchange hoặc GH Archive trong phiên bản đầu.

## 5. Câu Hỏi Kỹ Thuật Cần Trả Lời

* Làm thế nào tổ chức dữ liệu Public Jira theo kiến trúc Bronze - Silver - Gold?
* Làm thế nào chuyển đổi MongoDB dump hoặc dữ liệu extract của Public Jira thành Parquet có schema rõ?
* Làm thế nào tách Semantic Memory từ issue title/body/comment/resolution?
* Làm thế nào tách Episodic Memory từ changelog, status transition và comment timeline?
* Làm thế nào thiết kế graph schema liên kết issue, project, actor, comment, status, label/component và linked issue?
* Những thuật toán graph mining nào phù hợp để phân tích bottleneck, issue link, cụm vấn đề và node quan trọng?
* Graph-based retrieval có giúp truy xuất ngữ cảnh chính xác và gọn hơn baseline chỉ dựa trên text hay không?

## 6. Kết Quả Mong Đợi

Kết quả của đề tài là một nền tảng dữ liệu mẫu có thể chứng minh luồng xử lý end-to-end:

```text
Public Jira raw data
-> Bronze storage
-> extraction, cleaning and validation
-> Silver Parquet datasets
-> Neo4j Knowledge Graph
-> graph mining and analytics
-> GraphRAG/dashboard demo
```

Nền tảng này là bước chuẩn bị cho hướng phát triển dài hạn: xây dựng memory backend cho AI Agent hoặc hệ thống tự động hóa phân tích doanh nghiệp.
