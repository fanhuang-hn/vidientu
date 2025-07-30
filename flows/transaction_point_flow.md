# Chi tiết luồng nạp tiền - Điểm giao dịch
# Transaction Point Top-up Flow Details

## User Flow - Nạp tiền qua điểm giao dịch

### Các loại điểm giao dịch:
- Cửa hàng tiện lợi (Circle K, FamilyMart, GS25, B's Mart)
- Siêu thị (Co.opMart, Lotte Mart, Big C)
- Cửa hàng điện thoại (CellphoneS, FPT Shop, Thế Giới Di Động)
- Đại lý chính thức
- ATM có hỗ trợ (một số ngân hàng)

### Bước 1: Tìm điểm giao dịch
1. User mở ứng dụng ví điện tử
2. Chọn "Nạp tiền" từ màn hình chính
3. Chọn "Điểm giao dịch"
4. **Tìm điểm giao dịch gần nhất:**
   - Sử dụng GPS để xác định vị trí
   - Hiển thị bản đồ với các điểm giao dịch
   - Lọc theo loại điểm giao dịch
   - Hiển thị khoảng cách và thời gian hoạt động

### Bước 2: Tạo mã nạp tiền
1. Chọn điểm giao dịch từ danh sách
2. Nhập số tiền muốn nạp:
   - Tối thiểu: 50,000 VNĐ
   - Tối đa: 5,000,000 VNĐ/lần
   - Các mệnh giá cố định: 50K, 100K, 200K, 500K, 1M, 2M, 5M
3. Hệ thống tạo mã nạp tiền (Top-up Code):
   - Mã 8-12 chữ số
   - Có thời hạn 24 giờ
   - QR code tương ứng

### Bước 3: Đến điểm giao dịch
1. User đến điểm giao dịch đã chọn
2. Xuất trình mã nạp tiền hoặc QR code
3. Thông báo cho nhân viên số tiền muốn nạp

### Bước 4: Thanh toán tại điểm giao dịch
1. **Nhân viên quét mã:**
   - Quét QR code hoặc nhập mã nạp tiền
   - Hệ thống POS hiển thị thông tin giao dịch
   - Xác nhận số tiền và thông tin user

2. **User thanh toán:**
   - Thanh toán bằng tiền mặt
   - Hoặc thẻ ATM/tín dụng tại POS
   - Nhận biên lai xác nhận

### Bước 5: Xác nhận giao dịch
1. POS gửi xác nhận đến hệ thống ví
2. Hệ thống cập nhật số dư user
3. Gửi push notification xác nhận
4. Cập nhật lịch sử giao dịch

### Trạng thái giao dịch trong luồng:
```
CODE_GENERATED → PENDING_PAYMENT → PAID → CONFIRMED → SUCCESS
```

## Quy trình cho nhân viên điểm giao dịch

### Đào tạo nhân viên:
1. Hướng dẫn sử dụng thiết bị POS
2. Quy trình xử lý giao dịch nạp tiền
3. Xử lý các trường hợp lỗi
4. Customer service cơ bản

### Thiết bị cần thiết:
- Máy POS tích hợp QR scanner
- Kết nối internet ổn định
- Máy in hóa đơn
- Tài khoản đại lý trong hệ thống

### Quy trình xử lý:
1. Nhận yêu cầu từ khách hàng
2. Quét QR code hoặc nhập mã
3. Xác nhận thông tin trên màn hình POS
4. Nhận tiền từ khách hàng
5. Xác nhận giao dịch trên POS
6. In và đưa biên lai cho khách hàng

## Quản lý mã nạp tiền

### Tạo mã:
- Mã unique không trùng lặp
- Thuật toán mã hóa an toàn
- Tích hợp checksum để verify

### Hết hạn mã:
- Thời hạn: 24 giờ kể từ khi tạo
- Tự động hủy mã hết hạn
- Thông báo user về mã sắp hết hạn

### Sử dụng lại mã:
- Mỗi mã chỉ sử dụng 1 lần
- Đánh dấu đã sử dụng ngay khi thanh toán
- Không thể reactive mã đã dùng

## Xử lý lỗi

### Lỗi thường gặp:
1. **Mã không tồn tại**
   - Mã lỗi: INVALID_CODE
   - Nguyên nhân: Nhập sai mã hoặc mã giả
   - Giải pháp: Kiểm tra lại mã hoặc tạo mã mới

2. **Mã đã hết hạn**
   - Mã lỗi: EXPIRED_CODE
   - Nguyên nhân: Quá 24 giờ từ khi tạo
   - Giải pháp: Tạo mã mới

3. **Mã đã được sử dụng**
   - Mã lỗi: CODE_USED
   - Nguyên nhân: Mã đã thanh toán trước đó
   - Giải pháp: Kiểm tra lịch sử hoặc tạo mã mới

4. **Điểm giao dịch không hoạt động**
   - Mã lỗi: AGENT_INACTIVE
   - Nguyên nhân: Đại lý tạm ngưng dịch vụ
   - Giải pháp: Chọn điểm giao dịch khác

5. **Lỗi kết nối POS**
   - Mã lỗi: POS_CONNECTION_ERROR
   - Nguyên nhân: Mất kết nối internet
   - Giải pháp: Thử lại sau hoặc báo kỹ thuật

6. **Số tiền không khớp**
   - Mã lỗi: AMOUNT_MISMATCH
   - Nguyên nhân: Số tiền thanh toán ≠ số tiền yêu cầu
   - Giải pháp: Hoàn tiền hoặc nạp bù số tiền thiếu

## Reconciliation và settlement

### Đối soát hàng ngày:
- Tổng hợp giao dịch theo từng điểm
- So sánh với báo cáo từ đại lý
- Phát hiện sai lệch và xử lý

### Settlement:
- Chuyển tiền cho đại lý hàng tuần
- Trừ phí dịch vụ và hoa hồng
- Xuất báo cáo settlement

### Audit trail:
- Log tất cả thao tác
- Lưu trữ biên lai điện tử
- Backup dữ liệu định kỳ

## Thời gian xử lý
- Tạo mã: Tức thì
- Thanh toán tại điểm: 1-2 phút
- Cập nhật số dư: 30 giây - 2 phút
- Thời hạn mã: 24 giờ

## Phí giao dịch
- Phí user: 2,000 - 5,000 VNĐ (tùy loại điểm giao dịch)
- Hoa hồng đại lý: 0.5% - 1% doanh thu
- Phí xử lý hệ thống: 0.2% doanh thu

## Giờ hoạt động
- Cửa hàng tiện lợi: 24/7
- Siêu thị: 6:00 - 22:00
- Cửa hàng điện thoại: 8:00 - 21:00
- Đại lý: Theo quy định riêng
- ATM: 24/7 (một số máy)