# Báo Cáo Phân Tích Lỗi & Chi Tiết Các Bản Patch Thư Viện Crack Lib (Liên Quân Mobile 1.63.1.5)

Kho lưu trữ này chứa các phiên bản phân tích dịch ngược, can thiệp mã máy ARM64 và các bản vá lỗi (patches) cho thư viện mod `libtgpa.so` của Liên Quân Mobile v1.63.1.5.

---

## 📑 Danh Sách Các Phiên Bản Thư Viện (`crack_lib`)

| Tên Tệp | Kích Thước | Mô Tả Tóm Tắt |
| :--- | :--- | :--- |
| `libtgpa_V1.so` | 2,824,960 bytes | Thư viện gốc tải trực tiếp từ server mod |
| `libtgpa_V2.so` | 2,824,960 bytes | Thư viện nhúng thay thế trực tiếp `libLoader.so` |
| `libtgpa_V3.so` | 2,824,960 bytes | Patch 4 hàm xác thực online cơ bản (`isKeyValid`, `isKeyExpired`, ...) |
| `libtgpa_V4.so` | 2,824,960 bytes | Patch 7 vị trí (triệt tiêu phản hồi lỗi từ server HTTP) |
| `libtgpa_V5.so` | 2,824,960 bytes | Bản can thiệp ngắt sớm Auth Controller (Gây lỗi Crash do Stack) |
| `libtgpa_V6.so` | 2,824,960 bytes | Bản sửa crash (Gặp lỗi ẩn cả 2 Menu do cờ Auth State 0x1afd20) |
| `libtgpa_V7.so` | 2,824,960 bytes | Bản thử nghiệm (Khôi phục State Controller nhưng vẫn vướng cờ lỗi HTTP "KEY Không Tồn Tại") |
| `libtgpa_V8.so` | 2,824,960 bytes | **Bản Master hoàn chỉnh 100%** (Bypass toàn bộ bảng nhập Key + Triệt tiêu lỗi HTTP + Mở trực tiếp Menu Hack) |

---

## 🔍 Chi Tiết Các Lỗi Đã Gặp & Cách Khắc Phục

### 1. Lỗi Cài Đặt APK Trên Giả Lập / Điện Thoại (Install Verification Error)
- **Hiện tượng**: Giả lập Android / thiết bị báo lỗi *"Rất tiếc, không thể cài đặt ứng dụng này. Vui lòng xác minh tệp cài đặt trước khi thử lại"*.
- **Nguyên nhân**: File APK sau khi giải nén và nén lại chưa được căn chỉnh 4-byte (4-byte Zipalign) cho các thư viện `.so` và thiếu chữ ký Android Scheme v2/v3.
- **Cách khắc phục**:
  - Sử dụng công cụ `uber-apk-signer.jar` để thực hiện zipalign 4-byte chuẩn cho toàn bộ `.so`, `.arsc`, `.png`.
  - Ký đồng thời cả 3 chuẩn chữ ký: `v1`, `v2`, `v3`.

---

### 2. Lỗi Bảng Nhập Key Vẫn Xuất Hiện & Báo "KEY Không Tồn Tại" (Tris_V3 & Tris_V4)
- **Hiện tượng**: Dù đã patch các hàm `isKeyValid` và `isKeyExpired`, khi mở game vẫn xuất hiện cửa sổ `"ONLYTRIS LQM | XÁC MINH BẢN QUYỀN"`, nhập key bất kỳ báo dòng chữ đỏ `"KEY Không Tồn Tại"`.
- **Nguyên nhân**:
  - Khi người dùng bấm *"Kích hoạt thiết bị"*, một tiến trình HTTP gửi key lên server mod.
  - Server trả về JSON báo lỗi `{"status":"error", "message":"KEY Không Tồn Tại"}`.
  - Hàm **Master Auth Evaluator** (tại offset `0x1b9618`) và bộ đánh giá phản hồi server (tại `0x1b90f4`) ghi nhận cờ lỗi từ HTTP response và điều khiển **ImGui State Machine** (tại `0x1b8ac0`) nhảy vào nhánh hiển thị thông báo lỗi.
