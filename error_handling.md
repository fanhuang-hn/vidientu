# Xử lý lỗi và các trường hợp ngoại lệ
# Error Handling and Edge Cases

## Nguyên tắc xử lý lỗi tổng quát
## General Error Handling Principles

### 1. Phân loại lỗi (Error Classification)
- **Lỗi hệ thống (System Errors)**: Lỗi kỹ thuật, mất kết nối
- **Lỗi nghiệp vụ (Business Errors)**: Vượt hạn mức, không đủ số dư
- **Lỗi bảo mật (Security Errors)**: Xác thực thất bại, gian lận
- **Lỗi người dùng (User Errors)**: Nhập sai thông tin, hủy giao dịch

### 2. Chiến lược retry (Retry Strategy)
- **Exponential Backoff**: Tăng dần thời gian chờ giữa các lần retry
- **Circuit Breaker**: Tạm ngưng sau nhiều lần thất bại liên tiếp
- **Jitter**: Thêm random delay để tránh thundering herd

## Xử lý lỗi theo từng phương thức

### Bank Transfer - Chuyển khoản ngân hàng

#### Lỗi thường gặp:
1. **Sai nội dung chuyển khoản**
   ```json
   {
     "error_code": "BT001",
     "message": "Nội dung chuyển khoản không đúng định dạng",
     "details": "Cần bao gồm mã giao dịch TXN_XXXXXXXX",
     "resolution": "Kiểm tra lại nội dung và thực hiện chuyển khoản mới"
   }
   ```

2. **Chuyển khoản thiếu/thừa tiền**
   ```json
   {
     "error_code": "BT002",
     "message": "Số tiền chuyển khoản không khớp",
     "expected_amount": 100000,
     "received_amount": 95000,
     "resolution": "Chuyển bù 5,000 VNĐ hoặc liên hệ support"
   }
   ```

3. **Timeout webhook**
   ```json
   {
     "error_code": "BT003", 
     "message": "Không nhận được xác nhận từ ngân hàng",
     "timeout_duration": "30 minutes",
     "resolution": "Kiểm tra lại lịch sử giao dịch ngân hàng"
   }
   ```

#### Xử lý tự động:
- **Partial matching**: Tìm giao dịch có số tiền gần đúng (±5%)
- **Content fuzzy matching**: So khớp mã giao dịch với độ tương tự >80%
- **Auto-refund**: Hoàn tiền thừa sau 24 giờ nếu không claim

### Card Payment - Thanh toán thẻ

#### Lỗi thường gặp:
1. **3D Secure timeout**
   ```json
   {
     "error_code": "CP001",
     "message": "3D Secure verification timeout",
     "timeout_duration": "5 minutes",
     "resolution": "Thử lại với cùng thẻ"
   }
   ```

2. **Thẻ bị từ chối**
   ```json
   {
     "error_code": "CP002",
     "message": "Thẻ bị từ chối bởi ngân hàng",
     "bank_response_code": "05",
     "resolution": "Liên hệ ngân hàng phát hành thẻ"
   }
   ```

3. **CVV sai nhiều lần**
   ```json
   {
     "error_code": "CP003",
     "message": "Nhập sai CVV quá 3 lần",
     "lockout_duration": "15 minutes",
     "resolution": "Chờ hết thời gian khóa hoặc dùng thẻ khác"
   }
   ```

#### Anti-fraud measures:
- **Velocity check**: Tối đa 5 giao dịch/giờ/thẻ
- **Geolocation**: Cảnh báo giao dịch từ vị trí bất thường
- **Device fingerprinting**: Phát hiện thiết bị mới

### E-wallet Linking - Liên kết ví điện tử

#### Lỗi thường gặp:
1. **OAuth timeout**
   ```json
   {
     "error_code": "EW001",
     "message": "OAuth authorization timeout",
     "timeout_duration": "10 minutes",
     "resolution": "Khởi tạo lại quy trình liên kết"
   }
   ```

2. **Token hết hạn giữa chừng**
   ```json
   {
     "error_code": "EW002",
     "message": "Access token expired during transaction",
     "resolution": "Tự động refresh token và retry"
   }
   ```

3. **Ví nguồn bảo trì**
   ```json
   {
     "error_code": "EW003",
     "message": "E-wallet service under maintenance",
     "estimated_recovery": "2023-12-01T20:00:00Z",
     "resolution": "Thử lại sau khi hết bảo trì"
   }
   ```

