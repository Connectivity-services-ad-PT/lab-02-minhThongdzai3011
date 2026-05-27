# Event Contract sơ bộ — dùng cho dependency Queue async

> File này chỉ dùng cho các cặp Queue async ở Lab 02 để ghi nhận thỏa thuận ban đầu. Đặc tả chi tiết bằng AsyncAPI sẽ chuyển sang Lab 03.

## 1. Thông tin dependency

- Dependency số: 4
- Producer: Notification Service (B7)
- Consumer: Provider Service (B6)
- Cơ chế: Queue async
- Event/topic dự kiến: `core.notification.alerts`
- Người ghi: Trần Quang Tùng
- Ngày: 20/05/2026

## 2. Mục đích nghiệp vụ

Core Business phát event alert để Notification gửi thông báo đa kênh như Telegram, email hoặc app message đến user hoặc user group.

## 3. Event name / topic

| Mục | Giá trị |
|---|---|
| Topic name | `core.notification.alerts` |
| Event names | `alert.created`, `alert.escalated`, `alert.resolved` |
| Producer | Notification Service (B7) |
| Consumer | Core Business Service (B6) |

## 4. Payload tối thiểu

### alert.created
```json
{
  "eventId": "uuid",
  "eventType": "alert.created",
  "occurredAt": "2026-05-20T08:30:00Z",
  "correlationId": "uuid",
  "traceId": "uuid",
  "source": "core-business",
  "data": {
    "alertId": "uuid",
    "severity": "HIGH",
    "userId": "user-123",
    "title": "Phát hiện truy cập trái phép",
    "message": "Cửa chính - 20/05/2026"
  }
}
```

### alert.escalated
```json
{
  "eventId": "uuid",
  "eventType": "alert.escalated",
  "occurredAt": "2026-05-20T08:35:00Z",
  "correlationId": "uuid",
  "traceId": "uuid",
  "source": "core-business",
  "data": {
    "alertId": "uuid",
    "previousSeverity": "MEDIUM",
    "newSeverity": "HIGH",
    "reason": "Chưa xử lý sau 5 phút"
  }
}
```

### alert.resolved
```json
{
  "eventId": "uuid",
  "eventType": "alert.resolved",
  "occurredAt": "2026-05-20T09:00:00Z",
  "correlationId": "uuid",
  "traceId": "uuid",
  "source": "core-business",
  "data": {
    "alertId": "uuid",
    "resolvedBy": "admin",
    "resolutionNote": "Đã xử lý xong"
  }
}
```

## 5. Ràng buộc cần thống nhất

| Vấn đề | Quyết định tạm thời |
|---|---|
|Event id có bắt buộc không? |	Có, eventId (UUID) do Producer sinh ra|
|Có cần correlationId không? |	Có, dùng để tracing end-to-end|
|Có cho phép gửi trùng event không? |	Không, Consumer phải idempotent (cache eventId 24h)|
|Retry khi lỗi |	Queue retry tối đa 3 lần (1s, 2s, 4s)|
|Dead-letter queue |	Có, sau 3 lần retry thất bại|
|Timestamp format |ISO 8601 (occurredAt)|

## 6. Issue chuyển sang Lab 03

| Vấn đề | Mô tả |
|---|---|
| AsyncAPI specification | Viết `asyncapi.yaml` đầy đủ |
|||
