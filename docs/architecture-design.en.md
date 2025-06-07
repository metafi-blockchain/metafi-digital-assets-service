# 📐 Architecture Design Document
### Digital Asset Management System

---

## 1. 📘 Overview

Hệ thống Digital Asset Management gồm các thành phần phục vụ:
- Token hóa tài sản (Asset Tokenization)
- Xác thực định danh DID
- Phân quyền và xác thực người dùng
- Tương tác với blockchain Hyperledger Fabric
- Lưu trữ dữ liệu off-chain: metadata, tài liệu, cache

---

## 2. 🧱 Kiến trúc tầng (Layered Architecture)

```mermaid
graph TD
    subgraph "Client Layer"
        Web[Web App]
        Mobile[Mobile App]
        API[API Client]
        Explorer[UI Explorer]
    end

    subgraph "Middleware Layer"
        Gateway[Kong API Gateway]
        AuthN[AuthN Service]
        AuthZ[AuthZ Service]
        DID["DID Service"]
    end

    subgraph "Application Layer"
        Asset[Asset Service]
        Token[Token Service]
        Firefly[Firefly Service]
        ExplorerService[Explorer Service]
    end

    subgraph "Blockchain Layer"
        Fabric["Hyperledger Fabric<br/>Token SDK/Chaincode"]
        PublicChain[Public Blockchain<br/>  Smart contract]
    end

    subgraph "Storage Layer"
        DB[(PostgreSQL)]
        Cache[(Redis)]
        Storage[(IPFS / MinIO)]
    end

    subgraph "Event Streaming Layer"
        Kafka[(Kafka)]
    end

    Web --> Gateway
    Mobile --> Gateway
    API --> Gateway
    Explorer --> Gateway

    Gateway --> AuthN
    Gateway --> AuthZ
    Gateway --> Asset
    Gateway --> DID

    AuthN --> Asset
    AuthZ --> Asset
    DID --> Asset

    Asset --> Token
    Token --> Firefly
    Firefly --> |fabconnect| Fabric
    Firefly --> |evm connect| PublicChain

    Asset --> DB
    Asset --> Cache
    Asset --> Storage
    DID -->|ACA-Py| Fabric

    Asset --> Kafka
    Kafka --> Asset
    Token --> Kafka
    Kafka --> Token
    Firefly --> Kafka
    Kafka --> Firefly

    Explorer --> ExplorerService
    ExplorerService --> Firefly
```

---

## 3. 🧩 Component Design

### 3.1 Gateway (Kong)
- API routing, rate limit, auth enforcement
- Swagger/OpenAPI exposure

### 3.2 AuthN / AuthZ
- JWT issuance and validation
- Role-Based Access Control (RBAC)
- Session & token management (via Redis)

### 3.3 DID Service
- Tích hợp với ACA-Py & AnonCreds
- Quản lý định danh DID, định danh người dùng và chứng chỉ
- Tạo/mapping định danh với keystore Hyperledger Fabric (MSP ID)

### 3.4 Asset Service
- Quản lý vòng đời tài sản: tạo, cập nhật, yêu cầu sửa
- Xác minh quyền sở hữu thông qua DID
- Giao tiếp với Token Service để thực hiện token hóa
- Gửi/nhận sự kiện qua gRPC

### 3.5 Token Service
- Tạo và quản lý token đại diện tài sản
- Thực hiện các hành vi: mint, burn, transfer
- Giao tiếp gRPC với Asset Service và Blockchain layer

### 3.6 Fabric Blockchain Layer
- Hyperledger Fabric RAFT-based network
- Token SDK for managing token logic
- Chaincode viết bằng Golang

### 3.x 🗂️ Domain Model Diagram

Sơ đồ dưới đây mô tả các thực thể chính (Asset, Token, DID, User) và mối quan hệ giữa chúng trong hệ thống Digital Asset Management.

```mermaid
classDiagram
    %% User & Identity
    class User {
        +string user_id
        +string email
        +Role role
        +bool kyc_verified
    }

    class DID {
        +string did_id
        +string msp_id
        +KYCStatus status
    }

    %% Asset & Token
    class Asset {
        +string asset_id
        +string owner_did
        +AssetType type
        +string metadata_uri
        +string status
        +Timestamp created_at
    }

    class Token {
        +string token_id
        +string asset_id
        +uint total_supply
        +string symbol
        +uint decimals
        +string name
    }

    %% Transaction & Ownership
    class Transaction {
        +string tx_id
        +string from_did
        +string to_did
        +string token_id
        +uint amount
        +Timestamp created_at
    }

    class Ownership {
        +string asset_id
        +string did
        +float percentage
        +string ownership_type
    }

    %% Relationships
    User --> DID : mapped 1-1
    DID --> Asset : owns
    Asset --> Token : is_tokenized_by
    Token --> Transaction : used_in
    DID --> Transaction : initiates_or_receives
    Asset --> Ownership : has
    DID --> Ownership : owns_share
```

