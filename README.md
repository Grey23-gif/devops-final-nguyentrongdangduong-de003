# DevOps Final – Đề 003: Quản lý công việc

Bộ mã nguồn khởi đầu cho bài thi **DevOps Fundamentals – Cuối kỳ**.
Ứng dụng hiển thị **danh sách công việc** lấy từ MySQL thông qua Spring Boot, trả về **dạng JSON** qua Nginx (reverse proxy).

## Kiến trúc

```
Trình duyệt ── http://<IP_máy_chủ>:<APP_PORT>
    ▼
[nginx]  nginx:1.27-alpine        GET /*  → app:8080
    ▼
[app]    Spring Boot 4 (JRE 21)   GET /api/tasks → mysql:3306
    ▼
[mysql]  mysql:8.0                volume mysql_data, KHÔNG public cổng mysql
```

## Cấu trúc thư mục

Đổi tên thư mục/repository thành `devops-final-de003-<họ-tên>` (ví dụ `devops-final-de003-nguyenvana`).
Dấu ★ là phần sinh viên phải làm.

```
devops-final-de003-nguyenvana/     ← Mã nguồn Spring Boot 4 (Java 21, Gradle) – CÓ SẴN
├── src/main/java/com/ptit/devops/task/   ← Controller, Entity, Repository
├── src/main/resources/application.properties
├── build.gradle                   ← Build ra file build/libs/app.jar
├── settings.gradle
├── Dockerfile                     ← Multi-stage build (Gradle + JDK 21, runtime JRE 21)
├── .dockerignore
├── nginx/default.conf             ← Nginx reverse proxy tới app:8080
├── database/init.sql              ← Bảng tasks + 8 bản ghi mẫu – CÓ SẴN
├── docker-compose.yml             ← MySQL + Spring Boot + Nginx
├── .github/workflows/ci-cd.yml    ← Build/test → push GHCR → deploy qua SSH
├── student.env                    ← Thông tin sinh viên và cấu hình triển khai
├── screenshots/                   ← ★ Ảnh minh chứng (Phần 5)
├── submission-content/            ← ★ Toàn bộ tệp minh chứng khi nộp bài
├── .gitignore                     ← CÓ SẴN
└── README.md
```

Phần **sinh viên phải làm** (xem đề thi):

| File | Việc cần làm |
|------|--------------|
| `student.env` | Điền thông tin sinh viên và thông số triển khai |
| `screenshots/` | `01-actions.png`, `02-ghcr.png`, `03-compose-ps.png`, `04-website.png` |

## CI/CD

Workflow chạy khi push lên nhánh `main` hoặc được khởi chạy thủ công. Nó build và smoke-test toàn bộ stack bằng Docker Compose, push image có tag commit SHA và `latest` lên GHCR, rồi deploy qua SSH. Cấu hình các GitHub Actions Secrets `SERVER_HOST`, `SERVER_USER`, và `SERVER_SSH_KEY` trước khi chạy workflow. Máy chủ deploy cần Git, Docker Compose và quyền truy cập package GHCR của repository.

## API của ứng dụng

| Method | Đường dẫn | Mô tả |
|--------|-----------|-------|
| GET | `/api/tasks` | Danh sách công việc |

## Biến môi trường mà ứng dụng cần

| Biến | Ý nghĩa |
|------|---------|
| `DB_HOST`, `DB_PORT` | Địa chỉ MySQL (mặc định `mysql`, `3306`) |
| `DB_NAME`, `DB_USER`, `DB_PASSWORD` | Database và tài khoản kết nối |
| `APP_VERSION` | Phiên bản (truyền vào lúc build image, ví dụ commit SHA) |

## Chạy thử

```bash
docker compose --env-file student.env up -d --build
docker compose --env-file student.env ps
curl http://localhost:<APP_PORT>/api/tasks
docker compose --env-file student.env down
```
