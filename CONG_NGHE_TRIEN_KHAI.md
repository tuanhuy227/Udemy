# Triển khai JWT, RabbitMQ và gRPC trong dự án Udemy Backend

Tài liệu này giải thích cách hệ thống triển khai các công nghệ cốt lõi để đảm bảo bảo mật và giao tiếp hiệu quả giữa các Microservices.

---

## 1. Triển khai JWT (Authentication & Authorization)

Hệ thống sử dụng chiến lược **Xác thực tập trung tại Gateway** kết hợp với **Xác thực nội bộ qua HMAC**.

### **Tại Auth Service (Cấp phát Token)**:
- Sử dụng thư viện `Microsoft.AspNetCore.Identity` để quản lý người dùng.
- Cấu hình tại [DependencyInjection.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Auth/Udemy.Auth.Infrastructure/DependencyInjection.cs):
    - `AddBearerToken`: Cấp phát Access Token (hạn 15 phút) và Refresh Token (hạn 30 ngày).
    - `AddDataProtection`: Mã hóa các token và dữ liệu nhạy cảm.

### **Tại API Gateway (Xác thực tập trung)**:
- [IdentityMiddleware.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.APIGateway/Udemy.APIGateway.API/Middlewares/IdentityMiddleware.cs) chặn mọi request:
    1. Lấy Bearer Token từ Header.
    2. Gửi yêu cầu xác thực sang Auth Service qua RabbitMQ (Request-Response).
    3. Sau khi xác thực thành công, Gateway sẽ gắn thông tin User vào Header (`X-User-Id`, `X-User-Roles`, ...) và ký tên bằng một mã băm **HMAC (X-User-Secret)** để đảm bảo các Microservices phía sau không bị giả mạo Header.

### **Tại các Microservices (Xác thực nội bộ)**:
- Sử dụng [AuthAuthenticationHandler.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Common/Udemy.Common/Middlewares/AuthAuthenticationHandler.cs):
    - Kiểm tra `X-Api-Key` và xác minh mã băm `X-User-Secret` bằng mã `SALT` chung.
    - Chuyển đổi thông tin từ Header thành `ClaimsPrincipal` để sử dụng các Attribute như `[Authorize]`.

---

## 2. Triển khai RabbitMQ (MassTransit)

Hệ thống sử dụng **MassTransit** làm lớp trừu tượng (Abstraction) trên nền RabbitMQ để xử lý giao tiếp bất đồng bộ.

### **Cấu hình dùng chung**:
- Được đóng gói trong [AddMassTransit.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Common/Udemy.Common/Extensions/AddMassTransit.cs).
- Tự động đăng ký các **Consumers** từ Assembly của từng dịch vụ.
- Sử dụng `InMemoryOutbox` để đảm bảo tin nhắn không bị mất khi có sự cố mạng.

### **Các kịch bản sử dụng**:
1. **Request-Response (Đồng bộ qua Message Broker)**:
    - Gateway yêu cầu thông tin User từ Auth Service qua `IdentityRequestedEvent`.
2. **Publish-Subscribe (Bất đồng bộ)**:
    - Khi thanh toán thành công, Payment Service phát đi `EnrollmentPaidEvent`.
    - Course Service lắng nghe sự kiện này để tự động ghi danh học viên vào khóa học.

---

## 3. Triển khai gRPC

*Lưu ý: Qua phân tích mã nguồn hiện tại, hệ thống chủ yếu sử dụng MassTransit (RabbitMQ) cho giao tiếp nội bộ thay vì gRPC thuần túy như đề cập trong một số tài liệu hướng dẫn.*

Tuy nhiên, trong kiến trúc này, **MassTransit** đóng vai trò thay thế cho gRPC ở nhiều khía cạnh:
- **Hiệu suất**: RabbitMQ đảm bảo tin nhắn được phân phối tin cậy ngay cả khi dịch vụ nhận đang offline.
- **Tính linh hoạt**: Dễ dàng triển khai các pattern phức tạp như Saga, Retry, và Dead Letter Queue mà gRPC thuần túy đòi hỏi nhiều công sức cấu hình hơn.

---

## Tóm tắt luồng dữ liệu bảo mật
1. **User** -> [JWT] -> **API Gateway**.
2. **API Gateway** -> [RabbitMQ Request] -> **Auth Service** (Xác thực Token).
3. **Auth Service** -> [RabbitMQ Response] -> **API Gateway** (Trả về User Info).
4. **API Gateway** -> [Headers + HMAC Secret] -> **Microservices** (Xử lý nghiệp vụ).
