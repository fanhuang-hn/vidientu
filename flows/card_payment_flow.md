# Chi tiết luồng nạp tiền - Thẻ tín dụng/ghi nợ
# Credit/Debit Card Top-up Flow Details

## User Flow - Thẻ tín dụng/ghi nợ

### Bước 1: Khởi tạo giao dịch
1. User mở ứng dụng ví điện tử
2. Chọn "Nạp tiền" từ màn hình chính
3. Chọn "Thẻ tín dụng/ghi nợ"
4. Nhập số tiền muốn nạp (tối thiểu 20,000 VNĐ, tối đa 20,000,000 VNĐ/ngày)
5. Xem trước phí giao dịch (2.9% cho thẻ tín dụng, 1.9% cho thẻ ghi nợ)

### Bước 2: Nhập thông tin thẻ
1. **Thẻ mới:**
   - Nhập số thẻ (16-19 chữ số)
   - Nhập tên chủ thẻ
   - Nhập ngày hết hạn (MM/YY)
   - Nhập mã CVV (3-4 chữ số)
   - Tùy chọn lưu thẻ cho lần sau

2. **Thẻ đã lưu:**
   - Chọn thẻ từ danh sách đã lưu
   - Nhập mã CVV để xác nhận

### Bước 3: Xác thực giao dịch
1. Hệ thống gửi thông tin đến payment gateway
2. **3D Secure verification:**
   - Chuyển hướng đến trang ngân hàng
   - Nhập OTP hoặc mật khẩu giao dịch
   - Xác nhận giao dịch

3. **Biometric verification (nếu có):**
   - Vân tay hoặc Face ID
   - PIN của ứng dụng

### Bước 4: Xử lý kết quả
1. Nhận kết quả từ payment gateway
2. Cập nhật trạng thái giao dịch
3. Cập nhật số dư ví (nếu thành công)
4. Gửi thông báo và email xác nhận

### Trạng thái giao dịch trong luồng:
```
INITIATED → PROCESSING → 3DS_VERIFICATION → SUCCESS/FAILED
```

## Bảo mật thông tin thẻ

### Tokenization
- Không lưu trữ thông tin thẻ thực
- Sử dụng token để thay thế số thẻ
- Token có thời hạn và chỉ dùng cho merchant

### PCI DSS Compliance
- Mã hóa SSL/TLS cho tất cả giao dịch
- Không log thông tin nhạy cảm
- Audit trail đầy đủ
- Penetration testing định kỳ

## Xử lý lỗi

### Lỗi thường gặp:
1. **Thẻ không đủ số dư**
   - Mã lỗi: INSUFFICIENT_FUNDS
   - Thông báo: "Thẻ không đủ số dư để thực hiện giao dịch"
   - Giải pháp: Thử với số tiền nhỏ hơn hoặc thẻ khác

2. **Thẻ đã hết hạn**
   - Mã lỗi: EXPIRED_CARD
   - Thông báo: "Thẻ đã hết hạn"
   - Giải pháp: Cập nhật thông tin thẻ mới

3. **CVV không đúng**
   - Mã lỗi: INVALID_CVV
   - Thông báo: "Mã CVV không chính xác"
   - Giải pháp: Kiểm tra lại và nhập đúng CVV

4. **3D Secure thất bại**
   - Mã lỗi: 3DS_FAILED
   - Thông báo: "Xác thực 3D Secure thất bại"
   - Giải pháp: Thử lại hoặc liên hệ ngân hàng

5. **Vượt quá hạn mức**
   - Mã lỗi: LIMIT_EXCEEDED
   - Thông báo: "Vượt quá hạn mức giao dịch"
   - Giải pháp: Giảm số tiền hoặc thử lại ngày mai

6. **Thẻ bị khóa**
   - Mã lỗi: CARD_BLOCKED
   - Thông báo: "Thẻ tạm thời bị khóa"
   - Giải pháp: Liên hệ ngân hàng để mở khóa

## Thời gian xử lý
- Giao dịch thành công: 30 giây - 2 phút
- 3D Secure timeout: 5 phút
- Hoàn tiền (nếu cần): 3-5 ngày làm việc

## Phí giao dịch
- Thẻ ghi nợ nội địa: 1.9%
- Thẻ tín dụng nội địa: 2.9%
- Thẻ quốc tế: 3.5%
- Phí tối thiểu: 2,000 VNĐ