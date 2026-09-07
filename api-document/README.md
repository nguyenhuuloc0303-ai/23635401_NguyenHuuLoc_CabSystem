# HƯỚNG DẪN SỬ DỤNG TÀI LIỆU API (API DOCUMENTATION) - CAB SYSTEM

> **Học viên**: Nguyễn Hữu Lộc  
> **MSSV**: 23635401  
> **Đề tài**: Nền tảng Đặt xe Trực tuyến & Điều phối Thông minh (CAB System)  
> **Chuẩn tài liệu**: OpenAPI 3.0.3 (Swagger Spec) & Postman Collection  

---

## 1. Cấu trúc Thư mục `api-document`

Thư mục này được chia thành các file đặc tả độc lập theo đúng kiến trúc **5 Microservices** của đồ án, kèm theo 1 file tổng hợp Master:

| Tên File | Phân hệ Microservice | Các Use Cases & Chức năng bao phủ |
| :--- | :--- | :--- |
| [`openapi.yaml`](file:///t:/New%20folder/23635401_NguyenHuuLoc_CabSystem/api-document/openapi.yaml) | **Master Spec (Toàn hệ thống)** | Tổng hợp toàn bộ 5 phân hệ, 17 Use Cases và 32 FRs để dễ dàng import 1-Click vào Swagger/Postman |
| [`auth-service.yaml`](file:///t:/New%20folder/23635401_NguyenHuuLoc_CabSystem/api-document/auth-service.yaml) | **Auth & User Service** | `UC01` (Đăng ký, SMS OTP), `UC02` (Đăng nhập JWT, Quên mật khẩu), `UC03` (Hồ sơ cá nhân), `UC10` (Đăng ký phương tiện tài xế) |
| [`ride-service.yaml`](file:///t:/New%20folder/23635401_NguyenHuuLoc_CabSystem/api-document/ride-service.yaml) | **Ride & Dispatch Service** | `UC04` (Tính cước ước tính & Đặt xe), `UC05` (Hủy chuyến & Phạt 10k), `UC06` (Tracking GPS), `UC09` (Lịch sử), `UC11` (Bật/Tắt Online), `UC12` (Nhận cuốc 20s), `UC13` (Từ chối cuốc), `UC14` (Tiến trình cuốc xe) |
| [`payment-service.yaml`](file:///t:/New%20folder/23635401_NguyenHuuLoc_CabSystem/api-document/payment-service.yaml) | **Fare & Payment Service** | `UC07` (Chốt cước thực tế, Thanh toán tiền mặt/thẻ số Tokenization, Fallback sang tiền mặt khi lỗi cổng ngoài, Hóa đơn điện tử) |
| [`notification-service.yaml`](file:///t:/New%20folder/23635401_NguyenHuuLoc_CabSystem/api-document/notification-service.yaml) | **Notification Service** | `FR23` (SMS OTP), `FR24` (Push Notifications thời gian thực khi tìm thấy xe, tài xế đến nơi, cuốc hoàn tất) |
| [`admin-service.yaml`](file:///t:/New%20folder/23635401_NguyenHuuLoc_CabSystem/api-document/admin-service.yaml) | **Admin & Analytics Service** | `UC08` (Đánh giá tài xế), `UC15` (Khóa/Mở tài khoản, duyệt xe), `UC16` (Live Monitor bản đồ, can thiệp sự cố cưỡng chế), `UC17` (Báo cáo KPI doanh thu, Audit Logs bắt buộc) |

---

## 2. Hướng dẫn Xem và Chạy thử với Swagger Editor

Theo bài hướng dẫn trên **200Lab Blog**:

### Cách 1: Sử dụng Swagger Editor Online (Nhanh nhất)
1. Mở trình duyệt và truy cập: [https://editor.swagger.io](https://editor.swagger.io)
2. Trên thanh menu, chọn **File** $\rightarrow$ **Import File**.
3. Chọn file [`openapi.yaml`](file:///t:/New%20folder/23635401_NguyenHuuLoc_CabSystem/api-document/openapi.yaml) (hoặc bất kỳ file service riêng nào như `ride-service.yaml`, `auth-service.yaml`...).
4. Giao diện Swagger UI tương tác trực quan sẽ hiện ra ở khung bên phải với đầy đủ tài liệu, schemas, mã phản hồi và nút **Try it out** để gọi thử API.

### Cách 2: Chạy Swagger Editor qua Docker cục bộ
Mở Terminal hoặc Command Prompt và chạy lệnh:
```bash
docker pull swaggerapi/swagger-editor
docker run -d -p 8080:8080 swaggerapi/swagger-editor
```
Sau đó mở trình duyệt tại: `http://localhost:8080`.

---

## 3. Hướng dẫn Import vào Postman

1. Mở ứng dụng **Postman** trên máy tính.
2. Chọn nút **Import** (góc trên bên trái màn hình).
3. Kéo thả file [`openapi.yaml`](file:///t:/New%20folder/23635401_NguyenHuuLoc_CabSystem/api-document/openapi.yaml) hoặc nhấn **Select files**.
4. Khi Postman hiển thị hộp thoại tùy chọn, chọn định dạng: **OpenAPI 3.0 with a Postman Collection**.
5. Nhấn **Import**.
6. Postman sẽ tự động sinh ra toàn bộ danh mục Requests phân nhóm theo 5 Microservices, bao gồm sẵn:
   - Header mẫu (`Authorization: Bearer <token>`, `Content-Type: application/json`)
   - URL Parameters & Query Parameters
   - Body mẫu JSON cho từng Request và các kịch bản phản hồi mẫu (Examples).

---

## 4. Các Tham số & Quy tắc Nghiệp vụ cốt lõi trong API
- **Biểu phí cước (ASM-01)**:
  - CabBike: 12.000đ (2km đầu) | 4.500đ/km tiếp | 300đ/phút
  - CabCar 4 chỗ: 20.000đ (2km đầu) | 10.000đ/km tiếp | 600đ/phút
  - CabCar 7 chỗ: 25.000đ (2km đầu) | 12.500đ/km tiếp | 800đ/phút
- **Bán kính quét tìm xe (ASM-02)**: Quét mặc định 3 km qua Redis Geospatial; tự động mở rộng lên 5 km.
- **Timeout nhận chuyến (ASM-03)**: Bộ đếm 20 giây cho tài xế.
- **Hủy chuyến & Phạt hủy (ASM-04)**: Miễn phí khi đang tìm xe hoặc trong vòng 2 phút sau khi nhận; phạt 10.000 VNĐ nếu hủy sau 2 phút.
- **Fallback thanh toán (RULE-06 / EX05)**: Tự động chuyển sang thu Tiền mặt nếu thanh toán thẻ thất bại hoặc cổng thanh toán sập quá 30 giây.
- **Lưu vết kiểm toán (RULE-07)**: Bắt buộc nhập lý do khi quản trị viên can thiệp cuốc xe/khóa tài khoản; lưu vào bảng `audit_logs` ở chế độ chỉ đọc.
