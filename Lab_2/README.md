# BÁO CÁO THỰC HÀNH AN TOÀN VÀ BẢO MẬT THÔNG TIN — LAB 2: MÃ HÓA ĐỐI XỨNG HIỆN ĐẠI (JAVA SWING – MAVEN – BOUNCYCASTLE – SQL SERVER)

## 1. Thông tin sinh viên
- **Họ và tên:** Phạm Đỗ Hoàng Luân
- **Mã số sinh viên (MSSV):** 1150070026
- **Lớp:** 11_ĐH_TMĐT
- **Link YouTube: https://www.youtube.com/watch?v=At8ZvP4h550 
- **Học phần:** An toàn và Bảo mật Thông tin (Thực hành)
- **Nền tảng & Công nghệ sử dụng:**
  - **Ngôn ngữ & SDK:** Java (JDK 17 / JDK 21) & Python 3.10+ (`cryptography`, `tkinter`)
  - **Môi trường phát triển (IDE):** Apache NetBeans IDE 31 (NetBeans GUI Builder – Java Swing `JFrame`)
  - **Quản lý dự án & Phụ thuộc:** Apache Maven (`pom.xml`: thư viện `com.microsoft.sqlserver:mssql-jdbc:12.8.1.jre11` và `org.bouncycastle:bcprov-jdk18on:1.78.1`)
  - **Hệ quản trị Cơ sở dữ liệu:** Microsoft SQL Server (`127.0.0.1`, TCP Port `1433`, CSDL `LAB2_MaHoaHienDai`) & SQL Server Management Studio (SSMS)

---

## 2. Danh sách tệp báo cáo và tài nguyên trong thư mục


  - `pom.xml`: Khai báo cấu hình biên dịch Java 17 (`maven.compiler.release = 17`), lớp chính `week_02.frm_Menu`, thư viện kết nối SQL Server `mssql-jdbc` và thư viện mật mã `bcprov-jdk18on` (BouncyCastle).
  - **Các lớp tiện ích & truy cập dữ liệu dùng chung (Phần 1):**
    - `CipherUtil.java`: Chuyển đổi chuỗi ↔ mảng byte `UTF-8`, mã hóa/giải mã `Base64` và `HEX`, sinh số ngẫu nhiên an toàn (`SecureRandom`), ghép mảng byte (`concat`) và dẫn xuất khóa `PBKDF2WithHmacSHA256`.
    - `FileTextIO.java`: Đọc và lưu tệp văn bản `.txt` chuẩn `UTF-8` thông qua `JFileChooser` (`openText`, `saveText`, `openString`, `saveString`).
    - `DbConfig.java`, `HistoryDao.java`, `UserDao.java`: Quản lý kết nối JDBC tới SQL Server, đăng ký/đăng nhập người dùng với bảng `dbo.app_users` (lưu `salt` 16 byte + `password_hash` PBKDF2 32 byte, cấp vé phiên `Ticket` 128 bit) và ghi nhật ký thao tác vào bảng `dbo.crypto_history`.
    - `frm_KetNoiSQL.java` (`.form`) & `frm_LichSu.java` (`.form`): Giao diện cấu hình/kiểm tra kết nối SQL Server và bảng xem 50 dòng lịch sử thao tác mới nhất.
  - **Các cặp lớp Thuật toán & Form Giao diện (Bài 2.1 – 2.6):**
    1. `DESCipher.java` & `frm_DES.java` (`.form`): Mật mã khối DES (`DES/CBC/PKCS5Padding`, khóa 8 byte, IV ngẫu nhiên 8 byte).
    2. `TripleDESCipher.java` & `frm_TripleDES.java` (`.form`): Mật mã khối 3DES (`DESede/CBC/PKCS5Padding`, khóa 24 byte, IV ngẫu nhiên 8 byte).
    3. `AESCipher.java` & `frm_AES.java` (`.form`): Mật mã khối xác thực AES-256-GCM (`AES/GCM/NoPadding`, dẫn xuất khóa PBKDF2 120.000 vòng) tích hợp Đăng ký, Đăng nhập SQL Server và kiểm soát phiên bằng `Ticket`.
    4. `RC4Cipher.java` & `frm_RC4.java` (`.form`): Mật mã dòng RC4 (thuật toán lập lịch khóa KSA + sinh dòng khóa PRGA, bản mã hiển thị dạng HEX).
    5. `TwofishCipher.java` & `frm_Twofish.java` (`.form`): Mật mã khối Twofish 256-bit chế độ CBC đệm PKCS7 (thư viện BouncyCastle, khóa `SHA-256(Key)`, hiển thị tường minh IV 16 byte = 32 ký tự HEX, lưu tệp kèm dòng `IV=...`).
    6. `AESCTRCipher.java` & `frm_AES_CTR.java` (`.form`) / `aes_cbc_gui.py`: Bài tập nâng cao AES-256 chế độ CTR trên Java và AES-256 chế độ CBC trên Python Tkinter.
    7. `frm_Menu.java` (`.form`): Cửa sổ Menu điều hướng chính của toàn bộ bài Lab 2.

