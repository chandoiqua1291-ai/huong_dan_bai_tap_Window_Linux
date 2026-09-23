Nhiệm vụ 1:Nhập:
```bash
sudo setfacl -m u:user1:rwx project
```
```bash
sudo setfacl -m u:user2:r project
```
```bash
sudo setfacl -m u:user3:rx project
```
Để kiểm tra lại,nhập:
```bash
sudo getfacl project
```
Nhiệm vụ 2: Nhập
```bash
sudo setfacl -dR -m u:user1:rwx project
```
```bash
sudo setfacl -dR -m u:user2:r project
```
```bash
sudo setfacl -dR -m u:user3:rx project
```
Để kiểm tra lại,nhập:
```bash
sudo getfacl project
```
Nhiệm vụ 3: Tạo thư mục con và tệp bằng lệnh:
```bash
sudo mkdir project/subdir1
```
```bash
sudo touch project/file
```
Kiểm tra quyền bằng lệnh:
```bash
sudo getfacl project/subdir1 project/file1
```
Nhiệm vụ 4:
Chạy 2 lệnh
```bash
sudo setfacl -m u:user2:rwx project     # cập nhật access ACL hiện tại
```
```bash
sudo setfacl -dR -m u:user2:rwx project    # cập nhật default ACL đệ quy
```
Kiểm tra lại bằng lệnh:
```bash
sudo getfacl project
```
Nhiệm vụ 5:Nhập
```bash
sudo mkdir project/subdir2
```
```bash
sudo touch project/file2
```
Để kiểm tra lại: 
```bash
sudo getfacl project/subdir2 project/file2
```
