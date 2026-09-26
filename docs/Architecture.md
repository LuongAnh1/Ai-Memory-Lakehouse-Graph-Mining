# Kiến Trúc Dự Kiến

## 1. Tổng Quan

Đồ án sử dụng kiến trúc mini-lakehouse / lakehouse-style theo mô hình Medallion, kết hợp với Knowledge Graph để biểu diễn quan hệ giữa issue tracking data, tri thức xử lý và các sự kiện lifecycle.

```text
Public Jira Dataset v7
  -> Bronze Layer (MinIO raw objects)
  -> Processing Pipeline (extract, clean, validate)
  -> Silver Layer (Parquet datasets)
  -> Gold Layer (Neo4j Knowledge Graph)
  -> Analytics & GraphRAG Demo (Streamlit)
```

Trong phạm vi đồ án, MinIO + Parquet được xem là kiến trúc lakehouse-style ở mức nhỏ. Delta Lake hoặc Apache Iceberg chưa được đưa vào triển khai chính để giữ phạm vi phù hợp, nhưng có thể được ghi nhận là hướng nâng cấp khi cần ACID transaction, schema evolution và time travel.

## 2. Các Tầng Kiến Trúc

### 2.1. Data Sources

Nguồn dữ liệu đã chốt:

* **Primary**: Public Jira Dataset v7.
* **Optional semantic extension**: Stack Exchange subset.
* **Future event-stream extension**: GH Archive.

Trong phiên bản đầu, pipeline chỉ cần tập trung vào Public Jira Dataset v7. Các nguồn phụ chỉ đưa vào khi pipeline lõi đã ổn định.

### 2.2. Bronze Layer

Bronze lưu dữ liệu nguyên bản, chưa chỉnh sửa.

Vai trò:

* đảm bảo khả năng truy vết nguồn dữ liệu;
* cho phép chạy lại pipeline khi logic xử lý thay đổi;
* lưu raw ZIP/MongoDB dump/extracted files từ Public Jira;
* lưu metadata về nguồn, thời gian tải, version và license.

Cấu trúc gợi ý:

```text
bronze/
  public_jira/
    version=v7/
      raw/
      extracted/
      metadata/
```

### 2.3. Silver Layer

Silver chứa dữ liệu đã được làm sạch, chuẩn hóa và validate. Schema cụ thể sẽ chốt sau EDA.

Dataset dự kiến:

```text
silver/
  public_jira/
    issues.parquet
    comments.parquet
    change_events.parquet
    projects.parquet
    actors.parquet
    labels.parquet
    components.parquet
    issue_links.parquet
    text_chunks.parquet
```

Vai trò:

* chuẩn hóa timestamp, issue_id, project_id, actor_id;
* tách issue body/comment/resolution thành chunk nếu cần;
* chuẩn hóa changelog thành event records;
* loại bỏ hoặc đánh dấu bản ghi lỗi/missing;
* tạo schema thống nhất cho dữ liệu phân tích;
* làm đầu vào ổn định cho Neo4j.

### 2.4. Gold Layer

Gold là Knowledge Graph trong Neo4j.

Vai trò:

* lưu quan hệ giữa issue, project, actor, comment, status, label, component, change event và linked issue;
* phục vụ graph traversal;
* phục vụ graph mining;
* tạo subgraph làm context cho lớp demo GraphRAG.

### 2.5. Consumer Layer

Consumer layer là lớp demo, không phải trọng tâm chính.

Thành phần:

* dashboard Streamlit;
* một số câu truy vấn Cypher mẫu;
* module truy xuất subgraph theo issue id, project, label/component hoặc status;
* tùy chọn gọi LLM để tạo câu trả lời từ context đã truy xuất.

## 3. Graph Schema Dự Kiến

Graph schema chi tiết chưa chốt cứng trước khi EDA. Tuy nhiên, với Public Jira Dataset v7, hướng schema dự kiến gồm:

| Nhóm node | Ví dụ |
| --- | --- |
| Issue tracking | `Issue`, `Project` |
| Actor | `Actor`, `Assignee`, `Reporter`, `Commenter` |
| Lifecycle | `Status`, `ChangeEvent` |
| Classification | `Label`, `Component`, `Priority`, `IssueType` |
| Knowledge text | `Comment`, `Chunk` |
| Relationship artifact | `IssueLink` hoặc relationship trực tiếp giữa hai `Issue` |

Chi tiết được tách trong [Graph-Schema.md](Graph-Schema.md).

## 4. Graph Construction Và Graph Mining

Cần tách rõ hai phần:

* **Graph Construction**: trích xuất thực thể, chuẩn hóa id, tạo node/edge và nạp vào Neo4j.
* **Graph Mining**: chạy thuật toán hoặc truy vấn phân tích trên graph đã có.

Graph construction trả lời câu hỏi “dữ liệu được đưa vào graph như thế nào?”. Graph mining trả lời câu hỏi “từ graph đó khai phá được điều gì?”.

## 5. Luồng Xử Lý End-to-End

```text
1. Tải Public Jira Dataset v7 và lưu metadata nguồn.
2. Ingest raw dataset vào Bronze.
3. Extract MongoDB dump hoặc file nguồn thành dữ liệu đọc được.
4. Chạy EDA để xác định grain, schema và field quan trọng.
5. Chuẩn hóa issue/comment/changelog/link records.
6. Chunk issue body/comment/resolution nếu cần.
7. Validate dữ liệu bằng schema.
8. Ghi dữ liệu sạch sang Silver Parquet.
9. Load issue, project, actor, status, event, link và text chunk vào Neo4j.
10. Tạo quan hệ giữa các node.
11. Chạy graph mining và Cypher analytics.
12. Hiển thị dashboard hoặc truy xuất subgraph cho GraphRAG demo.
```

## 6. Rủi Ro Kiến Trúc

| Rủi ro | Cách giảm thiểu |
| --- | --- |
| Public Jira dump có cấu trúc phức tạp | bắt đầu bằng sample nhỏ, viết EDA/data dictionary trước |
| Schema graph quá phức tạp | bắt đầu với node/edge tối thiểu, mở rộng sau |
| Dữ liệu 5.8GB ZIP nặng khi xử lý local | xử lý theo batch/partition, dùng Polars lazy scan hoặc PyArrow |
| Dataset không có SOP riêng | dùng issue body/comment/resolution làm Semantic Memory |
| GraphRAG vượt phạm vi | giữ ở mức demo truy xuất subgraph và trả lời câu hỏi mẫu |
| Gọi Lakehouse bị bắt bẻ | dùng thuật ngữ mini-lakehouse / lakehouse-style, nêu rõ chưa triển khai Delta/Iceberg |
