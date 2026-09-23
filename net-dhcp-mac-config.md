Nhiệm vụ 1:
Bước 1: Lấy địa chỉ MAC (trên client)
```bash
ip link show eth0
```
Nếu ko lấy dc link thì dùng mã này để lấy 
```bash
cat /sys/class/net/eth0/address
```
Tìm dòng link/ether xx:xx:xx:xx:xx:xx, đó là địa chỉ MAC. Bạn ghi lại hoặc chụp màn hình để dùng ở bước 3.

Bước 2: Cài DHCP Server (trên server)
```bash
sudo apt update
```
```bash
sudo apt-get install -y isc-dhcp-server
```
Cài xong, dịch vụ có thể báo lỗi khi khởi động. Không sao, vì chưa cấu hình.

Bước 3: Cấu hình cấp IP tĩnh theo MAC (trên server)
```bash
sudo nano /etc/dhcp/dhcpd.conf
```
Kéo xuống cuối tệp và thêm khối sau. Thay AA:BB:CC:DD:EE:FF bằng MAC thật của client ở bước 1:
```bash
subnet 20.20.0.0 netmask 255.255.255.0 {
    host client {
        hardware ethernet AA:BB:CC:DD:EE:FF;
        fixed-address 20.20.0.50;
    }
}
```
Khối host client nằm bên trong khối subnet, đúng như gợi ý của đề. Lưu và thoát: Ctrl+O, Enter, Ctrl+X.

Bước 4: Chỉ định giao diện lắng nghe (trên server)
```bash
sudo nano /etc/default/isc-dhcp-server
```
Tìm dòng INTERFACESv4="" và sửa thành:
```bash
INTERFACESv4="eth0"
```
Lưu và thoát như trên.

Bước 5: Khởi động lại và kiểm tra
```bash
sudo systemctl restart isc-dhcp-server
```
```bash
sudo systemctl status isc-dhcp-server
```
Phải thấy active (running). Bấm q để thoát màn hình status.

Nhiệm vụ 2:

Bước này làm trên máy client.

Bước 1: Cài DHCP Client
```bash
sudo apt update
```
```bash
sudo apt-get install -y isc-dhcp-client
```
Phải cài xong trước khi flush IP ở bước 3, vì flush xong có thể mất mạng và apt sẽ không tải được gói.

Bước 2: Kiểm tra phiên bản
```bash
dpkg-query -W isc-dhcp-client
```
Ra dòng kiểu isc-dhcp-client 4.4.x... là cài thành công.

Bước 3: Xóa IP cũ trên eth0
```bash
sudo ip addr flush dev eth0
```
Bước 4: Xin cấp IP và ghi log
```bash
sudo dhclient -v 2>&1 | tee client.stdout
```
Bạn sẽ thấy các dòng DHCPDISCOVER, DHCPOFFER, DHCPREQUEST, DHCPACK, và cuối cùng là bound to 20.20.0.50.

Nếu lệnh không tự thoát, đợi vài giây hoặc bấm Ctrl+C, log vẫn đã được ghi vào client.stdout.

Kiểm tra kết quả
```bash
ip a show eth0
```
Phải thấy inet 20.20.0.50/24 trên eth0.

Nhiệm vụ 3:
Bước 1 và 2 làm trên client, bước 3 làm trên server.

Bước 1: Kiểm tra IP (client)
```bash
ip a show eth0
```
Phải thấy dòng inet 20.20.0.50/24 trên eth0.

Bước 2: Ping server (client)
```bash
ping 20.20.0.5 | tee ping.stdout
```
Bạn sẽ thấy các dòng 64 bytes from 20.20.0.5: icmp_seq=1 ttl=64 time=.... Lệnh này chạy mãi, nên đợi khoảng 4-5 gói rồi bấm Ctrl+C.

Nếu muốn nó tự dừng, dùng ping -c 4 20.20.0.5 | tee ping.stdout, kết quả ghi vào ping.stdout như nhau.

Bước 3: Xem log DHCP (server)
```bash
sudo tail -f /var/log/syslog
```
Nó chạy mãi và chỉ hiện log mới, nên phải bấm Ctrl+C mới thoát được. Khi nó đang chạy, bạn không gõ lệnh khác được.
