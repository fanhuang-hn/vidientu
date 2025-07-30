# Chi tiết luồng nạp tiền - Liên kết ví điện tử khác
# E-wallet Linking Top-up Flow Details

## User Flow - Liên kết ví điện tử khác

### Các ví điện tử được hỗ trợ:
- MoMo
- ZaloPay
- ViettelMoney
- VNPay
- ShopeePay
- GrabPay

### Bước 1: Khởi tạo giao dịch
1. User mở ứng dụng ví điện tử
2. Chọn "Nạp tiền" từ màn hình chính
3. Chọn "Ví điện tử khác"
4. Chọn loại ví muốn liên kết từ danh sách
5. Nhập số tiền muốn nạp (tối thiểu 10,000 VNĐ, tối đa 10,000,000 VNĐ/ngày)

### Bước 2: Liên kết tài khoản (lần đầu)
1. **Xác thực danh tính:**
   - Nhập số điện thoại đăng ký ví nguồn
   - Xác nhận OTP từ ví nguồn
   - Đồng ý điều khoản liên kết

2. **OAuth2 Authorization:**
   - Chuyển hướng đến ứng dụng ví nguồn
   - Đăng nhập vào ví nguồn
   - Cấp quyền truy cập cho ví đích
   - Nhận authorization code

3. **Lưu thông tin liên kết:**
   - Tạo access token và refresh token
   - Lưu thông tin liên kết với mã hóa
   - Hiển thị trạng thái liên kết thành công

### Bước 3: Thực hiện giao dịch
1. **Với tài khoản đã liên kết:**
   - Chọn ví từ danh sách đã liên kết
   - Nhập số tiền muốn chuyển
   - Xác nhận thông tin giao dịch

2. **API call đến ví nguồn:**
   - Gửi yêu cầu chuyển tiền qua API
   - Bao gồm: số tiền, mã giao dịch, thông tin xác thực
   - Chờ phản hồi từ ví nguồn

### Bước 4: Xác nhận từ ví nguồn
1. User nhận push notification từ ví nguồn
2. Mở ví nguồn để xác nhận giao dịch
3. Nhập PIN/OTP để xác thực
4. Ví nguồn gửi webhook xác nhận

### Bước 5: Hoàn tất giao dịch
1. Nhận xác nhận từ ví nguồn
2. Cập nhật số dư ví đích
3. Gửi thông báo hoàn tất
4. Cập nhật lịch sử giao dịch

### Trạng thái giao dịch trong luồng:
```
INITIATED → PENDING_APPROVAL → PROCESSING → SUCCESS/FAILED
```

## Quản lý liên kết ví

### Thêm ví mới:
1. Vào "Cài đặt" → "Ví liên kết"
2. Chọn "Thêm ví mới"
3. Thực hiện quy trình liên kết

### Hủy liên kết:
1. Chọn ví muốn hủy liên kết
2. Xác nhận hủy liên kết
3. Revoke access token
4. Thông báo đến ví nguồn

### Gia hạn liên kết:
- Tự động refresh token khi gần hết hạn
- Thông báo user nếu cần xác thực lại
- Tự động hủy liên kết nếu không thể gia hạn

## Xử lý lỗi

### Lỗi thường gặp:
1. **Ví nguồn không đủ số dư**
   - Mã lỗi: SOURCE_INSUFFICIENT_FUNDS
   - Thông báo: "Ví nguồn không đủ số dư"
   - Giải pháp: Nạp tiền vào ví nguồn trước

2. **Liên kết hết hạn**
   - Mã lỗi: LINK_EXPIRED
   - Thông báo: "Liên kết ví đã hết hạn"
   - Giải pháp: Thực hiện liên kết lại

3. **Ví nguồn tạm khóa**
   - Mã lỗi: SOURCE_WALLET_LOCKED
   - Thông báo: "Ví nguồn tạm thời bị khóa"
   - Giải pháp: Liên hệ support ví nguồn

4. **Vượt hạn mức giao dịch**
   - Mã lỗi: DAILY_LIMIT_EXCEEDED
   - Thông báo: "Vượt hạn mức giao dịch ngày"
   - Giải pháp: Thử lại vào ngày hôm sau

5. **User từ chối giao dịch**
   - Mã lỗi: USER_DECLINED
   - Thông báo: "Giao dịch bị từ chối"
   - Giải pháp: Thử lại hoặc chọn phương thức khác

6. **Lỗi kết nối API**
   - Mã lỗi: API_CONNECTION_ERROR
   - Thông báo: "Lỗi kết nối, vui lòng thử lại"
   - Giải pháp: Retry với exponential backoff

## Bảo mật

### Mã hóa dữ liệu:
- Token được mã hóa AES-256
- Thông tin liên kết được hash
- Log không chứa thông tin nhạy cảm

### Rate limiting:
- Tối đa 5 giao dịch/phút/user
- Tối đa 3 lần thử liên kết/ngày
- Auto-lock sau 5 lần thất bại liên tiếp

## Thời gian xử lý
- Liên kết ví: 2-3 phút
- Giao dịch thành công: 1-2 phút
- Timeout: 5 phút
- Refresh token: 30 ngày

## Phí giao dịch
- Miễn phí cho các giao dịch từ MoMo, ZaloPay
- 0.5% cho ViettelMoney, VNPay (tối đa 5,000 VNĐ)
- 1% cho ShopeePay, GrabPay (tối đa 10,000 VNĐ)