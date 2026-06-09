# Giải thích Chi tiết: Xác thực danh tính qua MassTransit (Request-Response Pattern)

Tài liệu này đi sâu vào cách giải quyết thử thách xác thực người dùng trong hệ thống Microservices bằng cách sử dụng **MassTransit** và **RabbitMQ**.

---

## 1. Bài toán: Xác thực chéo giữa các Service

Trong hệ thống Microservices, mỗi dịch vụ (như Course hay Payment) cần biết người dùng hiện tại là ai để thực hiện các hành động (ví dụ: Học viên A có quyền xem bài học B không?).

- **Vấn đề**: Các dịch vụ Course/Payment không nên truy cập trực tiếp vào Database của Auth Service.
- **Giải pháp**: Sử dụng cơ chế nhắn tin (Messaging) để gửi yêu cầu xác thực đến Auth Service và nhận kết quả phản hồi ngay lập tức.

---

## 2. Luồng xử lý (Workflow)

Hệ thống sử dụng **Request-Response Pattern** của MassTransit:
1. **Dịch vụ gửi yêu cầu (Client)**: Phát đi một `IdentityRequestedEvent` kèm theo JWT Token.
2. **Auth Service (Server)**: Lắng nghe sự kiện, xác thực Token và trả về thông tin người dùng qua `IdentityRequestFinalizedEvent`.

---

## 3. Chi tiết Mã nguồn

### **A. Định nghĩa Sự kiện (Events)**
Nằm tại [IdentityRequestedEvent.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Common/Udemy.Common/Events/Auth/IdentityRequestedEvent.cs) và [IdentityRequestFinalizedEvent.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Common/Udemy.Common/Events/Auth/IdentityRequestFinalizedEvent.cs).

```csharp
// Sự kiện gửi đi yêu cầu
public record IdentityRequestedEvent(string Token);

// Sự kiện phản hồi kết quả
public class IdentityRequestFinalizedEvent
{
    public bool IsSuccess { get; set; }
    public IdentityRequestSucceededEvent? Success { get; set; }
    public IdentityRequestFailedEvent? Failed { get; set; }
}
```

### **B. Xử lý tại Auth Service (Handler)**
File: [IdentityRequestedEventHandler.cs](file:///Users/tuanhuytran/Downloads/Udemy_Backend/Udemy.Auth/Udemy.Auth.Application/Handlers/IdentityRequestedEventHandler.cs)

```csharp
public class IdentityRequestedEventHandler(IAuthService authService) : IConsumer<IdentityRequestedEvent>
{
    public async Task Consume(ConsumeContext<IdentityRequestedEvent> context)
    {
        var message = context.Message;
        
        // 1. Gọi Service để giải mã và kiểm tra Token
        var ticket = _authService.GetIdentityFromToken(message.Token, context.CancellationToken);

        var resultEvent = new IdentityRequestFinalizedEvent();

        if (ticket == null)
        {
            // 2. Trả về lỗi nếu Token không hợp lệ
            resultEvent.Failed = new IdentityRequestFailedEvent("Token is not valid.", HttpStatusCode.BadRequest);
            await context.RespondAsync(resultEvent);
            return;
        }

        // 3. Trích xuất thông tin từ Identity Ticket
        var id = ticket.Properties.Items["Id"];
        
        // 4. Tạo sự kiện thành công kèm thông tin User (Roles, Id, Name)
        resultEvent.IsSuccess = true;
        resultEvent.Success = new IdentityRequestSucceededEvent(
            ticket.Principal.Claims.Where(c => c.Type == ClaimTypes.Role).Select(c => c.Value).ToList(),
            ticket.Principal.Identity!.IsAuthenticated,
            ticket.Principal.Identity!.Name,
            ticket.Principal.Identity.AuthenticationType,
            Guid.Parse(id!)
        );

        // 5. Gửi phản hồi ngược lại cho service yêu cầu
        await context.RespondAsync(resultEvent);
    }
}
```

---

## 4. Tại sao đây là một Task quan trọng cho Junior?

1. **Hiểu về Middleware & Auth**: Bạn học được cách JWT hoạt động bên dưới và cách trích xuất thông tin từ `Claims`.
2. **Làm quen với Message Broker**: Thay vì gọi API trực tiếp (HTTP), bạn sử dụng RabbitMQ, giúp hệ thống chịu tải tốt hơn và giảm độ trễ (latency) khi có nhiều request đồng thời.
3. **Tư duy hướng sự kiện (Event-driven)**: Đây là bước đầu để làm quen với các pattern phức tạp hơn như Saga hay Outbox sau này.
4. **Clean Code**: Việc tách biệt định nghĩa sự kiện vào thư viện Common và xử lý logic tại Application Layer giúp mã nguồn dễ đọc và tái sử dụng.
