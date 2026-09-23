Nhiệm vụ 1:
Đọc nội dung tệp tin cấu hình người dùng /etc/passwd để hiểu cấu trúc danh sách tài khoản:
```bash
sudo cat /etc/passwd
```
Đọc nội dung tệp mật khẩu mã hóa /etc/shadow:
```bash
sudo cat /etc/shadow
```
Sử dụng lệnh grep để tìm thông tin tài khoản root trong tệp /etc/shadow và lưu kết quả vào tệp log để hệ thống đánh giá:
```bash
sudo grep "^root:" /etc/shadow | tee lin.stdout
```
Nhiệm vụ 2:
Tạo một nhóm người dùng mới có tên là group2 trước khi khởi tạo người dùng:
sudo groupadd group2
Tạo 5 người dùng mới từ user2 đến user6:
```bash
sudo useradd -m user2
sudo useradd -m user3
sudo useradd -m user4
sudo useradd -m user5
sudo useradd -m user6
```
Đặt mật khẩu cho cả 5 user
```bash
sudo passwd user2
sudo passwd user3
sudo passwd user4
sudo passwd user5
sudo passwd user6
```
Thêm cả 5 user vào group2
```bash
sudo usermod -aG group2 user2
sudo usermod -aG group2 user3
sudo usermod -aG group2 user4
sudo usermod -aG group2 user5
sudo usermod -aG group2 user6
```
```bash
getent group group2 | tee -a lin.stdout
```
Nhiệm vụ 3:Tạo một thư mục mới có tên là shared_dir nằm trong thư mục gốc của người dùng user2:
```bash
sudo mkdir -p /home/user2/shared_dir
```
Thay đổi nhóm sở hữu của thư mục vừa tạo thành nhóm group2:
```bash
sudo chgrp group2 /home/user2/shared_dir
```
Thiết lập quyền 770:
```bash
sudo chmod 770 /home/user2/shared_dir
```
Kiểm tra:
```bash
ls -ld /home/user2/shared_dir | tee -a lin.stdout
```
Nhiệm vụ 4:
Tiến hành cài đặt gói module kiểm soát độ phức tạp mật khẩu libpam-pwquality:
```bash
sudo apt-get update
```
```bash
sudo apt-get install libpam-pwquality -y
```
Mở file cấu hình PAM cho mật khẩu
```bash
sudo nano /etc/pam.d/common-password
```
Thêm/sửa dòng cấu hình

Tìm xem file đã có sẵn dòng nào chứa pam_pwquality.so hay chưa (thường có sẵn 1 dòng mặc định dạng password requisite pam_pwquality.so retry=3).

Nếu đã có sẵn → sửa lại thành đúng dòng yêu cầu:
```bash
password requisite pam_pwquality.so retry=3 minlen=8 dcredit=-1 ucredit=-1 ocredit=-1 lcredit=-1
```
Kiểm tra:
```bash
grep "pam_pwquality.so" /etc/pam.d/common-password | tee -a lin.stdout
```
Nhiệm vụ 5:
Chạy lệnh đổi mật khẩu và ghi log:
```bash
sudo passwd user2 2>&1 | tee -a lin.stdout
```
Nhập thử mật khẩu yếu trước, sau đó mật khẩu đúng chuẩn
Khi hệ thống hỏi:

New password:

a) Thử mật khẩu yếu trước (để hệ thống từ chối, chứng minh chính sách hoạt động), ví dụ:

abc123

→ Hệ thống sẽ báo lỗi tương tự:

BAD PASSWORD: The password is shorter than 8 characters

hoặc

BAD PASSWORD: The password fails the dictionary check - it does not contain enough character classes

Nó sẽ hỏi lại New password: (tối đa 3 lần theo retry=3 đã cấu hình).

b) Sau đó nhập mật khẩu đạt chuẩn (đủ 4 loại ký tự: chữ hoa, chữ thường, số, ký tự đặc biệt, tối thiểu 8 ký tự), ví dụ:

Abcd123!

Nhập lại 1 lần nữa để xác nhận (Retype new password:) → cùng giá trị Abcd123!

→ Kết quả cuối:

passwd: password updated successfully

Kiểm tra:
```bash
cat lin.stdout
```
