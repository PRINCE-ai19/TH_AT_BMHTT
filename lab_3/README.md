# BÁO CÁO THỰC HÀNH AN TOÀN VÀ BẢO MẬT THÔNG TIN — LAB 3: MÃ HÓA KHÓA CÔNG KHAI (RSA – ECC – HỆ MÃ LAI RSA + AES – CHỮ KÝ SỐ ECDSA – SQL SERVER)

## 1. Thông tin sinh viên
- **Họ và tên:** Phạm Đỗ Hoàng Luân
- **Mã số sinh viên (MSSV):** 1150070026
- **Lớp:** 11_ĐH_TMĐT
- **Link YouTube:https://www.youtube.com/watch?v=Ih1ngxHfY_8
- **Học phần:** An toàn và Bảo mật Thông tin (Thực hành)
- **Ngày thực hành:** 23/11/2026
- **Nền tảng & Công nghệ sử dụng:**
  - **Ngôn ngữ & SDK:** Java (JDK 17) – sử dụng thư viện mật mã chuẩn `java.security` và `javax.crypto`
  - **Môi trường phát triển (IDE):** Apache NetBeans IDE 31 (NetBeans GUI Builder – Java Swing `JFrame`)
  - **Quản lý dự án & Phụ thuộc:** Apache Maven (`pom.xml`: thư viện `com.microsoft.sqlserver:mssql-jdbc:12.8.1.jre11`)
  - **Hệ quản trị Cơ sở dữ liệu:** Microsoft SQL Server 15.0.2200.2 (`127.0.0.1`, TCP Port `14333`, CSDL `LAB3_MaHoaKhoaCongKhai`, tài khoản phân quyền tối thiểu `lab3_user`) & SQL Server Management Studio 20 (SSMS)

---

## 2. Danh sách tệp báo cáo và tài nguyên trong thư mục
- **Báo cáo Word hoàn chỉnh (đã điền đầy đủ thông tin, thuyết minh & bảng phân tích chi tiết 37 hình):**
- **Tài liệu hướng dẫn thực hành gốc:**
- **Cấu trúc project Maven `LAB3_MaHoaKhoaCongKhai` (package `week_03`):**
  - `pom.xml`: Cấu hình biên dịch Java 17 (`maven.compiler.release = 17`), lớp chính `week_03.frm_Menu` và thư viện JDBC `mssql-jdbc:12.8.1.jre11`.
  - **Các lớp tiện ích & truy cập CSDL dùng chung (Phần 1):**
    - `CryptoSupport.java`: Tiện ích `SecureRandom`, chuyển đổi `Base64` (`b64`/`unb64`), `UTF-8`, ghép/cắt mảng byte (`concat`/`slice`), mã hóa/giải mã xác thực `AES-256-GCM` (`gcmEncrypt`/`gcmDecrypt`), dẫn xuất khóa `HKDF-SHA256` (`hkdf`), và chuẩn hóa tệp khóa 2 dòng `PUBLIC=...` / `PRIVATE=...` (`toKeyFile`/`fromKeyFile`).
    - `FileTextIO.java`: Hộp thoại `JFileChooser` mở/lưu tệp văn bản `.txt` chuẩn `UTF-8`.
    - `DbConfig.java` & `HistoryDao.java`: Lưu thông tin kết nối SQL Server trong bộ nhớ RAM (mật khẩu dạng `char[]`, không hardcode), ghi và đọc lịch sử thao tác bảng `dbo.History` bằng `PreparedStatement` chống SQL Injection.
    - `frm_KetNoiSQL.java` & `frm_LichSu.java`: Biểu mẫu cấu hình/kiểm tra kết nối SQL Server và hiển thị 50 dòng nhật ký mới nhất từ `dbo.History`.
  - **Các cặp lớp Thuật toán & Giao diện (Bài 3.1 – 3.5):**
    1. `RSACipher.java` & `RSADemo.java`: Mật mã RSA (`RSA/ECB/OAEPWithSHA-256AndMGF1Padding`) chạy Console đo thời gian tạo khóa 2048/3072 bit.
    2. `frm_RSA.java`: Biểu mẫu Java Swing cho mật mã RSA (chọn kích thước khóa 2048/3072 bit, mã hóa/giải mã và ghi log SQL).
    3. `ECCCipher.java` & `frm_ECC.java`: Mật mã đường cong Elliptic (`secp256r1` / P-256) theo sơ đồ ECIES (`ECDH` + `HKDF-SHA256` + `AES-256-GCM`), hỗ trợ lưu/nạp tệp khóa (`ecc_keys.txt`).
    4. `RSA_AES_Cipher.java` & `frm_RSA_AES.java`: Hệ mã lai RSA-2048 + AES-256-GCM cho văn bản dài (> 1.000 ký tự), bổ sung nút **Sửa 1 ký tự bản mã** (`btn_tamper`) kiểm chứng toàn vẹn.
    5. `ECCSignature.java` & `frm_ECC_Signature.java`: Chữ ký số ECDSA (`SHA256withECDSA` trên `secp256r1`), ký bằng khóa riêng và xác minh bằng khóa công khai.
    6. `frm_Menu.java`: Cửa sổ Menu điều hướng chính của toàn bộ bài Lab 3.

