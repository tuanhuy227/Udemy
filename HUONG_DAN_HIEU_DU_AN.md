# Hướng dẫn Tìm hiểu Sâu Dự án Udemy Backend

Để hiểu kĩ một dự án Microservices phức tạp như thế này, bạn nên tiếp cận theo lộ trình từ "ngoài vào trong" và từ "tổng quan đến chi tiết". Dưới đây là 5 giai đoạn để bạn làm chủ mã nguồn này.

---

## Giai đoạn 1: Hiểu "Bản đồ" Hệ thống (The Big Picture)

Trước khi đọc code, hãy nắm vững cách các mảnh ghép khớp với nhau:
1.  **Đọc tài liệu tổng quan**: Xem lại [HE_THONG.md](file:///Users/tuanhuytran/Downloads/Udemy_Backend/HE_THONG.md) để biết các Microservices làm gì.
2.  **Xem cấu trúc thư mục**: Nhận diện mô hình **Clean Architecture**. Mỗi Service đều có 4 lớp: `.API`, `.Application`, `.Domain`, `.Infrastructure`.
3.  **Cơ sở dữ liệu**: Đọc [DATABASE.md](file:///Users/tuanhuytran/Downloads/Udemy_Backend/DATABASE.md) để hiểu các thực thể nghiệp vụ (Entities) và mối quan hệ giữa chúng.

---

## Giai đoạn 2: Luyện tập "Đi theo dòng Request" (Trace the Flow)

Cách tốt nhất để hiểu code là chọn một chức năng và đi theo nó từ đầu đến cuối.
**Ví dụ: Chức năng tạo khóa học (`CreateCourse`)**
1.  **Entry Point**: Bắt đầu tại [CourseController.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Course/Udemy.Course.API/Controllers/CourseController.cs). Xem cách nó nhận Request.
2.  **Application Logic**: Xem [CourseService.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Course/Udemy.Course.Application/Services/CourseService.cs) xử lý nghiệp vụ, xác thực dữ liệu.
3.  **Data Persistence**: Xem [CourseRepository.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Course/Udemy.Course.Infrastructure/Repositories/CourseRepository.cs) lưu dữ liệu vào PostgreSQL như thế nào.

---

## Giai đoạn 3: Tìm hiểu "Chất keo" kết nối (The Glue)

Trong Microservices, giao tiếp là quan trọng nhất.
1.  **Thư viện dùng chung**: Khám phá [Udemy.Common](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Common/Udemy.Common). Đây là nơi chứa các logic lặp lại (xử lý lỗi, middleware, base classes). Nếu bạn hiểu file này, bạn đã hiểu 30% toàn bộ project.
2.  **Giao tiếp bất đồng bộ**: Đọc [CHI_TIET_MASSTRANSIT.md](file:///Users/tuanhuytran/Downloads/Udemy_Backend/CHI_TIET_MASSTRANSIT.md). Hiểu cách `IdentityRequestedEvent` giúp các service nói chuyện với nhau mà không bị phụ thuộc cứng.

---

## Giai đoạn 4: Đào sâu vào các Thử thách Kỹ thuật

Hãy xem cách dự án giải quyết các vấn đề khó:
1.  **Tìm kiếm**: Xem cách [ElasticSearchRepository.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Course/Udemy.Course.Infrastructure/Repositories/ElasticSearchRepository.cs) được triển khai để tăng tốc tìm kiếm.
2.  **Thanh toán**: Xem [IyzipayRepository.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Payment/Udemy.Payment.Infrastructure/Repositories/IyzipayRepository.cs) để hiểu quy trình tích hợp bên thứ ba phức tạp.
3.  **Bảo mật**: Xem cách dùng [DataProtectionProvider.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Auth/Udemy.Auth.Infrastructure/DataProtection/DataProtectionProvider.cs) trong Auth Service.

---

## Giai đoạn 5: Thử nghiệm và Mở rộng (Hands-on)

Đừng chỉ đọc, hãy thử thay đổi:
1.  **Đặt Breakpoint**: Chạy dự án bằng Docker, đặt breakpoint trong Controller và dùng Postman gọi API để xem dữ liệu biến đổi qua từng lớp.
2.  **Thêm tính năng nhỏ**: Thử thêm một trường mới vào `Course` Entity, chạy Migration và cập nhật API.
3.  **Đọc Test**: Xem các tệp trong [Udemy.Tests](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Tests) để biết cách hệ thống được kiểm chứng tự động.

---

### Mẹo nhỏ cho Junior:
- **Đừng cố hiểu hết cùng một lúc**: Hãy tập trung vào một Service (ví dụ `Udemy.Course`) cho đến khi thực sự hiểu nó, rồi mới chuyển sang service tiếp theo.
- **Sử dụng tài liệu chuẩn bị phỏng vấn**: Đọc [PHONG_VAN.md](file:///Users/tuanhuytran/Downloads/Udemy_Backend/PHONG_VAN.md) để biết các "điểm nóng" kỹ thuật mà bạn nên chú ý.
