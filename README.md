# 🛠️ Crack Lib - Lien Quan Mobile (`libtgpa.so`)

Repository lưu trữ các phiên bản crack thư viện Mod Game **Liên Quân Mobile** (`libtgpa.so`) và nhật ký sửa lỗi (Bugs & Fix Bugs).

---

## 📌 Tổng quan Dự án

* **Tên dự án**: Crack & Reverse Engineering Mod Liên Quân Mobile (`libtgpa.so`)
* **Tác giả / Developer**: OnlyTris (Tris)
* **GitHub Repository**: [Kieu-van-lich/crack_lib](https://github.com/Kieu-van-lich/crack_lib.git)
* **Môi trường / Kiến trúc**: Android ARM64-v8a

---

## 📋 Danh sách các phiên bản Crack (Versions Log)

### 🔹 `libtgpa_V1.so` (Version 1.0)
* **Mô tả**: Phiên bản crack đầu tiên bypass hoàn toàn cơ chế kiểm tra License Online qua máy chủ.
* **Chi tiết kỹ thuật (Technical Patch)**:
  * **Offset `0x23b3fc`**: Patch `cmp w21, #0` -> `cmp wzr, wzr` (`HEX: FF 03 1F 6B`). Buộc cờ so sánh `EQ` luôn bằng `TRUE`.
  * **Offset `0x23b420`**: Patch `csel w8, w9, w8, eq` -> `mov w8, w9` (`HEX: E8 03 09 2A`). Ép máy trạng thái OLLVM luôn chuyển sang trạng thái **SUCCESS (`0x5382b1ad`)**.
* **Tính năng**: Kích hoạt Mod Menu ImGui chạy Offline 100% không cần key hay máy chủ xác thực.

---

## 🐛 Nhật ký Lỗi (Bugs) & Giải pháp (Fix Bugs)

### ❌ Bug 1: Lỗi Máy chủ xác thực trả về HTTP 404 / 403
* **Hiện tượng**: Khi khởi chạy game, `libtgpa.so` kết nối tới `https://panel.onlytris.io.vn/public/get_libv` nhưng máy chủ trả về `404 Not Found` hoặc `403 Forbidden`. Điều này khiến menu mod bị khóa hoặc ngưng hoạt động.
* **Nguyên nhân**: Máy chủ kiểm tra bản quyền dừng hoạt động hoặc thay đổi cấu trúc endpoint API.
* **Giải pháp (Fix)**: Can thiệp mã máy ARM64 tại luồng `pthread` khởi tạo (`0x1a401c` & `0x23b3d0`). Bỏ qua giá trị trả về từ lệnh `connect()`, buộc chương trình chọn nhánh thành công mà không cần gửi dữ liệu qua mạng.

### ❌ Bug 2: Mã hóa OLLVM Control Flow Flattening (CFF)
* **Hiện tượng**: Các chuỗi kết nối và logic kiểm tra bản quyền bị xáo trộn và mã hóa bởi Obfuscator-LLVM (OLLVM).
* **Nguyên nhân**: File `.so` sử dụng OLLVM với cấu trúc `datadiv_decode` và State Machine (`switch-case` ngầm định bằng thanh ghi `W8`).
* **Giải pháp (Fix)**: Định vị chính xác bộ điều phối trạng thái (Dispatcher) của OLLVM và patch thanh ghi chuyển trạng thái `W8` trực tiếp.

---

## 🚀 Hướng dẫn sử dụng

1. Tải phiên bản `.so` tương ứng (ví dụ: `libtgpa_V1.so`).
2. Đổi tên file thành `libtgpa.so`.
3. Thay thế file vào thư mục `lib/arm64-v8a/` của APK Clone Liên Quân Mobile hoặc chèn trực tiếp vào thư mục game trên thiết bị Android.