---

## 3. Tóm tắt Lý thuyết & Kết quả Kiểm thử Thực tế

### 3.1. Bảng tổng hợp dữ liệu kiểm thử thực tế trên máy thực hành

| Bài | Thuật toán / Chức năng | Lớp xử lý & Form GUI | Dữ liệu đầu vào | Tham số / Khóa | Kết quả đầu ra / Quan sát |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1.4–1.5** | **SQL Server & Lịch sử** | `DbConfig`, `HistoryDao` / `frm_KetNoiSQL`, `frm_LichSu` | `127.0.0.1`, Port `14333`, DB `LAB3_MaHoaKhoaCongKhai` | User: `lab3_user` (chỉ có quyền `SELECT`, `INSERT` trên `dbo.History`) | `Kết nối thành công: Microsoft SQL Server 15.00.2200 - database LAB3_MaHoaKhoaCongKhai` |
| **3.1** | **RSA Console (OAEP)** | `RSACipher`, `RSADemo` | `HELLO RSA` | 2048 bit (`362 ms`) & 3072 bit (`860 ms`) | Bản mã Base64 dài `344` ký tự (2048 bit) và `512` ký tự (3072 bit); `Khoi phuc dung: true` |
| **3.2** | **RSA Swing GUI** | `RSACipher` / `frm_RSA` | `HELLO RSA` | RSA-2048 (tối đa 190 byte) | Bản mã 344 ký tự Base64 (`OsMubabG8Q...`); Giải mã khôi phục đúng `HELLO RSA` |
| **3.3** | **ECC (ECIES)** | `ECCCipher` / `frm_ECC` | `HELLO ECC` | Đường cong `secp256r1` (P-256), khóa Base64 | Mã hóa & giải mã chính xác `HELLO ECC`; lưu/nạp khóa qua `ecc_keys.txt` |
| **3.4** | **Hệ mã lai RSA + AES-GCM** | `RSA_AES_Cipher` / `frm_RSA_AES` | Văn bản tiếng Việt dài **1.000 ký tự** | RSA-2048 bọc khóa AES-256 (32 byte) + nonce 12 byte | Giải mã toàn văn 1.000 ký tự; Sửa 1 ký tự bản mã $\rightarrow$ `AEADBadTagException: Tag mismatch` |
| **3.5** | **Chữ ký số ECDSA** | `ECCSignature` / `frm_ECC_Signature` | `THONG DIEP A` & `THONG DIEP B` | `SHA256withECDSA` (`secp256r1`) | Tạo chữ ký Base64 96 ký tự (`MEQCICzEdV...`); sửa thông điệp cho ra chữ ký khác hoàn toàn |

### 3.2. Bảng đo đạc hiệu năng RSA-2048 và RSA-3072 (Bảng 3.7 – Bài 3.1)

