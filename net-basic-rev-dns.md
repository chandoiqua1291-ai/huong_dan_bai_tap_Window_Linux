Nhiệm vụ 1:
Trên máy server, cài đặt BIND9 và bộ công cụ tiện ích. Chạy lệnh sau để cập nhật danh sách gói và cài đặt.
```bash
sudo apt update && sudo apt install bind9 bind9utils -y
```
Kiểm tra phiên bản BIND để xác nhận cài đặt thành công.
```bash
named -v
```
Ra dòng kiểu BIND 9.x.x ... là cài thành công.
Khởi động và bật tự chạy
```bash
sudo systemctl start bind9
```
```bash
sudo systemctl enable named
```
Nếu lệnh enable named báo lỗi kiểu Refusing to operate on alias name, dùng lệnh sau thay thế, hai tên này là một dịch vụ.
```bash
sudo systemctl enable bind9
```
Kiểm tra trạng thái dịch vụ, đảm bảo hiển thị active (running).
```bash
sudo systemctl status bind9
```
Khai báo zone
```bash
sudo nano /etc/bind/named.conf.local
```
Kéo xuống cuối tệp và dán đoạn này vào (trong nano có thể dùng chuột phải hoặc Ctrl+Shift+V để dán):
```bash
// Zone thuận – phân giải hostname → IP
zone "lab.local" {
    type master;
    file "/etc/bind/db.lab.local";
    allow-update { none; };
};

// Zone ngược – phân giải IP → hostname (Reverse DNS)
zone "11.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.11.168.192";
    allow-update { none; };
};
```
Kiểm tra lại:
```bash
cat /etc/bind/named.conf.local
```
Nhiệm vụ 2:
Tạo file zone thuận
```bash
sudo nano /etc/bind/db.lab.local
```
Nano mở ra một màn hình trống. Dán nội dung sau vào (chuột phải hoặc Ctrl+Shift+V):
```bash
$TTL    604800
; Khai báo máy chủ có thẩm quyền (SOA) và các thông số đồng bộ
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                          2         ; Serial
                     604800         ; Refresh
                      86400         ; Retry
                    2419200         ; Expire
                     604800 )       ; Negative Cache TTL
;
; Khai báo Name Server của zone
@       IN      NS      ns1.lab.local.
; Bản ghi A – địa chỉ của Name Server
ns1     IN      A       192.168.11.10
; Bản ghi A – máy chủ nội bộ
server  IN      A       192.168.11.30
```
Lưu và thoát

Tạo file zone ngược
```bash
sudo nano /etc/bind/db.11.168.192
```
```bash
$TTL    604800
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                          2
                     604800
                      86400
                    2419200
                     604800 )
;
@       IN      NS      ns1.lab.local.
; Bản ghi PTR – octet cuối của IP → hostname đầy đủ (FQDN)
10      IN      PTR     ns1.lab.local.
30      IN      PTR     server.lab.local.
```
Lưu và thoát

Kiểm tra nội dung file zone ngược
```bash
cat /etc/bind/db.11.168.192
```
Kiểm tra cú pháp
```bash
sudo named-checkconf
```
```bash
sudo named-checkzone lab.local /etc/bind/db.lab.local
```
```bash
sudo named-checkzone 11.168.192.in-addr.arpa /etc/bind/db.11.168.192
```
named-checkconf không in gì là đúng. Hai lệnh named-checkzone phải in loaded serial 2 và OK.

Khởi động lại BIND9
```bash
sudo systemctl restart bind9
```
```bash
sudo systemctl status bind9
```
Thấy active (running) là xong, bấm q để thoát.
Nhiệm vụ 3:
Làm trên terminal của máy client nhé.
Cài dnsutils
```bash
sudo apt update
```
```bash
sudo apt install dnsutils -y
```
Trỏ client về DNS Server
```bash
sudo nano /etc/resolv.conf
```
Trong nano, thêm (hoặc sửa dòng nameserver đang có thành) dòng này:
nameserver 192.168.11.10
Lưu và thoát.Sau đó kiểm tra lại:
```bash
cat /etc/resolv.conf
```
Phải thấy dòng nameserver 192.168.11.10.
```bash
dig -x 192.168.11.30
```
Trong phần ANSWER SECTION của kết quả, xác nhận xuất hiện dòng:
```bash
30.11.168.192.in-addr.arpa.  604800  IN  PTR  server.lab.local.
```