- **Cách khắc phục (Patch V4)**:
  - Patch `0x1b9618`: `mov w8, #0` (`08 00 80 52`) -> Ép cờ lỗi Master Auth luôn bằng 0.
  - Patch `0x1b90f4`: `mov w8, #0` (`08 00 80 52`) -> Triệt tiêu phản hồi lỗi từ server.
  - Patch `0x1b8ac0`: `csel w8, w9, w8, al` (`08 01 88 9a`) -> Ép bộ chuyển trạng thái ImGui luôn nhảy vào Menu Hack.

---

### 3. Lỗi Văng Game Ngay Khi Khởi Động (Crash on Startup - Tris_V5)
- **Hiện tượng**: Bản Tris_V5 khi mở lên bị crash / thoát ứng dụng ngay lập tức (`SIGSEGV`).
- **Nguyên nhân**:
  - Tại bản V5, hàm khởi tạo Auth Controller (`0x1afad4`) bị can thiệp lệnh `ret` ngay tại đầu hàm (`mov w0, #1 ; ret`).
  - Việc thoát hàm quá sớm làm bỏ qua quá trình thiết lập Stack Frame và không khởi tạo các biến con trỏ (`x23`, `x24`, `x28`). Khi hàm cha tiếp tục thực thi và truy xuất các con trỏ này, ứng dụng bị lỗi truy cập bộ nhớ (`Null Pointer / Memory Access Violation`) dẫn đến crash game.

---

### 4. Lỗi Ẩn Cả Menu Login Nhập Key & Menu Hack (Bản V6 Cũ)
- **Hiện tượng**: Ở bản V6 cũ, khi mở game thì bảng nhập Key và Menu Hack đều bị ẩn khỏi màn hình.
- **Nguyên nhân**:
  - Lệnh `csel w8, w20, w28, al` tại offset `0x1afd20` đã ép trạng thái nội bộ của Controller về nhánh ngắt UI rendering. Cờ hiển thị cửa sổ đồ họa ImGui bị đặt về `false / Hidden`, khiến bộ render OpenGL bỏ qua toàn bộ lệnh vẽ giao diện (`ImGui::Begin()`).
- **Cách khắc phục (Bản V7 Fixed)**:
  - **Khôi phục State Controller**: Bỏ patch `0x1afd20`, giữ nguyên cấu trúc khởi tạo Stack Frame và luồng hiển thị giao diện ImGui.
  - **Patch Socket Bypass & State Machine direct**:
    - Patch `0x23b3fc`: `cmp wzr, wzr` (`ff 03 1f 6b`) -> Ép cờ so sánh socket kết nối luôn đúng.
    - Patch `0x23b420`: `mov w8, w9` (`e8 03 09 2a`) -> Bắt buộc ImGui State Machine chuyển thẳng sang giao diện Menu Hack chính mà không hiển thị bảng đăng nhập key nữa.

---

## 🛠️ Tổng Hợp Mã Máy ARM64 Master (Bản V7 Fixed)

```asm
1. [0x1b1ae4 - isKeyValid]          : 20 00 80 52 c0 03 5f d6 (mov w0, #1 ; ret)
2. [0x1b7f50 - isKeyExpired]        : 00 00 80 52 c0 03 5f d6 (mov w0, #0 ; ret)
3. [0x1b82d4 - verifyKeyOnline]     : 20 00 80 52 c0 03 5f d6 (mov w0, #1 ; ret)
4. [0x1b8558 - checkLicenseKey]     : 20 00 80 52 c0 03 5f d6 (mov w0, #1 ; ret)
5. [0x23b3fc - Socket Flag Compare] : ff 03 1f 6b             (cmp wzr, wzr)
6. [0x23b420 - State Machine Direct]: e8 03 09 2a             (mov w8, w9)
```
