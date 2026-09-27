# Nghiệp Vụ Harness Agent Và Memory Backend

## 1. Mục Đích

Folder này ghi lại nghiệp vụ dự kiến mà bộ nhớ Semantic/Episodic cung cấp cho
một hệ thống harness agent.

Trong phạm vi đồ án, repo này không xây dựng một AI Agent hoàn chỉnh. Tuy nhiên,
dữ liệu lakehouse, Knowledge Graph và graph mining được thiết kế để trở thành
**memory backend** cho một agent harness như Niko Agent.

Nói ngắn gọn:

```text
Niko Agent Harness
  -> cần context đáng tin cậy để phân tích
  -> gọi Memory Backend
  -> Memory Backend truy xuất Semantic/Episodic Memory + Knowledge Graph
  -> LLM nhận context đã chuẩn hóa, có nguồn, có quan hệ
```

## 2. Nghiệp Vụ Dự Kiến

Nghiệp vụ dự kiến là:

> Agent Memory Management cho hệ thống IT Helpdesk, Developer Support hoặc AI
> Support Agent nội bộ doanh nghiệp.

Đây là nghiệp vụ phụ trợ cho agent, không phải chỉ là lưu file hay lưu log. Nó
có nhiệm vụ tổ chức dữ liệu vận hành để LLM có thể đọc, truy xuất, liên kết và
phân tích vấn đề.

## 3. Vì Sao Đây Có Thể Xem Là Một Nghiệp Vụ Riêng

Memory backend có vòng đời và quy tắc riêng:

1. Thu thập dữ liệu từ hội thoại, ticket, issue, comment, changelog, trace và
   tài liệu.
2. Chuẩn hóa dữ liệu thành các bản ghi có schema rõ ràng.
3. Tách Semantic Memory và Episodic Memory.
4. Liên kết memory bằng Knowledge Graph khi nghiệp vụ đủ rõ.
5. Truy xuất context phù hợp cho LLM theo câu hỏi hoặc task.
6. Ghi trace để biết LLM đã dùng dữ liệu nào khi trả lời.
7. Đánh giá chất lượng retrieval, độ chính xác và mức độ tiết kiệm thời gian.

Do đó, có thể xem đây là một domain riêng:

```text
Domain: Agent Memory Management
Mục tiêu: giúp agent nhớ, truy xuất, liên kết và phân tích thông tin tốt hơn
Đầu vào: chat log, ticket, issue event, trace, tài liệu, graph data
Đầu ra: memory context, subgraph, evidence, dashboard, GraphRAG response
```

## 4. Người Dùng Và Tác Nhân Liên Quan

| Vai trò | Nhu cầu |
| --- | --- |
| Support engineer | Tra cứu issue tương tự, hướng xử lý cũ, nguyên nhân lặp lại |
| Developer | Tìm lỗi liên quan, component bị ảnh hưởng, lịch sử thay đổi |
| Team lead/manager | Xem bottleneck, issue aging, nhóm vấn đề lặp lại |
| AI Agent | Cần context có nguồn để phân tích và trả lời |
| Data engineer | Cần pipeline chuẩn hóa dữ liệu và đo chất lượng |

## 5. Đầu Vào Của Memory Backend

Nguồn dữ liệu chính trong đồ án là Public Jira Dataset v7.

Dữ liệu có thể được ánh xạ như sau:

| Nhóm dữ liệu | Ví dụ | Vai trò trong memory |
| --- | --- | --- |
| Text tri thức | issue title, body, comment, resolution | Semantic Memory |
| Event theo thời gian | created, assigned, status changed, reopened, closed | Episodic Memory |
| Quan hệ nghiệp vụ | issue link, component, project, actor, label | Knowledge Graph |
| Trace agent | route, retrieval, answer, error, followup | Harness observability |

## 6. Đầu Ra Mà Harness Agent Cần

Memory backend không chỉ trả về text. Đầu ra nên có cấu trúc để LLM dùng được:

| Đầu ra | Ý nghĩa |
| --- | --- |
| Relevant facts | tri thức/fact liên quan tới câu hỏi |
| Relevant episodes | sự kiện/tác vụ tương tự đã xảy ra |
| Related issues | issue gần giống hoặc có quan hệ graph |
| Evidence/source | nguồn dữ liệu để truy vết |
| Subgraph context | cụm node/edge liên quan để phân tích |
| Summary | bản tóm tắt ngắn gọn đưa vào prompt |

Ví dụ context cho LLM:

```text
Question: Tại sao issue này hay bị reopen?

Retrieved context:
- Issue A từng chuyển Open -> Resolved -> Reopened 3 lần.
- Các comment liên quan nhắc tới lỗi timeout ở component sync-service.
- Issue B và Issue C có link duplicate/relates với Issue A.
- Component sync-service có reopen rate cao hơn các component khác.
- Resolution gần nhất chỉ xử lý retry, chưa xử lý root cause.
```

## 7. Vì Sao Không Chỉ Phân Tích Theo Cách Truyền Thống

BI/SQL/dashboard truyền thống rất tốt với dữ liệu có cấu trúc:

```text
số issue theo tháng
thời gian xử lý trung bình
số ticket theo status
top component có nhiều bug
```

Nhưng nó yếu hơn khi câu hỏi cần hiểu nội dung tự nhiên và quan hệ ngầm:

```text
Những issue nào giống nhau dù label khác nhau?
Vấn đề này từng được xử lý chưa?
Nguyên nhân gốc có nằm trong comment cũ nào không?
Issue này liên quan tới cụm lỗi nào?
Tại sao một nhóm issue hay bị reopen?
```

LLM có giá trị khi nó được dùng như lớp đọc hiểu và tổng hợp phía trên dữ liệu
đã được chuẩn hóa, không phải như một nguồn sự thật độc lập.

## 8. Giá Trị Kinh Doanh

Doanh nghiệp có thể muốn chi tiền cho LLM khi lợi ích lớn hơn chi phí vận hành:

1. Giảm thời gian đọc ticket, comment, log và tài liệu thủ công.
2. Rút ngắn thời gian tìm issue tương tự hoặc hướng xử lý cũ.
3. Phát hiện vấn đề lặp lại và bottleneck nhanh hơn.
4. Giảm phụ thuộc vào một vài người nhớ hệ thống.
5. Hỗ trợ nhân sự mới hiểu lịch sử xử lý nhanh hơn.
6. Tăng tốc phân tích nguyên nhân gốc và ra quyết định.

Công thức đánh giá đơn giản:

```text
Giá trị LLM = thời gian con người tiết kiệm được
            + chất lượng tri thức được tái sử dụng
            + tốc độ phát hiện vấn đề
            - chi phí triển khai/vận hành/kiểm soát rủi ro
```

## 9. Mức Độ Tin Tưởng

Hệ thống không nên yêu cầu doanh nghiệp tin LLM một cách mù quáng.

Nguyên tắc đúng là:

```text
LLM là lớp phân tích/ngôn ngữ.
Dữ liệu thật nằm trong lakehouse/database/graph.
Câu trả lời phải được grounding bằng dữ liệu nguồn.
```

Các cơ chế cần có:

- truy vết nguồn dữ liệu;
- giới hạn context truy xuất;
- citation/evidence cho câu trả lời;
- trace mỗi lượt agent chạy;
- phân quyền dữ liệu theo user/project nếu triển khai thật;
- mask/anonymize dữ liệu nhạy cảm;
- human-in-the-loop cho quyết định quan trọng;
- đánh giá retrieval và hallucination bằng bộ câu hỏi mẫu.

## 10. Quan Hệ Với Knowledge Graph

Nếu nghiệp vụ đủ rõ, memory backend có thể có graph riêng.

Với nghiệp vụ Jira/support, graph giúp liên kết:

```text
Issue -> Project -> Component
Issue -> Comment -> Actor
Issue -> ChangeEvent -> Status
Issue -> LinkedIssue
Issue -> Chunk/SemanticFact
Episode -> Issue/Task/Conversation
```

Graph không tồn tại chỉ để trực quan hóa. Graph giúp agent trả lời các câu hỏi
cần đi qua nhiều quan hệ, ví dụ:

```text
issue hiện tại
  -> component liên quan
  -> issue cũ cùng component
  -> comment/resolution từng xử lý
  -> episode từng phân tích
  -> context đưa cho LLM
```

## 11. Ranh Giới Với Niko Agent

Niko Agent Harness là hệ thống chạy thật để chứng minh luồng agent:

```text
Telegram -> Router -> Fast/Deep Agent -> Memory Context -> Reply -> Trace/Ops
```

Repo lakehouse/graph này là hướng cải tiến memory:

```text
raw operational data
  -> Bronze/Silver/Gold
  -> Semantic/Episodic Memory
  -> Knowledge Graph
  -> Graph Mining
  -> context tốt hơn cho harness agent
```

Vì vậy, hai phần bổ trợ cho nhau:

- Niko Agent chứng minh harness chạy thật và có baseline memory đơn giản.
- Lakehouse/Graph repo chứng minh cách tổ chức dữ liệu để memory tốt hơn.

