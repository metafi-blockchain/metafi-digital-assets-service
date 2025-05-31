# Functional Requirements Document - Digital Asset Management System

## Table of Contents
1. [Overview](#1-overview)
2. [System Architecture](#2-system-architecture)
3. [Functional Requirements](#3-functional-requirements)
4. [Non-Functional Requirements](#4-non-functional-requirements)
5. [Service Interfaces](#5-service-interfaces)
6. [User Roles and Permissions](#6-user-roles-and-permissions)
7. [Business Processes](#7-business-processes)
8. [Deployment and Operations](#8-deployment-and-operations)

## 1. Overview

### 1.1 Objectives
Build a Digital Asset Service integrated with authentication and authorization services, supporting the tokenization and management of traditional assets such as real estate, certificates of deposit, and investment funds.

### 1.2 Scope
* Tokenization of physical and financial assets
* Ownership and transaction management
* Integration with AuthN Service for user authentication
* Integration with AuthZ Service for access control
* Integration with DID Service for identity management

### 1.3 Target Users
* Asset owners
* Investors
* System administrators
* Partners and third parties

## 2. System Architecture

### 2.1 System Overview

```mermaid
graph TD
    subgraph "Client Layer"
        Web[Web App]
        Mobile[Mobile App]
        API[API Client]
    end

    subgraph "Service Layer"
        AuthN[AuthN Service]
        AuthZ[AuthZ Service]
        DID[DID Service]
        Asset[Asset Service]
    end

    subgraph "Blockchain Layer"
        Fabric[Fabric Network]
        TokenSDK[Token SDK/Chaincode]
        MSP[MSP Identity]
    end

    subgraph "Storage Layer"
        DB[(Database)]
        Cache[(Redis Cache)]
        Storage[(IPFS/MinIO)]
    end

    Web --> Asset
    Mobile --> Asset
    API --> Asset

    Asset --> AuthN
    Asset --> AuthZ
    Asset --> DID

    Asset --> TokenSDK
    TokenSDK --> Fabric
    DID --> Fabric

    Asset --> DB
    Asset --> Cache
    Asset --> Storage

    %% Interface Labels
    Asset -.->|"Asset ↔ AuthN"| AuthN
    Asset -.->|"Asset ↔ AuthZ"| AuthZ
    Asset -.->|"Asset ↔ DID"| DID
    Asset -.->|"Asset ↔ TokenSDK"| TokenSDK
```

### 2.2 Core Components
* **Asset Service**: 
  * Asset information and metadata management
  * Tokenization and token lifecycle management through Token SDK
  * Token transaction processing on Fabric Network
  * Fabric and DID Service integration
  * Balance and state management

* **Token SDK/Chaincode**:
  * Provides basic token functions (mint, transfer, burn)
  * Manages token state on blockchain
  * Validates token transactions
  * Integrates with Fabric Network

* **AuthN Service**:
  * User authentication
  * Session management
  * JWT issuance

* **AuthZ Service**:
  * Access control
  * Role management
  * Permission checking

* **DID Service**:
  * Identity management
  * KYC verification
  * MSP Identity issuance

### 2.3 Asset Processing Flow

```mermaid
sequenceDiagram
    participant User
    participant AuthN
    participant AuthZ
    participant DID
    participant Asset
    participant TokenSDK
    participant Fabric
    
    User->>AuthN: Login
    AuthN-->>User: JWT Token
    
    User->>Asset: Tokenization Request
    Asset->>AuthN: Validate JWT
    AuthN-->>Asset: Valid
    
    Asset->>AuthZ: Check Permission
    AuthZ-->>Asset: Permission Granted
    
    Asset->>DID: Get DID Info
    DID-->>Asset: DID & MSP Identity
    
    Asset->>TokenSDK: Create Token
    TokenSDK->>Fabric: Submit Transaction
    Fabric->>Fabric: Validate & Commit
    Fabric-->>TokenSDK: Token Created
    TokenSDK-->>Asset: Token Created
    
    Asset-->>User: Asset Tokenized
```

### 2.4 Transaction Flow

```mermaid
sequenceDiagram
    participant User
    participant AuthN
    participant AuthZ
    participant Asset
    participant TokenSDK
    participant Fabric
    
    User->>AuthN: Validate Session
    AuthN-->>User: Session Valid
    
    User->>Asset: Transfer Request
    Asset->>AuthZ: Check Permission
    AuthZ-->>Asset: Permission Granted
    
    Asset->>TokenSDK: Submit Transaction
    TokenSDK->>Fabric: Execute Transaction
    Fabric->>Fabric: Validate & Commit
    Fabric-->>TokenSDK: Transaction Complete
    TokenSDK-->>Asset: Transaction Complete
    
    Asset-->>User: Transfer Confirmed
```

## 5. Service Interfaces

### 5.1 Asset ↔ AuthN Interface

```protobuf
service AuthNService {
    // Validate JWT token
    rpc ValidateToken(ValidateTokenRequest) returns (ValidateTokenResponse);
    
    // Get user information from token
    rpc GetUserInfo(GetUserInfoRequest) returns (GetUserInfoResponse);
    
    // Validate session
    rpc ValidateSession(ValidateSessionRequest) returns (ValidateSessionResponse);
}

message ValidateTokenRequest {
    string jwt_token = 1;
}

message ValidateTokenResponse {
    bool is_valid = 1;
    string user_id = 2;
    repeated string roles = 3;
    int64 expires_at = 4;
}

message GetUserInfoRequest {
    string user_id = 1;
}

message GetUserInfoResponse {
    string user_id = 1;
    string email = 2;
    string full_name = 3;
    repeated string roles = 4;
    bool is_active = 5;
}

message ValidateSessionRequest {
    string session_id = 1;
}

message ValidateSessionResponse {
    bool is_valid = 1;
    string user_id = 2;
    int64 expires_at = 3;
}
```

### 5.2 Asset ↔ AuthZ Interface

```protobuf
service AuthZService {
    // Check access permission
    rpc CheckPermission(CheckPermissionRequest) returns (CheckPermissionResponse);
    
    // Get user's permissions
    rpc GetUserPermissions(GetUserPermissionsRequest) returns (GetUserPermissionsResponse);
    
    // Check asset ownership
    rpc CheckAssetOwnership(CheckAssetOwnershipRequest) returns (CheckAssetOwnershipResponse);
}

message CheckPermissionRequest {
    string user_id = 1;
    string resource = 2;
    string action = 3;
}

message CheckPermissionResponse {
    bool allowed = 1;
    string reason = 2;
}

message GetUserPermissionsRequest {
    string user_id = 1;
}

message GetUserPermissionsResponse {
    repeated string permissions = 1;
    map<string, string> constraints = 2;
}

message CheckAssetOwnershipRequest {
    string user_id = 1;
    string asset_id = 2;
}

message CheckAssetOwnershipResponse {
    bool is_owner = 1;
    string ownership_type = 2; // FULL, PARTIAL, NONE
    double ownership_percentage = 3;
}
```

### 5.3 Asset ↔ DID Interface

```protobuf
service AssetService {
    // Create new asset
    rpc CreateAsset(CreateAssetRequest) returns (CreateAssetResponse);
    
    // Update asset information
    rpc UpdateAsset(UpdateAssetRequest) returns (UpdateAssetResponse);
    
    // Get asset information
    rpc GetAsset(GetAssetRequest) returns (GetAssetResponse);
    
    // Verify ownership
    rpc VerifyOwnership(VerifyOwnershipRequest) returns (VerifyOwnershipResponse);
}

message CreateAssetRequest {
    string owner_did = 1;
    AssetType asset_type = 2;
    string metadata_uri = 3;
    map<string, string> properties = 4;
}

message CreateAssetResponse {
    string asset_id = 1;
    string token_id = 2;
    string status = 3;
}

enum AssetType {
    REAL_ESTATE = 0;
    CERTIFICATE_OF_DEPOSIT = 1;
    INVESTMENT_FUND = 2;
    STABLECOIN = 3;
}
```

### 5.4 Implementation Notes

* **gRPC Communication**:
  * Use gRPC for all internal service communication
  * Implement retry mechanism for service calls
  * Use circuit breaker pattern
  * Implement request timeouts

* **Error Handling**:
  * Define clear error codes for each service
  * Implement proper error propagation
  * Log detailed error information
  * Implement retry mechanism for temporary errors

* **Security**:
  * Encrypt all internal communication
  * Implement service-to-service authentication
  * Validate input data
  * Rate limit all endpoints

* **Monitoring**:
  * Track latency for all service calls
  * Monitor error rates
  * Set up alerts for issues
  * Log detailed debugging information

## 6. User Roles and Permissions

### 6.1 Role Definitions

```protobuf
enum UserRole {
    // System administration roles
    SYSTEM_ADMIN = 0;      // System administrator
    COMPLIANCE_OFFICER = 1; // Compliance officer
    AUDITOR = 2;           // Auditor
    
    // Asset management roles
    ASSET_OWNER = 10;      // Asset owner
    ASSET_MANAGER = 11;    // Asset manager
    ASSET_OPERATOR = 12;   // Asset operator
    
    // Investment roles
    INVESTOR = 20;         // Investor
    INSTITUTIONAL_INVESTOR = 21; // Institutional investor
    RETAIL_INVESTOR = 22;  // Retail investor
    
    // Partner roles
    BROKER = 30;           // Broker
    CUSTODIAN = 31;        // Custodian
    LEGAL_ADVISOR = 32;    // Legal advisor
}
```

### 6.2 Role Permissions

#### 6.2.1 System Administration
* **SYSTEM_ADMIN**:
  * Full system management
  * System configuration
  * User and role management
  * Access to all logs and metrics
  * Highest system privileges

* **COMPLIANCE_OFFICER**:
  * KYC review and approval
  * Transaction monitoring
  * Compliance reporting
  * Risk assessment
  * No system configuration access

* **AUDITOR**:
  * Full transaction history access
  * System log access
  * Audit report generation
  * No modification rights

#### 6.2.2 Asset Management
* **ASSET_OWNER**:
  * Asset creation and management
  * Token issuance
  * Distribution policy decisions
  * Asset reporting
  * No system configuration access

* **ASSET_MANAGER**:
  * Asset operation management
  * Transaction execution
  * Management reporting
  * No token issuance rights

* **ASSET_OPERATOR**:
  * Operational activities
  * Asset status updates
  * No financial management rights

#### 6.2.3 Investors
* **INVESTOR** (Base role):
  * Asset information access
  * Transaction execution
  * Investment reporting
  * No asset creation rights

* **INSTITUTIONAL_INVESTOR**:
  * All INVESTOR rights
  * Large volume trading
  * Dedicated API access
  * Enhanced KYC requirements

* **RETAIL_INVESTOR**:
  * Limited trading
  * Basic information access
  * Basic KYC requirements

#### 6.2.4 Partners
* **BROKER**:
  * Order creation and management
  * Market information access
  * No direct trading rights

* **CUSTODIAN**:
  * Physical asset management
  * Ownership verification
  * No trading rights

* **LEGAL_ADVISOR**:
  * Legal document access
  * Legal report generation
  * No modification rights

### 6.3 Role Assignment Process

```mermaid
sequenceDiagram
    participant User
    participant AuthN
    participant AuthZ
    participant DID
    participant Compliance
    
    User->>AuthN: Register account
    AuthN->>DID: Create DID
    DID-->>AuthN: DID Created
    
    User->>Compliance: Submit KYC
    Compliance->>Compliance: Verify KYC
    Compliance->>AuthZ: Update role
    
    AuthZ->>AuthZ: Apply policy
    AuthZ-->>User: Role & Permissions
```

### 6.4 Policy Management

```protobuf
message RolePolicy {
    string role = 1;
    repeated string permissions = 2;
    map<string, string> constraints = 3;
    int64 max_transaction_amount = 4;
    repeated string allowed_asset_types = 5;
}

message UserPolicy {
    string user_id = 1;
    string role = 2;
    KYCStatus kyc_status = 3;
    repeated string additional_permissions = 4;
    map<string, string> custom_constraints = 5;
}
```

### 6.5 Implementation Notes

* **Role Hierarchy**:
  * Implement role inheritance
  * Support custom roles
  * Allow permission overrides
  * Audit log all changes

* **KYC Integration**:
  * KYC level affects permissions
  * Automatic role updates after KYC
  * Support enhanced KYC
  * Store KYC history

* **Compliance**:
  * Role-based compliance checks
  * Role-based transaction limits
  * Violation reporting
  * Anomaly alerts

* **Monitoring**:
  * Track role changes
  * Monitor permission usage
  * Alert on policy violations
  * Regular compliance reports

## 7. Business Processes

### 7.1 Asset Tokenization Process
```mermaid
sequenceDiagram
    participant Owner
    participant System
    participant Compliance
    participant Blockchain
    
    Owner->>System: Tokenization request
    System->>Compliance: Compliance check
    Compliance-->>System: Approval
    System->>Blockchain: Create token
    Blockchain-->>System: Confirmation
    System-->>Owner: Completion notification
```

### 7.2 Trading Process
```mermaid
sequenceDiagram
    participant Buyer
    participant Seller
    participant System
    participant Blockchain
    
    Buyer->>System: Place buy order
    Seller->>System: Place sell order
    System->>System: Match orders
    System->>Blockchain: Execute transaction
    Blockchain-->>System: Confirmation
    System-->>Buyer: Update balance
    System-->>Seller: Update balance
```

## 8. Deployment and Operations

### 8.1 Deployment Requirements
* Kubernetes cluster
* Hyperledger Fabric network
* Database cluster
* Monitoring system

### 8.2 Operational Procedures
* Monitoring and alerting
* Backup and restore
* Scaling and load balancing
* Security patching

### 8.3 Deployment Plan
* Phase 1: Core services
* Phase 2: Token management
* Phase 3: Trading features
* Phase 4: Advanced features

*Last Updated: 31/05/2025* 