# Tài Liệu Thiết Kế Kỹ Thuật - Hệ Thống Quản Lý Tài Sản Số

## 1. Kiến Trúc Hệ Thống

### 1.1 Kiến Trúc Tổng Quan

```mermaid
graph TD
    %% Client Layer
    subgraph "Client Layer"
        Web[Web App]
        Mobile[Mobile App]
        API[API Client]
    end

    %% Middleware Layer
    subgraph "Middleware Layer"
        Gateway[Kong API Gateway]
        AuthN[AuthN Service]
        AuthZ[AuthZ Service]
        DID[DID Middleware<br/>ACA-Py]
    end

    %% Application Layer
    subgraph "Application Layer"
        Asset[Asset Service]
        Token[Token Service]
    end

    %% Blockchain Layer
    subgraph "Blockchain Layer"
        Fabric[Fabric Network<br/>Token SDK]
        Indy[Hyperledger Indy]
    end

    %% Storage Layer
    subgraph "Storage Layer"
        DB[(PostgreSQL<br/>Database)]
        Cache[(Redis Cache)]
        Storage[(IPFS / MinIO)]
    end

    %% Flows from Clients
    Web --> Gateway
    Mobile --> Gateway
    API --> Gateway

    %% Gateway routing
    Gateway --> AuthN
    Gateway --> AuthZ
    Gateway --> Asset
    Gateway --> DID

    %% Auth middleware interaction
    AuthN --> Asset
    AuthZ --> Asset
    DID --> Asset

    %% AssetService integration
    Asset --> Token

    %% Token → Blockchain
    Token --> Fabric

    %% DID interaction with Indy (off-chain)
    DID -->|ACA-Py API| DIDAgent[(Indy Agent)] --> Indy

    %% Asset data persistence
    Asset --> DB
    Asset --> Cache
    Asset --> Storage
```

### 1.2 Tổng Quan Thành Phần

#### 1.2.1 Client Layer
* **Ứng Dụng Web**
  * Frontend (React/Next.js)
  * TypeScript
  * Material-UI/Tailwind CSS
  * gRPC-web client
  * Redux/Context API

* **Ứng Dụng Di Động**
  * React Native
  * TypeScript
  * Native Base
  * gRPC client
  * Redux

* **API Client**
  * REST/gRPC client
  * SDK cho các ngôn ngữ phổ biến

#### 1.2.2 Middleware Layer
* **Kong API Gateway**
  * Quản lý routing
  * Rate limiting
  * Load balancing
  * API documentation (Swagger/OpenAPI)

* **AuthN Service**
  * Quản lý token JWT
  * Quản lý phiên
  * Xác thực đa yếu tố (MFA)

* **AuthZ Service**
  * Kiểm soát truy cập dựa trên vai trò (RBAC)
  * Quản lý quyền
  * Policy enforcement

* **DID Middleware (ACA-Py)**
  * Quản lý danh tính (DID)
  * Quản lý chứng chỉ
  * Xác thực KYC
  * Tích hợp với Indy Agent
  * Cấp phát MSP Identity

#### 1.2.3 Application Layer
* **Asset Service**
  * Máy chủ gRPC
  * Quản lý metadata tài sản
  * Xác thực DID chủ sở hữu
  * Quản lý vòng đời tài sản
  * Xử lý từ chối và yêu cầu sửa đổi
  * Ghi log audit
  * Streaming thời gian thực

* **Token Service** (External Service)
  * Giao tiếp qua gRPC
  * Quản lý vòng đời token
  * Tích hợp với Fabric Network

#### 1.2.4 Blockchain Layer
* **Fabric Network (Token SDK)**
  * Mạng blockchain riêng
  * ChainCode
  * Tích hợp Token SDK
  * Hệ thống sự kiện
  * Smart contract quản lý token

#### 1.2.5 Storage Layer
* **PostgreSQL Database**
  * Lưu trữ metadata tài sản
  * Quản lý trạng thái
  * Audit logs
  * Transaction history

* **Redis Cache**
  * Session management
  * Rate limiting
  * Temporary data storage
  * Real-time data caching

* **IPFS / MinIO**
  * Lưu trữ metadata tài sản
  * Asset documents
  * Immutable storage
  * Content addressing

## 2. Stack Công Nghệ

### 2.1 Backend Services

* **Asset Service**
  * Golang
  * gRPC
  * PostgreSQL
  * Redis (cache)
  * IPFS/MinIO (storage)
  * Prometheus/Grafana (monitoring)
* **Token Service**
  * NestJs
  * gRPC
  * PostgreSQL
  * Redis (cache)
  * IPFS/MinIO (storage)
  * Prometheus/Grafana (monitoring)

