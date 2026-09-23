Nhiệm vụ 1:Sử dụng lệnh useradd để tạo 2 người dùng mới là user1 và user2:
```bash
sudo useradd -m user1
```
```bash
sudo useradd -m user2
```
Thiết lập mật khẩu cho 2 người dùng vừa tạo:
```bash
sudo passwd user1
```
```bash
sudo passwd user2
```
Kiểm tra trạng thái khởi tạo thành công và xuất kết quả ra tệp log chấm điểm:
```bash
getent passwd user1 | tee -a lin.stdout
```
```bash
getent passwd user2 | tee -a lin.stdout
```
Nhiệm vụ 2:Sử dụng lệnh groupadd để khởi tạo nhóm mới có tên là group1:
```bash
sudo groupadd group1
```
Sử dụng lệnh usermod để thêm người dùng user1 vào nhóm group1:
```bash
sudo usermod -aG group1 user1
```
Kiểm tra thông tin nhóm của user1 và ghi nhận log chấm điểm:
```bash
id user1 | grep group1 | tee -a lin.stdout
```
Nhiệm vụ 3: Thực hiện chuyển đổi sang tài khoản người dùng user1:
```bash
su - user1
```
Chạy lệnh xác nhận tài khoản đang hoạt động:
```bash
whoami
```
Tạo một thư mục mới có tên là testdir và tạo một tệp tin myfile.txt bên trong thư mục đó:
```bash
mkdir testdir
```
```bash
touch testdir/myfile.txt
```
Thoát khỏi phiên làm việc của user1 để quay lại tài khoản ban đầu:
exit
Sau đó dùng lệnh để kiểm tra:
```bash
sudo ls -l /home/user1/testdir/myfile.txt | tee -a lin.stdout
```
Nhiệm vụ 4: Mở tệp cấu hình đặc quyền sudoers an toàn bằng lệnh visudo:
```bash
sudo visudo
```
Di chuyển con trỏ xuống cuối tệp tin và thêm dòng cấu hình sau để cho phép user1 chạy toàn bộ các lệnh mà không cần mật khẩu:
```bash
user1 ALL=(ALL) NOPASSWD: ALL
```
Lưu file và thoát.

 Kiểm tra đặc quyền bằng cách đăng nhập vào user1 và thử nghiệm các lệnh mkdir và rm dưới quyền sudo:
```bash
su - user1
```
```bash
sudo mkdir /tmp/testdir_user1
```
```bash
sudo rm -rf /tmp/testdir_user1
```
```bash
echo "thanh cong" | sudo tee -a /home/ubuntu/lin.stdout
```
```bash
exit
```
Nhiệm vụ 5:
Thay đổi chủ sở hữu nhóm và điều chỉnh quyền truy cập của thư mục và tệp tin đã tạo ở nhiệm vụ 3:
```bash
sudo chown -R user1:group1 /home/user1/testdir
```
```bash
sudo chmod 750 /home/user1/testdir
```
```bash
sudo chmod 640 /home/user1/testdir/myfile.txt
```
Đăng nhập vào tài khoản user2 để thực hiện hành vi kiểm thử truy cập trực quan:
```bash
su - user2
```
Thử đọc tệp tin nằm trong thư mục bảo mật của user1 để quan sát thông báo lỗi hiển thị trực tiếp:
```bash
cat /home/user1/testdir/myfile.txt
```
Thoát khỏi phiên đăng nhập của user2 để quay về tài khoản ban đầu:
```bash
exit
```
```bash
sudo -u user2 touch /home/user1/testdir/myfile.txt 2>&1 | tee -a /home/ubuntu/lin.stdout
```
