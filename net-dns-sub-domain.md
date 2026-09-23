Nhiệm vụ 1:Làm trên máy server

Bước 1: Cài BIND9
```bash
sudo apt update && sudo apt install -y bind9 bind9utils
```
```bash
named -v
```
Ra dòng BIND 9.x.x ... là cài thành công.

Bước 2: Khởi động và bật tự chạy
```bash
sudo systemctl start bind9
```
```bash
sudo systemctl enable named
```
```bash
sudo systemctl status bind9
```
Phải thấy active (running), bấm q để thoát. Nếu enable named báo lỗi alias, dùng sudo systemctl enable bind9.

Bước 3: Khai báo zone
```bash
sudo nano /etc/bind/named.conf.local
```
Kéo xuống cuối tệp và dán:
```bash
zone "lab.local" {
    type master;
    file "/etc/bind/db.lab.local";
    allow-update { none; };
};
```
Lưu và thoát: Ctrl+O, Enter, Ctrl+X

Bước 4: Tạo file zone
```bash
sudo nano /etc/bind/db.lab.local
```
Dán nội dung sau
```bash
$TTL    604800
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                          2         ; Serial
                     604800         ; Refresh
                      86400         ; Retry
                    2419200         ; Expire
                     604800 )       ; Negative Cache TTL

; Cấu hình máy chủ tên miền gốc của zone
@       IN      NS      ns1.lab.local.

; Bản ghi địa chỉ A trỏ tên máy chủ ns1 và server về IP của Server
ns1     IN      A       192.168.14.10
server  IN      A       192.168.14.10
```
Lưu và thoát: Ctrl+O, Enter, Ctrl+X.
Bước 5: Kiểm tra cú pháp và xuất file chấm điểm

Trước hết về thư mục cá nhân, vì đề yêu cầu file check_config nằm ở thư mục gốc của tài khoản:
```bash
cd ~
```
```bash
sudo named-checkconf
```
```bash
sudo named-checkzone lab.local /etc/bind/db.lab.local > check_config
```
```bash
cat check_config
```
named-checkconf không in gì là đúng.

cat check_config phải hiện hai dòng:
```bash
zone lab.local/IN: loaded serial 2
OK
```
Không chạy lệnh này khi đang đứng trong /etc/bind, vì dấu > do shell của bạn (không phải sudo) ghi file, nên sẽ báo Permission denied.

Đó là lý do phải cd ~ trước. Kiểm tra file đã có chưa bằng ls ~/check_config.
Nhiệm vụ 2:Bước này làm trên máy server.

Bước 1: Mở tệp zone
```bash
sudo nano /etc/bind/db.lab.local
```
Bước 2: Thêm bản ghi api

Kéo xuống dòng cuối cùng của tệp (dùng Ctrl+End), xuống một dòng mới rồi gõ hoặc dán:
```bash
api     IN      A       192.168.14.11\
Sau khi thêm, cuối tệp sẽ trông như sau:
ns1     IN      A       192.168.14.10
server  IN      A       192.168.14.10
api     IN      A       192.168.14.11
```
Lưu và thoát: Ctrl+O, Enter, Ctrl+X.
Bước 3: In nội dung tệp
```bash
cat /etc/bind/db.lab.local
```
Phải thấy đủ ba bản ghi A: ns1, server, api. Bước này quan trọng vì đề dùng lịch sử lệnh để chấm, nên đừng bỏ qua.

Nhiệm vụ 3:Làm trên máy server
Bước 1: Mở file zone
```bash
sudo nano /etc/bind/db.lab.local
```
Bước 2: Thêm bản ghi A cho shop
Thêm vào cuối file (sau dòng api đã thêm ở Nhiệm vụ 2):
```bash
shop    IN      A       192.168.14.12
```
File hoàn chỉnh lúc này:
```bash
$TTL    604800
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                          2         ; Serial
                     604800         ; Refresh
                      86400         ; Retry
                    2419200         ; Expire
                     604800 )       ; Negative Cache TTL

@       IN      NS      ns1.lab.local.

ns1     IN      A       192.168.14.10
server  IN      A       192.168.14.10
api     IN      A       192.168.14.11
shop    IN      A       192.168.14.12
```
Bước 3: In nội dung ra terminal
```bash
cat /etc/bind/db.lab.local
```
Kết quả phải thấy đầy đủ cả 4 bản ghi A: ns1, server, api, shop.

Nhiệm vụ 4:Làm trên server
Bước 1: Mở file zone
```bash
sudo nano /etc/bind/db.lab.local
```
Bước 2: Thêm bản ghi A cho mail và bản ghi MX
Thêm vào cuối file (sau dòng shop):
```bash
mail    IN      A       192.168.14.13
@       IN      MX      10 mail.lab.local.
```
File hoàn chỉnh lúc này:
```bash
$TTL    604800
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                          2         ; Serial
                     604800         ; Refresh
                      86400         ; Retry
                    2419200         ; Expire
                     604800 )       ; Negative Cache TTL

@       IN      NS      ns1.lab.local.
@       IN      MX      10 mail.lab.local.

ns1     IN      A       192.168.14.10
server  IN      A       192.168.14.10
api     IN      A       192.168.14.11
shop    IN      A       192.168.14.12
mail    IN      A       192.168.14.13
```
Bước 3: Kiểm tra và in nội dung
```bash
sudo named-checkzone lab.local /etc/bind/db.lab.local
```
```bash
cat /etc/bind/db.lab.local
```
Kết quả named-checkzone phải ra:
```bash
zone lab.local/IN: loaded serial X
OK
```
Bước 4: Khởi động lại dịch vụ
```bash
sudo systemctl restart bind9
```
Nhiệm vụ 5:Làm trên client

Bước 1: Cập nhật gói
```bash
sudo apt update
```
Bước 2: Cấu hình resolv.conf
```bash
sudo nano /etc/resolv.conf
```
Tìm và sửa dòng cấu hình nameserver hoặc thêm dòng mới lên trên đầu:
```bash
nameserver 192.168.14.10
```
Lưu và đóng tệp tin.

Bước 3: Kiểm tra bằng dig
```bash
dig server.lab.local
```
Xác nhận kết quả trả về hiển thị chính xác địa chỉ IP 192.168.14.10 của máy chủ DNS BIND9.

Bước 4: Thực hiện xác thực phân giải subdomain api.lab.local.
```bash
ping -c 3 api.lab.local
```
Nhiệm vụ 6 Trên client

Bước 1: Chạy lệnh ping tới subdomain shop.lab.local.
```bash
ping -c 3 shop.lab.local
```
Bước 2: Xác nhận IP phân giải

Nhìn vào dòng đầu tiên của kết quả:
```bash
PING shop.lab.local (192.168.14.12) 56(84) bytes of data.
```
Nếu IP trong ngoặc đúng là 192.168.14.12 → phân giải DNS đã chính xác.
Nhiệm vụ 7 Tiếp tục trên terminal client

Bước 1: Ping subdomain mail
```bash
ping -c 3 mail.lab.local
```
Quan sát màn hình terminal để đảm bảo kết quả phân giải trả về địa chỉ IP chính xác là 192.168.14.13.

Bước 2: Sử dụng công cụ dig để truy vấn chi tiết bản ghi MX của tên miền.
```bash
dig lab.local MX
```
Xác nhận phần ANSWER SECTION của kết quả trả về hiển thị dòng bản ghi MX trỏ về mail.lab.local. với độ ưu tiên là 10.
