# Crack Library Collection - Liên Quân Mobile (LQM)

Tập hợp các phiên bản thư viện `libtgpa.so` và phân tích chi tiết lỗi, nguyên nhân, cách khắc phục qua từng phiên bản crack.

---

## 📌 Bảng Tổng Hợp Các Phiên Bản Crack

| Phiên bản | Tên File | Trạng Thái | Mô Tả & Bản Chất Kỹ Thuật |
| :--- | :--- | :--- | :--- |
| **V1** | `libtgpa_V1.so` | ❌ Crash | Patch thử nghiệm hàm kiểm tra cơ bản. |
| **V2** | `libtgpa_V2.so` | ❌ Sai kiến trúc | Thư viện 32-bit (armeabi-v7a) không tương thích trên nền tảng 64-bit (arm64-v8a). |
| **V3** | `libtgpa_V3.so` | ❌ Crash khởi động | Thiếu patch con trỏ xác thực online tại `verifyKeyOnline`. |
| **V4** | `libtgpa_V4.so` | ❌ Lỗi chữ ký APK | APK chưa được căn chỉnh 4-byte zipalign và thiếu chữ ký V2/V3 Scheme. |
| **V5** | `libtgpa_V5.so` | ❌ Crash (SIGSEGV) | Đặt lệnh `ret` trực tiếp tại `0x1afad4` làm hỏng con trỏ ngăn xếp Stack Frame (`x23`, `x24`, `x28`). |
| **V6** | `libtgpa_V6.so` | ❌ Ẩn giao diện ImGui | Lệnh `0x1afd20 (csel w8, w20, w28, al)` ép nhầm cờ trạng thái Controller sang nhánh ẩn UI rendering. |
| **V7** | `libtgpa_V7.so` | ❌ Chưa kích hoạt Key | Khôi phục rendering ImGui thành công nhưng chưa can thiệp tầng Button Action Handlers. |
| **V8** | `libtgpa_V8.so` | ❌ Báo Key không tồn tại | Xử lý các cờ HTTP và Socket nhưng chưa ép rẽ nhánh tại bộ chọn hành động nút bấm. |
| **V9** | `libtgpa_V9.so` | ❌ Ẩn UI | Ép `ret` tại `0x1ba3a0` và `0x1a48d4` làm ngắt chu trình khởi tạo ngữ cảnh ImGui Context. |
| **V10** | `libtgpa_V10.so` | 🏆 MASTER FIXED | Giữ nguyên ngữ cảnh ImGui, ép chuyển trạng thái thành công tuyệt đối tại bộ điều hướng `0x1b166c` & `0x1b55c8`. |

---

## 🛠️ Tổng Hợp Mã Máy ARM64 Master (Bản V10 Fixed)

```asm
1. [0x1b1ae4 - isKeyValid]          : 20 00 80 52 c0 03 5f d6 (mov w0, #1 ; ret)
2. [0x1b7f50 - isKeyExpired]        : 00 00 80 52 c0 03 5f d6 (mov w0, #0 ; ret)
3. [0x1b82d4 - verifyKeyOnline]     : 20 00 80 52 c0 03 5f d6 (mov w0, #1 ; ret)
4. [0x1b8558 - checkLicenseKey]     : 20 00 80 52 c0 03 5f d6 (mov w0, #1 ; ret)
5. [0x1b9618 - Master Auth Eval]    : 08 00 80 52             (mov w8, #0)
6. [0x1b90f4 - Server Response Eval]: 08 00 80 52             (mov w8, #0)
7. [0x1b166c - Button Action State] : 08 03 96 9a             (csel w8, w28, w22, al)
8. [0x1b55c8 - Key Action State]    : 08 02 97 9a             (csel w8, w19, w23, al)
```
