---
comments: true
---
# Cài đặt Redis trên k8s

## 1. Giới thiệu

Redis (Remote Dictionary Server) là một kho lưu trữ dữ liệu mã nguồn mở hoạt động hoàn toàn trong bộ nhớ (in-memory store). Được phát triển bởi Salvatore Sanfilippo vào năm 2009, Redis đã trở thành một trong những cơ sở dữ liệu NoSQL phổ biến nhất hiện nay, nổi bật với tốc độ cực nhanh và độ trễ thấp, khiến nó trở thành sự lựa chọn tuyệt vời cho nhiều ứng dụng và tình huống cần xử lý dữ liệu theo thời gian thực (real-time application).

Trong dự án này mình sẽ dùng redis để thực hiện caching dữ liệu.

## 2. Các mô hình caching 
### 2.1. So sánh các mô hình Redis Caching

| **Mô Hình**        | **Cách Hoạt Động**                                                            | **Ưu Điểm**                                                          | **Nhược Điểm**                                                        | **Hiệu Suất Đọc** | **Hiệu Suất Ghi** | **Độ Phức Tạp** | **Tính Nhất Quán**       |
|--------------------|--------------------------------------------------------------------------------|-----------------------------------------------------------------------|------------------------------------------------------------------------|--------------------|--------------------|------------------|--------------------------|
| **Cache-Aside**     | Ứng dụng kiểm tra cache, nếu miss thì truy DB và ghi lại cache                | Đơn giản, chỉ cache khi cần thiết                                     | Cache miss gây trễ, phải xử lý đồng bộ khi dữ liệu thay đổi            | Cao                | Trung Bình         | Thấp             | Trung Bình               |
| **Write-Through**   | Ghi vào cache trước, sau đó ghi DB                                            | Đọc nhanh, cache luôn đồng bộ                                         | Ghi chậm vì phải ghi cả cache và DB                                    | Cao                | Thấp               | Trung Bình       | Cao                      |
| **Write-Behind**    | Ghi cache trước, ghi DB sau (batch hoặc async)                                | Ghi cực nhanh, giảm tải DB                                            | Nguy cơ mất dữ liệu nếu Redis sập, đồng bộ phức tạp                     | Cao                | Rất Cao            | Cao              | Thấp → Trung Bình        |
| **Read-Through**    | Ứng dụng luôn gọi cache, Redis tự truy DB nếu cache miss                      | Logic tập trung, đơn giản phía ứng dụng                               | Redis cần hỗ trợ truy DB, không linh hoạt với nhiều nguồn dữ liệu      | Cao                | Trung Bình         | Trung Bình       | Trung Bình               |
| **Cache Invalidation** | Xóa hoặc cập nhật cache khi DB thay đổi                                    | Giảm nguy cơ dữ liệu cũ, thích hợp TTL và event-based                 | Khó đồng bộ khi nhiều nguồn cập nhật dữ liệu                            | Cao                | Trung Bình         | Trung Bình       | Cao (nếu kiểm soát tốt)  |
| **Full Page Cache** | Cache toàn bộ HTML/JSON response cho endpoint                                 | Phản hồi cực nhanh, giảm tải backend                                  | Không phù hợp với dữ liệu thay đổi thường xuyên                         | Rất Cao            | Không áp dụng      | Thấp             | Thấp → Trung Bình        |
| **Hybrid Cache**    | Kết hợp nhiều mô hình (VD: Read-Through + Write-Behind)                      | Cân bằng hiệu suất và độ tin cậy                                      | Cấu hình phức tạp, khó bảo trì                                         | Rất Cao            | Rất Cao            | Rất Cao          | Tuỳ vào cách kết hợp     |

---

### 2.2. Gợi ý lựa chọn theo tình huống

- **CRUD phổ thông (blog, tin tức):** `Cache-Aside` + TTL
- **Ứng dụng đọc nhiều (e-commerce, search):** `Write-Through` hoặc `Read-Through`
- **Ứng dụng real-time, tốc độ cao (game, IoT):** `Write-Behind`
- **Trang tĩnh, ít thay đổi (landing page):** `Full Page Cache`

---

## 3. Cài đặt thủ công trên k8s

### 3.1. Tạo PV và PVC 


```yaml title="redis-pv.yaml"  linenums="1"
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-pv-volume
  namespace: dev
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /mnt/nfs_share/redis
    server: 192.168.1.110  # Thay bằng IP NFS Server
    readOnly: false
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-pv-claim
  namespace: dev
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi


```

### 3.2. Cài đặt Deployment và Service


```yaml title="redis-dp.yaml"  linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7.4.0
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-data
          mountPath: /data
      volumes:
      - name: redis-data
        persistentVolumeClaim:
          claimName: redis-pv-claim
---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: dev
spec:
  selector:
    app: redis
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
      nodePort: 30008
  type: NodePort


```

!!! tip "Chú ý"
    Vì đang dev nên mình sẽ dùng type NodePort cổng 30008 để export redis với cổng 30008 để dễ dàng cấu hình khi coding. Khi trên production type [ClusterIP](../../Kubernetes (K8s)/Cài đặt K8S/Lý thuyết.md) 


### 3.3. Kiểm tra kết nối với redis
Trên window tải [redis cli Redis-x64-5.0.14.1.zip](https://github.com/tporadowski/redis/releases) 

```bash 
./redis-cli -h 192.168.1.100 -p 30008
```
!!! tip "Chú ý"
    192.168.1.100 là địa chỉ của master, bạn cũng có thể sử dụng 192.168.1.101, 192.168.1.102 là các địa chỉ ip của worker để test redis khi dùng ClusterIP 

```bash 
192.168.1.100:30008> set test_key "hello"
OK
```
```bash 
192.168.1.100:30008> get test_key
"hello"
```

## 4. Tài liệu tham khảo


- [Redis Caching Strategies - Redis.io](https://redis.io/docs/latest/)
- [Caching best practices - Microsoft Learn](https://learn.microsoft.com/en-us/azure/architecture/best-practices/caching)