* **AuthN/AuthZ**
  * Golang
  * gRPC protocol
  * JWT
  * Redis (session)
  * PostgreSQL

* **DID Middleware**
  * Golang
  * gRPC protocol
  * Aries Cloud Agent Python (ACA-Py)
  * Hyperledger Indy
  * PostgreSQL
  * Redis (cache)

### 2.2 Frontend

* **Ứng Dụng Web**
  * React/Next.js
  * TypeScript
  * Material-UI/Tailwind CSS
  * gRPC-web client
  * Redux/Context API

* **Ứng Dụng Di Động**
  * React Native
  * TypeScript
  * Native Base
  * gRPC client
  * Redux

### 2.3 Hạ Tầng

* **Cơ Sở Dữ Liệu**
  * PostgreSQL (cơ sở dữ liệu chính)
  * Redis (cache)
  * MongoDB (tùy chọn cho phân tích)

* **Lưu Trữ**
  * IPFS/MinIO (lưu trữ metadata)
  * Lưu trữ tương thích S3

* **Blockchain**
  * Hyperledger Fabric
  * Fabric Token SDK
  * Chaincode (Go)

## 3. Thiết Kế Chi Tiết

### 3.1 Asset Service

#### 3.1.1 Vai Trò Chính

- Quản lý metadata tài sản
- Quản lý vòng đời tài sản: tạo, cập nhật, token hóa
- Kiểm tra quyền sở hữu qua DID
- Tích hợp với Token Service
- Cung cấp API/gRPC cho frontend và các dịch vụ khác

#### 3.1.2 Giao Diện Dịch Vụ

```go
// Giao Diện Dịch Vụ Asset
interface AssetService {
    // Quản lý tài sản
    CreateAsset(ctx context.Context, req *CreateAssetRequest) (*Asset, error)
    UpdateAsset(ctx context.Context, req *UpdateAssetRequest) (*Asset, error)
    GetAsset(ctx context.Context, id string) (*Asset, error)
    ListAssets(ctx context.Context, filter *AssetFilter) ([]*Asset, error)
    
    // Token hóa
    TokenizeAsset(ctx context.Context, req *TokenizeAssetRequest) (*Asset, error)
    TransferOwnership(ctx context.Context, req *TransferOwnershipRequest) error
    
    // Quản lý trạng thái
    UpdateAssetState(ctx context.Context, req *UpdateStateRequest) error
    RejectAsset(ctx context.Context, req *RejectRequest) error
    RequestModification(ctx context.Context, req *ModificationRequest) error
    
    // Sự kiện
    SubscribeToEvents(callback EventCallback) error
    ProcessEvents(event *AssetEvent) error
}

// Cấu trúc dữ liệu
type Asset struct {
    ID          string          `json:"id"`
    Name        string          `json:"name"`
    Type        string          `json:"type"`
    OwnerDID    string          `json:"owner_did"`
    Value       decimal.Decimal `json:"value"`
    Status      string          `json:"status"` // DRAFT, SUBMITTED, APPROVED, REJECTED, AWAITING_FIX, TOKENIZED, ARCHIVED
    Metadata    json.RawMessage `json:"metadata"`
    CreatedAt   time.Time       `json:"created_at"`
    UpdatedAt   time.Time       `json:"updated_at"`
}
```

#### 3.1.3 Schema Cơ Sở Dữ Liệu

```sql
-- Bảng Assets
CREATE TABLE assets (
    id UUID PRIMARY KEY,
    name TEXT NOT NULL,
    type TEXT NOT NULL,
    owner_did TEXT NOT NULL,
    value DECIMAL NOT NULL,
    status TEXT NOT NULL,
    metadata JSONB,
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP NOT NULL
);

-- Bảng Asset Events
CREATE TABLE asset_events (
    id UUID PRIMARY KEY,
    asset_id UUID REFERENCES assets(id),
    event_type TEXT NOT NULL,
    data JSONB,
    created_at TIMESTAMP NOT NULL
);

-- Bảng Asset Audit Logs
CREATE TABLE asset_audit_logs (
    id UUID PRIMARY KEY,
    asset_id UUID REFERENCES assets(id),
    trace_id TEXT NOT NULL,
    operation TEXT NOT NULL,
    level TEXT NOT NULL,
    message TEXT NOT NULL,
    error TEXT,
    metadata JSONB,
    ip_address TEXT,
    user_agent TEXT,
    session_id TEXT,
    created_at TIMESTAMP NOT NULL
);
```

#### 3.1.4 Token Service Interface (gRPC)

