# Bài 5: Docker Compose và File .env

## 1. Mục tiêu

- Sử dụng Docker Compose V2 để quản lý nhiều container.
- Khai báo dịch vụ bằng Infrastructure as Code (IaC).
- Tách cấu hình và thông tin đăng nhập ra khỏi file Compose.
- Tạo Docker network và named volume tự động.
- Kiểm tra kết nối nội bộ giữa Backend và PostgreSQL.

## 2. Kiến trúc

```text
Docker Compose
      |
      +-- backend (app-container)
      |       |
      |       | DB_HOST=database
      |       |
      +-------+------------------+
              backend-net        |
                                 |
                         database (db-server)
                                 |
                              pgdata
                                 |
                   /var/lib/postgresql/data
```

## 3. Các file trong bài

| File | Chức năng |
|---|---|
| docker-compose.yml | Khai báo services, networks, volumes |
| .env | Cấu hình môi trường thực tế, không commit |
| .env.example | Mẫu cấu hình môi trường |
| .gitignore | Loại trừ file chứa thông tin nhạy cảm |
| README.md | Tài liệu hướng dẫn |

## 4. Cấu hình dịch vụ

### Database

- Image: `postgres:15-alpine`
- Container: `db-server`
- Database và tài khoản được cấu hình qua biến môi trường.
- Named volume `pgdata` lưu trữ dữ liệu.
- Healthcheck kiểm tra trạng thái sẵn sàng của PostgreSQL.

### Backend

- Container: `app-container`
- Kết nối Database qua hostname `database`.
- Sử dụng `depends_on` với điều kiện `service_healthy`.
- Tham gia mạng nội bộ `backend-net`.

Trong bài này, Backend là container Alpine dùng để kiểm thử mạng, chưa phải ứng dụng Web API thực tế.

## 5. Cách triển khai trên VPS

Yêu cầu Docker Engine và Docker Compose V2.

Tạo file cấu hình môi trường:

```bash
cp .env.example .env
nano .env
chmod 600 .env
```

Thay mật khẩu mẫu bằng mật khẩu riêng trước khi triển khai.

Kiểm tra cấu hình:

```bash
docker compose config
```

Khởi chạy các dịch vụ:

```bash
docker compose up -d
```

Kiểm tra trạng thái:

```bash
docker compose ps
```

Xem log:

```bash
docker compose logs -f
```

Nhấn `Ctrl + C` để thoát theo dõi log; các container vẫn tiếp tục chạy.

## 6. Kiểm tra kết nối

Kiểm tra phân giải DNS của Database từ Backend:

```bash
docker compose exec backend getent hosts database
```

Kiểm tra kết nối TCP tới PostgreSQL:

```bash
docker compose exec backend sh -c \
  'nc -z -w 3 database 5432'
```

Nếu lệnh `getent` hoặc `nc` không có trong image Alpine, có thể kiểm tra trực tiếp bằng:

```bash
docker compose exec database \
  pg_isready -U appuser -d appdb
```

## 7. Kiểm tra Network và Volume

```bash
docker network ls
docker volume ls
docker compose config --volumes
```

Docker Compose tự tạo network và volume khi khởi chạy dự án.

## 8. Dừng dịch vụ

```bash
docker compose down
```

Lệnh trên xóa các container và network do dự án tạo ra, nhưng giữ named volume mặc định.

Không sử dụng `docker compose down -v` nếu muốn giữ dữ liệu PostgreSQL.

## 9. Kết quả mong đợi

- Docker Compose đọc thành công cấu hình.
- Hai dịch vụ `backend` và `database` chạy.
- PostgreSQL đạt trạng thái healthy.
- Backend phân giải được hostname `database`.
- Network `backend-net` và volume `pgdata` được tạo.
- File `.env` không được đưa lên GitHub.

## 10. Kết luận

Docker Compose giúp khai báo và tự động hóa triển khai nhiều container bằng một file cấu hình. File `.env` hỗ trợ tách cấu hình môi trường khỏi mã nguồn, còn `.gitignore` giúp tránh commit thông tin đăng nhập.

Lưu ý: Alpine trong bài là container kiểm thử mạng. Khi có ứng dụng Backend thực tế, cần thay image và lệnh khởi chạy tương ứng với ứng dụng đó.