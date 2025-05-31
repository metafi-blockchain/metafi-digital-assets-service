Business Analysis - Digital Asset Service

1. Mục tiêu hệ thống

Hệ thống Digital Asset Service được xây dựng nhằm hỗ trợ việc số hóa các tài sản thực (Real World Assets - RWA) bằng cách sử dụng công nghệ blockchain Hyperledger Fabric. Mục tiêu là cung cấp một nền tảng tin cậy, minh bạch, bảo mật để phát hành, giao dịch và quản lý tài sản số dựa trên mô hình phân quyền.

2. Bối cảnh nghiệp vụ
	•	Tài sản truyền thống như bất động sản, chứng chỉ tiền gửi, quỹ đầu tư… khó giao dịch, thiếu thanh khoản.
	•	Người đầu tư nhỏ lẻ cần cơ hội tiếp cận và phân mảnh sở hữu tài sản lớn.
	•	Các tổ chức phát hành cần một nền tảng tin cậy để số hóa, theo dõi và quản lý tài sản.
	•	Cần tích hợp với các hệ thống định danh (DID), xác thực và phân quyền (AuthZ), cũng như hỗ trợ phân phối lợi tức tự động.

3. Đối tượng sử dụng

Vai trò	Nhiệm vụ
Nhà đầu tư cá nhân	Đầu tư, mua bán token đại diện cho tài sản số
Doanh nghiệp phát hành	Token hóa tài sản thật, quản lý phân phối, lợi tức
Quản trị hệ thống	Giám sát giao dịch, phân quyền, xử lý vi phạm
Cơ quan quản lý	Xem báo cáo, truy vết lịch sử token hóa tài sản

4. Chức năng chính

4.1 Token hóa tài sản số
	•	Phát hành token từ tài sản truyền thống
	•	Lưu metadata: thông tin pháp lý, định giá, tài liệu đính kèm
	•	Giao token đến người sở hữu ban đầu (investor hoặc issuer)

4.2 Giao dịch và chuyển nhượng
	•	Chuyển token từ người này sang người khác theo mô hình UTXO
	•	Giao dịch peer-to-peer hoặc theo chỉ định từ hệ thống

4.3 Phân phối lợi tức định kỳ
	•	Lãi suất, cổ tức, tiền thuê được phân phối đến người sở hữu
	•	Tự động hóa theo kỳ (tháng, quý)

4.4 Đăng ký đầu tư định kỳ (SIP)
	•	Người dùng có thể đăng ký đầu tư định kỳ vào tài sản số
	•	Hệ thống tự động xử lý và ghi nhận quyền sở hữu tương ứng

4.5 Kiểm soát và phân quyền
	•	Kiểm tra quyền thông qua AuthZ trước khi thực hiện giao dịch
	•	Chỉ người có vai trò phù hợp mới được tạo, chuyển, hủy token

4.6 Truy vấn, báo cáo, kiểm toán
	•	Tra cứu số dư, lịch sử giao dịch, trạng thái tài sản
	•	Xuất báo cáo chi tiết cho cơ quan quản lý hoặc nội bộ

5. Quy trình nghiệp vụ mẫu

Mint Token:
	1.	Nhà phát hành gửi yêu cầu mint token
	2.	Token Service gọi DID để xác định danh tính
	3.	Gọi AuthZ để kiểm tra quyền “token:mint”
	4.	Gọi Fabric Token SDK để phát hành token
	5.	Ghi metadata tài sản và phân phối token

Transfer Token:
	1.	Người dùng yêu cầu chuyển token
	2.	Token Service xác thực danh tính qua DID
	3.	Kiểm tra quyền chuyển token qua AuthZ
	4.	Gọi Fabric để thực hiện Transfer
	5.	Ghi nhận giao dịch và thông báo đến các bên

6. Tích hợp hệ sinh thái

Dịch vụ	Vai trò
DID Service	Xác định người dùng, ánh xạ MSP ID và chứng chỉ
AuthZ Service	RBAC - kiểm tra quyền người dùng theo vai trò
Fabric Network	Ghi nhận và xử lý giao dịch token hóa
IPFS / MinIO	Lưu metadata, file pháp lý, hình ảnh tài sản
AuthN Service	Xác thực JWT từ người dùng trước khi gọi API

7. Rủi ro và khuyến nghị

Rủi ro	Biện pháp giảm thiểu
Người dùng không xác thực đúng	Bắt buộc xác thực DID + JWT
Sai lệch khi phân phối token	Luôn ghi lại audit trail và dùng UTXO trace
Tấn công mạng, làm giả danh tính	Ký giao dịch bằng MSP cert từ Fabric
Sai lệch NAV hoặc dữ liệu tài sản	Đồng bộ dữ liệu từ Oracle hoặc nguồn pháp lý tin cậy

8. Kết luận

Digital Asset Service là nền tảng chủ chốt giúp hiện thực hóa việc quản lý và phân phối tài sản số trong môi trường doanh nghiệp hoặc tổ chức tài chính. Việc tích hợp với các module như DID, AuthZ, và sử dụng Fabric làm lớp cơ sở hạ tầng giúp hệ thống đảm bảo minh bạch, an toàn và mở rộng.

⸻

Cập nhật: 31/05/2025