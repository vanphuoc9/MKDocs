---
comments: true
---
# Mô hình kiến trúc dự án
## 1. Mô hình tổng thể

                    +---------------------------+
                    |    Người dùng (Client)    |
                    +---------------------------+
                            |
                +-------------------------------+
                |      Giao diện người dùng     |
                |  - Angular (Web Frontend)     |
                |  - React Native (Mobile App)  |
                +-------------------------------+
                            |
                    Gọi API (HTTPS)
                            |
            +------------------------------------+
            |       API Gateway / Ingress        | ← Kubernetes hoặc NGINX Ingress
            +------------------------------------+
                            |
            +----------------+-------------------+
            |                                    |
    +-------------------+              +--------------------------+
    |     Spring Boot   |              |     Keycloak (SSO)       |
    |  (RESTful API)    |◄────────────►|   Authentication Server  |
    |                   |              +--------------------------+
    +--------+----------+
            |
    +--------+--------+-----------------------+
    |                 |                       |
    |     PostgreSQL  |     MongoDB           |    Redis
    | (Dữ liệu quan hệ)| (Dữ liệu phi quan hệ)| (Cache, session/token)
    +-----------------+-----------------------+

## 2. Các dịch vụ của hệ thống

Dưới đây là các dịch vụ trong hệ thống:

| Tên Dịch vụ         | Mô Tả Chức Năng                                                | Đường Dẫn GitHub                                         |
|---------------------|---------------------------------------------------------------|---------------------------------------------------------|
| **AuthService**      | Xử lý xác thực và ủy quyền người dùng, quản lý session, JWT. Sử dụng keycloack.  |  |
| **UserService**      | Quản lý thông tin người dùng như đăng ký, đăng nhập, chỉnh sửa hồ sơ. |  |
| **MovieService**     | Cung cấp thông tin phim như tên, mô tả, thể loại, trailer, v.v.  | [https://github.com/vanphuoc9/movie.git](https://github.com/vanphuoc9/movie.git) |
| **FileService**     | Cung cấp dịch vụ download file, upload file, delete file...  | [https://github.com/vanphuoc9/file.git](https://github.com/vanphuoc9/file.git) |
| **ReviewService**    | Quản lý đánh giá và bình luận của người dùng về các bộ phim.    |  |
| **RatingService**    | Quản lý hệ thống đánh giá (stars, like/dislike) của các bộ phim. |  |
| **PaymentService**   | Xử lý giao dịch thanh toán cho thuê hoặc mua phim.             |  |
| **NotificationService** | Gửi thông báo qua email, SMS hoặc thông báo đẩy cho người dùng. | |
