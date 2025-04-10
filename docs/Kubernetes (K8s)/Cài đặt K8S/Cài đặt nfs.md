---
comments: true
---
# Cài đặt NFS server

## 1. Cài đặt NFS Server

```bash 
sudo apt update
sudo apt install -y nfs-kernel-server
```



## 2. Tạo thư mục chia sẻ

```bash 
sudo mkdir -p /mnt/nfs_share
sudo chown nobody:nogroup /mnt/nfs_share
sudo chmod 777 /mnt/nfs_share
```



## 3. Cấu hình NFS Export

```bash 
sudo nano /etc/exports
```

```bash 
/mnt/nfs_share *(rw,sync,no_root_squash,no_subtree_check)
```

!!! note "Lưu ý"
    
    - Cho phép tất cả các client truy cập (có thể thay thế bằng 192.168.1.0/24 để chỉ cho phép subnet 192.168.1.x).<br>
    - rw → Cho phép đọc/ghi.<br>
    - sync → Đồng bộ hóa dữ liệu ngay khi ghi.<br>
    - no_root_squash → Cho phép root trên client có quyền root trên NFS server.<br>
    - no_subtree_check → Tránh kiểm tra toàn bộ thư mục con để tăng hiệu suất.<br>
    - Lưu và thoát (Ctrl + X, nhấn Y, rồi Enter).


## 4. Áp dụng cấu hình và khởi động dịch vụ

Chạy lệnh sau để tải lại cấu hình:


```bash 
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```

Kiểm tra trạng thái:

```bash 
sudo systemctl status nfs-kernel-server
```

## 5. Mở firewall (nếu cần)

```bash 
sudo ufw allow from 192.168.1.0/24 to any port 2049
sudo ufw reload
```

