# 📋 YÊU CẦU DỰ ÁN DNU SHOP

## 🎯 1. Tổng quan

### 1.1. Mục đích dự án

**DNU Shop** là hệ thống thương mại điện tử (E-commerce) được xây dựng để:
- Phục vụ nhu cầu mua sắm trực tuyến của khách hàng
- Quản lý kho hàng và đơn hàng cho admin
- Thống kê doanh thu và bán hàng
- Học tập và thực hành kỹ năng Full-stack Development

### 1.4. Mô tả bài toán chi tiết (Scenario)

*Dưới đây là mô tả chi tiết về hoạt động của hệ thống, được dùng làm đầu vào duy nhất cho quá trình phân tích.*

**"Hệ thống DNU Shop là một nền tảng thương mại điện tử chuyên cung cấp các thiết bị công nghệ. Hệ thống có 2 nhóm người dùng chính: Khách hàng (Customer) và Quản trị viên (Admin). Tất cả người dùng đều có thông tin cơ bản gồm Họ tên, Email (duy nhất) và Mật khẩu (đã được mã hóa).**

**Đối với SẢN PHẨM (Product): Mỗi sản phẩm được bày bán đều thuộc về một Danh mục (Category) cụ thể (ví dụ: Điện thoại, Laptop). Sản phẩm cần lưu trữ các thông tin chi tiết gồm: Tên sản phẩm, Giá bán hiện tại, Hình ảnh minh họa, Mô tả kỹ thuật và Số lượng tồn kho. Một danh mục có thể chứa nhiều sản phẩm, nhưng mỗi sản phẩm chỉ thuộc một danh mục.**

**Quy trình MUA HÀNG (Ordering): Khách hàng sau khi đăng nhập có thể chọn mua nhiều sản phẩm. Khi khách hàng quyết định 'Đặt hàng', hệ thống sẽ tạo ra một Đơn hàng (Order). Một Đơn hàng phải chứa thông tin về: Ngày đặt hàng, Tổng tiền thanh toán, Tên người nhận, SĐT người nhận và Địa chỉ giao hàng. Đặc biệt, một đơn hàng bao gồm nhiều dòng sản phẩm, mỗi dòng được gọi là Chi tiết đơn hàng (OrderDetail). Mỗi Chi tiết đơn hàng ghi nhận lại Sản phẩm nào được mua, Số lượng bao nhiêu và Giá bán tại thời điểm mua (để lưu vết lịch sử giá nếu giá gốc thay đổi).**

**Quy trình QUẢN LÝ (Management): Quản trị viên cần theo dõi trạng thái của Đơn hàng. Một đơn hàng sẽ có các trạng thái chuyển đổi tuần tự: Mới (New) -> Đang giao (Shipping) -> Hoàn thành (Completed) hoặc bị Hủy (Cancelled). Quản trị viên cũng quản lý kho hàng bằng cách cập nhật Số lượng tồn kho của sản phẩm."**

### 1.2. Phạm vi dự án

**Trong phạm vi:**
- ✅ Quản lý sản phẩm (CRUD)
- ✅ Quản lý đơn hàng
- ✅ Quản lý người dùng và phân quyền
- ✅ Giỏ hàng và checkout
- ✅ Dashboard thống kê
- ✅ Upload ảnh sản phẩm

**Ngoài phạm vi (có thể mở rộng sau):**
- ❌ Thanh toán trực tuyến (Payment Gateway)
- ❌ Gửi email xác nhận
- ❌ Đánh giá sản phẩm
- ❌ Mã giảm giá (Coupon)
- ❌ Quản lý kho hàng nâng cao (Inventory Management)

### 1.3. Đối tượng sử dụng

**1. Khách hàng (Customer/User)**
- Xem danh sách sản phẩm
- Tìm kiếm và lọc sản phẩm
- Xem chi tiết sản phẩm
- Thêm vào giỏ hàng
- Đặt hàng

**2. Quản trị viên (Admin)**
- Đăng nhập vào hệ thống
- Quản lý sản phẩm (thêm, sửa, xóa)
- Quản lý đơn hàng (xem, cập nhật trạng thái)
- Xem dashboard thống kê
- Upload ảnh sản phẩm

## 2. Phân hệ Khách hàng (Storefront)
- **Xem danh sách sản phẩm**: Hiển thị dạng lưới, có phân trang (10 SP/trang).
- **Tìm kiếm**: Theo tên sản phẩm.
- **Lọc**: Theo danh mục, theo khoảng giá.
- **Chi tiết sản phẩm**: Xem ảnh lớn, mô tả, giá, số lượng tồn kho.
- **Giỏ hàng**: Thêm/Sửa/Xóa sản phẩm, tính tổng tiền tự động.
- **Đặt hàng**: Nhập thông tin giao hàng (Tên, SĐT, Địa chỉ) -> Lưu đơn hàng.

## 3. Phân hệ Quản trị (Admin Portal)
- **Đăng nhập**: Chỉ Admin mới được vào.
- **Dashboard**:
    - Tổng doanh thu tháng này.
    - Số đơn hàng mới chưa duyệt.
    - Top 5 sản phẩm bán chạy.
- **Quản lý Sản phẩm**:
    - Xem danh sách (Table).
    - Thêm mới (Upload ảnh).
    - Sửa thông tin.
    - Xóa (Soft delete - chỉ ẩn đi).
- **Quản lý Đơn hàng**:
    - Xem danh sách đơn hàng.
    - Cập nhật trạng thái: Mới -> Đang giao -> Hoàn thành / Hủy.

## 4. Yêu cầu Phi chức năng
- **Giao diện**: Responsive (dùng tốt trên điện thoại).
- **Bảo mật**: Password phải được mã hóa (Hash). API phải có Token.
- **Hiệu năng**: Tải trang dưới 2 giây.
