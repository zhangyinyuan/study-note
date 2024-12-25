# 搭建一个kubernetes集群

## 安装工具[kubectl](https://kubernetes.io/zh-cn/docs/tasks/tools/#kubectl)

### 安装

```shell
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
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
ip route show | grep "default via" 
default via 10.140.0.1 dev ens4 proto dhcp src 10.140.0.7 metric 100

sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```



```shell
cat /etc/kubernetes/kubelet.conf | grep server
```

重置k8s集群

```shell
sudo kubeadm reset
rm -rf /etc/cni/net.d
rm -rf $HOME/.kube/config
rm -rf /etc/kubernetes
```



```shell
docker run -d --name=rancher --privileged --network nginx_ngu_network --memory="2g"  rancher/rancher:latest 
```



```shell
dd if=/dev/zero of=/swapfile bs=1M count=2048
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

```shell
 docker logs --tail 10 nginx
```



## 列出所有命名空间下的所有的服务

> 可以看到端口

```shell
kubectl get svc --all-namespaces
NAMESPACE                         NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
cattle-fleet-local-system         fleet-agent            ClusterIP   None             <none>        <none>                   30m
cattle-provisioning-capi-system   capi-webhook-service   ClusterIP   10.101.178.158   <none>        443/TCP                  29m
cert-manager                      cert-manager           ClusterIP   10.103.216.116   <none>        9402/TCP                 35m
cert-manager                      cert-manager-webhook   ClusterIP   10.102.68.102    <none>        443/TCP                  35m
default                           kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP                  3h46m
default                           nginx-service          NodePort    10.104.63.107    <none>        80:30001/TCP             79m
kube-system                       kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   3h46m
```

删除服务

```shell
kubectl delete svc nginx-service -n default
kubectl delete deployment nginx -n default
root@tw:/app/docker/nginx/nginx/conf.d# kubectl get deployments -n default
No resources found in default namespace.
```

删除 Namespace 

```shell
kubectl get namespace cattle-system -o json
kubectl patch namespace cattle-system -p '{"metadata":{"finalizers":null}}'
kubectl get namespaces
kubectl delete all --all -n cattle-system
kubectl get all -n cattle-system
kubectl delete namespace cattle-system --force --grace-period=0


kubectl get namespace cattle-system -o json > cattle-system.json
kubectl delete validatingwebhookconfiguration rancher.cattle.io
kubectl delete validatingwebhookconfiguration cert-manager-webhook

```

### 方法一：将服务类型修改为 `LoadBalancer`

如果您使用的是云环境（如 GCP、AWS、Azure），可以通过将服务类型改为 `LoadBalancer` 来使 Rancher 获得一个外部 IP 地址。

执行以下命令来修改 Rancher 服务类型：

```
bash


复制代码
kubectl -n cattle-system patch svc rancher -p '{"spec":{"type":"LoadBalancer"}}'
```

此时，您可以通过 `EXTERNAL-IP` 来访问 Rancher。例如：

```
bash


复制代码
kubectl -n cattle-system get svc rancher
```

您将看到 `EXTERNAL-IP` 字段已分配一个外部 IP 地址，您可以使用该 IP 地址在浏览器中访问 Rancher UI。

### 方法二：将服务类型修改为 `NodePort`

如果您没有负载均衡器或不希望使用它，您可以将服务类型修改为 `NodePort`，然后使用集群节点的外部 IP 和端口访问 Rancher。

执行以下命令来修改 Rancher 服务类型：

```
bash


复制代码
kubectl -n cattle-system patch svc rancher -p '{"spec":{"type":"NodePort"}}'
```

然后，使用以下命令查看端口映射：

```
bash


复制代码
kubectl -n cattle-system get svc rancher
```

您将看到类似如下的输出：

```
scss复制代码NAME      TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
rancher   NodePort   10.104.162.113 <none>        80:30001/TCP   1m
```



如果 Rancher 部署了多个 Pod，可以通过以下命令查看所有 Pod 的日志：

```
bash


复制代码
kubectl logs -n cattle-system -l app=rancher --follow
```

### helm安装的rancher重置admin密码

```shell
cp /root/.kube/config .
KUBECONFIG=./config
kubectl --kubeconfig $KUBECONFIG -n cattle-system exec $(kubectl --kubeconfig $KUBECONFIG -n cattle-system get pods -l app=rancher --no-headers | head -1 | awk '{ print $1 }') -c rancher -- reset-password
```



` kubectl get svc --namespace default`等价于 `kubectl get svc --namespace default`



`kubectl get svc --all-namespaces`表示列出所有空间下的服务



```shell
# dokcer方式搭建
docker run -d --privileged -p 60080:80 -p 60443:443 -v ./rancher_data:/var/lib/rancher --restart=always --name rancher-2.7.1 rancher/rancher:v2.7.1


helm方式搭建
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=k8s.nguone.eu.org  --set bootstrapPassword=Ni4phughiotieCoo
```



```shell
kubectl apply -f nginx-pvc.yaml
kubectl get pvc
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
kubectl delete pvc nginx-pvc
kubectl apply -f nginx-test-pod.yaml

```

