# GitOps: Jenkins + Argocd + Github + Docker hub

## 1. Giới thiệu
GitOps là một phương pháp triển khai và quản lý hạ tầng, ứng dụng dựa trên Git. Nó sử dụng các công cụ CI/CD để tự động hóa việc triển khai và đồng bộ hệ thống với trạng thái mong muốn được lưu trữ trong Git repository

## 2. Chức năng các công cụ trong GitOps

### 2.1. GitHub
- **Chức năng**: Lưu trữ mã nguồn và cấu hình Kubernetes (manifests, Helm charts).
- **Vai trò**: GitHub là nơi chứa cấu hình và tài nguyên ứng dụng.

### 2.2. Jenkins
- **Chức năng**: Tự động hóa quy trình build và deploy.
- **Vai trò**: Jenkins xây dựng ứng dụng, tạo Image Docker và đẩy lên Docker Hub.

### 2.3. Docker Hub
- **Chức năng**: Lưu trữ Image Docker.
- **Vai trò**: Docker Hub là nơi lưu trữ Image Docker sẵn sàng để triển khai.

### 2.3. ArgoCD
- **Chức năng**: Triển khai và quản lý ứng dụng trên Kubernetes.
- **Vai trò**: ArgoCD đồng bộ hóa trạng thái ứng dụng trong Kubernetes với cấu hình trong Git.

## 3. Quy trình GitOps
1. **Lưu trữ mã nguồn và cấu hình trên GitHub**.
2. **Jenkins** xây dựng ứng dụng và đẩy Image Docker lên **Docker Hub**.
3. **ArgoCD** tự động triển khai và đồng bộ ứng dụng trên Kubernetes từ GitHub.


!!! tip "Chú ý"
    Có thể thay [GitHub](https://github.com/) bằng [Gitlab](https://docs.gitlab.com/user/get_started/) và Docker Hub bằng [Harbor registry private](https://goharbor.io/) hoặc [Gitlab registry](https://docs.gitlab.com/user/packages/container_registry/) để sử dụng registry riêng (private).

![gitops](images/gitops.webp)
