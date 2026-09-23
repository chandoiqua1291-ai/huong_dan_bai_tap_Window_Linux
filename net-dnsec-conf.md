Nhiệm vụ 1:
Bước 1: Cài BIND9 (server)
```bash
sudo apt update && sudo apt install -y bind9 bind9utils
```
Xác định thành công
```bash
named -v
```
Ra dòng BIND 9.x.x ... là cài thành công.

Bước 2: Khởi động và bật tự chạy (server)
```bash
sudo systemctl start bind9
```
```bash
sudo systemctl enable named
```
```bash
sudo systemctl status bind9
```
Phải thấy active (running), bấm q để thoát. Nếu enable named báo lỗi alias, dùng: 
```bash
sudo systemctl enable bind9
```
Bước 3: Khai báo zone (server)
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
Lưu: Ctrl+O, Enter, Ctrl+X.
Bước 4: Tạo file zone (server)
```bash
sudo nano /etc/bind/db.lab.local
```
Dán nội dung sau:
```bash
$TTL    604800
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                          2         ; Serial
                     604800         ; Refresh
                      86400         ; Retry
                    2419200         ; Expire
                     604800 )       ; Negative Cache TTL

; Cấu hình máy chủ tên miền gốc
@       IN      NS      ns1.lab.local.

; Các bản ghi địa chỉ A trỏ về IP tĩnh
ns1     IN      A       192.168.12.10
server  IN      A       192.168.12.30
```
Lưu và thoát: Ctrl+O, Enter, Ctrl+X.
Bước 5: Kiểm tra cú pháp và restart
```bash
sudo named-checkconf
```
```bash
sudo named-checkzone lab.local /etc/bind/db.lab.local
```
```bash
sudo systemctl restart bind9
```
named-checkconf không in gì là đúng.

named-checkzone phải in loaded serial 2 và OK.

Trên máy client
Bước 6: Cài dnsutils và trỏ DNS
```bash
sudo apt update && sudo apt install -y dnsutils
```
```bash
sudo nano /etc/resolv.conf
```
Thêm hoặc sửa thành:
```bash
nameserver 192.168.12.10
```
Lưu và thoát, rồi chạy:
```bash
dig server.lab.local
```
Kết quả đúng phải có:

status: NOERROR

Trong ANSWER SECTION: server.lab.local. 604800 IN A 192.168.12.30

Nhiệm vụ 2:
Bước này làm trên máy server.

Bước 1: Vào thư mục BIND9
```bash
cd /etc/bind
```
Bước 2: Tạo khóa Zone Signing Key ZSK.
```bash
sudo dnssec-keygen -a RSASHA256 -b 2048 -n ZONE lab.local
```
Hệ thống sẽ sinh ra hai tệp khóa dạng Klab.local.+008+[ZSK_ID].key và Klab.local.+008+[ZSK_ID].private. Hãy ghi nhớ mã số định danh ZSK_ID này.
Bước 3: Tạo khóa Key Signing Key KSK.
```bash
sudo dnssec-keygen -a RSASHA256 -b 4096 -n ZONE -f KSK lab.local
```
Hệ thống sẽ sinh ra hai tệp khóa dạng Klab.local.+008+[KSK_ID].key và Klab.local.+008+[KSK_ID].private. Hãy ghi nhớ mã số định danh KSK_ID này.

Bước 4: Chèn các khóa DNSKEY đã tạo vào tệp zone.
```bash
sudo nano /etc/bind/db.lab.local
```
Thêm hai dòng chỉ thị chèn tệp khóa tương ứng vào cuối tệp tin, thay thế [ZSK_ID] và [KSK_ID] bằng mã số định danh thực tế vừa tạo ở các bước trên:
```bash
$INCLUDE "/etc/bind/Klab.local.+008+12345.key"
$INCLUDE "/etc/bind/Klab.local.+008+67890.key"
```
Chỉ dùng tệp .key, không dùng tệp .private. Lưu và thoát: Ctrl+O, Enter, Ctrl+X.
Bước 5: Hiển thị nội dung tệp
```bash
cat /etc/bind/db.lab.local
```
Phải thấy nội dung zone cũ (SOA, NS, hai bản ghi A) và hai dòng $INCLUDE ở cuối.

Nhiệm vụ 3:

Bước này làm trên máy server.

Bước 1: Ký zone

Trước tiên vào thư mục BIND9 và xem lại ID khóa (đã tạo ở nhiệm vụ 2):
```bash
cd /etc/bind
```
```bash
grep DNSKEY /etc/bind/Klab.local*.key
```bash
Dòng có số 257 là KSK, dòng có số 256 là ZSK. Tên tệp .key tương ứng cho biết ID. Hoặc xem nhanh bằng ls /etc/bind/Klab.local*.key.
Chạy lệnh ký, thay [KSK_ID] và [ZSK_ID] bằng số thật
sudo dnssec-signzone -o lab.local -k /etc/bind/Klab.local.+008+[KSK_ID].key /etc/bind/db.lab.local /etc/bind/Klab.local.+008+[ZSK_ID].key
Thành công sẽ thấy các dòng như Verifying the zone using the following algorithms: RSASHA256 và Zone fully signed, cuối cùng là db.lab.local.signed.

Kiểm tra tệp đã được tạo:
```bash
ls /etc/bind/db.lab.local.signed
```
Bước 2: Sửa named.conf.local
```bash
sudo nano /etc/bind/named.conf.local
```
Sửa dòng file thành db.lab.local.signed, để khối zone như sau:
```bash
zone "lab.local" {
    type master;
    file "/etc/bind/db.lab.local.signed";
    allow-update { none; };
};
```
Lưu và thoát: Ctrl+O, Enter, Ctrl+X.
Bước 3: Hiển thị nội dung tệp cấu hình
```bash
cat /etc/bind/named.conf.local
```
Phải thấy file "/etc/bind/db.lab.local.signed";.

Nhiệm vụ 4:
Bước này làm trên máy server.

Bước 1: Kiểm tra cú pháp và xuất file
```bash
sudo named-checkconf
```
```bash
sudo named-checkzone lab.local /etc/bind/db.lab.local.signed > ~/config_check
```
```bash
cat ~/config_check
```
named-checkconf không in gì là đúng.

cat ~/config_check phải hiện dòng zone lab.local/IN: loaded serial ... và dòng OK ở cuối.
Bước 2: Restart BIND9
```bash
sudo systemctl restart bind9
```
```bash
sudo systemctl status bind9
```
Phải thấy active (running), bấm q để thoát.

Nhiệm vụ 5:

Bước này làm trên máy client.

Bước 1: Truy vấn kèm DNSSEC
```bash
dig ns1.lab.local +dnssec
```
Nếu resolv.conf của client vẫn trỏ nameserver 192.168.12.10 (đã làm ở nhiệm vụ 1) thì lệnh này chạy được ngay.

Chưa chắc thì kiểm tra bằng cat /etc/resolv.conf, hoặc chỉ định thẳng server:
```bash
dig ns1.lab.local +dnssec @192.168.12.10
```
Bước 2: Xác nhận RRSIG

Trong kết quả, ở ANSWER SECTION bạn cần thấy hai dòng:
```bash
ns1.lab.local.   604800  IN  A      192.168.12.10
ns1.lab.local.   604800  IN  RRSIG  A 8 3 604800 ... lab.local. (chuỗi chữ ký dài)
```
Để lọc nhanh xem có RRSIG hay không:
```bash
dig ns1.lab.local +dnssec | grep RRSIG
```

