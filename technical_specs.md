# Đặc tả kỹ thuật - API và Database
# Technical Specifications - API & Database

## API Endpoints

### 1. Initiate Top-up Transaction
```
POST /api/v1/topup/initiate
Content-Type: application/json
Authorization: Bearer {user_token}

Request Body:
{
  "method": "bank_transfer|card_payment|ewallet_link|transaction_point",
  "amount": 100000,
  "currency": "VND",
  "metadata": {
    "bank_code": "VCB", // for bank transfer
    "card_type": "credit", // for card payment
    "ewallet_type": "momo", // for ewallet
    "agent_id": "12345" // for transaction point
  }
}

Response:
{
  "status": "success",
  "transaction_id": "TXN_20231201_123456",
  "payment_data": {
    // Method-specific payment data
  },
  "expires_at": "2023-12-01T18:00:00Z"
}
```

### 2. Get Transaction Status
```
GET /api/v1/topup/status/{transaction_id}
Authorization: Bearer {user_token}

Response:
{
  "status": "success",
  "transaction": {
    "id": "TXN_20231201_123456",
    "status": "PROCESSING",
    "amount": 100000,
    "fee": 2000,
    "method": "bank_transfer",
    "created_at": "2023-12-01T17:30:00Z",
    "updated_at": "2023-12-01T17:35:00Z"
  }
}
```

### 3. Card Payment Processing
```
POST /api/v1/topup/card/process
Content-Type: application/json
Authorization: Bearer {user_token}

Request Body:
{
  "transaction_id": "TXN_20231201_123456",
  "card_data": {
    "number": "4111111111111111",
    "holder_name": "NGUYEN VAN A",
    "expiry_month": "12",
    "expiry_year": "25",
    "cvv": "123"
  },
  "save_card": true
}

Response:
{
  "status": "success",
  "requires_3ds": true,
  "redirect_url": "https://3ds.bank.com/auth?token=xyz",
  "transaction_status": "PENDING_3DS"
}
```

### 4. E-wallet Link Authorization
```
POST /api/v1/topup/ewallet/authorize
Content-Type: application/json
Authorization: Bearer {user_token}

Request Body:
{
  "ewallet_type": "momo",
  "phone_number": "0901234567",
  "authorization_code": "AUTH_CODE_FROM_MOMO"
}

Response:
{
  "status": "success",
  "link_id": "LINK_123456",
  "access_token": "encrypted_token",
  "expires_in": 2592000
}
```

### 5. Generate Transaction Point Code
```
POST /api/v1/topup/point/generate
Content-Type: application/json
Authorization: Bearer {user_token}

Request Body:
{
  "transaction_id": "TXN_20231201_123456",
  "agent_id": "AGENT_001",
  "amount": 100000
}

Response:
{
  "status": "success",
  "topup_code": "12345678",
  "qr_code": "data:image/png;base64,iVBORw0KGgoAAAANSU...",
  "expires_at": "2023-12-02T17:30:00Z"
}
```

## Database Schema

### Transactions Table
```sql
CREATE TABLE transactions (
  id VARCHAR(50) PRIMARY KEY,
  user_id VARCHAR(50) NOT NULL,
  method ENUM('bank_transfer', 'card_payment', 'ewallet_link', 'transaction_point') NOT NULL,
  amount DECIMAL(15,2) NOT NULL,
  fee DECIMAL(15,2) DEFAULT 0,
  currency VARCHAR(3) DEFAULT 'VND',
  status ENUM('INITIATED', 'PENDING', 'PROCESSING', 'SUCCESS', 'FAILED', 'CANCELLED', 'TIMEOUT') NOT NULL,
  metadata JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  completed_at TIMESTAMP NULL,
  
  INDEX idx_user_id (user_id),
  INDEX idx_status (status),
  INDEX idx_created_at (created_at)
);
```

### Bank Transfer Details Table
```sql
CREATE TABLE bank_transfer_details (
  transaction_id VARCHAR(50) PRIMARY KEY,
  bank_account VARCHAR(50) NOT NULL,
  account_holder VARCHAR(100) NOT NULL,
  transfer_content VARCHAR(200) NOT NULL,
  qr_code TEXT,
  bank_transaction_id VARCHAR(100) NULL,
  
  FOREIGN KEY (transaction_id) REFERENCES transactions(id)
);
```

### Card Payment Details Table
```sql
CREATE TABLE card_payment_details (
  transaction_id VARCHAR(50) PRIMARY KEY,
  card_token VARCHAR(100) NOT NULL,
  card_last4 VARCHAR(4) NOT NULL,
  card_type VARCHAR(20) NOT NULL,
  payment_gateway VARCHAR(50) NOT NULL,
  gateway_transaction_id VARCHAR(100) NULL,
  three_ds_status ENUM('NOT_REQUIRED', 'REQUIRED', 'COMPLETED', 'FAILED') DEFAULT 'NOT_REQUIRED',
  
  FOREIGN KEY (transaction_id) REFERENCES transactions(id)
);
```

