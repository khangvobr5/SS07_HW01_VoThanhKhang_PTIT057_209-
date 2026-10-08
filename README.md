# Bài 1: Quản lý Người Dùng Giới Hạn và Truyền Tải Dữ Liệu qua SFTP trên Windows

## 📋 Mục tiêu

- Tạo tài khoản người dùng giới hạn (`sftp-user`) phục vụ cho các tác vụ truyền nhận tệp tin từ xa.
- Làm chủ quy trình cài đặt và kết nối SFTP bằng phần mềm client trên Windows.
- Thực hiện truyền tải tệp tin nhật ký (logs) an toàn từ máy chủ Linux về máy tính cá nhân.

---

## 🛠️ Môi trường thực hiện

| Thành phần | Chi tiết |
|---|---|
| **Máy chủ (Server)** | Linux Ubuntu (VPS) |
| **Máy khách (Client)** | Windows 10/11 |
| **Phần mềm SFTP Client** | Bitvise SSH Client |
| **Giao thức kết nối** | SFTP (SSH File Transfer Protocol) - Port 22 |

---

## ⚙️ Các bước thực hiện

### Bước 1: Tạo tài khoản người dùng `sftp-user` trên máy chủ Linux

Đăng nhập vào VPS bằng tài khoản `root` hoặc tài khoản có quyền `sudo`, sau đó chạy lệnh:

```bash
sudo adduser sftp-user
```

> Hệ thống sẽ yêu cầu nhập mật khẩu mạnh (tối thiểu 12 ký tự, kết hợp chữ hoa, chữ thường, số và ký tự đặc biệt) và các thông tin tùy chọn.

**Kiểm tra người dùng vừa tạo:**

```bash
id sftp-user
```

**Kết quả mong đợi:**

```
uid=1001(sftp-user) gid=1001(sftp-user) groups=1001(sftp-user)
```

> ✅ Xác nhận `sftp-user` **không** thuộc nhóm `sudo` — không có quyền thực thi lệnh đặc quyền.

---

### Bước 2: Tạo thư mục log và tệp log giả lập

```bash
# Tạo thư mục chứa log
sudo mkdir -p /var/log/app-backup/

# Tạo tệp log giả lập
sudo touch /var/log/app-backup/backup-check.log

# Ghi nội dung giả lập vào file log
sudo bash -c 'echo "Backup status: SUCCESS at $(date)" > /var/log/app-backup/backup-check.log'
```

**Kiểm tra nội dung tệp:**

```bash
cat /var/log/app-backup/backup-check.log
```

**Ví dụ kết quả:**

```
Backup status: SUCCESS at Thu Oct  8 12:30:00 UTC 2026
```

---

### Bước 3: Cấp quyền đọc cho `sftp-user`

```bash
# Gán ownership: root sở hữu file, sftp-user thuộc group sở hữu
sudo chown -R root:sftp-user /var/log/app-backup

# Cấp quyền thư mục: owner=rwx, group=r-x, others=---
sudo chmod 750 /var/log/app-backup

# Cấp quyền tệp tin: owner=rw-, group=r--, others=---
sudo chmod 640 /var/log/app-backup/backup-check.log
```

**Kiểm tra quyền hạn:**

```bash
ls -l /var/log/app-backup/backup-check.log
```

**Kết quả mong đợi:**

```
-rw-r----- 1 root sftp-user 47 Oct  8 12:30 /var/log/app-backup/backup-check.log
```

> ✅ `sftp-user` có thể **đọc** file log nhưng **không thể ghi** hay thực thi.

---

### Bước 4: Kết nối SFTP từ Windows bằng Bitvise SSH Client

#### 4.1 Cài đặt Bitvise SSH Client

Tải và cài đặt Bitvise SSH Client từ trang chính thức: https://www.bitvise.com/ssh-client-download

#### 4.2 Cấu hình kết nối

Mở **Bitvise SSH Client** và điền thông tin:

| Trường | Giá trị |
|---|---|
| **Host** | `<IP_của_VPS>` |
| **Port** | `22` |
| **Username** | `sftp-user` |
| **Initial method** | `password` |
| **Password** | *(mật khẩu đã đặt ở Bước 1)* |

#### 4.3 Thực hiện đăng nhập

1. Nhấn nút **Log in**.
2. Nếu đây là lần kết nối đầu tiên, hệ thống sẽ hiển thị cảnh báo xác minh **Host Key** — nhấn **Accept and Save** để tin tưởng máy chủ.
3. Kết nối thành công sẽ hiển thị trạng thái **"Connected"** màu xanh lá.

