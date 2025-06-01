# Functional Requirements - Digital Asset Management System

## 1. System Objectives

Build a digital asset management system integrated with Hyperledger Fabric blockchain, supporting the tokenization of traditional assets such as real estate, certificates of deposit, and investment funds. The system must ensure identity verification through DID, access control through AuthZ, and support token transactions through Fabric Token SDK.

---

## 2. Overall System Architecture

* **DID Service**: Provides DID (Decentralized Identifiers) and maps users to system identities (MSP, cert).
* **AuthZ Service**: Manages access control based on user roles (RBAC).
* **Token Service**: Implements token logic (mint, transfer, burn, view) through Fabric Token SDK.
* **Client App**: Frontend interface for user interaction.
* **Fabric Network**: Records tokenized asset transactions on the private blockchain network.

---

## 3. Functional Requirements

### 3.1 Digital Assets Tokenization

* Support tokenization of physical and financial assets:
  * Real Estate
  * Certificates of Deposit (CDs)
  * Investment Fund Certificates (IFCs)
  * Stablecoins (backed by fiat, commodities, or crypto)
* Each token represents full or partial ownership of the asset
* Tokenization process includes:
  * Asset identification and valuation
  * Asset custody (if required)
  * Token structure design (fungible / NFT / fractional NFT)
  * Token issuance (mint) through smart contracts
  * Ownership recording on blockchain
  * Token trading and transfer through on-chain mechanisms
* Integration of metadata (IPFS or external storage) attached to tokens
* Support for whitelist/blacklist to ensure only KYC-verified users can interact with specific legal tokens

### 3.2 Token Creation and Issuance

* Receive asset information from users/administrators
* Call DID to map owner's MSP/certificate
* Verify mint token permissions through AuthZ
* Call Fabric Token SDK to create token (UTXO based)
* Store metadata describing the asset and corresponding token

### 3.3 Token Transfer & Trading

* Allow users to transfer tokens to others
* Verify permissions through AuthZ
* Call DID → get sender/receiver cert
* Execute transactions using Token SDK's `Transfer` function
* Support P2P trading and marketplace integration
* Implement atomic delivery vs payment (DvP) mechanism

### 3.4 Token Burn

* Allow token owners to actively burn tokens
* Verify permissions through AuthZ
* Call DID → verify identity and cert
* Call `Burn` SDK to remove token from network

### 3.5 Balance and Transaction History Query

* Allow users to view their token balance
* Enable transaction history queries (txID, timestamp, status...)
* Integrate Fabric Event or chaincode for real-time event reception

### 3.6 Automatic Dividend Distribution

* Tokens related to certificates of deposit, rental real estate...
* Periodic dividend distribution through smart contract logic or scheduled jobs
* Store dividend transaction information and send notifications to owners
* Support automatic reinvestment (SIP)

### 3.7 Reporting and Data Integration

* Aggregate token ownership reports by time and asset type
* Support data export for regulatory bodies (CSV, API)
* Integrate DID for user identification and AuthZ for report viewing restrictions
* Track transactions on-chain or hash off-chain data for verification

---

## 4. Non-Functional Requirements

### 4.1 Security

* Authentication through JWT and middleware
* Transactions signed with cert from DID Service (MSP Identity)
* Detailed permission checking for each action through AuthZ Service
* Prevent attacks such as:
  * Reentrancy
  * Oracle Manipulation
  * Replay attack

### 4.2 Scalability

* Support multiple asset types and multiple owning organizations
* Microservice design for easy module separation by business function (mint, transfer...)
* Prepare for multi-chain deployment capability

### 4.3 Availability and Recovery

* Support HA (High Availability) and periodic backups
* Ability to recover tokens from last state snapshot
* 24/7 system operation with failover mechanism
* Support horizontal scaling

### 4.4 Integration Capability

* Integration with:
  * DID Service for user identification
  * AuthN/AuthZ for access control
  * Hyperledger Fabric Token SDK (via Gateway)
  * Chainlink oracle for NAV and price updates
  * IPFS or MinIO for metadata storage

---

## 5. Initial Implementation Priority (MVP)

| Function                          | Priority   |
| --------------------------------- | ---------- |
| Asset Token Creation (mint)       | High       |
| Token Transfer                    | High       |
| Token Burn                        | High       |
| Balance & Transaction History     | Medium     |
| DID and AuthZ Integration         | High       |
| SIP & Dividend Automation         | Medium     |
| Reporting System                  | Medium     |
| Cross-chain Support               | Low        |

---

## 6. Technical Implementation Notes

* Chaincode uses Fabric Token SDK in external mode
* DID Service can be written in Go or NestJS depending on environment
* Transactions should be executed through gRPC or HTTP API Gateway
* Database stores tokenized asset metadata (PostgreSQL or MongoDB depending on scale)
* IPFS or MinIO can be used for certificate/metadata file storage if needed
* Support for future multi-chain deployment
* Integration with Chainlink for price feeds and NAV updates

*Last Updated: 31/05/2025* 

graph LR
    subgraph "Client Layer"
        Web[Web App]
        Mobile[Mobile App]
        API[API Client]
    end

    subgraph "Middleware Layer"
        Gateway[API Gateway]
        AuthN[AuthN Service]
        AuthZ[AuthZ Service]
        DID[DID Service]
    end

    subgraph "Application Layer"
        Asset[Asset Service]
        Token[Token Service]
    end

    subgraph "Blockchain Layer"
        Fabric[Fabric Network]
    end

    Web --> Gateway
    Mobile --> Gateway
    API --> Gateway

    Gateway --> AuthN
    Gateway --> AuthZ
    Gateway --> Asset

    AuthN --> Asset
    AuthZ --> Asset
    DID --> Asset

    Asset --> Token
    Token --> Fabric

    %% Interface Labels
    Gateway -.->|"Client ↔ Gateway"| Asset
    Asset -.->|"Asset ↔ Token Service"| Token
    Token -.->|"Token ↔ Fabric"| Fabric
    DID -.->|"DID ↔ Asset Service"| Asset 