**Giải thích:**
- Mỗi User chỉ có một DID (1-1 mapping).
- Một User có thể sở hữu nhiều Asset (thông qua DID).
- Asset được sở hữu bởi một DID, và có thể được token hóa thành nhiều Token.
- Token đại diện cho quyền sở hữu/tài sản, liên kết với Asset và tham gia các Transaction.
- DID có thể là chủ sở hữu (Ownership) của nhiều Asset, và tham gia các Transaction.

---

## 4. 🗄️ Data & Storage Design

| Layer | Tech | Purpose |
|-------|------|---------|
| Database | PostgreSQL | Tài sản, lịch sử, trạng thái |
| Cache | Redis | Session, caching real-time data |
| Storage | IPFS / MinIO | File đính kèm, chứng chỉ, metadata |

---

## 5. 📡 Protocols & Communication

| Service | Protocol | Description |
|---------|----------|-------------|
| Frontend → Gateway | HTTPS (REST/gRPC-web) | Truy cập dịch vụ backend |
| Gateway → Services | gRPC | Hiệu suất cao, typed APIs |
| Token ↔ Fabric | Fabric SDK | Submit & query transaction |
| DID ↔ ACA-Py | HTTPS (REST) | API định danh với Aries agent |

---

## 6. 🔒 Security Considerations

- mTLS giữa các service (gRPC mutual TLS)
- Token validation tại gateway và middleware
- DID resolution + validation để gắn identity
- RBAC enforced by AuthZ per route & action
- Audit log với trace ID theo chuẩn JSON logging

---

## 7. 📈 Scalability

- Horizontal scaling: gateway, services
- Independent deployment: Asset / Token service
- Stateless gRPC services + Redis session caching
- Asynchronous events: chuẩn bị tích hợp Kafka / NATS

---

## 8. 🔄 Sequence Diagrams

### 8.1 Asset Tokenization Flow

```mermaid
sequenceDiagram
    participant User as End User
    participant Web as Web/Mobile Client
    participant Gateway as API Gateway (Kong)
    participant Asset as Asset Service
    participant AuthN as AuthN
    participant DID as "DID Middleware"
    participant Token as Token Service
    participant Fabric as "Hyperledger Fabric"

    User->>Web: Submit Tokenization Request
    Web->>Gateway: gRPC /tokenizeAsset
    Gateway->>AuthN: Verify JWT
    Gateway->>DID: Resolve Owner DID
    Gateway->>Asset: TokenizeAsset()
    Asset->>Token: CreateToken(asset_id, owner_did, amount)
    Token->>Fabric: Mint Token via Fabric SDK
    Fabric-->>Token: Transaction success
    Token-->>Asset: Token created
    Asset-->>Gateway: Asset status: TOKENIZED
    Gateway-->>Web: Return success
```

---

## 9. 🖥️ Deployment Diagram

```mermaid
graph TD
    subgraph K8s Cluster
        subgraph API Layer
            GatewayPod[Kong API Gateway Pod]
        end

        subgraph Services
            AssetPod[Asset Service Pod]
            TokenPod[Token Service Pod]
            AuthNPod[AuthN Service]
            AuthZPod[AuthZ Service]
            DIDPod["DID Middleware"]
        end

        subgraph Blockchain
            FabricPeer1["Fabric Peer Org1"]
            FabricPeer2["Fabric Peer Org2"]
            Orderer[Orderer Node]
            Indy[Indy Node]
        end

        subgraph Storage
            DBPod[(PostgreSQL)]
            RedisPod[(Redis)]
            IPFSPod[(IPFS / MinIO)]
        end

        subgraph Event Streaming Layer
            KafkaPod[(Kafka)]
        end

        GatewayPod -->|gRPC| AssetPod
        GatewayPod --> AuthNPod
        GatewayPod --> AuthZPod
        GatewayPod --> DIDPod

        AssetPod --> TokenPod
        TokenPod --> FabricPeer1
        DIDPod -->|HTTP| ACA-PyPod

        AssetPod --> DBPod
        TokenPod --> DBPod
        AuthNPod --> RedisPod
        DIDPod --> RedisPod
        AssetPod --> IPFSPod
        ACA-PyPod --> Indy

        AssetPod --> KafkaPod
        KafkaPod --> AssetPod
        TokenPod --> KafkaPod
        TokenPod --> KafkaPod
        AuthZPod --> KafkaPod
        AuthNPod --> KafkaPod
        DIDPod --> KafkaPod
    end
```

---

## 10. Optional Extensions

| Diagram | Description |
|---------|-------------|
| ✅ Use Case Diagram | Đã thể hiện ở RD |
| ✅ Component Diagram | Bao phủ trong phần "Component Design" |
| ✅ Domain Model Diagram | Đã bổ sung, làm rõ các thực thể: Asset, Token, DID, User |
| ⏳ Event Flow Diagram | Đề xuất bổ sung khi tích hợp Kafka / NATS để minh họa các event chính (AssetCreated, TokenMinted, OwnershipTransferred...) và các service liên quan |
| ✅ Metrics / Logging Flow | Đã mô tả trong phần Monitoring
