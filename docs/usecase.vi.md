# Use Case Chi Tiết - Hệ Thống Quản Lý Tài Sản Số

## Use Case 1: Token hóa tài sản

- **Tên:** Token hóa tài sản
- **Tác nhân:** Chủ sở hữu tài sản, Quản trị viên
- **Mục tiêu:** Chuyển đổi tài sản vật lý/thực thành token số trên blockchain
- **Tiền điều kiện:** Chủ sở hữu đã xác thực KYC, tài sản đã được phê duyệt
- **Luồng chính:**
  1. Chủ sở hữu gửi yêu cầu token hóa (cung cấp metadata, giấy tờ…)
  2. Hệ thống xác thực thông tin, kiểm tra KYC
  3. Quản trị viên phê duyệt tài sản
  4. Hệ thống gọi Token Service để mint token
  5. Token được ghi nhận trên blockchain, trả về token_id cho chủ sở hữu
- **Luồng ngoại lệ:**
  - Thông tin không hợp lệ → từ chối, yêu cầu bổ sung
  - KYC không đạt → dừng quy trình, thông báo
  - Mint token thất bại → rollback, thông báo lỗi
- **Kết quả:** Tài sản được token hóa thành công, chủ sở hữu nhận token_id
- **Ghi chú:** Lưu log toàn bộ quá trình, gửi thông báo cho các bên liên quan

---

## Use Case 2: Chuyển nhượng token

- **Tên:** Chuyển nhượng token
- **Tác nhân:** Chủ sở hữu token, Người nhận
- **Mục tiêu:** Chuyển một phần hoặc toàn bộ token cho người khác
- **Tiền điều kiện:** Chủ sở hữu có đủ số dư, người nhận đã xác thực danh tính
- **Luồng chính:**
  1. Chủ sở hữu tạo yêu cầu chuyển nhượng (nhập số lượng, DID người nhận)
  2. Hệ thống xác thực quyền, kiểm tra số dư
  3. Hệ thống gọi Token Service thực hiện giao dịch
  4. Blockchain xác nhận giao dịch
  5. Hệ thống cập nhật số dư, lịch sử giao dịch cho cả hai bên
- **Luồng ngoại lệ:**
  - Số dư không đủ → báo lỗi
  - Người nhận chưa xác thực → báo lỗi
  - Giao dịch blockchain thất bại → rollback, thông báo lỗi
- **Kết quả:** Token được chuyển thành công, cập nhật lịch sử giao dịch
- **Ghi chú:** Gửi thông báo cho cả hai bên, lưu log audit

---

## Use Case 3: Hủy token (Burn)

- **Tên:** Hủy token (Burn)
- **Tác nhân:** Chủ sở hữu token
- **Mục tiêu:** Hủy bỏ token không còn giá trị sử dụng
- **Tiền điều kiện:** Chủ sở hữu xác thực danh tính, token còn hiệu lực
- **Luồng chính:**
  1. Chủ sở hữu gửi yêu cầu hủy token
  2. Hệ thống xác thực quyền sở hữu và trạng thái token
  3. Hệ thống gọi Token Service thực hiện burn
  4. Blockchain xác nhận giao dịch burn
  5. Hệ thống cập nhật trạng thái, lịch sử token
- **Luồng ngoại lệ:**
  - Token không hợp lệ hoặc đã bị khóa → báo lỗi
  - Giao dịch burn thất bại → rollback, thông báo lỗi
- **Kết quả:** Token bị hủy thành công, cập nhật lịch sử
- **Ghi chú:** Lưu log, gửi thông báo cho chủ sở hữu

---

## Use Case 4: Phân phối lợi tức định kỳ

