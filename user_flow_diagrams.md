# Sơ đồ User Flow - Luồng người dùng
# User Flow Diagrams

## Luồng tổng quan (Overview User Flow)

```
[Màn hình chính] 
       │
       ▼
[Chọn "Nạp tiền"]
       │
       ▼
[Chọn phương thức nạp tiền]
       │
    ┌──┴────────────────────────────────┐
    │                                   │
    ▼                                   ▼
[Nhập số tiền]                    [Xem số dư hiện tại]
    │                                   │
    ▼                                   │
[Xác nhận thông tin]                    │
    │                                   │
    ▼                                   │
[Thực hiện giao dịch] ◄─────────────────┘
    │
    ▼
[Chờ xử lý]
    │
    ▼
[Kết quả giao dịch]
    │
    ├── Thành công → [Cập nhật số dư] → [Thông báo thành công]
    │
    └── Thất bại → [Hiển thị lỗi] → [Thử lại hoặc chọn PP khác]
```

## 1. Bank Transfer User Flow

```
[Chọn "Chuyển khoản ngân hàng"]
            │
            ▼
[Nhập số tiền (50K-50M VNĐ)]
            │
            ▼
[Hiển thị phí giao dịch]
            │
            ▼
[Xác nhận tạo lệnh]
            │
            ▼
[Tạo mã giao dịch TXN_XXXXXX]
            │
            ▼
[Hiển thị thông tin chuyển khoản]
├── Số TK: 1234567890
├── Tên: CONG TY VI DIEN TU
├── Số tiền: 100,000 VNĐ
├── Nội dung: TOPUP TXN_XXXXXX
└── QR Code
            │
            ▼
[User mở app ngân hàng]
            │
            ▼
[Thực hiện chuyển khoản]
            │
         ┌──┴──┐
         │     │
         ▼     ▼
    [Quét QR] [Nhập thủ công]
         │     │
         └──┬──┘
            │
            ▼
[Chờ webhook từ ngân hàng]
            │
         ┌──┴──┐
         │     │
         ▼     ▼
    [Nhận được] [Timeout 30p]
         │         │
         │         ▼
         │    [Hủy giao dịch]
         │
         ▼
[Xác minh thông tin]
         │
      ┌──┴──┐
      │     │
      ▼     ▼
  [Đúng]  [Sai nội dung]
      │        │
      │        ▼
      │   [Chờ xử lý thủ công]
      │
      ▼
[Cập nhật số dư]
      │
      ▼
[Thông báo thành công]
```

## 2. Card Payment User Flow

```
[Chọn "Thẻ tín dụng/ghi nợ"]
            │
            ▼
[Nhập số tiền (20K-20M VNĐ)]
            │
            ▼
[Hiển thị phí: 1.9%-3.5%]
            │
            ▼
    ┌───────┴───────┐
    │               │
    ▼               ▼
[Thẻ mới]      [Thẻ đã lưu]
    │               │
    ▼               ▼
[Nhập thông tin thẻ] [Chọn thẻ từ danh sách]
├── Số thẻ          │
├── Tên chủ thẻ      ▼
├── MM/YY         [Nhập CVV]
├── CVV              │
└── ☑ Lưu thẻ        │
    │               │
    └───────┬───────┘
            │
            ▼
[Xác nhận thông tin]
            │
            ▼
[Gửi đến Payment Gateway]
            │
            ▼
    ┌───────┴───────┐
    │               │
    ▼               ▼
[Cần 3D Secure]  [Không cần 3DS]
    │               │
    ▼               │
[Chuyển hướng đến bank] │
    │               │
    ▼               │
[Nhập OTP/Password]     │
    │               │
    ▼               │
[Xác thực thành công]   │
    │               │
    └───────┬───────┘
            │
            ▼
    ┌───────┴───────┐
    │               │
    ▼               ▼
[Thành công]    [Thất bại]
    │               │
    ▼               ▼
[Cập nhật số dư] [Hiển thị lỗi]
    │               │
    ▼               ▼
[Thông báo]     [Thử lại]
```

## 3. E-wallet Linking User Flow

