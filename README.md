# Báo cáo Bài 2: Cấu hình phân quyền Nhóm và sudoers bằng visudo

## 1. Mục tiêu
- Biết cách tạo tài khoản người dùng (User) và nhóm (Group) mới trên Linux.
- Hiểu được cơ chế hoạt động của đặc quyền quản trị sudo và tệp tin cấu hình `/etc/sudoers`.
- Biết cách phân quyền giới hạn cho một nhóm người dùng chỉ được thực hiện một số lệnh hệ thống nhất định (NOPASSWD).

## 2. Quá trình cấu hình

Đầu tiên, em tạo một nhóm người dùng mới tên là `devops-admin`:
```bash
sudo groupadd devops-admin
```

Tiếp theo, tạo tài khoản người dùng `deployer` và thêm trực tiếp vào nhóm `devops-admin`:
```bash
sudo adduser deployer
sudo usermod -aG devops-admin deployer
```

Mở file cấu hình sudoers một cách an toàn bằng lệnh:
```bash
sudo visudo
```

Kéo xuống cuối file và thêm dòng cấu hình sau để cho phép nhóm `devops-admin` có quyền chạy các lệnh quản lý dịch vụ `systemctl` (start, stop, restart, status) mà không cần nhập mật khẩu:
```text
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```
Lưu lại cấu hình và thoát khỏi `visudo`.

## 3. Kết quả kiểm tra
Để kiểm tra quyền đã được thiết lập đúng hay chưa, em chuyển đổi sang tài khoản `deployer`:
```bash
su - deployer
```

Sau đó, em sử dụng lệnh `sudo -l` để liệt kê các quyền sudo của tài khoản này.
**Đầu ra của lệnh `sudo -l` (ảnh minh họa):**

```text
Matching Defaults entries for deployer on server:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User deployer may run the following commands on server:
    (ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

Chạy thử lệnh restart lại dịch vụ hệ thống `cron`:
```bash
sudo systemctl restart cron
```
Lệnh chạy thành công, tiến trình khởi động lại không yêu cầu nhập mật khẩu của user `deployer`, chứng tỏ việc phân quyền đã hoạt động chính xác.