- **Tên:** Phân phối lợi tức định kỳ
- **Tác nhân:** Hệ thống, Chủ sở hữu token
- **Mục tiêu:** Tự động phân phối lợi tức cho các chủ sở hữu token theo tỷ lệ sở hữu
- **Tiền điều kiện:** Có lợi tức phát sinh, thông tin chủ sở hữu cập nhật mới nhất
- **Luồng chính:**
  1. Hệ thống tính toán tổng lợi tức cần phân phối
  2. Xác định danh sách chủ sở hữu và tỷ lệ sở hữu
  3. Thực hiện phân phối, ghi nhận giao dịch
  4. Gửi thông báo và cập nhật lịch sử lợi tức
- **Luồng ngoại lệ:**
  - Lỗi tính toán → dừng phân phối, thông báo admin
  - Lỗi cập nhật số dư → rollback, thông báo lỗi
- **Kết quả:** Lợi tức được phân phối thành công, chủ sở hữu nhận thông báo
- **Ghi chú:** Lưu log, hỗ trợ truy xuất lịch sử

---

## Use Case 5: Đăng ký đầu tư định kỳ (SIP)

- **Tên:** Đăng ký đầu tư định kỳ (SIP)
- **Tác nhân:** Nhà đầu tư
- **Mục tiêu:** Đăng ký và thực hiện đầu tư định kỳ vào tài sản số
- **Tiền điều kiện:** Nhà đầu tư đã xác thực danh tính, có đủ số dư
- **Luồng chính:**
  1. Nhà đầu tư đăng ký SIP (chọn tài sản, số tiền, chu kỳ)
  2. Hệ thống xác thực thông tin, kiểm tra số dư
  3. Đến kỳ, hệ thống tự động thực hiện giao dịch đầu tư
  4. Cập nhật số dư, lịch sử đầu tư
  5. Gửi thông báo cho nhà đầu tư
- **Luồng ngoại lệ:**
  - Số dư không đủ tại thời điểm đầu tư → thông báo lỗi, dừng SIP
  - Lỗi giao dịch → rollback, thông báo lỗi
- **Kết quả:** Đầu tư định kỳ được thực hiện thành công, cập nhật lịch sử
- **Ghi chú:** Cho phép hủy hoặc thay đổi SIP, lưu log toàn bộ quá trình

---

## Use Case 6: Truy vấn số dư và lịch sử giao dịch

- **Tên:** Truy vấn số dư và lịch sử giao dịch
- **Tác nhân:** Chủ sở hữu token, Quản trị viên
- **Mục tiêu:** Xem số dư token và lịch sử giao dịch
- **Tiền điều kiện:** Đã xác thực danh tính
- **Luồng chính:**
  1. Người dùng gửi yêu cầu truy vấn số dư/lịch sử
  2. Hệ thống xác thực quyền truy cập
  3. Hệ thống truy vấn dữ liệu và trả về kết quả
- **Luồng ngoại lệ:**
  - Không có quyền truy cập → báo lỗi
  - Lỗi truy vấn dữ liệu → thông báo lỗi
- **Kết quả:** Người dùng nhận được thông tin số dư/lịch sử giao dịch
- **Ghi chú:** Hỗ trợ lọc, phân trang, xuất báo cáo

---

## Use Case 7: Quản lý phân quyền và KYC

- **Tên:** Quản lý phân quyền và KYC
- **Tác nhân:** Người dùng, Quản trị viên
- **Mục tiêu:** Đảm bảo chỉ người dùng hợp lệ, đủ điều kiện mới được thực hiện các nghiệp vụ
- **Tiền điều kiện:** Người dùng đã đăng ký tài khoản
- **Luồng chính:**
  1. Người dùng đăng ký, cung cấp thông tin KYC
  2. Hệ thống xác thực thông tin, kiểm tra KYC
  3. Quản trị viên phê duyệt hoặc từ chối
  4. Hệ thống cập nhật trạng thái tài khoản
- **Luồng ngoại lệ:**
  - Thông tin KYC không hợp lệ → yêu cầu bổ sung
  - Từ chối KYC → khóa tài khoản, thông báo
- **Kết quả:** Người dùng được phân quyền phù hợp, đủ điều kiện sử dụng hệ thống
- **Ghi chú:** Lưu log toàn bộ quá trình, hỗ trợ audit 