#### 4.4 Mở cửa sổ SFTP và tải file về máy

1. Trong giao diện Bitvise, nhấn nút **New SFTP window** để mở trình quản lý file.
2. **Khung bên phải (Remote files)**: Điều hướng đến `/var/log/app-backup/`.
3. **Khung bên trái (Local files)**: Chọn thư mục đích trên máy Windows (ví dụ: `C:\Users\Toan\Downloads\`).
4. **Kéo thả** (hoặc nhấn chuột phải → **Download**) tệp `backup-check.log` sang khung bên trái.

---

## ✅ Kết quả kiểm tra

### Trên VPS — Kiểm tra user và quyền hạn

**Lệnh:**
```bash
id sftp-user
ls -l /var/log/app-backup/backup-check.log
```

**Kết quả thực tế:**
```
uid=1001(sftp-user) gid=1001(sftp-user) groups=1001(sftp-user)
-rw-r----- 1 root sftp-user 47 Oct  8 12:30 /var/log/app-backup/backup-check.log
```

### Trên Windows — Kiểm tra tệp đã tải về

Mở tệp `backup-check.log` bằng Notepad hoặc bất kỳ trình soạn thảo văn bản nào:

```
Backup status: SUCCESS at Thu Oct  8 12:30:00 UTC 2026
```

> ✅ Nội dung tệp tải về **khớp hoàn toàn** với file gốc trên VPS.

---

## 📸 Ảnh chụp màn hình minh chứng

### Ảnh 1: Giao diện Bitvise SSH Client — Cấu hình kết nối

> *(Chèn ảnh chụp màn hình giao diện đăng nhập Bitvise với thông tin Host và Username đã điền)*

### Ảnh 2: Kết nối thành công — Trạng thái "Connected"

> *(Chèn ảnh chụp màn hình Bitvise hiển thị trạng thái kết nối thành công)*

### Ảnh 3: Cửa sổ SFTP — Duyệt file trên máy chủ

> *(Chèn ảnh chụp màn hình SFTP window, khung phải đang hiển thị `/var/log/app-backup/backup-check.log`)*

### Ảnh 4: Tải file thành công — File xuất hiện trên máy Windows

> *(Chèn ảnh chụp màn hình thư mục Windows sau khi tải file về)*

### Ảnh 5: Kiểm tra nội dung tệp trên Windows

> *(Chèn ảnh chụp màn hình Notepad đang mở `backup-check.log` với nội dung hiển thị)*

### Ảnh 6: Kết quả lệnh kiểm tra trên VPS

> *(Chèn ảnh chụp màn hình terminal hiển thị kết quả lệnh `id sftp-user` và `ls -l`)*

---

## 🔐 Phân tích bảo mật

### Tại sao dùng tài khoản riêng `sftp-user` thay vì `root`?

| Tiêu chí | `root` | `sftp-user` |
|---|---|---|
| **Quyền hạn** | Toàn quyền hệ thống | Chỉ đọc file được cấp phép |
| **Rủi ro nếu bị lộ** | Toàn bộ hệ thống bị xâm phạm | Giới hạn ở file log |
| **Nguyên tắc** | Vi phạm Least Privilege | ✅ Tuân thủ Least Privilege |
| **Phù hợp** | Quản trị hệ thống | Truyền file an toàn |

### Phân tích phân quyền

| Quyền | Ý nghĩa |
|---|---|
| `chmod 750 /var/log/app-backup/` | `root` toàn quyền; `sftp-user` (group) vào được thư mục và liệt kê file; người khác không có quyền gì |
| `chmod 640 backup-check.log` | `root` đọc/ghi; `sftp-user` (group) chỉ đọc; người khác không có quyền gì |

---

## 📝 Tổng kết

| Yêu cầu | Trạng thái |
|---|---|
| Tạo tài khoản `sftp-user` không có quyền sudo | ✅ Hoàn thành |
| Tạo thư mục `/var/log/app-backup/` và file log | ✅ Hoàn thành |
| Cấp quyền đọc cho `sftp-user` | ✅ Hoàn thành |
| Kết nối SFTP từ Windows bằng Bitvise | ✅ Hoàn thành |
| Tải tệp `backup-check.log` về máy Windows | ✅ Hoàn thành |
| Kiểm tra nội dung tệp khớp với file gốc | ✅ Hoàn thành |
