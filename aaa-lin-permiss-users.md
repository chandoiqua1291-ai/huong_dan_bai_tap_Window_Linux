Nhiệm vụ 1 Nhập:
```bash
ls /etc/passwd /etc/shadow
```
```bash
ls -l /etc/passwd /etc/shadow | tee -a lin.stdout
```
Nhiệm vụ 2 Nhập:
```bash
find / -perm -o=rw -type f -exec ls -l {} + 2>/dev/null > world_writable.txt
```
```bash
cat world_writable.txt | tee -a lin.stdout
```
Nhiệm vụ 3 Nhập:
```bash
openssl passwd -6 -salt mysalt password123 | tee enc_password.txt
```
```bash
cat enc_password.txt | tee -a lin.stdout
```
Sau khi nhập lệnh "cat enc_password.txt | tee -a lin.stdout" sẽ nhận dc 1 đoạn mã, copy đoạn mã đó để làm nvu 4

Nhiệm vụ 4 Nhập:
```bash
sudo nano /etc/shadow
```
Tìm dòng bắt đầu bằng "root:", sau đó xác định dấu ":" tiếp theo, xoá phần bên trong "root:" và dấu ":" đó rồi gán đoạn mã đã copy dc ở nvu 4 vào,rồi lưu và exit
Về lại màn hình terminal chính tiếp tục nhập:
```bash
sudo grep "^root:" /etc/shadow | tee -a lin.stdout
```
```bash
su - root
```
Ở dây sẽ yêu cầu mk chưa mã hoá, nhập mk chưa mã hoá là "password123"

Nhiệm vụ 5 Nhập:
```bash
sudo apt-get update && sudo apt-get install -y inotify-tools
```
Khởi tạo hai tài khoản người dùng kiểm thử user1 và user2 bằng cú pháp 
```bash
sudo useradd -m <tên_người_dùng>
```
Sau khi tạo xong tiếp tục nhập:
```bash
sudo touch /tmp/trap_file.log
sudo chown user1:user2 /tmp/trap_file.log
sudo chmod 640 /tmp/trap_file.log
```
```bash
nano monitor.sh
```
sẽ mở ra file kịch bản, ta nhập đoạn sau rồi lưu lại:
```bash
#!/bin/bash
inotifywait -m -e open /tmp/trap_file.log | while read dir event file; do
    echo "user2 Logged Access: $file was opened" >> /tmp/trap.log
done
```
Tiếp tục nhập:
```bash
chmod +x monitor.sh
```
```bash
sudo ./monitor.sh &
```
```bash
sudo su - user2
```
```bash
cat /tmp/trap_file.log
```
```bash
exit
```
```bash
cat /tmp/trap.log | tee -a lin.stdout
```
