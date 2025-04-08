# Toàn luồng Jenkins + Argocd + Github + Docker hub trên k8s

## 1. Giới thiệu

Trong bài viết trước Jenkins đã lấy source từ github sau đó dùng kaniko để build Dockerfile có trong source, sau đó đẩy image lên docker hub, và argocd dùng image của trên docker hub để deployment ứng dụng lên k8s. Bài viết này mình sẽ giới thiệu kết hợp toàn trình luồng gitops: Jenkins + Argocd + Github + Docker hub trên k8s

2. Cài đặt

2.1 Cấu hình jenkins

Mã nguồn github: [https://github.com/vanphuoc9/complete-prodcution-e2e-pipeline.git](https://github.com/vanphuoc9/complete-prodcution-e2e-pipeline.git)