# 搭建一个kubernetes集群

## 安装工具[kubectl](https://kubernetes.io/zh-cn/docs/tasks/tools/#kubectl)

### 下载

```shell
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

### 安装

```shell
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# 查看版本 
kubectl version --client
```

### 安装 bash-completion

```shell
apt-get install bash-completion -y && source /usr/share/bash-completion/bash_completion
```

### 全局启用启动 kubectl 自动补全功能

```shell
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
sudo chmod a+r /etc/bash_completion.d/kubectl
source ~/.bashrc
```

### [安装 `kubectl convert` 插件](https://kubernetes.io/zh-cn/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin)

```shell
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert"
sudo install -o root -g root -m 0755 kubectl-convert /usr/local/bin/kubectl-convert
rm kubectl-convert kubectl-convert.sha256
```

## 安装集群:开发/测试环境

### 硬件要求

- 2 个或更多 CPU
- 2GB 可用内存
- 20GB 可用磁盘空间
- 互联网连接
- 容器或虚拟机管理器，例如：[Docker](https://minikube.sigs.k8s.io/docs/drivers/docker/)、[QEMU](https://minikube.sigs.k8s.io/docs/drivers/qemu/)、[Hyperkit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/)、[Hyper-V](https://minikube.sigs.k8s.io/docs/drivers/hyperv/)、[KVM](https://minikube.sigs.k8s.io/docs/drivers/kvm2/)、[Parallels](https://minikube.sigs.k8s.io/docs/drivers/parallels/)、[Podman](https://minikube.sigs.k8s.io/docs/drivers/podman/)、[VirtualBox](https://minikube.sigs.k8s.io/docs/drivers/virtualbox/)或[VMware Fusion/Workstation](https://minikube.sigs.k8s.io/docs/drivers/vmware/)

### 安装minikube 

[minikube 是**本地 Kubernetes**，专注于让 Kubernetes 更易于学习和开发。](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download#LoadBalancer)

```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

### 新建一个用户

```shell
sudo useradd -m -s /bin/bash minikube  # -m 创建主目录,-s /bin/bash 指定默认的shell
sudo usermod -aG sudo minikube
sudo usermod -aG docker minikube
```

### 启动集群

> 从具有管理员权限的终端（但不要用 root 身份登录）运行

```shell
su - minikube
minikube start
```
显示如下,表示成功
```log
* minikube v1.34.0 on Debian 12.8 (amd64)
* Using the docker driver based on existing profile
* Starting "minikube" primary control-plane node in "minikube" cluster
* Pulling base image v0.0.45 ...
* Updating the running docker "minikube" container ...
* Preparing Kubernetes v1.31.0 on Docker 27.2.0 ...
* Verifying Kubernetes components...
  - Using image docker.io/kubernetesui/dashboard:v2.7.0
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
  - Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
* Some dashboard features require the metrics-server addon. To enable all features please run:

	minikube addons enable metrics-server

* Enabled addons: storage-provisioner, default-storageclass, dashboard
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

## 安装集群:生产环境

安装netcat-openbsd

```shell
apt update && apt install netcat-openbsd -y

# 用于直观地看端口是否启用
root@tw:~# nc 127.0.0.1 443 -v
Connection to 127.0.0.1 443 port [tcp/https] succeeded!
```

### [安装 kubeadm](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

```shell
sudo apt-get update
# apt-transport-https 可能是一个虚拟包（dummy package）；如果是的话，你可以跳过安装这个包
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```



```shell
kubeadm init --control-plane-endpoint "loadbalancer.nguone.eu.org" --ignore-preflight-errors=Port-6443,Port-10259,Port-10257,Port-10250,Port-2379,Port-2380,FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,DirAvailable--var-lib-etcd
```

