# Technical Design - Digital Asset Service

## 1. Mục tiêu kỹ thuật

Cung cấp thiết kế chi tiết cho hệ thống quản lý tài sản số dựa trên Hyperledger Fabric, tích hợp với dịch vụ định danh (DID Service), xác thực (AuthN) và phân quyền truy cập (AuthZ). Dự án áp dụng mô hình microservices, REST/gRPC API, và sử dụng Fabric Token SDK để thao tác với tài sản.

---

## 2. Kiến trúc tổng thể

```
+-----------------+      +------------+     +-----------+
|  Client / App   +----->+ API Gateway+<--->+ AuthN     |
+-----------------+      +-----+------+     +-----------+
                               |
                    +----------v----------+
                    |  Digital Asset API  |
                    +----------+----------+
                               |
       +-----------------------+-----------------------+
       |                       |                       |
+------v-----+        +--------v------+        +--------v------+
| DID Service|        | AuthZ Service |        | Token SDK     |
| (Resolve)  |        | (RBAC Check)  |        | (Fabric Chain)|
+------------+        +---------------+        +---------------+
```

---

## 3. Thành phần hệ thống

### 3.1 Digital Asset API

* Framework: Go (Gin hoặc gRPC)
* Chức năng:

  * Mint/Transfer/Burn token
  * Truy vấn số dư và lịch sử
  * Đăng ký SIP
* Middleware:

  * JWT Validation
  * DID resolution
  * Role/Permission checking (AuthZ)

### 3.2 DID Service

* Input: JWT `sub`
* Output: DID → MSP ID + Certificate + Wallet Key
* Cache DID mapping để giảm độ trễ

### 3.3 AuthZ Service

* Kiểm tra quyền với `action`, `subject`, `resource`
* Có thể dùng Casbin hoặc RBAC custom

### 3.4 Token SDK

* Gọi tới Fabric Gateway (external builder)
* Xử lý UTXO-based token:

  * `Mint(tokenType, amount, owner)`
  * `Transfer(tokenID, to)`
  * `Burn(tokenID)`
* Sign transaction bằng cert của người dùng

### 3.5 Metadata Storage

* PostgreSQL/MongoDB: Lưu thông tin tài sản và giao dịch
* IPFS: Lưu thông tin file đính kèm (giấy tờ, ảnh...)

---

## 4. API Design

### 4.1 POST /v1/assets/mint

```json
{
  "token_type": "RWA",
  "amount": 1000000,
  "receiver_did": "did:fabric:org1:user1",
  "metadata": {
    "property_id": "house-001",
    "valuation": "10B"
  }
}
```

### 4.2 POST /v1/assets/transfer

```json
{
  "token_id": "token123",
  "to_did": "did:fabric:org2:user2"
}
```

### 4.3 POST /v1/assets/burn

```json
{
  "token_id": "token123"
}
```

### 4.4 GET /v1/assets/balance

→ Trả về danh sách token sở hữu theo DID hiện tại

### 4.5 GET /v1/assets/history

→ Trả về lịch sử giao dịch (txID, thời gian, đối tượng)

---

## 5. Quy trình nghiệp vụ

### 5.1 Mint Token Flow

1. Nhận JWT từ client
2. Giải mã và lấy DID từ DID Service
3. Kiểm tra quyền mint qua AuthZ
4. Gọi Fabric Token SDK để mint token
5. Lưu metadata và trả về `tx_id`

### 5.2 Transfer Token Flow

1. Kiểm tra quyền sở hữu token
2. Xác minh quyền `token:transfer`
3. Lấy cert DID của người gửi + người nhận
4. Thực hiện `Transfer` và ghi nhận `tx_id`

### 5.3 Burn Token Flow

1. Kiểm tra quyền sở hữu + quyền burn
2. Thực hiện `Burn` trên Fabric
3. Ghi log giao dịch

---

## 6. Deployment

* Docker + Kubernetes
* Tích hợp các Secret (JWT Key, Fabric MSP...) qua Kubernetes Secret
* Có thể dùng Helm chart riêng hoặc tích hợp với hệ thống AuthZ, DID hiện có

---

## 7. Logging & Observability

* Logging qua stdout + Elastic hoặc Loki
* Monitoring metrics qua Prometheus + Grafana
* Audit Trail lưu trên DB và có thể băm hash lưu on-chain nếu cần

---

## 8. Kết luận

Thiết kế kỹ thuật này đáp ứng các yêu cầu về chức năng, bảo mật và tích hợp cho một hệ thống tài sản số hoạt động trên nền tảng Fabric. Các service được tách biệt, dễ scale và triển khai độc lập hoặc tích hợp với hệ sinh thái DID/AuthZ sẵn có.

*Cập nhật: 31/05/2025*
