Nhiệm vụ 1:
Tại 1 terminal nhập:
```bash
ls -la /etc/pam.d/
```
```bash
cat /etc/pam.d/common-auth
```
```bash
sudo tail /var/log/auth.log
```
Nhiệm vụ 2:
Tiếp tục tại terminal đó nhập
```bash
sudo passwd user1
```
Nó sẽ yêu cầu mk mới, nhập mk bất kì.
Sau đó nhập tiếp:
```bash
sudo nano /etc/pam.d/common-auth
```
Terminal sẽ mở cấu hình xác thực:

Ở đây tìm dòng cấu hình sử dụng "module pam_unix.so" đang bị chú thích và tiến hành xóa dấu # ở đầu dòng:

auth    [success=1 default=ignore]    pam_unix.so nullok_secure

Sau đó ctrl O để lưu và ctrl X để exit về.

Nhập tiếp:
```bash
su user1
```
Yêu cầu mk là mk mình vừa đặt ở trên, sau khi đăng nhập dc nhập "exit" để thoát,rồi nhập tiếp 
```bash
sudo tail /var/log/auth.log
```
rồi exit

Nhiệm vụ 3:
Nhập:
```bash
sudo nano /etc/pam.d/common-auth
```
Hiện thị cấu hình để chỉnh sửa, di chuyển đến dòng có cấu hình "pam_unix.so" ấn xuống dòng và tiến hành nhập:
```bash
auth    required    pam_tally2.so deny=3 unlock_time=300
```
đảm bảo sao cho cấu hình "pam_tally2.so" nằm phía trên cấu hình "pam_unix.so".Sau đó ctrl O để lưu và ctrl x để thoát

Nhập:
```bash
sudo nano /etc/pam.d/common-account
```
Tiếp tục sửa như trên nhưng nhập câu lệnh:
```bash
account    required    pam_tally2.so
```
đảm bảo sao cho cấu hình "pam_tally2.so" nằm phía trên cấu hình "pam_unix.so".Sau đó ctrl O để lưu và ctrl x để thoát

Tiếp tục nhập:
```bash
su user1
```
Nhập sai mk 3 lần để khoá tài khoản sau đó nhập lệnh sau để mở
```bash
sudo pam_tally2 --user user1 --reset
```