---

## 3. Tóm tắt Lý thuyết & Đặc tả Kỹ thuật các Thuật toán Mã hóa Đối xứng Hiện đại

### 3.1. Bảng tổng hợp bộ dữ liệu kiểm thử thực tế trên máy thực hành

| Bài | Thuật toán / Chức năng | Lớp xử lý & Form GUI | Dữ liệu đầu vào (Plaintext / Cấu hình) | Khóa (Key) / IV / Ticket | Kết quả đầu ra (Ciphertext / Trạng thái) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1.6** | **Kết nối SQL & Lịch sử** | `DbConfig`, `HistoryDao` / `frm_KetNoiSQL`, `frm_LichSu` | Server: `127.0.0.1`, Port: `1433`, DB: `LAB2_MaHoaHienDai` | User: `sa` / Pass: `123` | `Kết nối thành công: Microsoft SQL Server 15.00.2200 - database LAB2_MaHoaHienDai` |
| **2.1** | **DES (CBC/PKCS5)** | `DESCipher` / `frm_DES` | `Xin chào Lab 2 - Mã hóa hiện đại !` | Key: `12345678` (8 byte) | `FhtjHFGNOCJhmoiJsUTJPzjYK5RMS/hSxA568fV7m0iOZb7EDQJpNyCrd7RZlSu8ozWezdy8Dlo=` |
| **2.2** | **TripleDES (3DES)** | `TripleDESCipher` / `frm_TripleDES` | `Xin chào Lab 2 - Mã hóa hiện đại` | Key: `123456789012345678901234` (24 byte) | `IBnl19tq6bhw/mFUu+RHQRlXv45qkg7ywi4/6Dds2NliP240gVjck/Lbwunb3jhX1qi2CnNhK9c=` |
| **2.3** | **AES-256-GCM + SQL Login** | `AESCipher`, `UserDao` / `frm_AES` | User: `sinhvien01`<br>Plaintext: `Xin chào Lab 2 - Mã hóa hiện đại!` | Key AES: `MatKhauAES@20226`<br>Ticket: `D948F08603CA14F844BA0E5D3EC0EC42` | `xfwuSbuNw5Oouy4IgJ2Wc7QcZm058bRY1wHFMFyMtrLuAU9i7OaS6t/gjS8Ew6ETGMWjbhkxhhJ+vQ++pfcrFgBhidTZ93RYltPgSeNTGZAtdR1ljg==` |
| **2.4** | **RC4 (Stream Cipher)** | `RC4Cipher` / `frm_RC4` | `Plaintext` (9 byte) | Key: `key` | **`5B0055844AFB1E323C`** (18 ký tự HEX) |
| **2.5** | **Twofish (CBC/PKCS7)** | `TwofishCipher` / `frm_Twofish` | `Xin chào Lab 2 - Mã Hóa hiện đại!` | Key: `Twofish@Lab2`<br>IV: `1CA146D461E92D2837DE115116EF25F8` | `aKvWs1qt8mrZfJ8YIDn2XlT7tlL+XWJF3msMvzFSeQj1AmN3UD6zjX/ckheYxdf6` |

