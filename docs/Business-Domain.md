# Nghiệp Vụ Dự Kiến

## 1. Nghiệp Vụ Chọn Cho Đồ Án

Nghiệp vụ của đồ án là **nền tảng phân tích dữ liệu issue/ticket tracking cho IT Helpdesk, developer support hoặc AI Support Agent nội bộ doanh nghiệp**.

Trong bối cảnh này, tổ chức có một hệ thống ghi nhận và xử lý vấn đề như Jira/GitHub Issues/helpdesk ticket. Mỗi issue có thể có mô tả, comment, label, component, assignee, status change, linked issue và thời gian xử lý.

Đồ án dùng **Public Jira Dataset v7** làm dữ liệu lõi để mô phỏng nghiệp vụ này.

## 2. Lý Do Chọn Nghiệp Vụ Này

Nghiệp vụ issue/ticket tracking phù hợp vì ánh xạ được trực tiếp vào ba lớp chính của đề tài:

| Loại dữ liệu | Ví dụ | Vai trò |
| --- | --- | --- |
| Issue body/comment/resolution | mô tả lỗi, thảo luận, hướng xử lý, ghi chú đóng issue | Semantic Memory |
| Issue changelog/event | created, status change, assigned, commented, linked, closed, reopened | Episodic Memory |
| Quan hệ nghiệp vụ | issue thuộc project nào, liên kết issue nào, ai xử lý, status nào | Knowledge Graph |
| Chỉ số phân tích | bottleneck status, issue aging, reopen rate, linked issue, community issue | BI/Analytics |

Nghiệp vụ này cũng dễ giải thích trong báo cáo vì không cần giả định quá nhiều. Public Jira đã có dữ liệu thật, đủ lớn và đủ quan hệ để xây dựng pipeline lẫn graph mining.

## 3. Luồng Nghiệp Vụ Mẫu

Một issue/ticket có thể diễn ra như sau:

```text
User tạo issue: "API trả lỗi 500 khi đồng bộ dữ liệu."
Issue được gán component: integration.
Issue được gán label: bug, high priority.
Issue được assigned cho một maintainer/team.
Các comment trao đổi nguyên nhân và hướng xử lý.
Issue được link với issue khác theo quan hệ duplicate/blocks/relates.
Issue chuyển trạng thái resolved/closed hoặc reopened.
```

Từ luồng này, hệ thống có thể tạo các thực thể:

* Issue;
* Project;
* Actor/User;
* Comment;
* Status;
* Label;
* Component;
* ChangeEvent;
* IssueLink;
* Chunk nếu cần tách text dài thành đơn vị truy xuất.

## 4. Câu Hỏi Phân Tích Có Thể Demo

Các câu hỏi phân tích sẽ bám vào cấu trúc thật sau EDA, nhưng định hướng chính gồm:

* Project nào có nhiều issue nhất?
* Component/label nào liên quan nhiều tới issue reopened?
* Status nào thường gây bottleneck lâu nhất?
* Actor nào tham gia nhiều issue/comment nhất?
* Issue nào có nhiều linked issue hoặc comment nhất?
* Cụm issue nào thường liên quan với nhau?
* Với một issue cụ thể, subgraph nào nên được truy xuất để làm ngữ cảnh cho LLM?

## 5. Dữ Liệu Mẫu

Dữ liệu mẫu đã chốt:

```text
Primary dataset:
  Public Jira Dataset v7

Primary Semantic Memory:
  issue title/body/comment/resolution

Primary Episodic Memory:
  changelog, status transition, assignment event, comment timeline, issue link

Optional semantic extension:
  Stack Exchange subset

Future event-stream extension:
  GH Archive
```

Sau khi tải dataset và chạy EDA, tài liệu này cần được cập nhật thêm:

* grain thực tế của từng collection/table;
* data dictionary;
* các cột quan trọng cho Semantic Memory;
* các cột quan trọng cho Episodic Memory;
* các câu hỏi phân tích có thể trả lời bằng dữ liệu thật.
