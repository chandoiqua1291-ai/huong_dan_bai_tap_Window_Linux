Nhiệm vụ 1:Trên máy chủ, mở tệp cấu hình PAM của dịch vụ SSH:
```bash
sudo nano /etc/pam.d/sshd
```
Cấu hình thêm dòng khai báo module pam_debug.so ở ngay đầu tệp tin:
```bash
auth       required     pam_debug.so
```
Note: cho đoạn mã trên ngay dưới đoạn mã này: # PAM configuration for the Secure Shell service

Lưu và đóng tệp tin cấu hình.

Hiển thị nội dung tệp tin cấu hình để kiểm tra:
```bash
cat /etc/pam.d/sshd
```
Khởi động lại dịch vụ xinetd để áp dụng cấu hình mới.Trong bài lab này, dịch vụ SSH được quản lý gián tiếp qua xinetd thay vì chạy độc lập:
```bash
sudo systemctl restart xinetd
```
Nhiệm vụ 2:Nhập mã sau để tìm ip máy chủ:
```bash
ip a
```
Đọc và tìm đoạn mã có dạng gần giống thế này: inet 172.16.10.2/24 brd 172.16.10.255 scope global eth0 thì phần "172.16.10.2" là ip,lưu ý có eth0 là ip máy chủ
Mở terminal của máy khách và thực hiện đăng nhập SSH thành công bằng mã:
```bash
ssh ubuntu@<IP_máy_chủ>
```
Nhập mật khẩu:
```bash
ubuntu
```
Sau khi đăng nhập gõ exit để thoát.
Sau đó thực hiện đăng nhập lại nhưng gõ sai mk để đăng nhập thất bại:
```bash
ssh ubuntu@<IP_máy_chủ>
```
sau đó quay lại máy chủ để nhập
```bash
sudo tail -n 20 /var/log/auth.log
```
Nhiệm vụ 3:
Nhập mã sau trên máy chủ để mở tệp mã nguồn có sẵn để tiến hành chỉnh sửa:
```bash
sudo nano /etc/pam.d/pam_custom.c
```
Sau đó

Thay thế chuỗi điều kiện kiểm tra người dùng strcmp(user, "admin") == 0 thành strcmp(user, "ubuntu") == 0.

Thay đổi thông điệp ghi log khi thành công từ "Admin login detected: %s" thành "login detected: %s".

Gợi ý: Trong nano, bạn có thể dùng Ctrl + W để tìm nhanh chuỗi strcmp hoặc Admin login rồi sửa tay, sau đó lưu và thoát

Sau đó nhập:
```bash
cd /etc/pam.d
```
```bash
sudo gcc -fPIC -shared -o pam_custom.so pam_custom.c -lpam
```
```bash
sudo cp pam_custom.so /lib/security/pam_custom.so
```
Nhiệm vụ 4:Trên máy chủ, mở tệp cấu hình PAM của dịch vụ SSH:
```bash
sudo nano /etc/pam.d/sshd
```
Thay thế dòng cấu hình module pam_debug.so đã tạo ở nhiệm vụ 1 bằng module tùy chỉnh:
```bash
auth       required     pam_custom.so
```
Lưu và đóng tệp tin cấu hình.

Kiểm tra lại bằng:
```bash
cat /etc/pam.d/sshd
```
Rồi khởi động lại dịch vụ xinetd để áp dụng cấu hình mới:
```bash
sudo systemctl restart xinetd
```
Nhiệm vụ 5:
Từ máy khách, thực hiện đăng nhập SSH bằng tài khoản hợp lệ ubuntu để kiểm thử trường hợp thành công:
```bash
ssh ubuntu@<IP_máy_chủ>
```
Nhập mk "ubuntu" rồi exit

Note ip máy chủ có thể xác đinh bằng mã: ip a

Từ máy khách, thực hiện đăng nhập SSH bằng tài khoản không hợp lệ để kiểm thử trường hợp thất bại:
```bash
ssh admin@<IP_máy_chủ>
```
Nhập mật khẩu bất kỳ và quan sát kết quả bị từ chối truy cập.

Trên máy chủ, chạy lệnh in log xác thực:
```bash
sudo tail -n 20 /var/log/auth.log
```
