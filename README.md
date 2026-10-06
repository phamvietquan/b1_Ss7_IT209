# Bài 1: Quản lý Người Dùng Giới Hạn và Truyền Tải Dữ Liệu qua SFTP trên Windows

## Thông tin bài nộp

| Thông tin | Chi tiết |
|-----------|----------|
| **Môn học** | IT209 – Quản trị Hệ thống Linux |
| **Session** | 07 |
| **Bài tập** | Bài 1 |
| **Sinh viên** | *(Điền tên của bạn)* |
| **MSSV** | *(Điền MSSV của bạn)* |
| **Ngày thực hiện** | 07/10/2026 |

---

## Mục tiêu

- Tạo tài khoản người dùng **giới hạn** (`sftp-user`) phục vụ truyền nhận tệp từ xa.
- Kết nối SFTP từ Windows bằng **Bitvise SSH Client** (hoặc WinSCP / FileZilla).
- Tải an toàn tệp log `/var/log/app-backup/backup-check.log` từ máy chủ Linux về Windows.

---

## Các Bước Thực Hiện

### Bước 1 – Tạo người dùng `sftp-user` trên VPS Linux

Đăng nhập vào VPS với quyền `root` hoặc tài khoản có `sudo`, sau đó chạy lệnh:

```bash
sudo adduser sftp-user
```

> Hệ thống sẽ yêu cầu nhập **mật khẩu mạnh** (kết hợp chữ hoa, chữ thường, số và ký tự đặc biệt) và các thông tin tuỳ chọn khác.

**Kết quả:** Tài khoản `sftp-user` được tạo thành công, **không** thuộc nhóm `sudo`.

---

### Bước 2 – Tạo thư mục log và tệp log giả lập

```bash
# Tạo thư mục backup log
sudo mkdir -p /var/log/app-backup/

# Tạo file log và ghi nội dung giả lập
sudo touch /var/log/app-backup/backup-check.log
sudo bash -c 'echo "Backup status: SUCCESS at $(date)" > /var/log/app-backup/backup-check.log'
```

---

### Bước 3 – Cấp quyền đọc cho `sftp-user`

```bash
# Gán ownership: root sở hữu, sftp-user thuộc group
sudo chown -R root:sftp-user /var/log/app-backup

# Cấp quyền: group được đọc/chạy thư mục
sudo chmod 750 /var/log/app-backup

# Cấp quyền: group được đọc file log
sudo chmod 640 /var/log/app-backup/backup-check.log
```

| Đối tượng | Quyền hạn |
|-----------|-----------|
| `root` (owner) | Đọc + Ghi + Thực thi |
| `sftp-user` (group) | Đọc thư mục + Đọc file |
| Người khác (others) | Không có quyền |

---

### Bước 4 – Kiểm tra trên VPS

```bash
# Kiểm tra thông tin user
id sftp-user

# Kiểm tra quyền hạn tệp tin
ls -l /var/log/app-backup/backup-check.log
```

**Kết quả mong đợi:**

```
uid=1001(sftp-user) gid=1001(sftp-user) groups=1001(sftp-user)
```
> ✅ `sftp-user` **không** thuộc nhóm `sudo` → không có quyền đặc quyền.

```
-rw-r----- 1 root sftp-user 45 Oct  7 02:00 /var/log/app-backup/backup-check.log
```

📸 **Screenshot kết quả lệnh kiểm tra:**


---

### Bước 5 – Kết nối SFTP từ Windows bằng Bitvise SSH Client

#### 5.1 Cấu hình kết nối

Mở **Bitvise SSH Client**, điền các thông tin sau vào tab **Login**:

| Trường | Giá trị |
|--------|---------|
| **Host** | `<IP_của_VPS>` |
| **Port** | `22` |
| **Username** | `sftp-user` |
| **Initial method** | `password` |
| **Password** | *(mật khẩu đã tạo ở Bước 1)* |

Nhấn nút **Log in** để kết nối.

📸 **Screenshot giao diện đăng nhập Bitvise:**


---

#### 5.2 Mở cửa sổ SFTP

Sau khi kết nối thành công, nhấn **New SFTP Window**.

- **Khung trái (Local files):** Điều hướng đến thư mục lưu trữ trên máy Windows (ví dụ: `C:\Downloads\sftp-logs\`).
- **Khung phải (Remote files):** Điều hướng đến `/var/log/app-backup/`.

📸 **Screenshot cửa sổ SFTP đã kết nối:**



---

#### 5.3 Tải file về máy Windows

Tìm file `backup-check.log` ở khung **Remote files** (phải), kéo thả sang khung **Local files** (trái).

📸 **Screenshot sau khi tải file thành công:**


---

### Bước 6 – Kiểm tra nội dung file đã tải về

Mở file `backup-check.log` trên Windows bằng Notepad hoặc VS Code:

```
Backup status: SUCCESS at Wed Oct  7 02:00:00 UTC 2026
```

> ✅ Nội dung **khớp hoàn toàn** với file gốc trên VPS.

📸 **Screenshot nội dung file đã tải:**


---

## Kết Quả Kiểm Tra

| Tiêu chí | Kết quả |
|----------|---------|
| `sftp-user` tồn tại và không thuộc nhóm `sudo` | ✅ Đạt |
| Thư mục `/var/log/app-backup/` tồn tại | ✅ Đạt |
| File `backup-check.log` có quyền `640` | ✅ Đạt |
| Kết nối SFTP từ Windows thành công | ✅ Đạt |
| File log được tải về máy Windows | ✅ Đạt |
| Nội dung file tải về khớp với file gốc | ✅ Đạt |

---

## Tổng Kết

Bài thực hành đã hoàn thành đầy đủ các yêu cầu:

1. **Tạo tài khoản giới hạn** `sftp-user` – không có quyền `sudo`, chỉ có thể đọc file log được chỉ định.
2. **Cấp quyền an toàn** theo nguyên tắc _Least Privilege_ (quyền tối thiểu cần thiết).
3. **Kết nối và tải file** thành công qua giao thức SFTP bằng giao diện đồ họa trên Windows.
4. **Xác minh tính toàn vẹn** của dữ liệu – nội dung file tải về khớp với file gốc trên VPS.

> 💡 **Ghi chú bảo mật:** Trong môi trường thực tế, nên sử dụng **SSH Key Authentication** thay vì mật khẩu để tăng tính bảo mật cho kết nối SFTP.

---

