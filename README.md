# Data Lakehouse & Graph Mining Pipeline for AI Memory Backend

Đây là đề tài theo định hướng Data Engineering/Data Mining, tập trung xây dựng hạ tầng dữ liệu cho bộ nhớ AI gồm dữ liệu tri thức tĩnh (Semantic Memory) và nhật ký sự kiện theo thời gian (Episodic Memory).

Phạm vi hiện tại không xây dựng một AI Agent hoàn chỉnh. Lớp GraphRAG/AI Agent chỉ đóng vai trò demo đầu ra để kiểm chứng khả năng truy xuất ngữ cảnh từ dữ liệu đã được xử lý.

## Tài liệu

* [Vấn đề cần giải quyết](docs/Problem.md)
* [Nguồn dữ liệu](docs/Data-Sources.md)
* [Nghiệp vụ dự kiến](docs/Business-Domain.md)
* [Nghiệp vụ harness agent và memory backend](docs/Harness-Agent-Business/README.md)
* [Kiến trúc dự kiến](docs/Architecture.md)
* [Graph schema dự kiến](docs/Graph-Schema.md)
* [Công nghệ sử dụng](docs/Technology-Stack.md)
* [Lộ trình triển khai](docs/Roadmap.md)
* [Kế hoạch đánh giá](docs/Evaluation.md)

## Định hướng đã chốt

* Nghiệp vụ demo: nền tảng phân tích dữ liệu issue/ticket tracking cho IT Helpdesk, developer support hoặc AI Support Agent.
* Dữ liệu lõi: [Public Jira Dataset v7](https://zenodo.org/records/15719919), bản open/anonymized, quy mô 5.8GB ZIP với 2.7M issues, 32M changes, 9M comments và 1M issue links.
* Semantic Memory: ưu tiên lấy từ issue title/body/comment/resolution trong Public Jira; có thể bổ sung Stack Exchange subset nếu cần knowledge base lớn hơn.
* Episodic Memory: ưu tiên lấy từ issue lifecycle, changelog, status change, assignee change, comment event và issue link trong Public Jira.
* Hướng mở rộng tương lai: GH Archive dùng để mô phỏng ingestion theo event stream nếu còn thời gian.
* Kiến trúc dữ liệu: mini-lakehouse / lakehouse-style architecture theo mô hình Bronze - Silver - Gold.
* Storage: MinIO + Parquet, chưa đưa Delta Lake hoặc Apache Iceberg vào phạm vi triển khai chính.
* Graph layer: Neo4j với graph schema dự kiến, điều chỉnh theo cấu trúc thật sau EDA.
* Graph mining: phân tích lifecycle issue/ticket, status bottleneck, issue links, assignee/project/component và cụm vấn đề liên quan.
* Demo: Streamlit + truy vấn Neo4j/GraphRAG ở mức Proof of Concept.
