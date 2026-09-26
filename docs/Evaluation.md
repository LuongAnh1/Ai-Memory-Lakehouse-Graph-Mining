# Kế Hoạch Đánh Giá

## 1. Mục Tiêu Đánh Giá

Đánh giá không nhằm chứng minh các tuyên bố tuyệt đối như “giảm 90% token” hay “loại bỏ hallucination”. Mục tiêu là đo lường có kiểm soát xem kiến trúc dữ liệu và graph retrieval có cải thiện việc truy xuất ngữ cảnh so với baseline chỉ dựa trên text hay không.

Vì dữ liệu lõi đã chốt là Public Jira Dataset v7, bộ chỉ số sẽ tập trung vào issue, comment, changelog, project, actor, component, status và issue link.

## 2. Nhóm Chỉ Số Dataset/EDA

| Chỉ số | Ý nghĩa |
| --- | --- |
| số issues | xác định quy mô thực thể trung tâm |
| số comments | đánh giá độ phong phú của Semantic Memory |
| số change events | đánh giá độ phong phú của Episodic Memory |
| số issue links | đánh giá khả năng tạo graph quan hệ |
| số projects/components/labels/statuses | xác định khả năng phân tích theo nhóm |
| dung lượng raw | kiểm tra quy mô dữ liệu |
| dung lượng Parquet | đo hiệu quả lưu trữ sau transform |
| missing values | xác định field cần làm sạch |
| độ dài text trung bình | đánh giá khả năng chunking/retrieval |
| số timestamp field | xác định khả năng phân tích lifecycle |

## 3. Nhóm Chỉ Số Pipeline

| Chỉ số | Ý nghĩa |
| --- | --- |
| số file/collection ingest thành công | kiểm tra khả năng nạp dữ liệu thô |
| số bản ghi hợp lệ/lỗi | đo chất lượng validation |
| thời gian xử lý pipeline | đo hiệu năng xử lý |
| số chunk tạo ra | đo kết quả xử lý text |
| kích thước dữ liệu raw và Parquet | so sánh hiệu quả lưu trữ |
| số partition | kiểm tra tổ chức dữ liệu theo source/version/project/thời gian |

Ví dụ báo cáo:

```text
Issues processed: 100000/100000
Comments processed: 320000/322000
Invalid records: 128
Text chunks generated: 450000
Pipeline runtime: 18m 24s
```

## 4. Nhóm Chỉ Số Graph

| Chỉ số | Ý nghĩa |
| --- | --- |
| số node theo từng label | kiểm tra graph coverage |
| số relationship theo từng type | kiểm tra mức độ liên kết dữ liệu |
| average degree | đánh giá mật độ liên kết |
| top degree nodes | tìm issue/project/actor/component nổi bật |
| community count | xem dữ liệu có tạo thành nhóm hành vi không |
| path length từ Issue tới LinkedIssue/Comment/Chunk | đo khả năng truy vết tri thức |
| số status transition | đánh giá khả năng phân tích lifecycle |

Câu hỏi đánh giá:

* Project nào có nhiều issue nhất?
* Actor nào liên kết với nhiều issue/comment nhất?
* Component hoặc label nào có độ trung tâm cao?
* Issue nào có nhiều linked issue nhất?
* Có cộng đồng issue liên quan nào nổi bật không?

## 5. Nhóm Chỉ Số Graph Mining

| Kỹ thuật | Mục tiêu |
| --- | --- |
| Degree Centrality | tìm node có nhiều kết nối |
| PageRank | tìm node có ảnh hưởng cao trong graph |
| Community Detection | tìm cụm issue liên quan |
| Path Traversal | truy vết issue -> linked issue/comment/chunk |
| Bottleneck Analysis | tìm status hoặc transition gây chậm |

Kết quả cần có nhận xét nghiệp vụ, không chỉ in ra bảng số.

## 6. Nhóm Chỉ Số Retrieval

So sánh ba cách truy xuất:

| Cách truy xuất | Mô tả |
| --- | --- |
| Full-context baseline | đưa toàn bộ text liên quan vào prompt |
| Keyword/vector baseline | tìm đoạn văn bản tương đồng hoặc chứa từ khóa |
| Graph-based retrieval | truy xuất subgraph theo issue/project/component/status/linked issue |

Chỉ số:

* số token context đầu vào;
* thời gian truy xuất;
* số nguồn được truy vết;
* độ đúng của câu trả lời theo bộ câu hỏi mẫu;
* mức độ giải thích được của kết quả;
* khả năng chỉ ra đường liên kết từ issue tới nguồn tri thức.

## 7. Bộ Câu Hỏi Kiểm Thử Mẫu

Câu hỏi cụ thể sẽ sinh sau EDA. Khung câu hỏi:

* Project nào có nhiều issue nhất?
* Status nào gây bottleneck lâu nhất?
* Component/label nào thường đi cùng các issue reopened?
* Actor nào tham gia nhiều issue/comment nhất?
* Issue nào có nhiều linked issue nhất?
* Với một issue cụ thể, comment/chunk nào liên quan nhất?
* Từ một issue, đường đi tới các issue liên quan hoặc tri thức xử lý là gì?

## 8. Tiêu Chí Hoàn Thành

Đề tài được xem là đạt yêu cầu kỹ thuật nếu:

* có EDA report cho Public Jira Dataset v7;
* pipeline xử lý được sample có ý nghĩa từ Bronze sang Silver;
* dữ liệu Silver có schema rõ và validate được;
* Neo4j chứa graph có đủ node/edge chính theo schema sau EDA;
* có ít nhất 3 truy vấn Cypher phân tích được ý nghĩa nghiệp vụ;
* có ít nhất 1 thuật toán hoặc kỹ thuật graph mining được áp dụng;
* demo truy xuất được subgraph/context cho một câu hỏi nghiệp vụ;
* báo cáo có số liệu đo lường thay vì chỉ mô tả định tính.

## 9. Cách Trình Bày Kết Quả

Kết quả nên trình bày theo hướng:

```text
Dataset -> EDA -> Pipeline -> Graph -> Mining -> Retrieval -> Đánh giá
```

Mỗi kết quả nên có:

* dữ liệu đầu vào;
* phương pháp xử lý;
* số liệu đo được;
* hình ảnh hoặc bảng minh họa;
* nhận xét nghiệp vụ.