| Kích thước khóa | Thời gian tạo khóa (ms) | Độ dài khóa công khai (Base64) | Độ dài bản mã (Base64) | Giới hạn bản rõ (OAEP-SHA256) |
| :--- | :--- | :--- | :--- | :--- |
| **2048 bit** | **362 ms** | 392 ký tự | 344 ký tự | 190 byte |
| **3072 bit** | **860 ms** *(chậm hơn ~2,38 lần)* | 564 ký tự | 512 ký tự | 318 byte |

### 3.3. Tóm tắt kiến trúc & cấu trúc bản mã của các hệ mật
1. **RSA-OAEP (`RSA/ECB/OAEPWithSHA-256AndMGF1Padding` – Bài 3.1 & 3.2):**
   - Mã hóa bằng **khóa công khai** $(n, e)$, giải mã bằng **khóa riêng** $(n, d)$.
   - Giới hạn độ dài bản rõ: $\text{bytes}(n) - 66$ (190 byte với 2048 bit; 318 byte với 3072 bit). Đệm OAEP thêm yếu tố ngẫu nhiên nên cùng một bản rõ mã hóa nhiều lần sẽ ra các bản mã khác nhau.
2. **ECC kiểu ECIES (`secp256r1` + ECDH + HKDF-SHA256 + AES-256-GCM – Bài 3.3):**
   - Khóa 256 bit trên đường cong `secp256r1` có mức an toàn tương đương RSA-3072 nhưng khóa công khai Base64 chỉ dài 124 ký tự.
   - Cấu trúc bản mã: $\text{Base64}(\text{Khóa công khai tạm } E_{91\text{B}} \parallel \text{Salt}_{16\text{B}} \parallel \text{Nonce}_{12\text{B}} \parallel \text{Ciphertext}_{\text{AES-GCM}} \parallel \text{Tag}_{16\text{B}})$.
3. **Hệ mã lai RSA + AES-256-GCM (Bài 3.4):**
   - Kết hợp tốc độ và khả năng mã hóa văn bản dài không giới hạn của AES-256-GCM với khả năng trao đổi khóa an toàn của RSA-2048 OAEP.
   - Cấu trúc bản mã: $\text{Base64}(\text{Khóa AES đã bọc RSA}_{256\text{B}} \parallel \text{Nonce}_{12\text{B}} \parallel \text{Ciphertext}_{\text{AES-GCM}} \parallel \text{Tag}_{16\text{B}})$.
4. **Chữ ký số ECDSA (`SHA256withECDSA` – Bài 3.5):**
   - **Ký** bằng **khóa riêng**, **xác minh** bằng **khóa công khai**. Đảm bảo tính **toàn vẹn**, **xác thực** và **chống chối bỏ** (không giữ bí mật nội dung).

---

## 4. Danh mục 37 Hình ảnh Minh chứng trong Báo cáo Word (`Huynh.docx`)

### 4.1. Phần 1 – CSDL SQL Server, Lớp dùng chung & Form Kết nối / Lịch sử (`Hình 1` $\rightarrow$ `Hình 5`)
- **Hình 1 (`image1.png`):** Thiết lập cơ sở dữ liệu `LAB3_MaHoaKhoaCongKhai`, bảng `dbo.History` và kiểm tra đăng nhập tài khoản `lab3_user` qua cổng `14333` trên SSMS.
- **Hình 2 (`image2.png`):** Khởi tạo dự án Maven `LAB3_MaHoaKhoaCongKhai`, các lớp dùng chung (`CryptoSupport`, `FileTextIO`, `DbConfig`, `HistoryDao`) và thiết kế giao diện `frm_KetNoiSQL.java`.
- **Hình 3 (`image3.png`):** Cài đặt mã nguồn xử lý sự kiện nút Kiểm tra kết nối (`btn_testActionPerformed`) trong `frm_KetNoiSQL.java`.
- **Hình 4 (`image4.png`):** Cài đặt hàm khởi tạo và phương thức tải dữ liệu lịch sử `loadData()` trong `frm_LichSu.java`.
- **Hình 5 (`image5.png`):** Chạy kiểm thử kết nối Microsoft SQL Server thành công trên biểu mẫu `frm_KetNoiSQL` (`127.0.0.1:14333`, `lab3_user`).

