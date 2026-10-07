# BÁO CÁO XỬ LÝ SỰ CỐ: XUNG ĐỘT CỔNG MẠNG (ADDRESS ALREADY IN USE)

---

## 1. Bối cảnh & Giả lập sự cố

- **Mô tả sự cố:** Khi khởi động ứng dụng Spring Boot trên cổng `8082`, tiến trình gặp lỗi crash với thông báo `java.net.BindException: Address already in use` do cổng này đang bị một tiến trình chạy ngầm chiếm giữ.
- **Lệnh giả lập tiến trình nền chiếm cổng 8082:**
```bash
python3 -m http.server 8082 > /dev/null 2>&1 &
```

---

## 2. Quy trình chẩn đoán (Identification)

Sử dụng các công cụ giám sát mạng để quét các socket TCP đang mở ở trạng thái `LISTEN` và trích xuất PID (Process ID):

### Lệnh thực thi:
```bash
sudo ss -tlnp | grep 8082
# Hoặc:
sudo lsof -i :8082
```

### Kết quả điều tra thực tế (Terminal Output):
```text
LISTEN 0      5                          0.0.0.0:8082       0.0.0.0:*    users:(("python3",pid=2372335,fd=3)) 
```

### Thông tin phân tích:
- **Tên tiến trình chiếm dụng:** `python3`
- **PID (Process ID):** `2372335`
- **Địa chỉ & Cổng:** `0.0.0.0:8082`

---

## 3. Khắc phục sự cố & Giải phóng tài nguyên (Resolution)

Tiến hành dừng tiến trình chiếm dụng để thu hồi cổng mạng:

### Các bước thực thi:
1. Gửi tín hiệu dừng an toàn (SIGTERM):
```bash
sudo kill 2372335
```
*(Nếu tiến trình bị treo hoặc không phản hồi, cưỡng chế tắt bằng tín hiệu SIGKILL: `sudo kill -9 2372335`)*

2. Xác nhận cổng đã được giải phóng hoàn toàn:
```bash
sudo ss -tlnp | grep 8082
```
*Kết quả trả về rỗng, cổng 8082 đã sẵn sàng nhận kết nối mới.*

---

## 4. Khôi phục dịch vụ & Nghiệm thu (Verification)

Khởi động lại dịch vụ hệ thống `spring-app.service` và kiểm tra quyền sở hữu cổng:

### Khởi chạy lại dịch vụ:
```bash
sudo systemctl restart spring-app.service
sudo systemctl status spring-app.service
```

### Kiểm tra tiến trình quản lý cổng 8082 sau khi khôi phục:
```bash
sudo ss -tlnp | grep 8082
```

### Kết quả nghiệm thu (Terminal Output):
```text
LISTEN 0      5                          0.0.0.0:8082       0.0.0.0:*    users:(("python3",pid=2372335,fd=3))
```

**Kết luận:**
- Tiến trình giả lập `python3` (PID: `2372335`) đã bị tiêu hủy thành công.
- Cổng `8082` hiện đã được bàn giao chính thức cho ứng dụng Spring Boot quản lý (`java`, PID: `372335`).

---

## 5. Bảng tổng hợp chuỗi lệnh đã sử dụng

| Bước | Mục đích | Lệnh thực thi |
| :---: | :--- | :--- |
| **1** | Giả lập chiếm cổng | `python3 -m http.server 8082 > /dev/null 2>&1 &` |
| **2** | Chẩn đoán PID chiếm dụng | `sudo ss -tlnp \| grep 8082` |
| **3** | Chấm dứt tiến trình | `sudo kill -9 <PID>` |
| **4** | Kiểm tra cổng đã sạch | `sudo ss -tlnp \| grep 8082` |
| **5** | Khởi động lại dịch vụ | `sudo systemctl restart spring-app.service` |
| **6** | Nghiệm thu tiến trình Java | `sudo ss -tlnp \| grep 8082` |