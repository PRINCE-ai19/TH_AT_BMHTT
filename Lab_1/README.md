# BÁO CÁO THỰC HÀNH AN TOÀN VÀ BẢO MẬT THÔNG TIN — LAB 1: MÃ HÓA ĐỐI XỨNG CỔ ĐIỂN (JAVA SWING – MAVEN – SQL SERVER)

## 1. Thông tin sinh viên
- **Họ và tên:** Phạm Đỗ Hoàng Luân
- **Mã số sinh viên (MSSV):** 1150070026
- **Lớp:** 11_ĐH_TMĐT
Linh youtube : 
- **Học phần:** An toàn và Bảo mật Thông tin (Thực hành)
- **Nền tảng & Công nghệ sử dụng:**
  - **Ngôn ngữ & SDK:** Java (JDK 17 / JDK 21)
  - **Môi trường phát triển (IDE):** Apache NetBeans IDE 31 (NetBeans GUI Builder – Java Swing `JFrame`)
  - **Quản lý dự án & Phụ thuộc:** Apache Maven (`pom.xml`, thư viện `com.microsoft.sqlserver:mssql-jdbc:12.8.1.jre11`)
  - **Hệ quản trị Cơ sở dữ liệu:** Microsoft SQL Server Express (`SQLEXPRESS`, TCP Port `14333`) & SQL Server Management Studio (SSMS)

---

