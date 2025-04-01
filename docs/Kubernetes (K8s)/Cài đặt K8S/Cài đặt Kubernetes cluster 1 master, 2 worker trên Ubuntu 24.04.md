# Cài đặt Kubernetes cluster 1 master, 2 worker trên Ubuntu 24.04

## Giới thiệu

Có nhiều cách để cài đặt k8s, trong tài liệu này giới thiệu cài đặt k8s trên ubuntu 24.04 để gần với môi trường production nhất

## Cấu hình

Trong môi trường thử nghiệm:

- **HP 240 GM Notebook Intel core I5 2.4Ghz, Ram: 16GB:** Cài đặt 4 máy ảo bằng virutalbox
- **Kubernetes cluster: 01 Master, 02 Worker, 01 NFS**
- **Phiên bản hệ điều hành: Ubuntu Server 24.04**

Lưu ý trên đây chỉ là mô hình thử nghiệm, còn môi trường production nên thực hiện ít nhất: **Kubernetes cluster: 03 Master, 08 Worker** để đảm bảo [High Availability (HA)](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/) hoặc nhiều hơn tùy thuộc vào độ lớn của dự án



| Tên máy/hostmane/Node          | Hệ điều hành | IP | Vai trò |
|----------------------|----------------|----------------|-----------|
| master.xtl    | ubuntu 24.04              | 192.168.1.100           | master  |
| worker1.xtl | ubuntu 24.04              | 192.168.1.101          | worker 1 |
| worker2.xtl  | ubuntu 24.04             | 192.168.1.102            | worker 2  |
| nfs.xtl  | ubuntu 24.04             | 192.168.1.110            | NFS  |

