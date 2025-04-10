---
comments: true
---
# Cài đặt Ingress Nginx

## 1. Giới thiệu Ingress Nginx

Ingress Nginx là một Ingress controller cho Kubernetes sử dụng NGINX như một reverse proxy và load balancer. [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) là một tài nguyên trong Kubernetes cho phép quản lý các kết nối đến các dịch vụ trong Kubernetes cluster. Ingress Nginx hỗ trợ khả năng căn bằng tải, SSL, URI rewrite và nhiều tính năng khác.

Thường thì Ingress Nginx sử dụng để cho phép bên ngoài truy cập vào ứng dụng thông qua tên miền ví dụ như: app1.example.com, app2.example.com.

![Ingress](images/ingress.svg)

## 2. Cài helm nếu chưa có

```bash 
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```


## 3. [Cài đặt Ingress Nginx bằng helm](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)
Có nhiều cách cài Ingress như: [HAProxy Ingress](https://github.com/haproxytech/kubernetes-ingress#readme), [NGINX Ingress Controller for Kubernetes](https://www.nginx.com/products/nginx-ingress-controller/),  [Kong Ingress Controller for Kubernetes](https://github.com/Kong/kubernetes-ingress-controller#readme),...

Trong bài viết này sẽ sử dụng [Ingress NGINX Controller](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)

```bash 
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

Sau khi cài đặt, kiểm tra lại

```bash 
kubectl get services -n ingress-nginx

NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.103.11.172    <none>         80:32000/TCP,443:32748/TCP   17d
ingress-nginx-controller-admission   ClusterIP      10.107.107.210   <none>         443/TCP                      17d


```

## 4. Tài liệu tham khảo:

- [Cài đặt Metallb và Ingress Nginx trên Bare metal Kubernetes cluster của anh nvtienanh](https://nvtienanh.info/blog/cai-dat-metallb-va-ingress-nginx-tren-bare-metal-kubernetes-cluster)
- [Ingress NGINX Controller](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)
- [Bare-metal considerations](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/)