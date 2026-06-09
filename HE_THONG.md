# Cấu trúc Hệ thống Udemy Backend (Microservices)

Tài liệu này giải thích chi tiết cấu trúc kiến trúc của hệ thống Udemy Backend, được xây dựng trên nền tảng .NET 8 với kiến trúc Microservices và Clean Architecture.

---

## 1. Tổng quan Kiến trúc (Architecture Overview)

Hệ thống được thiết kế theo mô hình **Microservices**, chia nhỏ các chức năng nghiệp vụ thành các dịch vụ độc lập. Mỗi dịch vụ có cơ sở dữ liệu riêng và giao tiếp với nhau thông qua cơ chế đồng bộ (REST API) hoặc bất đồng bộ (Message Broker).

### Các thành phần chính:
- **API Gateway**: Điểm đầu vào duy nhất cho các client, chịu trách nhiệm điều phối request.
- **Identity Service (Auth)**: Quản lý người dùng, đăng ký, đăng nhập và phân quyền (JWT).
- **Course Service**: Quản lý khóa học, bài học, bình luận, đánh giá và ghi danh.
- **Payment Service**: Xử lý thanh toán, tích hợp cổng thanh toán (Iyzipay).
- **CDN Service**: Quản lý lưu trữ tệp tin và video (sử dụng Minio).
- **Common Library**: Thư viện dùng chung chứa các thành phần cốt lõi, middleware và sự kiện.

---

## 2. Cấu trúc một Microservice (Clean Architecture)

Mỗi dịch vụ (như `Udemy.Course`, `Udemy.Auth`) được tổ chức theo các lớp của **Clean Architecture**:

- **`.API`**:
  - Chứa các `Controllers` để tiếp nhận request.
  - Cấu hình Dependency Injection (`Program.cs`).
  - Cấu hình Middleware và các file settings (`appsettings.json`).
- **`.Application`**:
  - Chứa logic nghiệp vụ (`Services`).
  - Các trình xử lý sự kiện (`Handlers`) cho MassTransit.
  - Các trình xác thực dữ liệu (`Validators`).
- **`.Domain`**:
  - Chứa các thực thể chính (`Entities`).
  - Các giao diện (`Interfaces`) cho Repository và Service.
  - Các hằng số, `Enums` và `DTOs`.
- **`.Infrastructure`**:
  - Triển khai các Repository (`Repositories`).
  - Cấu hình cơ sở dữ liệu (`DbContext`, `Configurations`).
  - Quản lý di trú dữ liệu (`Migrations`).
  - Tích hợp các dịch vụ bên ngoài (ElasticSearch, Minio, v.v.).

---

## 3. Thư viện dùng chung (Udemy.Common)

Đây là thành phần quan trọng giúp tái sử dụng mã nguồn và đảm bảo tính nhất quán giữa các dịch vụ:

- **Base**: Các lớp cơ sở như `BaseEntity`, `BaseRepository`.
- **Consul**: Hỗ trợ Service Discovery (đăng ký và tìm kiếm dịch vụ).
- **Events**: Định nghĩa các sự kiện (Events) để giao tiếp bất đồng bộ qua MassTransit (ví dụ: `EnrollmentPaidEvent`).
- **ExceptionMiddlewares**: Xử lý lỗi tập trung (`GlobalExceptionHandler`).
- **Extensions**: Các phương thức mở rộng để đăng ký nhanh MassTransit, API Versioning, v.v.
- **Middlewares**: Các bộ lọc trung gian như `AuthMiddleware`, `TransactionMiddleware`.

---

## 4. Công nghệ sử dụng (Tech Stack)

- **Ngôn ngữ**: C# (.NET 8)
- **Cơ sở dữ liệu**:
  - **PostgreSQL**: Lưu trữ dữ liệu quan hệ chính.
  - **ElasticSearch**: Hỗ trợ tìm kiếm khóa học nâng cao.
- **Giao tiếp**:
  - **MassTransit**: Message Broker (RabbitMQ) để giao tiếp bất đồng bộ.
  - **Consul**: Service Discovery.
