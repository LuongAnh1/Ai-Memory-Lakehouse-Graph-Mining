# Graph Schema Dự Kiến

## 1. Mục Đích

Graph schema hiện tại là **schema dự kiến cho Public Jira Dataset v7**. Schema cuối cùng sẽ được chốt sau khi tải dataset, extract dữ liệu và chạy EDA.

Lý do chưa chốt cứng: cần kiểm tra cấu trúc thật của MongoDB dump, tên collection, field nested, kiểu dữ liệu timestamp, cách biểu diễn changelog, comment, actor và issue link.

## 2. Nguyên Tắc Thiết Kế

Schema cần đi từ dữ liệu thật:

```text
Public Jira fields -> Silver schema -> Graph schema -> Cypher analytics -> Demo questions
```

Không bắt buộc mọi node/edge đều xuất hiện trong phiên bản đầu. Chỉ giữ các thành phần map được từ dataset sau EDA.

## 3. Node Labels Dự Kiến

| Node label | Nguồn dữ liệu gợi ý | Vai trò |
| --- | --- | --- |
| `Issue` | issue id, key, title, body, created, updated | thực thể trung tâm |
| `Project` | project id, project key, project name | nhóm issue theo project |
| `Actor` | creator, reporter, assignee, commenter, changelog author | người/tác nhân tham gia |
| `Comment` | issue comments | Semantic Memory và timeline tương tác |
| `Status` | current status, status transition | lifecycle state |
| `ChangeEvent` | changelog item/history | Episodic Memory |
| `Label` | labels/tags nếu có | phân loại issue |
| `Component` | Jira component/module nếu có | phân nhóm kỹ thuật/nghiệp vụ |
| `Priority` | priority nếu có | mức độ ưu tiên |
| `IssueType` | bug/task/improvement/story nếu có | loại issue |
| `Chunk` | text chunk từ title/body/comment/resolution | đơn vị truy xuất cho GraphRAG demo |

## 4. Relationship Types Dự Kiến

| Relationship pattern | Ý nghĩa |
| --- | --- |
| `(:Issue)-[:IN_PROJECT]->(:Project)` | issue thuộc project |
| `(:Actor)-[:CREATED]->(:Issue)` | actor tạo issue |
| `(:Actor)-[:REPORTED]->(:Issue)` | actor report issue |
| `(:Issue)-[:ASSIGNED_TO]->(:Actor)` | issue được assigned cho actor |
| `(:Actor)-[:COMMENTED]->(:Comment)` | actor viết comment |
| `(:Issue)-[:HAS_COMMENT]->(:Comment)` | issue chứa comment |
| `(:Issue)-[:HAS_STATUS]->(:Status)` | trạng thái hiện tại |
| `(:Issue)-[:HAS_LABEL]->(:Label)` | label/tag của issue |
| `(:Issue)-[:HAS_COMPONENT]->(:Component)` | component/module liên quan |
| `(:Issue)-[:HAS_PRIORITY]->(:Priority)` | mức độ ưu tiên |
| `(:Issue)-[:HAS_TYPE]->(:IssueType)` | loại issue |
| `(:Issue)-[:HAS_EVENT]->(:ChangeEvent)` | issue có lifecycle event |
| `(:ChangeEvent)-[:FROM_STATUS]->(:Status)` | event chuyển từ status |
| `(:ChangeEvent)-[:TO_STATUS]->(:Status)` | event chuyển tới status |
| `(:Actor)-[:TRIGGERED]->(:ChangeEvent)` | actor tạo thay đổi |
| `(:Issue)-[:LINKS_TO {type}]->(:Issue)` | issue liên kết issue khác |
| `(:Issue)-[:HAS_CHUNK]->(:Chunk)` | text của issue được chunk |
| `(:Comment)-[:HAS_CHUNK]->(:Chunk)` | text của comment được chunk |

## 5. Semantic Memory

Semantic Memory trong phiên bản đầu lấy từ Public Jira:

* issue title;
* issue description/body;
* comment text;
* resolution/closure note nếu có;
* label, component, issue type và priority như metadata ngữ nghĩa.

Các text dài có thể được tách thành `Chunk` để phục vụ truy xuất ngữ cảnh.

## 6. Episodic Memory

Episodic Memory trong phiên bản đầu lấy từ Public Jira:

* created event;
* status transition;
* assignment change;
* label/component change nếu có trong changelog;
* comment timeline;
* issue link created/updated nếu có;
* reopened/resolved/closed events.

Mục tiêu là tái dựng được một phần lifecycle của issue theo thời gian.

## 7. Graph Mining Dự Kiến

Các phân tích ưu tiên:

* Degree Centrality để tìm issue/project/actor/component nổi bật.
* PageRank để tìm issue hoặc knowledge node có ảnh hưởng cao.
* Community Detection để phát hiện cụm issue liên quan.
* Path Traversal để truy vết từ issue tới linked issue, comment, status hoặc chunk.
* Bottleneck analysis bằng thời gian issue nằm ở từng status.

## 8. Việc Cần Làm Sau EDA

Sau khi tải và inspect dataset, cần cập nhật file này bằng:

* danh sách collection/file thật;
* grain của từng bảng Silver;
* mapping field thật sang node/edge;
* node labels giữ lại trong phiên bản đầu;
* relationship types giữ lại trong phiên bản đầu;
* constraints/indexes Neo4j;
* 3-5 truy vấn Cypher mẫu dựa trên giá trị thật trong dataset.
