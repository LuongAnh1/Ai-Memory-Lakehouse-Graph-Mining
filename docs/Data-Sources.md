# Nguồn Dữ Liệu

## 1. Kết Luận Chọn Dữ Liệu

Nguồn dữ liệu chính thức của đồ án là **Public Jira Dataset v7**.

Lý do chọn nguồn này:

* đủ lớn để thể hiện năng lực Data Engineering, với file ZIP khoảng 5.8GB;
* có provenance rõ ràng, DOI và trang phát hành công khai trên Zenodo;
* là dữ liệu issue tracking thật từ các public Jira repositories;
* có cả dữ liệu văn bản và dữ liệu sự kiện theo thời gian;
* phù hợp tự nhiên với mô hình Semantic Memory, Episodic Memory và Knowledge Graph;
* bản v7 đã được anonymized, phù hợp hơn so với các bản cũ từng bị restricted do vấn đề personal information.

Nguồn: [The Public Jira Dataset v7 - Zenodo](https://zenodo.org/records/15719919)

## 2. Vai Trò Của Từng Nguồn

| Nguồn | Vai trò trong đồ án | Trạng thái |
| --- | --- | --- |
| [Public Jira Dataset v7](https://zenodo.org/records/15719919) | dữ liệu lõi cho pipeline, lakehouse, graph mining, analytics và demo | đã chốt |
| [Stack Exchange Data Dump](https://meta.stackexchange.com/questions/224873/all-stack-exchange-data-dump-releases) | bổ sung Semantic Memory nếu cần knowledge base lớn hơn | tùy chọn |
| [GH Archive](https://www.gharchive.org/) | mở rộng event-stream ingestion trong giai đoạn sau | dự phòng/mở rộng |
| [TAWOS](https://github.com/SOLAR-group/TAWOS) | dataset Jira nhỏ hơn để thử nghiệm nhanh nếu cần | phụ |
| [Hugging Face `s2pidape/support-ticket-dataset`](https://huggingface.co/datasets/s2pidape/support-ticket-dataset) | synthetic support tickets quy mô lớn, dùng để smoke test pipeline hoặc so sánh | phụ |
| [Kaggle Customer IT Support - Ticket Dataset](https://www.kaggle.com/dsv/10323677) | synthetic email/support tickets có label queue, priority, type, tags | phụ |
| [Kaggle Customer Support Enhancing Efficiency](https://www.kaggle.com/datasets/suvroo/customer-support-enhancing-efficiency/versions/1) | dataset nhỏ về AI-driven customer support, dùng tham khảo schema | phụ |
| [Datanemics Customer Support Tickets](https://datanemics.com/datasets/support-tickets/) | 4.000 support tickets, dùng để smoke test nhanh | phụ |

## 3. Public Jira Dataset v7

Public Jira Dataset v7 là dataset issue tracking công khai, được thu thập từ các Jira repositories bằng Jira API. Dataset gồm:

* 16 public Jira repositories;
* 1.822 projects;
* 2.7 triệu issues;
* 32 triệu change records;
* 9 triệu comments;
* 1 triệu issue links;
* file ZIP khoảng 5.8GB;
* dữ liệu người dùng đã được anonymized bằng UUID4 masks.

Dataset này phù hợp với đồ án vì một issue có thể chứa đồng thời:

* **Semantic Memory**: title, description/body, comment, resolution text, label, component;
* **Episodic Memory**: created time, updated time, status change, assignment change, comment event, issue link;
* **Graph Mining**: issue-project, issue-comment, issue-status, issue-actor, issue-component, issue-link.

## 4. Stack Exchange Là Semantic Extension

Stack Exchange không phải dữ liệu lõi của đồ án. Nguồn này chỉ nên dùng nếu sau khi xử lý Public Jira, phần Semantic Memory vẫn chưa đủ phong phú.

Vai trò phù hợp:

* bổ sung Q&A posts làm knowledge base;
* tạo các node `Post`, `Question`, `Answer`, `Tag`, `Chunk`;
* liên kết issue với bài Q&A theo keyword, tag hoặc similarity search.

Lưu ý:

* license thường theo CC BY-SA, cần attribution rõ ràng;
* full dump rất lớn, nên chỉ chọn subset theo site hoặc chủ đề;
* không nên đưa vào giai đoạn đầu nếu pipeline Public Jira chưa ổn.

## 5. GH Archive Là Hướng Mở Rộng Event Stream

GH Archive không làm dữ liệu lõi trong phiên bản đầu vì phạm vi rộng và nhiễu hơn Public Jira. Tuy nhiên, đây là nguồn tốt để mô phỏng ingestion theo thời gian nếu đồ án còn thời gian mở rộng.

Vai trò phù hợp:

* tải event theo giờ/ngày;
* lọc `IssuesEvent`, `IssueCommentEvent`, `PullRequestEvent`;
* mô phỏng incremental ingestion;
* so sánh batch dataset với event-stream dataset.

## 6. Vì Sao Không Chọn Dataset Ticket Nhỏ Làm Lõi?

Các dataset phụ từ Hugging Face, Kaggle hoặc Datanemics có thể dùng để smoke test pipeline, nhưng không nên làm lõi vì:

* quy mô thường nhỏ hơn yêu cầu 2-3GB;
* có khả năng synthetic hoặc provenance chưa mạnh;
* thiếu changelog/lifecycle đầy đủ;
* thiếu quan hệ issue-link hoặc project/component rõ ràng;
* khó bảo vệ độ nghiêm túc của đồ án Data Engineering.

Riêng Hugging Face `s2pidape/support-ticket-dataset` có quy mô lớn hơn các dataset phụ còn lại, nhưng vẫn là synthetic dataset nên chỉ nên dùng làm nguồn đối chiếu hoặc thử pipeline, không thay thế Public Jira Dataset v7.

## 7. Có Nên Cào Dữ Liệu Không?

Có thể cào dữ liệu nếu nguồn cho phép, nhưng với đồ án này không nên ưu tiên scraping HTML tự do. Hướng an toàn hơn là dùng public dump hoặc official API.

Nếu phải cào dữ liệu, cần tuân thủ:

* đọc và tôn trọng Terms of Service;
* tôn trọng robots.txt và rate limit;
* không bypass login, CAPTCHA hoặc cơ chế chống scraping;
* không thu thập dữ liệu cá nhân nhạy cảm;
* lưu metadata về nguồn, thời gian tải và điều kiện sử dụng;
* anonymize hoặc hash định danh nếu cần.

Với quyết định hiện tại, Public Jira Dataset v7 đã đủ lớn và đủ phù hợp nên scraping chỉ còn là phương án phụ.

## 8. Tác Động Tới Thiết Kế

Vì dữ liệu lõi đã chốt là Public Jira Dataset v7:

* Bronze layer sẽ lưu raw ZIP/MongoDB dump hoặc dữ liệu extract nguyên bản từ Public Jira.
* Silver layer ưu tiên các bảng `issues`, `comments`, `change_events`, `projects`, `actors`, `components`, `labels`, `issue_links`.
* Gold layer trong Neo4j xoay quanh `Issue`, `Project`, `Actor`, `Comment`, `Status`, `Component`, `Label`, `ChangeEvent`, `IssueLink`.
* Semantic Memory lấy trực tiếp từ title/body/comment/resolution trước khi dùng nguồn ngoài.
* Episodic Memory lấy từ changelog, status transition, assignment event, comment timeline và issue links.
* Graph schema chi tiết vẫn chốt sau EDA vì cần kiểm tra cấu trúc dump thực tế.
