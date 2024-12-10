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

## 安装集群
