# Sơ Đồ Luồng - Hệ Thống Quản Lý Tài Sản Số

## 1. Luồng Tạo Token

```mermaid
sequenceDiagram
    participant Client
    participant TokenService
    participant DIDService
    participant AuthService
    participant Fabric
    participant DB

    Client->>TokenService: Yêu Cầu Tạo Token
    TokenService->>AuthService: Xác Thực Token
    AuthService-->>TokenService: Token Hợp Lệ
    TokenService->>DIDService: Xác Minh Danh Tính
    DIDService-->>TokenService: Danh Tính Đã Xác Minh
    TokenService->>Fabric: Giao Dịch Tạo Token
    Fabric-->>TokenService: Giao Dịch Đã Xác Nhận
    TokenService->>DB: Lưu Metadata Token
    TokenService-->>Client: Phản Hồi Token Đã Tạo
```

## 2. Luồng Chuyển Token

```mermaid
sequenceDiagram
    participant Sender
    participant TokenService
    participant AuthService
    participant Fabric
    participant DB
    participant Receiver

    Sender->>TokenService: Yêu Cầu Chuyển
    TokenService->>AuthService: Xác Thực Người Gửi
    AuthService-->>TokenService: Người Gửi Hợp Lệ
    TokenService->>Fabric: Kiểm Tra Số Dư
    Fabric-->>TokenService: Số Dư Đủ
    TokenService->>Fabric: Giao Dịch Chuyển
    Fabric-->>TokenService: Giao Dịch Đã Xác Nhận
    TokenService->>DB: Cập Nhật Số Dư
    TokenService-->>Sender: Chuyển Thành Công
    TokenService-->>Receiver: Thông Báo Chuyển
```

## 3. Luồng Xác Thực

```mermaid
sequenceDiagram
    participant User
    participant AuthService
    participant DIDService
    participant Redis
    participant DB

    User->>AuthService: Yêu Cầu Đăng Nhập
    AuthService->>DIDService: Xác Minh Thông Tin
    DIDService-->>AuthService: Thông Tin Hợp Lệ
    AuthService->>DB: Lấy Dữ Liệu Người Dùng
    AuthService->>Redis: Tạo Phiên
    AuthService-->>User: Token JWT
    User->>AuthService: Yêu Cầu API với JWT
    AuthService->>Redis: Xác Thực Phiên
    AuthService-->>User: Phản Hồi Đã Phân Quyền
```

## 4. Luồng Sự Kiện Thời Gian Thực

```mermaid
sequenceDiagram
    participant Client
    participant TokenService
    participant Fabric
    participant Redis
    participant EventStream

    Client->>TokenService: Đăng Ký Nhận Sự Kiện
    TokenService->>Fabric: Đăng Ký Lắng Nghe Sự Kiện
    Fabric->>TokenService: Sự Kiện Xảy Ra
    TokenService->>Redis: Cache Sự Kiện
    TokenService->>EventStream: Phát Sự Kiện
    EventStream-->>Client: Stream Sự Kiện
```

## 5. Luồng KYC/AML

```mermaid
sequenceDiagram
    participant User
    participant DIDService
    participant KYCProvider
    participant AuthService
    participant DB

    User->>DIDService: Nộp Tài Liệu KYC
    DIDService->>KYCProvider: Xác Minh Tài Liệu
    KYCProvider-->>DIDService: Kết Quả Xác Minh
    DIDService->>DB: Lưu Trạng Thái KYC
    DIDService->>AuthService: Cập Nhật Trạng Thái Người Dùng
    DIDService-->>User: Cập Nhật Trạng Thái KYC
```

## 6. Luồng Xử Lý Lỗi

```mermaid
sequenceDiagram
    participant Client
    participant Service
    participant Logger
    participant Monitoring
    participant AlertSystem

    Client->>Service: Yêu Cầu
    Service->>Service: Xử Lý Yêu Cầu
    Service->>Logger: Ghi Log Lỗi
    Service->>Monitoring: Báo Cáo Lỗi
    Monitoring->>AlertSystem: Lỗi Nghiêm Trọng
    Service-->>Client: Phản Hồi Lỗi
    AlertSystem->>Service: Thử Lại/Khôi Phục
```

## 7. Luồng Khôi Phục Hệ Thống

```mermaid
sequenceDiagram
    participant Service
    participant HealthCheck
    participant Kubernetes
    participant Backup
    participant DB

    Service->>HealthCheck: Kiểm Tra Sức Khỏe
    HealthCheck->>Service: Không Khỏe Mạnh
    HealthCheck->>Kubernetes: Khởi Động Lại Service
    Kubernetes->>Service: Instance Mới
    Service->>Backup: Khôi Phục Trạng Thái
    Backup->>DB: Khôi Phục Dữ Liệu
    Service->>HealthCheck: Kiểm Tra Sức Khỏe
    HealthCheck-->>Service: Khỏe Mạnh
```

*Cập nhật: 31/05/2025* 