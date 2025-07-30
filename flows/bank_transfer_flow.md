# Chi tiết luồng nạp tiền - Chuyển khoản ngân hàng
# Bank Transfer Top-up Flow Details

## User Flow - Chuyển khoản ngân hàng

### Bước 1: Khởi tạo giao dịch
1. User mở ứng dụng ví điện tử
2. Chọn "Nạp tiền" từ màn hình chính
3. Chọn "Chuyển khoản ngân hàng"
4. Nhập số tiền muốn nạp (tối thiểu 50,000 VNĐ, tối đa 50,000,000 VNĐ/ngày)
5. Xác nhận số tiền và phí giao dịch (nếu có)

### Bước 2: Tạo lệnh chuyển khoản
1. Hệ thống tạo mã giao dịch duy nhất (TXN_ID)
2. Hiển thị thông tin chuyển khoản:
   - Số tài khoản ngân hàng của ví điện tử
   - Tên người thụ hưởng
   - Số tiền chuyển khoản
   - Nội dung chuyển khoản (bao gồm TXN_ID)
3. Hiển thị QR code cho việc chuyển khoản nhanh
4. Cung cấp tùy chọn sao chép thông tin

### Bước 3: Thực hiện chuyển khoản
1. User mở ứng dụng ngân hàng
2. Thực hiện chuyển khoản với thông tin đã cung cấp
3. Hoặc quét QR code để chuyển khoản nhanh

### Bước 4: Xác nhận giao dịch
1. Hệ thống nhận webhook từ ngân hàng
2. Xác minh thông tin giao dịch
3. Cập nhật trạng thái giao dịch
4. Thông báo cho user qua push notification

### Trạng thái giao dịch trong luồng:
```
INITIATED → PENDING → PROCESSING → SUCCESS/FAILED
```

## Xử lý lỗi

### Lỗi thường gặp:
1. **Sai nội dung chuyển khoản**
   - Hệ thống không thể matching giao dịch
   - Giải pháp: Liên hệ customer service với mã giao dịch ngân hàng

2. **Chuyển khoản thiếu tiền**
   - Số tiền nhận được < số tiền yêu cầu
   - Giải pháp: Hoàn tiền hoặc yêu cầu chuyển bù

3. **Timeout giao dịch**
   - Không nhận được tiền sau 30 phút
   - Giải pháp: Tự động hủy giao dịch, kiểm tra lại

4. **Chuyển khoản thừa tiền**
   - Số tiền nhận được > số tiền yêu cầu
   - Giải pháp: Nạp đúng số tiền yêu cầu, hoàn tiền thừa

## Thời gian xử lý
- Thời gian chờ tối đa: 30 phút
- Thời gian xử lý trung bình: 2-5 phút
- Làm việc: 24/7 (tùy theo ngân hàng)

## Phí giao dịch
- Miễn phí cho giao dịch >= 100,000 VNĐ
- Phí 5,000 VNĐ cho giao dịch < 100,000 VNĐ
- Phí chuyển khoản do ngân hàng của user quy định