### E-wallet Links Table
```sql
CREATE TABLE ewallet_links (
  id VARCHAR(50) PRIMARY KEY,
  user_id VARCHAR(50) NOT NULL,
  ewallet_type VARCHAR(20) NOT NULL,
  phone_number VARCHAR(20) NOT NULL,
  access_token TEXT NOT NULL,
  refresh_token TEXT,
  expires_at TIMESTAMP NOT NULL,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_user_ewallet (user_id, ewallet_type),
  INDEX idx_user_id (user_id),
  INDEX idx_expires_at (expires_at)
);
```

### Transaction Point Codes Table
```sql
CREATE TABLE transaction_point_codes (
  code VARCHAR(12) PRIMARY KEY,
  transaction_id VARCHAR(50) NOT NULL,
  agent_id VARCHAR(50) NOT NULL,
  amount DECIMAL(15,2) NOT NULL,
  qr_code TEXT NOT NULL,
  status ENUM('ACTIVE', 'USED', 'EXPIRED') DEFAULT 'ACTIVE',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  expires_at TIMESTAMP NOT NULL,
  used_at TIMESTAMP NULL,
  
  FOREIGN KEY (transaction_id) REFERENCES transactions(id),
  INDEX idx_agent_id (agent_id),
  INDEX idx_expires_at (expires_at)
);
```

### User Wallets Table
```sql
CREATE TABLE user_wallets (
  user_id VARCHAR(50) PRIMARY KEY,
  balance DECIMAL(15,2) DEFAULT 0,
  daily_topup_limit DECIMAL(15,2) DEFAULT 50000000,
  daily_topup_used DECIMAL(15,2) DEFAULT 0,
  last_reset_date DATE DEFAULT (CURRENT_DATE),
  is_verified BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

## Webhook Events

### 1. Bank Transfer Webhook
```json
{
  "event": "bank_transfer.completed",
  "data": {
    "bank_transaction_id": "VCB_20231201_789012",
    "amount": 100000,
    "transfer_content": "TOPUP TXN_20231201_123456",
    "sender_account": "1234567890",
    "receiver_account": "0987654321",
    "timestamp": "2023-12-01T17:45:00Z"
  }
}
```

### 2. Card Payment Webhook
```json
{
  "event": "card_payment.completed",
  "data": {
    "gateway_transaction_id": "GTW_789012345",
    "merchant_transaction_id": "TXN_20231201_123456",
    "status": "SUCCESS",
    "amount": 102000,
    "fee": 2000,
    "timestamp": "2023-12-01T17:45:00Z"
  }
}
```

### 3. E-wallet Transfer Webhook
```json
{
  "event": "ewallet_transfer.completed",
  "data": {
    "external_transaction_id": "MOMO_789012345",
    "our_transaction_id": "TXN_20231201_123456",
    "status": "SUCCESS",
    "amount": 100000,
    "sender_phone": "0901234567",
    "timestamp": "2023-12-01T17:45:00Z"
  }
}
```

## Security Considerations

### Authentication & Authorization
- JWT tokens with 1-hour expiration
- Refresh tokens with 30-day expiration
- Rate limiting: 100 requests/minute per user
- IP whitelisting for webhook endpoints

### Data Encryption
- AES-256 for sensitive data at rest
- TLS 1.3 for data in transit
- RSA-2048 for key exchange
- HMAC-SHA256 for webhook signatures

### PCI DSS Compliance
- No storage of full card numbers
- Card data tokenization
- Secure key management
- Regular security audits

### Fraud Prevention
- Velocity checks on transactions
- Device fingerprinting
- Geolocation verification
- Machine learning fraud detection

## Error Codes

| Code | Description | Action |
|------|-------------|--------|
| E001 | Invalid transaction amount | Check minimum/maximum limits |
| E002 | Insufficient balance | Top up source account |
| E003 | Invalid payment method | Choose different method |
| E004 | Transaction timeout | Retry transaction |
| E005 | Daily limit exceeded | Try again tomorrow |
| E006 | Invalid card details | Check card information |
| E007 | 3D Secure failed | Contact bank |
| E008 | E-wallet link expired | Re-link e-wallet |
| E009 | Invalid top-up code | Generate new code |
| E010 | Agent not available | Choose different agent |

## Monitoring & Logging

### Key Metrics
- Transaction success rate by method
- Average processing time
- Daily/monthly volume
- Error rate by type
- User adoption by method

### Alerts
- Transaction success rate < 95%
- Processing time > 5 minutes
- Error rate > 5%
- Suspicious transaction patterns
- System downtime

### Logs
```json
{
  "timestamp": "2023-12-01T17:30:00Z",
  "level": "INFO",
  "event": "transaction.initiated",
  "user_id": "USER_123456",
  "transaction_id": "TXN_20231201_123456",
  "method": "bank_transfer",
  "amount": 100000,
  "metadata": {
    "ip_address": "192.168.1.1",
    "user_agent": "VidienTu/1.0.0",
    "device_id": "DEVICE_789012"
  }
}
```