```protobuf
// Interface giao tiếp với Token Service
service TokenService {
    // Quản lý token
    rpc CreateToken(CreateTokenRequest) returns (Token);
    rpc TransferToken(TransferRequest) returns (Transaction);
    rpc BurnToken(BurnRequest) returns (Transaction);
    
    // Truy vấn
    rpc GetTokenBalance(BalanceRequest) returns (Balance);
    rpc GetTransactionHistory(HistoryRequest) returns (TransactionList);
    
    // Sự kiện
    rpc SubscribeToEvents(SubscribeRequest) returns (stream TokenEvent);
}

message CreateTokenRequest {
    string asset_id = 1;
    string owner_did = 2;
    string type = 3;
    string amount = 4;
    bytes metadata = 5;
}

message TransferRequest {
    string token_id = 1;
    string from_did = 2;
    string to_did = 3;
    string amount = 4;
}

message TokenEvent {
    string event_id = 1;
    string event_type = 2;
    string token_id = 3;
    bytes data = 4;
    int64 timestamp = 5;
}
```

### 3.2 Monitoring và Logging

#### 3.2.1 Metrics

```go
// Định nghĩa metrics
type Metrics struct {
    // Asset metrics
    AssetCreation     *prometheus.CounterVec
    AssetStateChanges *prometheus.CounterVec
    AssetOperations   *prometheus.HistogramVec
    
    // System metrics
    RequestLatency    *prometheus.HistogramVec
    ErrorRate         *prometheus.CounterVec
    ActiveConnections *prometheus.GaugeVec
}

// Khởi tạo metrics
func NewMetrics() *Metrics {
    return &Metrics{
        AssetCreation: prometheus.NewCounterVec(
            prometheus.CounterOpts{
                Name: "asset_creation_total",
                Help: "Tổng số tài sản được tạo",
            },
            []string{"type", "status"},
        ),
        // ... other metrics
    }
}
```

#### 3.2.2 Logging

```go
// Cấu trúc log event
type LogEvent struct {
    TraceID    string                 `json:"trace_id"`
    Service    string                 `json:"service"`
    Operation  string                 `json:"operation"`
    Level      string                 `json:"level"`
    Message    string                 `json:"message"`
    Error      string                 `json:"error,omitempty"`
    Metadata   map[string]interface{} `json:"metadata,omitempty"`
    IPAddress  string                 `json:"ip_address,omitempty"`
    UserAgent  string                 `json:"user_agent,omitempty"`
    SessionID  string                 `json:"session_id,omitempty"`
    Timestamp  time.Time             `json:"timestamp"`
}

// Xử lý log event
func (s *serviceImpl) logEvent(level zapcore.Level, operation string, message string, err error, metadata map[string]interface{}) {
    event := &LogEvent{
        TraceID:    trace.SpanContextFromContext(s.ctx).TraceID().String(),
        Service:    s.serviceName,
        Operation:  operation,
        Level:      level.String(),
        Message:    message,
        Timestamp:  time.Now(),
        Metadata:   metadata,
        IPAddress:  s.getClientIP(),
        UserAgent:  s.getUserAgent(),
        SessionID:  s.getSessionID(),
    }
    
    if err != nil {
        event.Error = err.Error()
    }
    
    // Ghi log vào hệ thống
    s.logger.Log(level, message, zap.Any("event", event))
    
    // Ghi log audit nếu cần
    if s.shouldAudit(level, operation) {
        s.auditLogger.Log(event)
    }
    
    // Cập nhật metrics
    s.metrics.AuditLogVolume.WithLabelValues(
        event.Service,
        event.Operation,
        event.Level,
    ).Inc()
}
```

## 4. Tính Năng Mở Rộng (Future Scope)

### 4.1 Compliance Service
- Kiểm tra tuân thủ
- Xác thực giao dịch
- Quản lý rủi ro

### 4.2 Order Service
- Quản lý sổ lệnh
- Khớp lệnh
- Quản lý giao dịch

### 4.3 Phân Phối Lợi Nhuận
- Tính toán lợi tức tự động
- Lập lịch phân phối
- Xử lý thanh toán

### 4.4 Quản Lý Quyền Biểu Quyết
- Tính toán quyền biểu quyết
- Quản lý cuộc bỏ phiếu
- Theo dõi kết quả


---

## 5. Phần mở rộng (Future Scope)

Các chức năng/phân hệ sau đây sẽ được xem xét phát triển ở các giai đoạn tiếp theo:

- **Phân phối lợi nhuận** (Profit Distribution/Dividend)
- **Voting / Quyền biểu quyết theo sở hữu**
- **Compliance Service** (Dịch vụ tuân thủ)
- **Order Service** (Sổ lệnh, khớp lệnh, giao dịch trên sàn)
- **Marketplace** (Sàn giao dịch tài sản, giao dịch fraction, đặt lệnh mua/bán)

Các nội dung này sẽ được bổ sung chi tiết trong các bản cập nhật tài liệu tiếp theo khi hệ thống mở rộng phạm vi. 