# Lộ Trình Triển Khai

## 1. Mục Tiêu Theo Giai Đoạn

Lộ trình được thiết kế cho khoảng 12 tuần. Dữ liệu lõi đã chốt là **Public Jira Dataset v7**, nhưng schema chi tiết vẫn cần EDA trước khi triển khai pipeline đầy đủ.

## 2. Phase 0: Data Access Và EDA

Thời gian dự kiến: tuần 1.

Mục tiêu:

* tải hoặc chuẩn bị sample từ Public Jira Dataset v7;
* lưu metadata nguồn: URL, DOI, version, ngày tải, license/terms;
* kiểm tra cấu trúc ZIP/MongoDB dump;
* xác định collection/file chính;
* kiểm tra cột, kiểu dữ liệu, nested fields, missing values;
* xác định grain: issue, comment, changelog event, issue link, project hay actor;
* ước lượng dung lượng raw/Parquet;
* quyết định Silver schema và graph schema phiên bản đầu.

Deliverables:

* data source chính thức: Public Jira Dataset v7;
* EDA notebook/report ngắn;
* data dictionary ban đầu;
* kế hoạch partition;
* phiên bản đầu của Silver schema và graph schema.

## 3. Phase 1: Setup Và Mini-PoC

Thời gian dự kiến: tuần 2-3.

Mục tiêu:

* dựng Docker Compose cho MinIO và Neo4j;
* tạo cấu trúc thư mục dự án;
* ingest sample nhỏ của Public Jira vào Bronze;
* extract sample thành bảng trung gian;
* tạo thử graph mini trong Neo4j.

Deliverables:

* `docker-compose.yml`;
* bucket/prefix Bronze mẫu;
* script kết nối MinIO;
* script kết nối Neo4j;
* graph nhỏ dựa trên field thật của Public Jira;
* quy ước partition theo source/version/project hoặc thời gian.

## 4. Phase 2: Data Pipeline Và Silver Layer

Thời gian dự kiến: tuần 4-6.

Mục tiêu:

* viết pipeline đọc dữ liệu từ Bronze;
* chuẩn hóa issues, comments, change events, projects, actors, labels/components và issue links;
* tách Semantic Memory từ issue title/body/comment/resolution;
* tách Episodic Memory từ changelog và timeline;
* validate schema;
* ghi dữ liệu sạch sang Parquet.

Deliverables:

* các file Parquet ở Silver theo schema sau EDA;
* log kết quả pipeline;
* báo cáo số bản ghi hợp lệ/lỗi;
* thống kê dung lượng raw và Parquet.

## 5. Phase 3: Knowledge Graph

Thời gian dự kiến: tuần 7-8.

Mục tiêu:

* hiện thực graph schema đã cập nhật sau EDA;
* viết graph loader từ Silver Parquet vào Neo4j;
* tạo constraints/indexes cần thiết;
* tạo node và relationship;
* viết bộ Cypher query kiểm tra dữ liệu.

Deliverables:

* graph schema và mapping từ field dữ liệu sang node/edge;
* script load graph;
* Cypher queries mẫu;
* ảnh chụp Neo4j Browser;
* thống kê số node/edge.

## 6. Phase 4: Graph Mining Và Analytics

Thời gian dự kiến: tuần 9-10.

Mục tiêu:

* chạy Degree Centrality hoặc PageRank để tìm issue/project/actor/component quan trọng;
* chạy Community Detection nếu graph đủ phong phú;
* phân tích đường đi từ issue tới linked issue, comment, status hoặc chunk;
* phân tích bottleneck theo thời gian ở từng status;
* tạo bảng kết quả phục vụ báo cáo.

Deliverables:

* top project/component/status/actor/issue;
* nhóm issue liên quan;
* ví dụ path từ issue tới linked issue hoặc knowledge chunk;
* nhận xét ý nghĩa nghiệp vụ.

## 7. Phase 5: Demo GraphRAG Và Dashboard

Thời gian dự kiến: tuần 11.

Mục tiêu:

* xây dashboard Streamlit;
* hiển thị thống kê pipeline và graph;
* cho phép truy vấn theo issue id, project, component, label/status hoặc actor;
* hiển thị subgraph/context liên quan;
* tùy chọn gọi LLM để sinh câu trả lời từ context.

Deliverables:

* app Streamlit;
* các câu hỏi demo dựa trên giá trị thật trong Public Jira;
* ảnh chụp UI;
* so sánh ngắn giữa truy xuất text baseline và truy xuất theo graph.

## 8. Phase 6: Hoàn Thiện Báo Cáo

Thời gian dự kiến: tuần 12.

Mục tiêu:

* hoàn thiện báo cáo;
* chuẩn hóa hình vẽ kiến trúc;
* tổng hợp kết quả đánh giá;
* chuẩn bị slide bảo vệ;
* ghi rõ giới hạn và hướng mở rộng.

Deliverables:

* báo cáo cuối;
* slide bảo vệ;
* bảng đánh giá;
* source code demo;
* hướng dẫn tải Public Jira Dataset v7 và tái chạy pipeline.

## 9. Nguyên Tắc Kiểm Soát Scope

Ưu tiên theo thứ tự:

1. Public Jira sample đã EDA và có data dictionary.
2. Pipeline chạy được end-to-end trên sample.
3. Dữ liệu Silver có schema rõ ràng.
4. Neo4j graph có node/edge đúng với dữ liệu thật.
5. Có kết quả graph mining giải thích được.
6. Có dashboard/demo cơ bản.
7. Stack Exchange/GH Archive chỉ làm khi các phần lõi đã ổn.
8. LLM/GraphRAG chỉ là demo sau khi dữ liệu và graph ổn định.