### 3.2. Nguyên lý hoạt động & Cấu trúc bản mã của từng hệ mật
1. **Mật mã khối DES (`DES/CBC/PKCS5Padding` – Bài 2.1):**
   - Kích thước khối 64 bit (8 byte), khóa 64 bit (8 ký tự ASCII = 8 byte, trong đó 56 bit thực dùng và 8 bit kiểm tra chẵn lẻ).
   - Mỗi lần mã hóa sinh một vector khởi tạo `IV` ngẫu nhiên 8 byte, thực hiện đệm `PKCS5Padding` và mã hóa chế độ `CBC`.
   - Cấu trúc bản mã: $\text{Ciphertext} = \text{Base64}(\text{IV}_{8\text{B}} \parallel E_K(\text{Plaintext}))$. Nhờ `IV` ngẫu nhiên, mã hóa cùng một bản rõ nhiều lần luôn tạo ra các chuỗi bản mã khác nhau.
2. **Mật mã khối TripleDES / 3DES (`DESede/CBC/PKCS5Padding` – Bài 2.2):**
   - Thực hiện 3 vòng DES liên tiếp theo mô hình **EDE** (Encrypt $K_1$ $\rightarrow$ Decrypt $K_2$ $\rightarrow$ Encrypt $K_3$) trên khối 8 byte.
   - Khóa dài 24 byte ($3 \times 8$ byte = 24 ký tự ASCII không dấu). Cấu trúc bản mã: $\text{Base64}(\text{IV}_{8\text{B}} \parallel E_{K_1,K_2,K_3}(\text{Plaintext}))$.
3. **Mật mã xác thực AES-256-GCM kết hợp PBKDF2 & SQL Server (`AES/GCM/NoPadding` – Bài 2.3):**
   - **Quản lý định danh & Xác thực:** Mật khẩu đăng ký tài khoản được băm bằng `PBKDF2WithHmacSHA256` (`120.000` vòng, `Salt` ngẫu nhiên 16 byte, đầu ra 256 bit = 32 byte) lưu vào bảng `dbo.app_users`. Khi đăng nhập, hệ thống dùng hàm so sánh thời gian hằng `MessageDigest.isEqual()` để chống tấn công đo thời gian (*Timing Attack*) và cấp mã phiên `Ticket` ngẫu nhiên 128 bit (32 ký tự HEX).
   - **Mã hóa xác thực (AEAD):** Từ mật khẩu `Key AES`, hệ thống sinh `Salt` ngẫu nhiên 16 byte và dẫn xuất khóa AES 256 bit bằng `PBKDF2` (120.000 vòng), kết hợp với `Nonce` ngẫu nhiên 12 byte để mã hóa ở chế độ `AES/GCM/NoPadding` kèm thẻ xác thực `Authentication Tag` 128 bit (16 byte).
   - Cấu trúc bản mã tự chứa: $\text{Ciphertext} = \text{Base64}(\text{Salt}_{16\text{B}} \parallel \text{Nonce}_{12\text{B}} \parallel \text{EncryptedData} \parallel \text{Tag}_{16\text{B}})$. Nếu bản mã bị sửa đổi dù chỉ 1 bit hoặc nhập sai khóa, quá trình giải mã sẽ phát hiện ngay qua ngoại lệ `AEADBadTagException`.
4. **Mật mã dòng RC4 (`KSA + PRGA` – Bài 2.4):**
   - **KSA (Key-Scheduling Algorithm):** Khởi tạo mảng hoán vị $S = [0..255]$ và trộn $S$ theo mảng byte của khóa $K$ (độ dài từ 1 đến 256 byte).
   - **PRGA (Pseudo-Random Generation Algorithm):** Sinh từng byte dòng khóa (keystream) và thực hiện phép $\text{XOR}$ ($\oplus$) trực tiếp với từng byte bản rõ.
   - Bản mã không cần đệm, có độ dài byte đúng bằng bản rõ và được hiển thị dưới dạng chuỗi thập lục phân (`HEX`).
