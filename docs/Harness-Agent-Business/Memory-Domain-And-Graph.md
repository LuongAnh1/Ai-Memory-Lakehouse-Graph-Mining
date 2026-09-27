# Memory Domain Và Graph Riêng Cho Harness Agent

## 1. Memory Có Phải Một Domain Riêng Không

Có. Trong bối cảnh agent harness, memory có thể được coi là một domain riêng vì
nó có dữ liệu, quy tắc, vòng đời và mục tiêu nghiệp vụ riêng.

Memory domain không chỉ lưu lại đoạn chat. Nó trả lời các câu hỏi:

- thông tin nào đáng nhớ lâu dài;
- sự kiện nào cần lưu theo thời gian;
- dữ liệu nào đủ tin cậy để đưa vào prompt;
- quan hệ nào giữa user, task, issue, fact và episode là quan trọng;
- khi LLM hỏi một vấn đề, context nào nên được truy xuất.

## 2. Hai Lớp Graph Có Thể Tồn Tại Song Song

Trong đồ án có thể phân biệt hai lớp graph:

| Lớp graph | Mục tiêu | Ví dụ node |
| --- | --- | --- |
| Business Graph | mô hình hóa nghiệp vụ Jira/support | `Issue`, `Project`, `Component`, `Actor`, `Status`, `Comment` |
| Agent Memory Graph | mô hình hóa bộ nhớ của harness agent | `Conversation`, `Turn`, `Task`, `Fact`, `Episode`, `Decision`, `Evidence` |

Business Graph giúp hiểu dữ liệu nghiệp vụ. Agent Memory Graph giúp hiểu agent
đã biết gì, từng phân tích gì, dùng nguồn nào và phản hồi ra sao.

Trong phiên bản đầu, không bắt buộc phải xây đầy đủ cả hai graph. Có thể ưu tiên
Business Graph trên Public Jira, sau đó mở rộng sang Agent Memory Graph khi tích
hợp sâu với harness.

## 3. Entity Dự Kiến Của Agent Memory Graph

| Entity | Ý nghĩa |
| --- | --- |
| `User` | người dùng tương tác với agent |
| `Conversation` | một phiên hoặc kênh hội thoại |
| `Turn` | một lượt user hỏi/agent trả lời |
| `Message` | nội dung message cụ thể |
| `Task` | tác vụ agent cần xử lý |
| `Fact` | semantic memory ổn định |
| `Episode` | sự kiện/tác vụ theo thời gian |
| `Decision` | quyết định routing hoặc kết luận phân tích |
| `Evidence` | nguồn dữ liệu được dùng để trả lời |
| `Issue` | issue/ticket liên quan từ business graph |
| `TraceEvent` | event kỹ thuật trong harness |

## 4. Relationship Dự Kiến

| Relationship | Ý nghĩa |
| --- | --- |
| `(:User)-[:SENT]->(:Message)` | user gửi message |
| `(:Conversation)-[:HAS_TURN]->(:Turn)` | conversation chứa turn |
| `(:Turn)-[:CREATED_TASK]->(:Task)` | turn tạo task cần xử lý |
| `(:Task)-[:USED_FACT]->(:Fact)` | task dùng semantic fact |
| `(:Task)-[:USED_EPISODE]->(:Episode)` | task dùng episode cũ |
| `(:Task)-[:PRODUCED_DECISION]->(:Decision)` | task sinh quyết định/kết luận |
| `(:Decision)-[:SUPPORTED_BY]->(:Evidence)` | kết luận có nguồn hỗ trợ |
| `(:Episode)-[:ABOUT_ISSUE]->(:Issue)` | episode liên quan tới issue |
| `(:Fact)-[:DESCRIBES]->(:Issue)` | fact mô tả issue hoặc tri thức liên quan |
| `(:TraceEvent)-[:OBSERVED]->(:Turn)` | trace ghi nhận một phần turn |

## 5. Liên Kết Semantic Và Episodic Memory

Semantic Memory trả lời câu hỏi:

```text
Hệ thống biết điều gì?
Luật, fact, kinh nghiệm, hướng xử lý nào đã được ghi nhận?
```

Episodic Memory trả lời câu hỏi:

```text
Điều gì đã xảy ra?
Ai làm gì, khi nào, trong task nào, kết quả ra sao?
```

Knowledge Graph giúp nối hai loại memory này:

```text
Episode xử lý Issue A
  -> sinh ra Fact F về nguyên nhân lỗi
  -> Fact F được dùng lại trong Task mới
  -> Task mới trỏ về Evidence là comment/resolution cũ
```

## 6. Vai Trò Của Graph Mining

Graph mining được dùng sau khi graph đã có dữ liệu.

Một số mục tiêu phù hợp:

- tìm issue/component/actor có độ trung tâm cao;
- phát hiện cụm issue liên quan;
- tìm bottleneck trong lifecycle;
- tìm path từ issue hiện tại tới tri thức xử lý cũ;
- tìm episode/fact thường được dùng lại;
- phát hiện memory bị cô lập hoặc ít giá trị.

Graph mining không thay LLM. Nó tạo insight và cấu trúc truy xuất tốt hơn để LLM
có context chính xác hơn.

## 7. Mức Độ Cần Làm Trong Đồ Án

Khuyến nghị phạm vi:

1. Làm chắc Business Graph từ Public Jira trước.
2. Chứng minh graph có thể truy xuất context tốt hơn tìm text đơn giản.
3. Ghi rõ Agent Memory Graph là hướng mở rộng khi tích hợp với Niko Agent.
4. Nếu còn thời gian, map thử một số trace/episode từ Niko vào graph.

Không nên cố xây một memory graph quá lớn trước khi có dữ liệu thật và EDA.