#### Retry logic:
```javascript
async function retryEwalletTransaction(transactionId, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const result = await processEwalletTransaction(transactionId);
      return result;
    } catch (error) {
      if (error.code === 'EW002' && attempt < maxRetries) {
        await refreshToken();
        await delay(Math.pow(2, attempt) * 1000); // Exponential backoff
        continue;
      }
      throw error;
    }
  }
}
```

### Transaction Point - Điểm giao dịch

#### Lỗi thường gặp:
1. **Mã không được quét được**
   ```json
   {
     "error_code": "TP001",
     "message": "QR code không đọc được",
     "resolution": "Tăng độ sáng màn hình hoặc nhập mã thủ công"
   }
   ```

2. **POS mất kết nối**
   ```json
   {
     "error_code": "TP002", 
     "message": "POS terminal offline",
     "last_connection": "2023-12-01T17:00:00Z",
     "resolution": "Chọn điểm giao dịch khác hoặc thử lại sau"
   }
   ```

3. **Agent hết tiền mặt**
   ```json
   {
     "error_code": "TP003",
     "message": "Agent temporarily out of cash",
     "resolution": "Chọn agent khác hoặc thanh toán bằng thẻ"
   }
   ```

## Xử lý các trường hợp edge case

### 1. Giao dịch trùng lặp (Duplicate Transactions)
```javascript
// Idempotency check
function checkDuplicateTransaction(userId, amount, method, timeWindow = 300) {
  const recentTx = getRecentTransactions(userId, timeWindow);
  return recentTx.some(tx => 
    tx.amount === amount && 
    tx.method === method && 
    tx.status !== 'FAILED'
  );
}
```

### 2. Split-brain scenario
- Giao dịch thành công ở payment gateway nhưng fail ở hệ thống
- Giải pháp: Reconciliation job định kỳ 5 phút

### 3. Partial refund
- User yêu cầu hoàn một phần tiền đã nạp
- Tạo transaction âm với reference đến transaction gốc

### 4. Multiple webhook delivery
- Cùng một webhook được gửi nhiều lần
- Giải pháp: Idempotency key trong webhook header

## Monitoring và alerting

### Real-time alerts:
1. **High error rate**: >5% trong 5 phút
2. **Payment gateway down**: Response time >10s
3. **Suspicious patterns**: >10 failed attempts từ cùng IP
4. **Balance inconsistency**: Total credited ≠ total in wallet

### Dashboard metrics:
- Error rate by payment method
- Average resolution time
- Customer complaint volume
- Manual intervention required

## Recovery procedures

### 1. Manual intervention workflow
```yaml
steps:
  - detect_issue: "Automated detection or customer report"
  - investigate: "Check logs and transaction details"
  - verify_payment: "Confirm payment with external provider"
  - resolve_transaction: "Update status and credit wallet"
  - notify_customer: "Send resolution notification"
  - document_case: "Record for future prevention"
```

### 2. Rollback scenarios
- **Failed database update**: Rollback to previous state
- **Incorrect amount credited**: Create compensating transaction
- **System crash during processing**: Resume from last checkpoint

### 3. Emergency procedures
- **Mass failure**: Pause new transactions, investigate root cause
- **Security breach**: Immediate lockdown, user notification
- **Regulatory compliance**: Report to authorities within required timeframe

## Customer communication

### Error message guidelines:
1. **Clear và actionable**: Nói rõ user cần làm gì
2. **Không technical**: Tránh thuật ngữ kỹ thuật
3. **Có timeline**: Nói rõ khi nào sẽ được giải quyết
4. **Cung cấp alternatives**: Đề xuất phương thức khác

### Example messages:
```json
{
  "user_friendly": "Giao dịch đang được xử lý. Vui lòng chờ 2-5 phút để tiền được cập nhật vào ví.",
  "technical": "Transaction TXN_123456 in PROCESSING state, awaiting webhook confirmation",
  "estimated_resolution": "2-5 minutes",
  "alternative_actions": ["Check transaction history", "Contact support", "Try different method"]
}
```

## Testing error scenarios

### Automated tests:
- Network timeout simulation
- Invalid payment data injection
- Webhook failure scenarios
- Race condition tests

### Manual testing:
- User journey testing with errors
- Support team escalation procedures
- Recovery time measurement
- Customer satisfaction surveys