- **Lưu trữ**: **Minio** (tương thích S3) cho tệp tin và video.
- **Triển khai**: **Docker & Docker Compose**.

---

## 5. Luồng hoạt động tiêu biểu

1. **Request**: Client gửi yêu cầu đến `APIGateway`.
2. **Routing**: Gateway kiểm tra quyền truy cập (qua `Auth`) và chuyển hướng đến dịch vụ tương ứng (ví dụ: `Course Service`).
3. **Processing**: Dịch vụ xử lý logic, tương tác với DB.
4. **Events**: Nếu có hành động cần thông báo cho dịch vụ khác (ví dụ: Thanh toán thành công), một sự kiện sẽ được gửi vào Message Broker.
5. **Consumption**: Dịch vụ liên quan (ví dụ: `Course Service`) lắng nghe sự kiện và cập nhật trạng thái (ví dụ: Kích hoạt khóa học cho người dùng).

---

## 6. Các chức năng chi tiết theo từng dịch vụ

### **6.1. Auth Service (Dịch vụ Xác thực)**
- **Đăng ký & Đăng nhập**: Quản lý tài khoản người dùng (Học viên và Giảng viên).
- **Phân quyền (RBAC)**: Cấp phát và xác thực JWT token dựa trên vai trò.
- **Bảo mật dữ liệu**: Sử dụng Data Protection để bảo vệ thông tin cá nhân.
- **Xác thực qua Email**: Tích hợp SMTP để gửi mã xác nhận hoặc khôi phục mật khẩu.

### **6.2. Course Service (Dịch vụ Khóa học)**
- **Quản lý Khóa học**: Tạo, cập nhật, xóa và phê duyệt khóa học (Giảng viên/Admin).
- **Quản lý Nội dung**: Tổ chức các chương mục (Lesson Categories) và bài học (Lessons).
- **Tìm kiếm nâng cao**: Tích hợp **ElasticSearch** để tìm kiếm khóa học theo từ khóa, danh mục, giá, v.v.
- **Hệ thống Tương tác**:
    - **Hỏi & Đáp**: Gửi câu hỏi và trả lời trong từng bài học.
    - **Bình luận & Đánh giá**: Đánh giá khóa học (Rating) và để lại ý kiến.
    - **Yêu thích & Like**: Lưu khóa học vào danh sách yêu thích.
- **Ghi danh (Enrollment)**: Quản lý danh sách học viên đã tham gia khóa học.

### **6.3. Payment Service (Dịch vụ Thanh toán)**
- **Giỏ hàng**: Quản lý các khóa học người dùng dự định mua.
- **Tích hợp Cổng thanh toán**: Kết nối với **Iyzipay** để xử lý giao dịch thẻ.
- **Thanh toán 3D Secure**: Hỗ trợ quy trình xác thực thanh toán an toàn.
- **Hoàn tiền (Refund)**: Xử lý yêu cầu hoàn tiền khi người dùng không hài lòng.
- **Lịch sử giao dịch**: Lưu trữ thông tin chi tiết các đơn hàng đã thanh toán.

### **6.4. CDN Service (Dịch vụ Nội dung số)**
- **Tải lên tệp tin**: Hỗ trợ tải lên video bài giảng, tài liệu đính kèm.
- **Quản lý Storage**: Sử dụng **Minio** để lưu trữ và quản lý quyền truy cập tệp tin.
- **Cung cấp nội dung**: Tạo URL tạm thời hoặc URL công khai để xem video và tải tài liệu.

### **6.5. API Gateway**
- **Định tuyến (Routing)**: Chuyển hướng yêu cầu từ Client đến Microservice tương ứng.
- **Xác thực tập trung**: Kiểm tra tính hợp lệ của Token trước khi gửi đến các dịch vụ nội bộ.
- **Service Discovery**: Kết nối với Consul để tự động tìm kiếm địa chỉ các dịch vụ đang chạy.