### 4.2. Bài 3.1 – Mật mã RSA Console (`Hình 6` $\rightarrow$ `Hình 7`)
- **Hình 6 (`image6.png`):** Chạy kiểm thử `RSADemo.java` với khóa **2048 bit** (`362 ms`, giới hạn 190 byte, bản mã 344 ký tự Base64, giải mã đúng `HELLO RSA`).
- **Hình 7 (`image7.png`):** Chạy kiểm thử `RSADemo.java` với khóa **3072 bit** (`860 ms`, giới hạn 318 byte, bản mã 512 ký tự Base64, giải mã đúng `HELLO RSA`).

### 4.3. Bài 3.2 – Mật mã RSA với giao diện Swing (`Hình 8` $\rightarrow$ `Hình 13`)
- **Hình 8 (`image8.png`):** Thiết kế giao diện biểu mẫu Mật mã RSA (`frm_RSA.java`) trên NetBeans GUI Builder.
- **Hình 9 (`image9.png`):** Cài đặt mã nguồn sự kiện nút `Generate Keys` (`btn_generateActionPerformed`) trong `frm_RSA.java`.
- **Hình 10 (`image10.png`):** Cài đặt mã nguồn sự kiện nút `Encrypt` (`btn_encryptActionPerformed`) trong `frm_RSA.java`.
- **Hình 11 (`image11.png`):** Cài đặt mã nguồn sự kiện nút `Decrypt` (`btn_decryptActionPerformed`) trong `frm_RSA.java`.
- **Hình 12 (`image12.png`):** Chạy kiểm thử mã hóa RSA-2048 với `HELLO RSA` trên `frm_RSA` (xuất 344 ký tự Base64).
- **Hình 13 (`image13.png`):** Chạy kiểm thử giải mã RSA-2048 khôi phục chính xác `HELLO RSA` trên `frm_RSA`.

### 4.4. Bài 3.3 – Mật mã đường cong Elliptic ECC (`Hình 14` $\rightarrow$ `Hình 20`)
- **Hình 14 (`image14.png`):** Thiết kế giao diện biểu mẫu Mật mã đường cong Elliptic (`frm_ECC.java`) trên NetBeans GUI Builder.
- **Hình 15 (`image15.png`):** Cài đặt mã nguồn sự kiện nút `Generate Keys` (`btn_generateActionPerformed`) trong `frm_ECC.java`.
- **Hình 16 (`image16.png`):** Cài đặt mã nguồn sự kiện nút `Save Keys` (`btn_savekeysActionPerformed` – `ecc_keys.txt`) trong `frm_ECC.java`.
- **Hình 17 (`image17.png`):** Cài đặt mã nguồn sự kiện nút `Load Keys` (`btn_loadkeysActionPerformed`) trong `frm_ECC.java`.
- **Hình 18 (`image18.png`):** Cài đặt mã nguồn sự kiện nút `Encrypt` (`btn_encryptActionPerformed`) trong `frm_ECC.java`.
- **Hình 19 (`image19.png`):** Cài đặt mã nguồn sự kiện nút `Decrypt` (`btn_decryptActionPerformed`) trong `frm_ECC.java`.
- **Hình 20 (`image20.png`):** Chạy kiểm thử mã hóa và giải mã thành công `HELLO ECC` trên `frm_ECC`.

