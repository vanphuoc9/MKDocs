---
comments: true
---
# Cài đặt Mongodb trên k8s

## 1. Giới thiệu

MongoDB là một NoSQL database, khác với cơ sở dữ liệu quan hệ (SQL như MySQL, PostgreSQL).

Dữ liệu trong MongoDB được lưu dưới dạng document có cấu trúc giống JSON, nhưng thực tế là BSON (Binary JSON), giúp mở rộng hiệu suất và hỗ trợ nhiều kiểu dữ liệu hơn.

MongoDB được thiết kế để mở rộng theo chiều ngang (horizontal scalability), phù hợp với các ứng dụng có dữ liệu lớn và thay đổi nhanh.



## 2. Cài đặt trên k8s

Github chứa file cài đặt: [https://github.com/vanphuoc9/install-mongodb.git](https://github.com/vanphuoc9/install-mongodb.git)

### 2.1. Tạo secret


```yaml title="mongo-secret.yaml"  linenums="1"
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
  namespace: dev
type: Opaque
data:
  mongo-root-username: dXNlcm5hbWU=
  mongo-root-password: cGFzc3dvcmQ=

```


### 2.2. Tạo PV và PVC 

```yaml title="mongodb-pv.yaml"  linenums="1"
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv-volume
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
    path: /mnt/nfs_share/mongodb
    server: 192.168.1.110  # Thay bằng IP NFS Server
    readOnly: false
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pv-claim
  namespace: dev
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi

```


### 2.3. Cài đặt Deployment và Service



```yaml title="mongodb-dp.yaml"  linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  namespace: dev
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom: 
            secretKeyRef:
              name: mongo-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom: 
            secretKeyRef:
              name: mongo-secret
              key: mongo-root-password
        volumeMounts:
        - name: mongodb-storage
          mountPath: /data/db 
      volumes:
        - name: mongodb-storage
          persistentVolumeClaim:
            claimName: mongodb-pv-claim
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-service
  namespace: dev
spec:
  type: NodePort
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017 # port service
      targetPort: 27017 # container port
      nodePort: 30011


```

### 2.4. Tạo Database bằng lệnh

Cài dặt shell
 [https://www.mongodb.com/try/download/shell](https://www.mongodb.com/try/download/shell)

Sau đó dăng nhập với tài khoản admin

```bash 
mongosh --host 192.168.1.100 --port 30011 -u username -p password 
```


Sử dụng db rebdb:

```bash 
use rebdb;
```

Gán roles

```bash 
db.createUser({
  user: "rebdb",
  pwd: "rebdb",
  roles: [{ role: "readWrite", db: "rebdb" }]
});
```

![gitops](images/mongodb1.png)

Dùng NoSQL for MongoDB để kiểm tra 

Đường dẫn kết nối: mongodb://rebdb:rebdb@192.168.1.101:30011/rebdb


![gitops](images/mongodb2.png)


![gitops](images/mongodb3.png)


## 3. Tham khảo:

[https://www.strongdm.com/blog/mongodb-create-user](https://www.strongdm.com/blog/mongodb-create-user)

