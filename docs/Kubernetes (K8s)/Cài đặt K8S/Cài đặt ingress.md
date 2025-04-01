# Cài đặt Ingress Nginx

## Giới thiệu Ingress Nginx

Ingress Nginx là một Ingress controller cho Kubernetes sử dụng NGINX như một reverse proxy và load balancer. [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) là một tài nguyên trong Kubernetes cho phép quản lý các kết nối đến các dịch vụ trong Kubernetes cluster. Ingress Nginx hỗ trợ khả năng căn bằng tải, SSL, URI rewrite và nhiều tính năng khác.

Thường thì Ingress Nginx sử dụng để cho phép bên ngoài truy cập vào ứng dụng thông qua tên miền ví dụ như: app1.example.com, app2.example.com.

![Ingress](images/ingress.svg)

## Cài helm nếu chưa có

```bash 
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```


## [Cài đặt Ingress Nginx bằng helm](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)
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

Chúng ta có thể thấy 2 port 80/443 trên Nginx được map tương ứng sang port 32000/32748, như vậy nếu muốn access vào App1 trên K8s thông qua Ingress Nginx thì cần truy cập như sau:

- **HTTP: http://app1.example.com:32000**
- **HTTPS: https://app1.example.com:32748**

Như vậy khá là bất tiện, chúng ta có thể:

- **Dùng chức năng port-forward trên router để mapping tiếp port 80->32000 và 443->32748. Về cơ bản đường đi của nó như sau: Internet (80/http) –>router (port-forward) –> Kuberentes IP (32000/http) –> Nginx (80/http)**

- **Dùng 1 Proxy như Haproxy hoặc Nginx đứng trước K8s cluster để điều phối**

Để tránh những phức tạp khi triển khai Ingress Nginx trên Bare metal K8s cluster, dưới đây mình sẽ giới thiệu cách triển khai Metallb và Ingress Nginx (Load Balancer)

Tài liệu tham khảo:

- [Cài đặt Metallb và Ingress Nginx trên Bare metal Kubernetes cluster của anh nvtienanh](https://nvtienanh.info/blog/cai-dat-metallb-va-ingress-nginx-tren-bare-metal-kubernetes-cluster)
- [Ingress NGINX Controller](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)
- [Bare-metal considerations](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/)