### 4.5. Bài 3.4 – Hệ mã lai RSA + AES-256-GCM (`Hình 21` $\rightarrow$ `Hình 29`)
- **Hình 21 (`image21.png`):** Thiết kế giao diện biểu mẫu Hệ mã lai RSA + AES-GCM (`frm_RSA_AES.java`) bổ sung nút `Sửa 1 ký tự bản mã`.
- **Hình 22 (`image22.png`):** Cài đặt mã nguồn sự kiện nút `Generate Keys` (`btn_generateActionPerformed`) trong `frm_RSA_AES.java`.
- **Hình 23 (`image23.png`):** Cài đặt mã nguồn sự kiện nút `Save Keys` (`btn_savekeysActionPerformed` – `rsa_aes_keys.txt`) trong `frm_RSA_AES.java`.
- **Hình 24 (`image24.png`):** Cài đặt mã nguồn sự kiện nút `Load Keys` (`btn_loadkeysActionPerformed`) trong `frm_RSA_AES.java`.
- **Hình 25 (`image25.png`):** Cài đặt mã nguồn sự kiện nút `Encrypt` (`btn_encryptActionPerformed`) trong `frm_RSA_AES.java`.
- **Hình 26 (`image26.png`):** Cài đặt mã nguồn sự kiện nút `Decrypt` (`btn_decryptActionPerformed`) trong `frm_RSA_AES.java`.
- **Hình 27 (`image27.png`):** Cài đặt mã nguồn sự kiện nút `Sửa 1 ký tự bản mã` (`btn_tamperActionPerformed`) trong `frm_RSA_AES.java`.
- **Hình 28 (`image28.png`):** Chạy kiểm thử mã hóa và giải mã thành công đoạn văn bản tiếng Việt dài **1.000 ký tự** trên `frm_RSA_AES`.
- **Hình 29 (`image29.png`):** Kiểm thử phát hiện bản mã bị sửa đổi 1 ký tự (`javax.crypto.AEADBadTagException: Tag mismatch`) trên `frm_RSA_AES`.

### 4.6. Bài 3.5 – Chữ ký số ECDSA (`Hình 30` $\rightarrow$ `Hình 37`)
- **Hình 30 (`image30.png`):** Thiết kế giao diện biểu mẫu Chữ ký số ECDSA (`frm_ECC_Signature.java`).
- **Hình 31 (`image31.png`):** Cài đặt mã nguồn sự kiện nút `Generate Keys` (`btn_generateActionPerformed`) trong `frm_ECC_Signature.java`.
- **Hình 32 (`image32.png`):** Cài đặt mã nguồn sự kiện nút `Save Keys` (`btn_savekeysActionPerformed` – `ecdsa_keys.txt`) trong `frm_ECC_Signature.java`.
- **Hình 33 (`image33.png`):** Cài đặt mã nguồn sự kiện nút `Load Keys` (`btn_loadkeysActionPerformed`) trong `frm_ECC_Signature.java`.
- **Hình 34 (`image34.png`):** Cài đặt mã nguồn sự kiện nút `Sign` (`btn_signActionPerformed`) trong `frm_ECC_Signature.java`.
- **Hình 35 (`image35.png`):** Cài đặt mã nguồn sự kiện nút `Verify` (`btn_verifyActionPerformed`) trong `frm_ECC_Signature.java`.
- **Hình 36 (`image36.png`):** Chạy kiểm thử ký số ECDSA với thông điệp `THONG DIEP A` trên `frm_ECC_Signature`.
- **Hình 37 (`image37.png`):** Chạy kiểm thử thay đổi thông điệp thành `THONG DIEP B` trên `frm_ECC_Signature` (chữ ký biến đổi hoàn toàn).

---

## 5. Hướng dẫn chạy lại dự án trên máy cá nhân
1. **Khởi động SQL Server:** Đảm bảo dịch vụ SQL Server đang chạy, bật `TCP/IP` cổng `14333` và bật `Mixed Mode`.
2. **Tạo CSDL & tài khoản:** Chạy `01_Tao_CSDL.sql` và `02_Tao_TaiKhoan_LAB.sql` trong SSMS để tạo `LAB3_MaHoaKhoaCongKhai`, bảng `dbo.History` và tài khoản `lab3_user`.
3. **Mở dự án trong NetBeans:** Mở project Maven `LAB3_MaHoaKhoaCongKhai` $\rightarrow$ **Clean and Build (`Shift+F11`)**.
4. **Chạy chương trình:**
   - Bài 3.1 Console: nhấp phải `RSADemo.java` $\rightarrow$ **Run File (`Shift+F6`)**.
   - Các bài Swing & SQL: bấm **Run Project (`F6`)** để mở `frm_Menu`, vào **Kết nối SQL Server** kiểm tra kết nối thành công trước khi thao tác để ghi nhận lịch sử vào `dbo.History`.
