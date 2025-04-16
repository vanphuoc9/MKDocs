# Lý thuyết K8S

## 1. Các loại Service trong Kubernetes (ngoài NodePort)

| **Loại Service**                             | **Mô tả**                                                                 | **Use case**                                         | **Khả năng truy cập**            |
|---------------------------------------------|---------------------------------------------------------------------------|------------------------------------------------------|----------------------------------|
| **ClusterIP (default)**                     | Chỉ truy cập được từ bên trong cluster.                                  | Giao tiếp nội bộ giữa các Pod                        | Nội bộ cluster                   |
| **NodePort**                                | Mở một cổng cố định trên mỗi Node để truy cập từ bên ngoài cluster        | Truy cập service từ bên ngoài (debug/test)           | Cổng node (IP:port)              |
| **LoadBalancer**                            | Cấp một địa chỉ IP public thông qua cloud provider hoặc MetalLB          | Truy cập service từ bên ngoài (production)           | IP Public (Cloud/MetalLB)        |
| **ExternalName**                            | Mapping DNS name đến một external service bên ngoài Kubernetes            | Kết nối dịch vụ bên ngoài (VD: DB, API)              | Chuyển hướng DNS                 |
| **Headless Service (`ClusterIP: None`)**    | Không tạo IP ảo, chỉ dùng DNS để truy cập trực tiếp từng Pod              | StatefulSet, truy cập từng Pod trực tiếp             | DNS nội bộ                       |
| **Ingress** *(không phải Service trực tiếp)* | Điều phối truy cập HTTP/HTTPS thông qua domain name                       | Truy cập nhiều service qua 1 IP hoặc domain công khai | HTTP/HTTPS entrypoint           |

---

Gợi ý sử dụng theo tình huống

- **Nội bộ giữa các Pod:** `ClusterIP`
- **Truy cập tạm thời (debug):** `NodePort`
- **Triển khai production (cloud hoặc bare metal):** `LoadBalancer` (kèm MetalLB nếu on-premises)
- **Truy cập theo tên miền:** `Ingress`
- **Kết nối đến API/DB bên ngoài:** `ExternalName`
- **Ứng dụng dạng Stateful (Zookeeper, Kafka…):** `Headless Service`
