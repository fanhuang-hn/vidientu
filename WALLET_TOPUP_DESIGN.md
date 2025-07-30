# Thiết kế luồng nạp tiền vào ví điện tử
# Electronic Wallet Top-up Flow Design

## Tổng quan (Overview)

Tài liệu này mô tả chi tiết các luồng nạp tiền cho ứng dụng ví điện tử, bao gồm:
- Các phương thức nạp tiền khác nhau
- User flow chi tiết cho từng phương thức
- Quản lý trạng thái giao dịch
- Xử lý lỗi và các trường hợp ngoại lệ

This document details the money top-up flows for the electronic wallet application, including:
- Different top-up methods
- Detailed user flows for each method
- Transaction state management
- Error handling and exception cases

## Các phương thức nạp tiền (Top-up Methods)

### 1. Chuyển khoản ngân hàng (Bank Transfer)
- **Mô tả**: Nạp tiền bằng cách chuyển khoản từ tài khoản ngân hàng
- **Chi tiết**: [Bank Transfer Flow](./flows/bank_transfer_flow.md)
- **Ưu điểm**: Không cần thẻ, hỗ trợ số tiền lớn
- **Nhược điểm**: Thời gian xử lý lâu hơn

### 2. Thẻ tín dụng/ghi nợ (Credit/Debit Card)  
- **Mô tả**: Nạp tiền trực tiếp bằng thẻ tín dụng/ghi nợ
- **Chi tiết**: [Card Payment Flow](./flows/card_payment_flow.md)
- **Ưu điểm**: Xử lý nhanh, tiện lợi
- **Nhược điểm**: Có phí giao dịch, giới hạn số tiền

### 3. Liên kết ví điện tử khác (Link Other E-wallets)
- **Mô tả**: Nạp tiền từ các ví điện tử khác đã liên kết
- **Chi tiết**: [E-wallet Linking Flow](./flows/ewallet_linking_flow.md)
- **Ưu điểm**: Tiện lợi cho user có nhiều ví
- **Nhược điểm**: Cần liên kết trước, phụ thuộc vào ví nguồn

### 4. Nạp qua điểm giao dịch (Transaction Point Top-up)
- **Mô tả**: Nạp tiền bằng tiền mặt/thẻ tại các điểm giao dịch
- **Chi tiết**: [Transaction Point Flow](./flows/transaction_point_flow.md)
- **Ưu điểm**: Hỗ trợ tiền mặt, không cần tài khoản ngân hàng
- **Nhược điểm**: Cần đến điểm giao dịch, giờ hoạt động hạn chế

## Trạng thái giao dịch (Transaction States)

| Trạng thái | Mô tả | Hành động tiếp theo |
|------------|-------|-------------------|
| INITIATED | Giao dịch được khởi tạo | Chờ xác thực |
| PENDING | Đang chờ xử lý | Chờ phản hồi từ ngân hàng/đối tác |
| PROCESSING | Đang xử lý | Hệ thống đang xử lý giao dịch |
| SUCCESS | Thành công | Cập nhật số dư ví |
| FAILED | Thất bại | Thông báo lỗi và hướng dẫn |
| CANCELLED | Đã hủy | Kết thúc giao dịch |
| TIMEOUT | Hết thời gian | Tự động hủy giao dịch |

## Kiến trúc hệ thống (System Architecture)

```
[Mobile App] --> [API Gateway] --> [Wallet Service] --> [Payment Gateway]
                                        |
                                   [Database] --> [Notification Service]
```

## Bảo mật và tuân thủ (Security & Compliance)

- Mã hóa end-to-end cho thông tin thanh toán
- Tuân thủ chuẩn PCI DSS
- Xác thực 2 yếu tố (2FA)
- Giám sát giao dịch real-time
- Tuân thủ quy định của Ngân hàng Nhà nước Việt Nam

## Tài liệu chi tiết (Detailed Documentation)

### Luồng nghiệp vụ (Business Flows)
- [Bank Transfer Flow](./flows/bank_transfer_flow.md) - Chi tiết luồng chuyển khoản ngân hàng
- [Card Payment Flow](./flows/card_payment_flow.md) - Chi tiết luồng thanh toán thẻ
- [E-wallet Linking Flow](./flows/ewallet_linking_flow.md) - Chi tiết luồng liên kết ví điện tử
- [Transaction Point Flow](./flows/transaction_point_flow.md) - Chi tiết luồng nạp qua điểm giao dịch

### Đặc tả kỹ thuật (Technical Specifications)
- [Technical Specs](./technical_specs.md) - API endpoints, database schema, webhooks
- [Error Handling](./error_handling.md) - Xử lý lỗi và các trường hợp ngoại lệ

### Sơ đồ luồng tổng quan (Overview Flow Diagram)

```
[User] --> [Select Top-up Method] 
    |
    ├── [Bank Transfer] --> [Generate Transfer Info] --> [Wait for Bank Webhook] --> [Update Balance]
    |
    ├── [Card Payment] --> [Enter Card Details] --> [3D Secure] --> [Process Payment] --> [Update Balance]
    |
    ├── [E-wallet Link] --> [OAuth Authorization] --> [Transfer from Linked Wallet] --> [Update Balance]
    |
    └── [Transaction Point] --> [Generate Code] --> [Visit Agent] --> [Pay Cash/Card] --> [Update Balance]
```

## Checklist triển khai (Implementation Checklist)

### Phase 1 - Core Infrastructure
- [ ] Database schema setup
- [ ] Base API endpoints
- [ ] Authentication system
- [ ] Transaction state management
- [ ] Basic error handling

### Phase 2 - Payment Methods
- [ ] Bank transfer integration
- [ ] Card payment gateway integration
- [ ] E-wallet OAuth implementations
- [ ] Transaction point POS system

### Phase 3 - Advanced Features  
- [ ] Fraud detection
- [ ] Advanced error handling
- [ ] Monitoring and alerting
- [ ] Customer support tools

### Phase 4 - Optimization
- [ ] Performance optimization
- [ ] Security audit
- [ ] Load testing
- [ ] User experience improvements