## 2. Danh sách tệp báo cáo và tài nguyên trong thư mục
- **Báo cáo Word hoàn chỉnh (đã điền đầy đủ thuyết minh & bảng phân tích 32 hình):** [`CHo_Huynh.docx`](file:///d:/Thuc_hanh_MonCHO/CHo_Huynh.docx)
- **Tài liệu hướng dẫn thực hành gốc:** [`LAB1_HuongDan.pdf`](file:///d:/Thuc_hanh_MonCHO/LAB1_HuongDan.pdf)
- **Cấu trúc project Maven `LAB1_MaHoaCoDien` (trong package `week_01`):**
  - `pom.xml`: Khai báo cấu hình biên dịch Java 17 (`maven.compiler.release = 17`), lớp chính `week_01.frm_Menu` và thư viện kết nối SQL Server `mssql-jdbc`.
  - `FileTextIO.java`: Lớp tiện ích dùng chung hỗ trợ mở (`openText`, `openString`) và lưu (`saveText`, `saveString`) tệp văn bản `.txt` chuẩn bảng mã `UTF-8` thông qua `JFileChooser`.
  - **6 cặp lớp Thuật toán & Form Giao diện cơ bản (Bài 1.1 – 1.6):**
    1. `CaesarCipher.java` & `Frm_Caesar.java` (`.form`): Mật mã dịch vòng Caesar.
    2. `VigenereCipher.java` & `Frm_Vigenere.java` (`.form`): Mật mã đa bảng Vigenère.
    3. `RailFenceCipher.java` & `frm_RailFence.java` (`.form`): Mật mã hoán vị Hàng rào (Rail Fence).
    4. `PlayfairCipher.java` & `Frm_Playfair.java` (`.form`): Mật mã thay thế cặp chữ Playfair trên ma trận khóa 5×5.
    5. `TranspositionCipher.java` & `Frm_Transposition.java` (`.form`): Mật mã Hoán vị cột (Columnar Transposition).
    6. `AutokeyCipher.java` & `Frm_Autokey.java` (`.form`): Mật mã khóa tự sinh Autokey.
  - **Các lớp mở rộng (Phần 3, Phần 4, Phần 5):**
    - `CipherTongHop.java`, `DbConfig.java`, `HistoryDao.java`, `frm_TongHop.java`, `frm_KetNoiSQL.java`, `frm_LichSu.java`: Chương trình tổng hợp 6 thuật toán và lưu lịch sử thao tác vào bảng `dbo.history` trên Microsoft SQL Server.
    - `AttackDemos.java`, `frm_TanCong.java`: Phân tích (tấn công) mật mã cổ điển (Vét cạn Caesar, Phân tích tần suất chữ cái, Biết trước bản rõ Vigenère).
    - `frm_Menu.java`: Cửa sổ Menu điều hướng chính của toàn bộ bài Lab.

---

## 3. Tóm tắt Lý thuyết & Đặc tả Kỹ thuật 6 Thuật toán Mã hóa Cổ điển

### 3.1. Bảng tổng hợp bộ dữ liệu kiểm thử chuẩn (Test Vectors)

| Bài | Thuật toán | Lớp xử lý & Form GUI | Plaintext | Key | Ciphertext chuẩn | Kết quả Giải mã |
| :--- | :--- | :--- | :---: | :---: | :---: | :---: |
| **1.1** | **Caesar** | `CaesarCipher` / `Frm_Caesar` | `HELLO` | `3` | **`KHOOR`** | `HELLO` |
| **1.2** | **Vigenère** | `VigenereCipher` / `Frm_Vigenere` | `HELLO` | `KEY` | **`RIJVS`** | `HELLO` |
| **1.3** | **Rail Fence** | `RailFenceCipher` / `frm_RailFence` | `HELLO WORLD` | `3` | **`HOREL OLLWD`** | `HELLO WORLD` |
| **1.4** | **Playfair (5×5)** | `PlayfairCipher` / `Frm_Playfair` | `HELLOWORLD` | `KEYWORD` | **`GYIZSCOKCFBU`** | `HELXLOWORLDX` |
| **1.5** | **Transposition** | `TranspositionCipher` / `Frm_Transposition` | `BAOMAT` | `2,3,1` | **`AAOTBM`** | `BAOMAT` |
| **1.6** | **Autokey** | `AutokeyCipher` / `Frm_Autokey` | `HELLO` | `KEY` | **`RIJSS`** | `HELLO` |

### 3.2. Nguyên lý hoạt động từng hệ mật
1. **Mật mã Caesar (Bài 1.1):**
   - Dịch chuyển mỗi chữ cái `A–Z` / `a–z` đi $k$ vị trí trong bảng chữ cái 26 ký tự:
     - Mã hóa: $C = (P + k) \bmod 26$
     - Giải mã: $P = (C - k) \bmod 26$
   - Sử dụng `Math.floorMod(original + shift, 26)` để xử lý chính xác cả khóa âm khi giải mã; các ký tự ngoài `A–Z` (khoảng trắng, chữ số, dấu câu) được giữ nguyên.
2. **Mật mã Vigenère (Bài 1.2):**
   - Lặp lại từ khóa chữ cái $K$ (chỉ gồm `A–Z`) theo độ dài bản rõ.
   - Công thức: $C_i = (P_i + K_{i \bmod m}) \bmod 26$ và $P_i = (C_i - K_{i \bmod m}) \bmod 26$.
   - Khoảng trắng và dấu câu được giữ nguyên và **không làm tăng chỉ số `keyIndex`**.
3. **Mật mã Rail Fence – Hàng rào (Bài 1.3):**
   - Xếp các ký tự bản rõ (bao gồm cả dấu cách) theo đường zíc-zắc đi xuống rồi đi lên trên $k$ hàng (`rails` từ 2 đến 1000) bằng mảng mẫu `railPattern(length, rails)`, sau đó đọc nối tiếp từng hàng từ hàng `0` đến hàng `rails - 1`.
4. **Mật mã Playfair – Ma trận 5×5 (Bài 1.4):**
   - Tạo ma trận khóa `5×5` từ các chữ cái không trùng nhau của khóa nối với bảng chữ cái (gộp `I` và `J` chung một ô).
   - Chuẩn hóa bản rõ: bỏ khoảng trắng, đổi `J -> I`, chia thành từng cặp 2 chữ (`makePairs`). Nếu 2 chữ trong một cặp giống nhau hoặc lẻ chữ cuối cùng thì chèn ký tự đệm `X` (hoặc `Q` nếu chữ đó là `X`).
   - Áp dụng 3 quy tắc biến đổi trên ma trận `5×5`:
     - **Cùng hàng:** Lấy chữ ngay bên phải (mã hóa: `+1 mod 5`) / bên trái (giải mã: `+4 mod 5`).
     - **Cùng cột:** Lấy chữ ngay bên dưới (mã hóa: `+1 mod 5`) / bên trên (giải mã: `+4 mod 5`).
     - **Khác hàng, khác cột (Hình chữ nhật):** Lấy chữ ở góc đối diện trên cùng hàng.
5. **Mật mã Hoán vị cột – Columnar Transposition (Bài 1.5):**
   - Khóa là danh sách thứ tự đọc các cột cách nhau bởi dấu phẩy (ví dụ `2,3,1` là hoán vị của `1..3`).
   - Điền bản rõ theo từng hàng vào lưới có số cột bằng độ dài khóa $m$, sau đó đọc xuất ra theo từng cột dựa trên thứ tự chỉ định trong khóa.
6. **Mật mã Autokey (Bài 1.6):**
   - Khắc phục tính tuần hoàn của khóa Vigenère bằng cách tạo dòng khóa gồm **khóa mồi (primer key)** nối tiếp với chính **các ký tự chữ cái của bản rõ** (`KEY` + `HELLO` $\rightarrow$ dòng khóa `KEYHE`).
   - Khi giải mã, ngay khi khôi phục được ký tự bản rõ nào (`transformed`), chương trình lập tức đẩy độ dịch của ký tự vừa tìm được vào danh sách `plainOffsets` để giải mã các ký tự tiếp theo.

---

## 4. Danh mục 32 Hình ảnh Minh chứng trong Báo cáo Word (`CHo_Huynh.docx`)

### 4.1. Bài 1.1 – Mật mã Caesar (`Hình 1` $\rightarrow$ `Hình 6`)
- **Hình 1 (`image1.png`):** Khởi tạo dự án Maven `LAB1_MaHoaCoDien`, thiết kế giao diện `Frm_Caesar.java` trên NetBeans GUI Builder và biên dịch dự án thành công (`BUILD SUCCESS`).
- **Hình 2 (`image2.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mã hóa (`btn_encryptActionPerformed`) trong `Frm_Caesar.java`.
- **Hình 3 (`image3.png`):** Cài đặt mã nguồn xử lý sự kiện nút Giải mã (`btn_decryptActionPerformed`) trong `Frm_Caesar.java`.
- **Hình 4 (`image4.png`):** Cài đặt mã nguồn xử lý sự kiện nút mở tệp bản mã (`btn_openActionPerformed`) gọi `FileTextIO.openText()`.
- **Hình 5 (`image5.png`):** Cài đặt mã nguồn xử lý sự kiện nút lưu tệp bản mã (`btn_saveActionPerformed` – `caesar_ciphertext.txt`) gọi `FileTextIO.saveText()`.
- **Hình 6 (`image6.png`):** Chạy kiểm thử chương trình `Frm_Caesar`: nhập `Plaintext = HELLO`, `Key = 3` $\rightarrow$ kết quả `Ciphertext = KHOOR`.

### 4.2. Bài 1.2 – Mật mã Vigenère (`Hình 7` $\rightarrow$ `Hình 11`)
- **Hình 7 (`image7.png`):** Xây dựng lớp thuật toán `VigenereCipher.java` trong package `week_01`.
- **Hình 8 (`image8.png`):** Thiết kế giao diện `Frm_Vigenere.java` bằng thao tác Refactor Copy từ `Frm_Caesar` với khóa mặc định `KEY`.
- **Hình 9 (`image9.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mã hóa (`btn_encryptActionPerformed`) trong `Frm_Vigenere.java`.
- **Hình 10 (`image10.png`):** Cài đặt mã nguồn xử lý sự kiện nút lưu tệp (`btn_saveActionPerformed` – `vigenere_ciphertext.txt`) trong `Frm_Vigenere.java`.
- **Hình 11 (`image11.png`):** Chạy kiểm thử chương trình `Frm_Vigenere`: nhập `Plaintext = HELLO`, `Key = KEY` $\rightarrow$ kết quả `Ciphertext = RIJVS`.

### 4.3. Bài 1.3 – Mật mã Rail Fence (`Hình 12` $\rightarrow$ `Hình 16`)
- **Hình 12 (`image12.png`):** Thiết kế giao diện `frm_RailFence.java` trong NetBeans GUI Builder với số hàng mặc định `3`.
- **Hình 13 (`image13.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mã hóa (`btn_encryptActionPerformed`) trong `frm_RailFence.java`.
- **Hình 14 (`image14.png`):** Cài đặt mã nguồn xử lý sự kiện nút Giải mã (`btn_decryptActionPerformed`) trong `frm_RailFence.java`.
- **Hình 15 (`image15.png`):** Cài đặt mã nguồn xử lý sự kiện nút lưu tệp (`btn_saveActionPerformed` – `railfence_ciphertext.txt`) trong `frm_RailFence.java`.
- **Hình 16 (`image16.png`):** Chạy kiểm thử chương trình `frm_RailFence`: nhập `Plaintext = HELLO WORLD`, `Key = 3` $\rightarrow$ kết quả `Ciphertext = HOREL OLLWD`.

### 4.4. Bài 1.4 – Mật mã Playfair 5×5 (`Hình 17` $\rightarrow$ `Hình 22`)
- **Hình 17 (`image17.png`):** Thiết kế giao diện `Frm_Playfair.java` bổ sung ô hiển thị Ma trận khóa 5×5 (`txt_matrix`, font `Monospaced 14 Bold`, `editable = false`) và nút `Generate Key Matrix` (`btn_matrix`).
- **Hình 18 (`image18.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mã hóa (`btn_encryptActionPerformed`) trong `Frm_Playfair.java`.
- **Hình 19 (`image19.png`):** Cài đặt mã nguồn xử lý sự kiện nút Giải mã (`btn_decryptActionPerformed`) trong `Frm_Playfair.java`.
- **Hình 20 (`image20.png`):** Cài đặt mã nguồn xử lý sự kiện nút lưu tệp (`btn_saveActionPerformed` – `playfair_ciphertext.txt`) trong `Frm_Playfair.java`.
- **Hình 21 (`image21.png`):** Cài đặt mã nguồn xử lý sự kiện nút tạo ma trận khóa 5×5 (`btn_matrixActionPerformed`) trong `Frm_Playfair.java`.
- **Hình 22 (`image22.png`):** Chạy kiểm thử chương trình `Frm_Playfair`: nhập `Plaintext = HELLOWORLD`, `Key = KEYWORD` $\rightarrow$ sinh ma trận khóa 5×5 và xuất `Ciphertext = GYIZSCOKCFBU`.

### 4.5. Bài 1.5 – Mật mã Hoán vị cột Transposition (`Hình 23` $\rightarrow$ `Hình 27`)
- **Hình 23 (`image23.png`):** Thiết kế giao diện `Frm_Transposition.java` trong NetBeans GUI Builder với khóa mặc định `2,3,1`.
- **Hình 24 (`image24.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mã hóa (`btn_encryptActionPerformed`) trong `Frm_Transposition.java`.
- **Hình 25 (`image25.png`):** Cài đặt mã nguồn xử lý sự kiện nút Giải mã (`btn_decryptActionPerformed`) trong `Frm_Transposition.java`.
- **Hình 26 (`image26.png`):** Cài đặt mã nguồn xử lý sự kiện nút lưu tệp (`btn_saveActionPerformed` – `transposition_ciphertext.txt`) trong `Frm_Transposition.java`.
- **Hình 27 (`image27.png`):** Chạy kiểm thử chương trình `Frm_Transposition`: nhập `Plaintext = BAOMAT`, `Key = 2,3,1` $\rightarrow$ kết quả `Ciphertext = AAOTBM`.

### 4.6. Bài 1.6 – Mật mã Autokey (`Hình 28` $\rightarrow$ `Hình 32`)
- **Hình 28 (`image28.png`):** Thiết kế giao diện `Frm_Autokey.java` trong NetBeans GUI Builder với khóa mồi mặc định `KEY`.
- **Hình 29 (`image29.png`):** Cài đặt mã nguồn xử lý sự kiện nút Mã hóa (`btn_encryptActionPerformed`) trong `Frm_Autokey.java`.
- **Hình 30 (`image30.png`):** Cài đặt mã nguồn xử lý sự kiện nút Giải mã (`btn_decryptActionPerformed`) trong `Frm_Autokey.java`.
- **Hình 31 (`image31.png`):** Cài đặt mã nguồn xử lý sự kiện nút lưu tệp (`btn_saveActionPerformed` – `autokey_ciphertext.txt`) trong `Frm_Autokey.java`.
- **Hình 32 (`image32.png`):** Chạy kiểm thử chương trình `Frm_Autokey`: nhập `Plaintext = HELLO`, `Key = KEY` $\rightarrow$ kết quả `Ciphertext = RIJSS`.

---

## 5. Tổng kết Đánh giá Điểm yếu & Phương pháp Tấn công Mật mã Cổ điển

| Thuật toán | Phân loại | Điểm yếu bảo mật chính | Phương pháp thám mã (Cryptanalysis) |
| :--- | :--- | :--- | :--- |
| **Caesar** | Thay thế đơn biểu | Không gian khóa cực nhỏ (chỉ có 26 khóa khả dĩ). | Vét cạn 26 khóa (**Brute Force**) kết hợp chấm điểm tần suất tiếng Anh; phân tích tần suất chữ cái (**Frequency Analysis**). |
| **Vigenère** | Thay thế đa biểu (khóa lặp) | Khóa lặp lại theo chu kỳ cố định $m$. | Xác định độ dài khóa bằng kiểm định **Kasiski / Friedman (Index of Coincidence)** rồi phân tích tần suất từng cột; hoặc tấn công **Known Plaintext** ($K_i = (C_i - P_i) \bmod 26$). |
| **Rail Fence** | Hoán vị (Zíc-zắc) | Số hàng nhỏ, tần suất chữ cái giữ nguyên 100% như bản rõ. | Thử vét cạn mọi số hàng `rails` từ $2..n$; kiểm tra phân bố tần suất chữ cái. |
| **Playfair** | Thay thế cặp chữ (Digram) | Cấu trúc ma trận 5×5 cố định 25 chữ cái, giữ nguyên tần suất cặp chữ (digram). | Phân tích tần suất cặp chữ (digram frequency analysis); tấn công **Known Plaintext** để dựng lại ma trận 5×5. |
| **Transposition** | Hoán vị cột | Chỉ đổi vị trí ký tự, giữ nguyên tần suất đơn tự của bản rõ. | Thử các độ dài khóa nhỏ, ghép cột (anagramming) dựa trên các cặp chữ cái thông dụng trong ngôn ngữ tự nhiên (`TH`, `HE`, `IN`, `ER`). |
| **Autokey** | Thay thế đa biểu (khóa tự sinh) | Khóa chứa chính nội dung của bản rõ. | Tấn công từ điển / đoán từ thông dụng (**Crib Dragging** với các từ như `THE`, `AND`) để khôi phục dần các đoạn bản rõ và khóa; tấn công **Known Plaintext**. |
