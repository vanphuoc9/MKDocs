
# Mẹo K8S


## 1. Connection is unauthorized

Issue: When working with Kubeadm cluster with Calico CNI plugin, the container fails to launch and if you describe the pod you may get the following error.
Warning  FailedCreatePodSandBox  16m                 kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed to create pod network sandbox k8s_webserver-667ddc69b6-wq689_default_ffa6a237-7dc6-4bc4-9444-b0146a5b7f21_0(6dfe713911b0d60f98cf464a11928b041c885ff9dd3c59323ca5271be1df632b): error adding pod default_webserver-667ddc69b6-wq689 to CNI network "k8s-pod-network": plugin type="calico" failed (add): error getting ClusterInformation: connection is unauthorized: Unauthorized


This error can be rectified by restarting the Calico pods running in the kube-system namespace.

Get the labels of calico pods using the following command.

```bash 
kubectl get pods -n kube-system --show-labels
```

First delete all the calico pods using the following command.

```bash 
kubectl delete pods -n kube-system -l k8s-app=calico-node
```


