# Flow Diagram - Digital Asset Management System

## 1. Token Creation Flow

```mermaid
sequenceDiagram
    participant Client
    participant TokenService
    participant DIDService
    participant AuthService
    participant Fabric
    participant DB

    Client->>TokenService: CreateToken Request
    TokenService->>AuthService: Validate Token
    AuthService-->>TokenService: Token Valid
    TokenService->>DIDService: Verify Identity
    DIDService-->>TokenService: Identity Verified
    TokenService->>Fabric: Create Token Transaction
    Fabric-->>TokenService: Transaction Confirmed
    TokenService->>DB: Store Token Metadata
    TokenService-->>Client: Token Created Response
```

## 2. Token Transfer Flow

```mermaid
sequenceDiagram
    participant Sender
    participant TokenService
    participant AuthService
    participant Fabric
    participant DB
    participant Receiver

    Sender->>TokenService: Transfer Request
    TokenService->>AuthService: Validate Sender
    AuthService-->>TokenService: Sender Valid
    TokenService->>Fabric: Check Balance
    Fabric-->>TokenService: Sufficient Balance
    TokenService->>Fabric: Transfer Transaction
    Fabric-->>TokenService: Transaction Confirmed
    TokenService->>DB: Update Balances
    TokenService-->>Sender: Transfer Success
    TokenService-->>Receiver: Transfer Notification
```

## 3. Authentication Flow

```mermaid
sequenceDiagram
    participant User
    participant AuthService
    participant DIDService
    participant Redis
    participant DB

    User->>AuthService: Login Request
    AuthService->>DIDService: Verify Credentials
    DIDService-->>AuthService: Credentials Valid
    AuthService->>DB: Get User Data
    AuthService->>Redis: Create Session
    AuthService-->>User: JWT Token
    User->>AuthService: API Request with JWT
    AuthService->>Redis: Validate Session
    AuthService-->>User: Authorized Response
```

## 4. Real-time Event Flow

```mermaid
sequenceDiagram
    participant Client
    participant TokenService
    participant Fabric
    participant Redis
    participant EventStream

    Client->>TokenService: Subscribe to Events
    TokenService->>Fabric: Register Event Listener
    Fabric->>TokenService: Event Occurred
    TokenService->>Redis: Cache Event
    TokenService->>EventStream: Publish Event
    EventStream-->>Client: Stream Event
```

## 5. KYC/AML Flow

```mermaid
sequenceDiagram
    participant User
    participant DIDService
    participant KYCProvider
    participant AuthService
    participant DB

    User->>DIDService: Submit KYC Documents
    DIDService->>KYCProvider: Verify Documents
    KYCProvider-->>DIDService: Verification Result
    DIDService->>DB: Store KYC Status
    DIDService->>AuthService: Update User Status
    DIDService-->>User: KYC Status Update
```

## 6. Error Handling Flow

```mermaid
sequenceDiagram
    participant Client
    participant Service
    participant Logger
    participant Monitoring
    participant AlertSystem

    Client->>Service: Request
    Service->>Service: Process Request
    Service->>Logger: Log Error
    Service->>Monitoring: Report Error
    Monitoring->>AlertSystem: Critical Error
    Service-->>Client: Error Response
    AlertSystem->>Service: Retry/Recovery
```

## 7. System Recovery Flow

```mermaid
sequenceDiagram
    participant Service
    participant HealthCheck
    participant Kubernetes
    participant Backup
    participant DB

    Service->>HealthCheck: Check Health
    HealthCheck->>Service: Unhealthy
    HealthCheck->>Kubernetes: Restart Service
    Kubernetes->>Service: New Instance
    Service->>Backup: Restore State
    Backup->>DB: Restore Data
    Service->>HealthCheck: Health Check
    HealthCheck-->>Service: Healthy
```

*Last Updated: 31/05/2025* 