# Công Nghệ Sử Dụng

## 1. Nguyên Tắc Chọn Công Nghệ

Stack được chọn theo ba tiêu chí:

* đủ thể hiện năng lực Data Engineering/Data Mining;
* dễ triển khai trong phạm vi bài tập lớn;
* có thể mở rộng lên kiến trúc production trong tương lai.

Đồ án ưu tiên công nghệ open-source, chạy được bằng Docker/local machine và có tài liệu rõ ràng.

## 2. Stack Đề Xuất

| Nhóm | Công nghệ | Vai trò |
| --- | --- | --- |
| Runtime | Python 3.10+ | ngôn ngữ chính cho pipeline |
| Container | Docker Compose | dựng môi trường MinIO, Neo4j, app demo |
| Data Acquisition | HTTP download, Zenodo download, MongoDB tools, 7z | tải và extract Public Jira Dataset v7 |
| Object Storage | MinIO | lưu Bronze/Silver dưới dạng S3-compatible object storage |
| File Format | Apache Parquet | lưu dữ liệu sạch dạng cột |
| DataFrame | Polars hoặc Pandas | xử lý issue/comment/event records, transform dữ liệu, ghi Parquet |
| Large File Handling | PyArrow, Polars lazy scan, chunked readers | xử lý dữ liệu nhiều GB theo batch/partition |
| SQL Analytics | DuckDB | kiểm tra/query trực tiếp dữ liệu Parquet |
| Validation | Pydantic | định nghĩa schema và validate bản ghi |
| Graph Database | Neo4j Community Edition | lưu Knowledge Graph |
| Graph Mining | Neo4j Graph Data Science hoặc Cypher | centrality, community detection, path analysis |
| UI Demo | Streamlit | dashboard và giao diện truy vấn |
| Visualization | PyVis hoặc Neo4j Browser | trực quan hóa graph |
| GraphRAG Demo | Neo4j driver hoặc LangChain | truy xuất subgraph và trả lời câu hỏi mẫu |

## 3. Storage: MinIO + Parquet

### 3.1. Vì sao dùng MinIO?

MinIO phù hợp vì:

* mô phỏng object storage theo chuẩn S3;
* dễ chạy local bằng Docker;
* phù hợp lưu raw ZIP/MongoDB dump, extracted files và Parquet datasets;
* giúp đồ án có màu sắc Data Lake/Data Platform rõ hơn so với chỉ lưu file local.

### 3.2. Vì sao dùng Parquet?

Parquet phù hợp vì:

* lưu dữ liệu dạng cột;
* nén tốt;
* đọc nhanh khi chỉ cần một số cột;
* hỗ trợ tốt bởi Polars, Pandas, DuckDB và nhiều engine phân tích khác.

### 3.3. Vì sao chưa dùng Delta Lake/Iceberg?

Trong phạm vi đồ án, dữ liệu chủ yếu là batch/append-style và quy mô vừa phải. Nếu đưa Delta Lake hoặc Apache Iceberg vào ngay, phần triển khai sẽ phức tạp hơn vì cần thêm table format, catalog hoặc engine tương thích.

Phương án chốt:

```text
Triển khai chính: MinIO + Parquet + quy ước metadata
Tên gọi: mini-lakehouse / lakehouse-style architecture
Hướng mở rộng: Delta Lake hoặc Apache Iceberg
```

Delta Lake/Iceberg chỉ nên được nhắc trong báo cáo như hướng nâng cấp khi hệ thống cần:

* ACID transaction;
* schema evolution;
* time travel;
* merge/update/delete;
* quản lý metadata quy mô lớn.

## 4. Pipeline

Pipeline nên chia thành các module rõ ràng:

```text
src/
  profiling/
  ingestion/
  extraction/
  transformation/
  validation/
  storage/
  graph_loader/
  analytics/
  app/
```

Các bước chính:

* tải Public Jira Dataset v7 hoặc chuẩn bị sample;
* ingest raw ZIP/MongoDB dump vào Bronze;
* extract dữ liệu từ dump thành các collection/table có thể xử lý;
* chạy EDA/profiling;
* parse và normalize issue/comment/changelog/link records;
* extract/chunk text từ title/body/comment/resolution;
* validate schema;
* ghi Parquet vào Silver;
* load dữ liệu vào Neo4j;
* chạy Cypher/GDS analytics;
* hiển thị kết quả trên Streamlit.

## 5. Graph Layer

Neo4j được chọn vì:

* biểu diễn node/edge tự nhiên;
* Cypher dễ đọc và dễ demo;
* có Neo4j Browser để trực quan hóa nhanh;
* có thể dùng Neo4j GDS cho thuật toán graph mining;
* phù hợp với truy vấn nhiều bước giữa issue, project, actor, status, linked issue và knowledge text.

Các phân tích nên ưu tiên:

* Degree Centrality: tìm issue/project/actor/component nổi bật.
* PageRank: tìm issue hoặc knowledge node có ảnh hưởng cao trong graph.
* Community Detection: gom cụm issue/label/project/comment có liên quan.
* Shortest Path/Path Traversal: truy vết từ issue tới linked issue, comment hoặc text chunk.
* Bottleneck Analysis: đo thời gian issue nằm ở từng status.

## 6. Demo Layer

Demo nên giữ vừa đủ:

* màn hình dashboard thống kê số issue, project, status, actor và graph nodes;
* bảng top labels/status/projects/actors/linked issues;
* form nhập issue id, project, label/status hoặc câu hỏi;
* kết quả hiển thị subgraph hoặc danh sách node/edge liên quan;
* tùy chọn sinh câu trả lời tự nhiên bằng LLM từ context đã truy xuất.

Nếu dùng LLM sinh Cypher tự động, nên cấu hình tài khoản Neo4j read-only cho demo để giảm rủi ro truy vấn ghi hoặc xóa dữ liệu.
