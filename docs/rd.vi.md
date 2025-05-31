# Yêu Cầu Chức Năng - Hệ Thống Quản Lý Tài Sản Số

## 1. Mục tiêu hệ thống

Xây dựng một hệ thống quản lý tài sản số tích hợp với blockchain Hyperledger Fabric, hỗ trợ việc token hóa tài sản truyền thống như bất động sản, chứng chỉ tiền gửi, và quỹ đầu tư. Hệ thống cần đảm bảo xác thực danh tính qua DID, kiểm soát truy cập qua AuthZ, và hỗ trợ giao dịch token thông qua Fabric Token SDK.

---

## 2. Kiến trúc hệ thống tổng thể

* **DID Service**: Cung cấp DID (Decentralized Identifiers) và ánh xạ người dùng đến danh tính trong hệ thống (MSP, cert).
* **AuthZ Service**: Quản lý quyền truy cập dựa trên vai trò người dùng (RBAC).
* **Token Service**: Thực hiện logic token (mint, transfer, burn, view) thông qua Fabric Token SDK.
* **Client App**: Giao diện frontend tương tác với người dùng.
* **Fabric Network**: Ghi nhận các giao dịch token hóa tài sản trên mạng blockchain riêng.

---

## 3. Yêu cầu chức năng

### 3.1 Token hóa tài sản số

* Hỗ trợ token hóa các tài sản vật lý và tài chính:
  * Bất động sản
  * Chứng chỉ tiền gửi (CDs)
  * Chứng chỉ quỹ đầu tư (IFCs)
  * Stablecoin (bảo chứng bởi tiền pháp định, hàng hóa, hoặc crypto)
* Mỗi token đại diện cho toàn bộ hoặc một phần quyền sở hữu tài sản
* Quy trình token hóa bao gồm:
  * Định danh và định giá tài sản
  * Lưu ký tài sản (nếu cần)
  * Thiết kế cấu trúc token (fungible / NFT / fractional NFT)
  * Phát hành token (mint) thông qua smart contract
  * Ghi nhận quyền sở hữu trên blockchain
  * Giao dịch và chuyển nhượng token qua các cơ chế on-chain
* Tích hợp metadata (IPFS hoặc lưu trữ ngoài) gắn kèm token
* Hỗ trợ whitelist/blacklist để đảm bảo chỉ người dùng đã KYC mới tương tác được với loại token pháp lý đặc thù

### 3.2 Tạo và phát hành Token

* Nhận thông tin tài sản từ người dùng/quản trị viên
* Gọi DID để ánh xạ MSP/certificate của chủ sở hữu
* Xác nhận quyền mint token qua AuthZ
* Gọi Fabric Token SDK để tạo token (UTXO based)
* Lưu metadata mô tả tài sản và token tương ứng

### 3.3 Chuyển nhượng & Giao dịch Token

* Cho phép người dùng chuyển token sang người khác
* Kiểm tra quyền qua AuthZ
* Gọi DID → lấy cert người gửi/người nhận
* Giao dịch sử dụng hàm `Transfer` của Token SDK
* Hỗ trợ giao dịch P2P và tích hợp marketplace
* Triển khai cơ chế thanh toán giao nhận nguyên tử (DvP)

### 3.4 Hủy Token

* Cho phép người sở hữu chủ động hủy bỏ token
* Kiểm tra quyền qua AuthZ
* Gọi DID → xác thực danh tính và cert
* Gọi `Burn` SDK để xóa token khỏi mạng

### 3.5 Truy vấn số dư và lịch sử giao dịch

* Cho phép người dùng xem số dư token của mình
* Cho phép truy vấn lịch sử giao dịch (txID, timestamp, status...)
* Tích hợp Fabric Event hoặc chaincode để nhận sự kiện realtime

### 3.6 Tự động phân phối lợi tức

* Token có liên quan đến chứng chỉ tiền gửi, bất động sản cho thuê...
* Định kỳ phân phối lợi tức qua logic hợp đồng thông minh hoặc job định kỳ
* Lưu thông tin giao dịch lợi tức và gửi thông báo đến người sở hữu
* Hỗ trợ tái đầu tư tự động (SIP)

### 3.7 Báo cáo và tích hợp dữ liệu

* Tổng hợp báo cáo sở hữu token theo thời gian, loại tài sản
* Hỗ trợ export dữ liệu cho cơ quan quản lý (CSV, API)
* Tích hợp DID để xác định người dùng và AuthZ để giới hạn quyền xem báo cáo
* Theo dõi giao dịch on-chain hoặc hash dữ liệu off-chain để kiểm chứng

---

## 4. Yêu cầu phi chức năng

### 4.1 Bảo mật

* Xác thực qua JWT và middleware
* Giao dịch được ký bằng cert từ DID Service (MSP Identity)
* Kiểm tra quyền chi tiết theo từng hành động qua AuthZ Service
* Ngăn chặn các tấn công như:
  * Reentrancy
  * Oracle Manipulation
  * Replay attack

### 4.2 Khả năng mở rộng

* Hỗ trợ nhiều loại tài sản và nhiều tổ chức sở hữu
* Thiết kế dạng microservice để dễ tách module theo nghiệp vụ (mint, transfer...)
* Chuẩn bị cho khả năng triển khai đa chuỗi (multi-chain)

### 4.3 Tính sẵn sàng và khôi phục

* Hỗ trợ HA (High Availability) và backup định kỳ
* Có khả năng khôi phục token theo snapshot trạng thái cuối cùng
* Hệ thống hoạt động 24/7 với cơ chế failover
* Hỗ trợ nhân bản ngang (horizontal scaling)

### 4.4 Khả năng tích hợp

* Tích hợp với:
  * DID Service để định danh người dùng
  * AuthN/AuthZ để kiểm tra quyền truy cập
  * Hyperledger Fabric Token SDK (qua Gateway)
  * Chainlink oracle để cập nhật NAV, giá
  * IPFS hoặc MinIO để lưu metadata

---

## 5. Ưu tiên triển khai giai đoạn đầu (MVP)

| Chức năng                         | Ưu tiên    |
| --------------------------------- | ---------- |
| Tạo Token tài sản (mint)          | Cao        |
| Chuyển Token (transfer)           | Cao        |
| Hủy Token (burn)                  | Cao        |
| Truy vấn số dư, lịch sử giao dịch | Trung bình |
| Tích hợp DID và AuthZ             | Cao        |
| SIP & tự động hóa lợi tức         | Trung bình |
| Hệ thống báo cáo                  | Trung bình |
| Hỗ trợ cross-chain                | Thấp       |

---

## 6. Ghi chú triển khai kỹ thuật

* Chaincode sử dụng Fabric Token SDK dạng external
* DID Service có thể được viết bằng Go hoặc NestJS tùy môi trường
* Giao dịch nên thực hiện qua gRPC hoặc HTTP API Gateway
* Cơ sở dữ liệu lưu thông tin metadata về tài sản token hóa (PostgreSQL hoặc MongoDB tùy quy mô)
* IPFS hoặc MinIO có thể dùng để lưu trữ file chứng nhận/metadata nếu cần
* Hỗ trợ triển khai đa chuỗi trong tương lai
* Tích hợp Chainlink để cập nhật giá và NAV

*Cập nhật: 31/05/2025* 