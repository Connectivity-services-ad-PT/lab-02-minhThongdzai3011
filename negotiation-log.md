# Biên bản đàm phán hợp đồng API

- Cặp đàm phán: Pair 04 - Core Business - Notification
- Product: B
- Provider: Notification Service (B7)
- Consumer: Core Business Service (B6)
- Phiên: v1.0
- Ngày: 20/05/2026

---

## Issue #1

- Raised by: Provider
- Event: `alert.created`
- Concern: Notification cần biết mức độ nghiêm trọng để quyết định kênh gửi (CRITICAL → Telegram, LOW → email). Consumer chưa gửi field severity
- Proposal: Consumer phải thêm field severity vào payload alert.created với enum [LOW, MEDIUM, HIGH, CRITICAL]
- Resolution: Accepted
- Rationale: Giúp Provider routing thông minh, tránh gửi spam cho alert không quan trọng
- Impact: Bổ sung `severity` là trường bắt buộc trong event `alert.created`

---

## Issue #2

- Raised by: Consumer
- Event: Tất cả events
- Concern: Consumer lo ngại không thể truy vết alert nếu thiếu thông tin trace
- Proposal: Consumer yêu cầu Provider phải đảm bảo giữ nguyên `correlationId` và `traceId` trong mọi event để tracing end-to-end
- Resolution: Accepted
- Rationale: Đảm bảo khả năng debug khi có sự cố giữa các service
- Impact: Provider cam kết không làm thay đổi hoặc mất các field này. Ghi nhận vào contract

---

## Issue #3

- Raised by: Provider
- Event: `alert.created`
- Concern: Provider muốn tự quyết định kênh gửi (Telegram/Email/App) dựa trên severity và user preference, không muốn Consumer gửi danh sách channel cứng nhắc
- Proposal: Consumer chỉ cần gửi `severity` và `userId`/`userGroupId`. Provider sẽ tra cứu kênh ưu tiên của user
- Resolution: Modified
- Rationale: iảm coupling, Provider linh hoạt hơn, Consumer không cần biết chi tiết cấu hình user
- Impact: Consumer không gửi field `channels`. Thay vào đó gửi `userId` hoặc `userGroupId`

---

## Issue #4

- Raised by: Provider
- Event: Tất cả events
- Concern: Queue có thể gửi lại event do lỗi mạng (at-least-once), dẫn đến Provider xử lý trùng lặp → gửi nhiều thông báo cho user
- Proposal: Mỗi event phải có `eventId` (UUID) duy nhất do Consumer sinh ra. Provider sẽ idempotent: không gửi trùng thông báo cho cùng `eventId`
- Resolution: Accepted
- Rationale: Đảm bảo tính đúng đắn, tránh spam user
- Impact: Thêm `eventId` là trường bắt buộc do Consumer cung cấp. Provider sẽ cache `eventId` đã xử lý trong 24h

---

## Issue #5

- Raised by: Provider
- Event: `alert.resolved`
- Concern: Provider muốn biết có cần gửi thông báo "alert đã được giải quyết" không, và ai đã xử lý để ghi log
- Proposal: Consumer phải gửi thêm field `resolvedBy` (tên người hoặc system) và `resolutionNote` (lý do) trong event `alert.resolved`
- Resolution: Accepted
- Rationale: Cung cấp thông tin đầy đủ cho user và audit log
- Impact: Thêm `resolvedBy` (string) và `resolutionNote` (string, optional) vào event `alert.resolved`

---

## Issue #6

- Raised by: Consumer
- Event: Tất cả events
- Concern: Consumer muốn thống nhất cơ chế retry khi Provider không xử lý được (lỗi database, timeout gửi email)
- Proposal: Queue retry tối đa 3 lần với exponential backoff (1s, 2s, 4s). Sau 3 lần thất bại → dead-letter queue để operator xử lý
- Resolution: Accepted
- Rationale: Cân bằng giữa khả năng phục hồi và tránh tắc nghẽn
- Impact: Ghi nhận cơ chế retry. Lab 03 sẽ đặc tả cụ thể bằng AsyncAPI

---

# Chốt hợp đồng v1.0

Provider sign-off: QuangTungMasterD
Consumer sign-off: 
Witness (GV/TA):    
Date: 20/05/2026

---

## Ghi chú warning nếu Spectral còn cảnh báo

| Warning | Lý do chấp nhận tạm thời | Kế hoạch sửa |
|---|---|---|
||||