5. **Mật mã khối Twofish (`Twofish/CBC/PKCS7Padding` – Bài 2.5):**
   - Sử dụng `TwofishEngine` của thư viện **BouncyCastle** với kích thước khối 128 bit (16 byte). Khóa 256 bit (32 byte) được tạo bằng cách băm chuỗi mật khẩu qua thuật toán `SHA-256`.
   - Vector khởi tạo `IV` dài 16 byte (32 ký tự HEX) được sinh ngẫu nhiên qua nút `Generate IV` và hiển thị tường minh trên form. Khi lưu ra tệp `.txt`, chương trình lưu 2 dòng gồm `IV=<32_ký_tự_HEX>` ở dòng 1 và bản mã `Base64` ở dòng 2 (không lưu Key).

---

## 4. Danh mục 35 Hình ảnh Minh chứng trong Báo cáo Word (`Doc1.docx`)

### 4.1. Phần 1 – Chuẩn bị CSDL SQL Server, Lớp dùng chung & Form Kết nối / Lịch sử (`Hình 1` $\rightarrow$ `Hình 6`)
- **Hình 1 (`image1.png`):** Khởi tạo dự án Maven `LAB2_MaHoaHienDai`, xây dựng các lớp nền tảng (`CipherUtil`, `FileTextIO`, `DbConfig`, `HistoryDao`, `UserDao`) và thiết kế giao diện `frm_ketNoiSQL.java`.
- **Hình 2 (`image2.png`):** Cài đặt mã nguồn xử lý sự kiện nút Kiểm tra kết nối (`btn_testActionPerformed`) trong `frm_ketNoiSQL`.
- **Hình 3 (`image3.png`):** Cài đặt mã nguồn xử lý sự kiện nút Xem lịch sử (`btn_historyActionPerformed`) mở biểu mẫu `frm_LichSu`.
- **Hình 4 (`image4.png`):** Cài đặt hàm khởi tạo (Constructor) và phương thức tải dữ liệu lịch sử `loadData()` trong `frm_LichSu.java`.
- **Hình 5 (`image5.png`):** Chạy kiểm thử kết nối Microsoft SQL Server trên biểu mẫu `frm_KetNoiSQL` (`Server = 127.0.0.1`, `Port = 1433`, `Database = LAB2_MaHoaHienDai`, `User = sa` $\rightarrow$ Kết nối thành công SQL Server 15.00.2200).
- **Hình 6 (`image6.png`):** Kiểm thử mở biểu mẫu Lịch sử thao tác (`frm_LichSu`) truy vấn bảng `dbo.crypto_history`.

