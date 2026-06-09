# Tài liệu chuẩn bị Phỏng vấn - Dự án Udemy Backend (Junior Perspective)

Tài liệu này được điều chỉnh để phù hợp với vị trí **Junior Backend Developer**, tập trung vào việc thực thi, học hỏi và giải quyết các vấn đề kỹ thuật cụ thể trong một hệ thống lớn.

---

## 1. Giới thiệu Tổng quan Dự án

**Dự án**: Udemy Clone Backend
**Mô hình**: Microservices Architecture
**Công nghệ sử dụng**: .NET 8, Clean Architecture, PostgreSQL, RabbitMQ (MassTransit), ElasticSearch, Docker.

**Mô tả ngắn gọn**: Em tham gia phát triển hệ thống Backend cho nền tảng học trực tuyến Udemy. Dự án giúp em làm quen với việc làm việc trong môi trường Microservices, hiểu cách các dịch vụ tách biệt giao tiếp với nhau và cách tổ chức code theo Clean Architecture để dễ bảo trì.

---

## 2. Vai trò của em trong dự án

Với vai trò là **Junior Backend Developer**, em đã đóng góp vào các phần sau:
- Tham gia phát triển các tính năng trong **Course Service** (Quản lý bài học, danh mục) và **Payment Service** (Xử lý giỏ hàng).
- Thực hiện các CRUD nâng cao và viết logic nghiệp vụ dựa trên các Interface đã được định nghĩa.
- Tìm hiểu và triển khai tích hợp các dịch vụ bên thứ ba như cổng thanh toán **Iyzipay** và công cụ tìm kiếm **ElasticSearch**.
- Viết Unit Test và Integration Test để đảm bảo chất lượng mã nguồn.

---

## 3. Các Task em thấy khó và Cách em xử lý

### **Thử thách 1: Hiểu và áp dụng Clean Architecture**
- **Vấn đề**: Lúc đầu em thấy bối rối vì một chức năng đơn giản nhưng phải đi qua nhiều lớp (Domain, Application, Infrastructure, API). Em chưa hiểu tại sao phải chia nhỏ như vậy.
- **Cách xử lý**:
    - Em đã dành thời gian đọc mã nguồn các phần đã có sẵn và tham khảo tài liệu về Clean Architecture.
    - Em nhận ra việc tách lớp giúp code không bị phụ thuộc vào Framework (như EF Core hay các thư viện bên thứ 3). Khi em cần thay đổi logic thanh toán trong `IyzipayRepository`, em chỉ cần sửa ở lớp Infrastructure mà không làm ảnh hưởng đến logic nghiệp vụ ở lớp Application.

### **Thử thách 2: Xử lý sự kiện bất đồng bộ với MassTransit**
- **Vấn đề**: Đây là lần đầu em làm việc với Message Broker. Em gặp khó khăn khi xử lý việc xác thực người dùng giữa các service.
- **Cách xử lý**:
    - Em đã học cách sử dụng **Request-Response pattern** trong MassTransit.
    - Em đã triển khai [IdentityRequestedEventHandler](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Auth/Udemy.Auth.Application/Handlers/IdentityRequestedEventHandler.cs) để phản hồi thông tin định danh cho các service khác. Qua task này, em hiểu được tầm quan trọng của việc không chia sẻ chung cơ sở dữ liệu giữa các microservices.

### **Thử thách 3: Tích hợp cổng thanh toán Iyzipay**
- **Vấn đề**: API của Iyzipay có nhiều tham số và quy trình xác thực phức tạp (3D Secure). Việc map dữ liệu từ hệ thống sang format của đối tác dễ xảy ra sai sót.
- **Cách xử lý**:
    - Em đã đọc kỹ tài liệu API của đối tác và xây dựng các lớp Helper như [IyzipayUtils](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Payment/Udemy.Payment.Domain/Utils/IyzipayUtils.cs) để chuẩn hóa việc tạo Request.
    - Em cũng học được cách sử dụng **Logger** để ghi lại các mã lỗi từ phía đối tác, giúp việc debug khi thanh toán thất bại trở nên dễ dàng hơn.

---

## 4. Bài học và Sự phát triển
- **Kỹ năng chuyên môn**: Thành thạo hơn với C# .NET 8, hiểu cách hoạt động của Dependency Injection, Middleware và cách viết code "sạch".
- **Kỹ năng hệ thống**: Hiểu được luồng dữ liệu trong Microservices, cách xử lý lỗi khi một dịch vụ không hoạt động (Retry, Queue).
- **Tư duy**: Thay vì chỉ tập trung vào việc "chạy được", em đã bắt đầu chú trọng vào việc "viết sao cho đúng pattern" để đồng đội dễ review và bảo trì.
