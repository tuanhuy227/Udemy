# Cấu trúc Cơ sở dữ liệu (Database Schema)

Hệ thống sử dụng mô hình **Database-per-Service**, trong đó mỗi Microservice quản lý cơ sở dữ liệu riêng của mình để đảm bảo tính độc lập.

---

## 1. Auth Service (Schema: `identity`)

Dịch vụ này sử dụng ASP.NET Core Identity để quản lý người dùng và phân quyền.

### Các bảng chính:
- **`AspNetUsers`**: Lưu trữ thông tin tài khoản người dùng (Email, PasswordHash, v.v.).
- **`AspNetRoles`**: Danh sách các vai trò trong hệ thống (Student, Instructor, Admin).
- **`AspNetUserRoles`**: Bảng trung gian kết nối Người dùng và Vai trò (n-n).
- **`DataProtectionKeys`**: Lưu trữ các khóa bảo mật để mã hóa dữ liệu cá nhân.

---

## 2. Course Service (Schema: `public`)

Đây là dịch vụ có cấu trúc dữ liệu phức tạp nhất, quản lý toàn bộ nội dung học tập.

### Các bảng chính và Mối quan hệ:
- **`Courses` (Khóa học)**:
    - `1 - 1` với **`CourseDetails`**: Lưu thông tin chi tiết, mô tả dài của khóa học.
    - `1 - n` với **`Lessons`**: Một khóa học có nhiều bài học.
    - `1 - n` với **`LessonCategories`**: Một khóa học được chia thành nhiều chương/mục.
    - `1 - n` với **`Enrollments`**: Danh sách học viên đăng ký khóa học.
    - `1 - n` với **`Comments`**: Các đánh giá và nhận xét của học viên.
    - `1 - n` với **`Tags`**: Các từ khóa phân loại khóa học.
- **`Lessons` (Bài học)**:
    - `1 - n` with **`Attachments`**: Các tài liệu đính kèm bài học (PDF, Code).
    - `1 - n` with **`Questions`**: Các câu hỏi thảo luận trong bài học.
- **`Questions` & `Answers`**:
    - `1 - n` giữa **`Questions`** và **`Answers`**: Một câu hỏi có thể có nhiều câu trả lời.
    - `1 - n` giữa **`Answers`** và **`Likes`**: Người dùng có thể like các câu trả lời hay.
- **`Favorites`**: Lưu danh sách khóa học yêu thích của từng người dùng.

---

## 3. Payment Service (Schema: `public`)

Quản lý giao dịch và thông tin thanh toán.

### Các bảng chính:
- **`Payments`**: Lưu trữ lịch sử giao dịch (Mã giao dịch, số tiền, trạng thái từ Iyzipay).
- **`Baskets` & `BasketItems`**:
    - **`Baskets`**: Giỏ hàng của người dùng.
    - **`BasketItems`**: Các khóa học cụ thể nằm trong giỏ hàng.
- **`Cards`**: Lưu trữ thông tin thẻ đã được mã hóa (Token) để tái sử dụng cho các lần thanh toán sau.
- **`UserDatas`**: Thông tin bổ sung của người dùng phục vụ cho việc xuất hóa đơn và xác thực thanh toán.

---

## 4. Đặc điểm thiết kế
- **BaseEntity**: Hầu hết các bảng đều kế thừa từ `BaseEntity`, bao gồm các trường dùng chung như `Id`, `CreatedAt`, `UpdatedAt`, `IsDeleted`.
- **Soft Delete**: Hệ thống sử dụng cờ `IsDeleted` để xóa mềm dữ liệu thay vì xóa vĩnh viễn (tùy cấu hình).
- **Audit Logs**: Các hành động quan trọng trên khóa học được ghi lại trong bảng `AuditLogs` để theo dõi lịch sử thay đổi.