```
[Chọn "Ví điện tử khác"]
            │
            ▼
[Chọn loại ví: MoMo/ZaloPay/...]
            │
            ▼
    ┌───────┴───────┐
    │               │
    ▼               ▼
[Lần đầu liên kết] [Đã liên kết]
    │               │
    ▼               │
[Nhập SĐT ví nguồn] │
    │               │
    ▼               │
[Xác nhận OTP]      │
    │               │
    ▼               │
[OAuth Authorization] │
    │               │
    ▼               │
[Chuyển đến app ví] │
    │               │
    ▼               │
[Đăng nhập ví nguồn] │
    │               │
    ▼               │
[Cấp quyền truy cập] │
    │               │
    ▼               │
[Lưu thông tin liên kết] │
    │               │
    └───────┬───────┘
            │
            ▼
[Nhập số tiền (10K-10M VNĐ)]
            │
            ▼
[Xác nhận giao dịch]
            │
            ▼
[Gửi API đến ví nguồn]
            │
            ▼
[Push notification ví nguồn]
            │
            ▼
[User mở ví nguồn]
            │
            ▼
[Xác nhận trong ví nguồn]
            │
            ▼
[Nhập PIN/OTP]
            │
            ▼
    ┌───────┴───────┐
    │               │
    ▼               ▼
[Chấp nhận]     [Từ chối]
    │               │
    ▼               ▼
[Webhook confirm] [Thông báo từ chối]
    │               │
    ▼               ▼
[Cập nhật số dư] [Hủy giao dịch]
    │
    ▼
[Thông báo thành công]
```

## 4. Transaction Point User Flow

```
[Chọn "Điểm giao dịch"]
            │
            ▼
[Hiển thị bản đồ điểm giao dịch]
            │
            ▼
[Lọc theo loại/khoảng cách]
            │
            ▼
[Chọn điểm giao dịch]
            │
            ▼
[Nhập số tiền]
├── Mệnh giá: 50K, 100K, 200K, 500K, 1M, 2M, 5M
└── Hoặc nhập tự do (50K-5M)
            │
            ▼
[Tạo mã nạp tiền]
├── Mã 8-12 chữ số
├── QR Code
└── Thời hạn: 24 giờ
            │
            ▼
[Đến điểm giao dịch]
            │
            ▼
[Xuất trình mã/QR cho nhân viên]
            │
            ▼
[Nhân viên quét mã]
            │
            ▼
[POS hiển thị thông tin giao dịch]
            │
            ▼
[Nhân viên xác nhận]
            │
            ▼
    ┌───────┴───────┐
    │               │
    ▼               ▼
[Thanh toán tiền mặt] [Thanh toán thẻ]
    │               │
    ▼               ▼
[Nhận tiền]     [Quẹt thẻ POS]
    │               │
    └───────┬───────┘
            │
            ▼
[Nhân viên xác nhận thanh toán]
            │
            ▼
[POS gửi confirm đến hệ thống]
            │
            ▼
[Cập nhật số dư user]
            │
            ▼
[Push notification]
            │
            ▼
[In biên lai cho khách]
```

## Luồng xử lý lỗi (Error Handling Flow)

```
[Phát hiện lỗi]
        │
        ▼
[Phân loại lỗi]
        │
     ┌──┴─────────────────────────┐
     │                          │
     ▼                          ▼
[Lỗi có thể retry]         [Lỗi không thể retry]
     │                          │
     ▼                          ▼
[Exponential backoff]      [Hiển thị lỗi cho user]
     │                          │
     ▼                          ▼
[Retry tự động]           [Đề xuất hành động]
     │                          │
  ┌──┴──┐                       ▼
  │     │                  [Liên hệ support]
  ▼     ▼                       │
[OK] [Fail]                     ▼
  │     │                  [Tạo ticket]
  │     ▼                       │
  │ [Manual intervention]       ▼
  │     │                  [Thông báo timeline]
  │     ▼
  │ [Customer support]
  │
  └─────┬─────────────────────────┘
        │
        ▼
[Cập nhật trạng thái]
        │
        ▼
[Thông báo user]
```

## Responsive Design Flow

### Mobile App Flow
- Touch-friendly buttons (min 44px)
- Swipe gestures cho navigation
- Pull-to-refresh cho cập nhật trạng thái
- Native alerts và notifications

### Web Admin Flow  
- Keyboard shortcuts
- Bulk operations
- Advanced filtering
- Export capabilities

### Accessibility Flow
- Screen reader support
- High contrast mode
- Large text options
- Voice navigation