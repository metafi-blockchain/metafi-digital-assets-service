# Yêu Cầu Chức Năng - Hệ Thống Quản Lý Tài Sản Số (Digital Asset Service)

## 1. Yêu cầu chức năng (Functional Requirements)

### 1.1 Token hóa tài sản số (Digital Assets Tokenization)

* Hỗ trợ token hóa các tài sản vật lý và tài chính:

  * Bất động sản
  * Chứng chỉ tiền gửi (CDs)
  * Chứng chỉ quỹ đầu tư (CCQs)
  * Stablecoin (bảo chứng bởi tiền pháp định, hàng hóa, hoặc crypto)
* Mỗi token đại diện cho toàn bộ hoặc một phần quyền sở hữu tài sản.
* Quy trình token hóa gồm:

  * Định danh và định giá tài sản
  * Lưu ký tài sản (nếu cần)
  * Thiết kế cấu trúc token (fungible / NFT / fractional NFT)
  * Phát hành token (mint) thông qua smart contract
  * Ghi nhận quyền sở hữu trên blockchain
  * Giao dịch và chuyển nhượng token qua các cơ chế on-chain
* Tích hợp metadata (IPFS hoặc lưu trữ ngoài) gắn kèm token
* Hỗ trợ whitelist/blacklist để đảm bảo chỉ người dùng đã KYC mới tương tác được với loại token pháp lý đặc thù như CCQ, BĐS

### 1.2 Quản lý quyền sở hữu

* Theo dõi quyền sở hữu token trên blockchain.
* Hỗ trợ mô hình sở hữu phân mảnh (fractional ownership).
* Kiểm tra quyền sở hữu trước khi chuyển nhượng.

### 1.3 Giao dịch & chuyển nhượng tài sản

* Hỗ trợ mua, bán, chuyển token qua hợp đồng thông minh.
* Triển khai cơ chế thanh toán giao nhận nguyên tử (Delivery vs Payment - DvP).
* Cho phép giao dịch token P2P hoặc trên marketplace.

### 1.4 Phân phối lợi tức

* Hợp đồng thông minh tự động phân phối:

  * Lãi từ tiền gửi
  * Cổ tức từ quỹ
  * Thu nhập cho thuê từ bất động sản
* Thực hiện định kỳ hàng tháng hoặc hàng quý.

### 1.5 Tuân thủ pháp lý

* Tích hợp KYC/AML cho tất cả người dùng.
* Chỉ cho phép ví đã KYC tương tác với hệ thống.
* Áp dụng whitelist/blacklist ở cấp hợp đồng thông minh.

### 1.6 Tự động hóa qua smart contract

* Tự động hóa các nghiệp vụ:

  * Tạo / hủy / chuyển token
  * Quản trị biểu quyết (governance)
  * Tái đầu tư lợi tức định kỳ (SIP)

### 1.7 Báo cáo & kiểm toán

* Cung cấp API để:

  * Truy xuất quyền sở hữu và lịch sử giao dịch
  * Truy vấn NAV và hiệu suất đầu tư
* Lưu vết giao dịch on-chain hoặc hash dữ liệu off-chain để kiểm chứng.

## 2. Yêu cầu phi chức năng (Non-Functional Requirements)

### 2.1 Bảo mật

* Xác thực dựa trên JWT và DID.
* Ký tất cả giao dịch với danh tính MSP của Hyperledger Fabric.
* Ngăn chặn các tấn công như:

  * Reentrancy
  * Oracle Manipulation
  * Replay attack

### 2.2 Khả năng mở rộng

* Hỗ trợ đồng thời nhiều loại tài sản.
* Thiết kế theo hướng microservice để dễ mở rộng.
* Chuẩn bị cho khả năng triển khai đa chuỗi (multi-chain).

### 2.3 Khả năng tích hợp

* Tích hợp với:

  * DID Service để định danh người dùng
  * AuthN/AuthZ để kiểm tra quyền truy cập
  * Hyperledger Fabric Token SDK (qua Gateway)
  * Chainlink oracle để cập nhật NAV, giá
  * IPFS hoặc MinIO để lưu metadata

### 2.4 Tuân thủ & Giám sát

* Ghi log các hành vi nhạy cảm và cảnh báo bất thường.
* Hỗ trợ xuất báo cáo định kỳ theo định dạng cơ quan quản lý.
* Cho phép truy xuất kiểm toán theo yêu cầu.

### 2.5 Tính khả dụng & độ tin cậy

* Hệ thống hoạt động 24/7 với cơ chế failover.
* Hỗ trợ nhân bản ngang (horizontal scaling).
* Sao lưu định kỳ và snapshot trạng thái token.

## 3. Phạm vi triển khai theo giai đoạn (MVP)

| Tính năng                  | Ưu tiên    |
| -------------------------- | ---------- |
| Tạo / Chuyển / Hủy token   | Cao        |
| Token hóa tài sản số       | Cao        |
| Quản lý quyền sở hữu token | Cao        |
| Phân phối lợi tức          | Trung bình |
| Giao dịch định kỳ (SIP)    | Thấp       |
| Cập nhật NAV qua oracle    | Thấp       |
| Hỗ trợ cross-chain         | Thấp       |