### 4.2. Bài 2.1 – Mật mã khối DES (`Hình 7` $\rightarrow$ `Hình 11`)
- **Hình 7 (`image7.png`):** Thiết kế giao diện biểu mẫu Mật mã DES (`frm_DES.java`) trên NetBeans GUI Builder với khóa mặc định 8 byte `12345678`.
- **Hình 8 (`image8.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mã hóa (`btn_encryptActionPerformed`) trong `frm_DES.java`.
- **Hình 9 (`image9.png`):** Cài đặt mã nguồn xử lý sự kiện nút Giải mã (`btn_decryptActionPerformed`) trong `frm_DES.java`.
- **Hình 10 (`image10.png`):** Cài đặt mã nguồn xử lý sự kiện nút Lưu tệp (`btn_saveActionPerformed` – `des_ciphertext.txt`) trong `frm_DES.java`.
- **Hình 11 (`image11.png`):** Chạy kiểm thử chương trình Mật mã DES (`frm_DES`) với dữ liệu tiếng Việt có dấu `Xin chào Lab 2 - Mã hóa hiện đại !` và `Key = 12345678`.

### 4.3. Bài 2.2 – Mật mã khối TripleDES / 3DES (`Hình 12` $\rightarrow$ `Hình 16`)
- **Hình 12 (`image12.png`):** Thiết kế giao diện biểu mẫu Mật mã TripleDES (`frm_TripleDES.java`) bằng thao tác Refactor Copy từ `frm_DES` với khóa 24 byte `123456789012345678901234`.
- **Hình 13 (`image13.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mã hóa (`btn_encryptActionPerformed`) trong `frm_TripleDES.java`.
- **Hình 14 (`image14.png`):** Cài đặt mã nguồn xử lý sự kiện nút Giải mã (`btn_decryptActionPerformed`) trong `frm_TripleDES.java`.
- **Hình 15 (`image15.png`):** Cài đặt mã nguồn xử lý sự kiện nút Lưu tệp và Mở tệp (`btn_saveActionPerformed` – `tripledes_ciphertext.txt` & `btn_openActionPerformed`) trong `frm_TripleDES.java`.
- **Hình 16 (`image16.png`):** Chạy kiểm thử chương trình Mật mã TripleDES (`frm_TripleDES`) với khóa 24 byte.

### 4.4. Bài 2.3 – Mật mã AES-256-GCM kết hợp Đăng ký / Đăng nhập SQL Server (`Hình 17` $\rightarrow$ `Hình 24`)
- **Hình 17 (`image17.png`):** Thiết kế giao diện biểu mẫu AES-GCM kết hợp Đăng ký / Đăng nhập SQL Server (`frm_AES.java`).
- **Hình 18 (`image18.png`):** Cài đặt mã nguồn xử lý sự kiện nút Kết nối SQL Server (`btn_sqlActionPerformed`) và Đăng ký tài khoản (`btn_registerActionPerformed`) trong `frm_AES.java`.
- **Hình 19 (`image19.png`):** Cài đặt mã nguồn xử lý sự kiện nút Đăng nhập và cấp vé phiên Ticket (`btn_loginActionPerformed`) trong `frm_AES.java`.
- **Hình 20 (`image20.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mã hóa AES-256-GCM (`btn_encryptActionPerformed`) kiểm tra quyền đăng nhập `currentUser` trong `frm_AES.java`.
- **Hình 21 (`image21.png`):** Cài đặt mã nguồn xử lý sự kiện nút Giải mã AES-256-GCM (`btn_decryptActionPerformed`) trong `frm_AES.java`.
- **Hình 22 (`image22.png`):** Cài đặt mã nguồn xử lý sự kiện nút Lưu tệp và Mở tệp (`btn_saveActionPerformed` – `aes_ciphertext.txt` & `btn_openActionPerformed`) trong `frm_AES.java`.
- **Hình 23 (`image23.png`):** Thực hiện kết nối SQL Server trực tiếp từ biểu mẫu `frm_AES` trước khi Đăng ký / Đăng nhập.
- **Hình 24 (`image24.png`):** Kiểm thử toàn trình Đăng ký, Đăng nhập tài khoản `sinhvien01` nhận Ticket `D948F08603CA14F844BA0E5D3EC0EC42` và Mã hóa AES-256-GCM lưu lịch sử vào SQL Server.

### 4.5. Bài 2.4 – Mật mã dòng RC4 (`Hình 25` $\rightarrow$ `Hình 29`)
- **Hình 25 (`image25.png`):** Thiết kế giao diện biểu mẫu Mật mã dòng RC4 (`frm_RC4.java`) bằng thao tác Refactor Copy.
- **Hình 26 (`image26.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mã hóa (`btn_encryptActionPerformed`) trong `frm_RC4.java`.
- **Hình 27 (`image27.png`):** Cài đặt mã nguồn xử lý sự kiện nút Giải mã (`btn_decryptActionPerformed`) trong `frm_RC4.java`.
- **Hình 28 (`image28.png`):** Cài đặt mã nguồn xử lý sự kiện nút Lưu tệp (`btn_saveActionPerformed` – `rc4_ciphertext.txt`) trong `frm_RC4.java`.
- **Hình 29 (`image29.png`):** Chạy kiểm thử chương trình Mật mã dòng RC4 (`frm_RC4`) với `Plaintext = Plaintext`, `Key = key` xuất bản mã HEX `5B0055844AFB1E323C`.

### 4.6. Bài 2.5 – Mật mã khối Twofish BouncyCastle (`Hình 30` $\rightarrow$ `Hình 35`)
- **Hình 30 (`image30.png`):** Thiết kế giao diện biểu mẫu Mật mã Twofish (`frm_Twofish.java`) bổ sung điều khiển Vector khởi tạo `IV (HEX)` và nút `Generate IV`.
- **Hình 31 (`image31.png`):** Cài đặt mã nguồn xử lý sự kiện nút Sinh Vector khởi tạo ngẫu nhiên 16 byte (`btn_ivActionPerformed`) trong `frm_Twofish.java`.
- **Hình 32 (`image32.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mã hóa Twofish-CBC (`btn_encryptActionPerformed`) trong `frm_Twofish.java`.
- **Hình 33 (`image33.png`):** Cài đặt mã nguồn xử lý sự kiện nút Giải mã Twofish-CBC (`btn_decryptActionPerformed`) trong `frm_Twofish.java`.
- **Hình 34 (`image34.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mở tệp khôi phục đồng thời IV và bản mã (`btn_openActionPerformed`) trong `frm_Twofish.java`.
- **Hình 35 (`image35.png`):** Chạy kiểm thử chương trình Mật mã Twofish (`frm_Twofish`) với khóa `Twofish@Lab2` và `IV = 1CA146D461E92D2837DE115116EF25F8`.

---

## 5. Tổng kết So sánh Đặc tính Kỹ thuật & An ninh của các Hệ mật Đối xứng Hiện đại

| Thuật toán | Loại mật mã | Kích thước khối | Kích thước khóa | Chế độ & Đệm trong Lab | Đánh giá an ninh & Khuyến nghị thực tế |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **DES** | Mã khối (Feistel 16 vòng) | 64 bit (8 byte) | 56 bit hiệu dụng (8 byte) | `CBC` + `PKCS5Padding` | **Đã lỗi thời:** Không gian khóa $2^{56}$ quá nhỏ, dễ dàng bị phá bằng vét cạn (Brute-force). Chỉ dùng cho mục đích học thuật. |
| **TripleDES (3DES)** | Mã khối (3 vòng DES EDE) | 64 bit (8 byte) | 112 / 168 bit (24 byte) | `CBC` + `PKCS5Padding` | **Ngừng khuyến nghị (NIST 2023):** Tốc độ chậm gấp 3 lần DES; khối 64 bit dễ bị tấn công va chạm khối (*Sweet32*). |
| **AES-256-GCM** | Mã khối (Mạng SPN 14 vòng) | 128 bit (16 byte) | 256 bit (32 byte từ PBKDF2) | `GCM` + `NoPadding` (AEAD) | **Chuẩn công nghiệp hiện hành:** Tốc độ cao (hỗ trợ phần cứng AES-NI), vừa bảo mật vừa chống giả mạo bản mã nhờ thẻ xác thực 128 bit. |
| **RC4** | Mã dòng (KSA + PRGA) | Từng byte (Stream) | 8 – 2048 bit (1 – 256 byte) | XOR dòng khóa (`NoPadding`) | **Đã bị cấm trong TLS (RFC 7465):** Các byte đầu của dòng khóa có độ lệch thống kê, dễ bị tấn công khi dùng lại khóa. |
| **Twofish** | Mã khối (Feistel 16 vòng) | 128 bit (16 byte) | 256 bit (`SHA-256(Key)`) | `CBC` + `PKCS7Padding` | **An toàn cao:** Ứng viên chung kết AES, thiết kế S-box phụ thuộc khóa rất mạnh; thường dùng trong các phần mềm mã hóa ổ đĩa/tệp (VeraCrypt, KeePass). |
| **AES-256-CTR** | Mã khối ở chế độ dòng | 128 bit (16 byte) | 256 bit (32 byte ngẫu nhiên) | `CTR` + `NoPadding` | **Hiệu năng cao:** Xử lý song song được, không cần đệm; tuyệt đối không được tái sử dụng cặp `(Key, Nonce)` và nên kết hợp thêm MAC/GCM để chống sửa đổi. |
