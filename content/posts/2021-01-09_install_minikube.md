---
title: "🛠️ 在腾讯云上安装 minikube"
date: 2022-01-09T09:22:35+08:00
draft: false
tags: [技术,k8s]
categories: [正文]
description: "在酒店隔离，闲着也是闲着，那就折腾一下 minikube 吧。"
featured_image: ""
comment : false
canonicalUrl: 
---

这几天隔离在酒店，颇为无聊。于是，我就想着趁此机会玩玩 Kubeflow 或者 KServe，感受一下 Kubernetes 上的 model serving 方案现在发展到了什么地步。

首先我需要安装、配置一个 K8s 环境。考虑到成本因素，我选择在腾讯云轻量服务器上搭建一个单机的 K8s 环境用于学习。具体配置是 2 CPU，4 GB 内存，Ubuntu 20.04 LTS，一个月67元（香港可用区）。我因为是腾讯云 TVP 的关系，实际支付 0 元 😆

我先后尝试了 Ubuntu 的 MicroK8s 和 Kubernetes 的 kind，效果都不太理想。一个是各种版本依赖问题，另一个问题是文档以及用户教程都不太丰富。所以最后还是选择了 minikube。

## ⚙️ minikube 安装前置条件

参考文档：https://minikube.sigs.k8s.io/docs/start/

- 2 CPUs or more
- 2GB of free memory
- 20GB of free disk space
- Internet connection
- Container or virtual machine manager, such as: [Docker](https://minikube.sigs.k8s.io/docs/drivers/docker/), [Hyperkit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/), [Hyper-V](https://minikube.sigs.k8s.io/docs/drivers/hyperv/), [KVM](https://minikube.sigs.k8s.io/docs/drivers/kvm2/), [Parallels](https://minikube.sigs.k8s.io/docs/drivers/parallels/), [Podman](https://minikube.sigs.k8s.io/docs/drivers/podman/), [VirtualBox](https://minikube.sigs.k8s.io/docs/drivers/virtualbox/), or [VMware Fusion/Workstation](https://minikube.sigs.k8s.io/docs/drivers/vmware/)

### 网络配置

#### 公有云防火墙设置

一般公有云只给服务器开通几个默认的端口，比如用于 http 服务的 80 端口，用于 ssh 服务的 22 端口等等。后续我们要访问 minikube 上的服务，需要在公有云控制台上的防火墙管理界面中提前开通外部访问的端口。我使用腾讯云，具体参考以下文档：
https://cloud.tencent.com/document/product/1207/44577

其他公有云服务商的文档，也很容易找到。

我为 minikube 预留了外网 50000-60000 的端口。

#### Linux 服务器上的端口转发设置

K8s 上的服务一般都使用 K8s 的集群地址 (Cluster IP) 提供服务。我们也可以配置 External load balancer，这样服务就可以基于外部 IP 地址提供服务。不过怎么配置 External load balancer 我还没搞定，所以我在后面采用的方法是将服务的端口由 K8s cluster IP 转发到本地 (127.0.0.1, localhost)。

为了能够从外部访问，还需要做一个从外网地址到本地的端口转发。最终的效果类似于，

1. 用户访问 http://public_ip:public_port
2. NAT from http://public_ip:public_port to http://127.0.0.1:local_port
3. K8s forward http://127.0.0.1:local_port to http://cluster_ip:node_port

上面第 2 步中的端口转发需要在 Linux 操作系统上配置。我使用的 Ubuntu 20.04 LTS，需要执行以下命令：

```bash
# 确认 eth0 是系统的主要网络设备，比较下面命令输出的 inet 地址是不是和你的公有云内网地址一致
ifconfig eth0
# 打开 eth0 的路由功能。
sudo sysctl -w net.ipv4.conf.eth0.route_localnet=1
# 打开 nat 功能，将所有 tcp 访问直接转发给 127.0.0.1。为了省事，我这里没有设置端口的映射规则，因此端口号将保持不变
sudo iptables -t nat -A PREROUTING -p tcp -j DNAT --to-destination 127.0.0.1
```

### 安装 Docker

在前置条件中有提到，minikube 需要一个容器或虚拟机管理软件，我这里用的是最常见的 Docker。

在 Ubuntu 上通过 apt 安装的大致过程：

```bash
# 如果有安装旧版本的 docker，需要先卸载
sudo apt-get remove docker docker-engine docker.io containerd runc
# 安装必要的工具
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release
# 下载 Docker 的 GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# 添加 Docker 的 apt 源
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee etc/apt/sources.list.d/docker.list > /dev/null
# 安装 Docker engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
# Docker 安装成功以后，记得把你的用户加到 docker 用户组里面。
sudo usermod -aG docker $USER && newgrp docker
# 验证 Docker 是否安装成功
sudo docker run hello-world
```

> [Docker 详细安装文档](https://docs.docker.com/engine/install/ubuntu/)

### 安装 kubectl

kubectl 是 Kubernetes 的命令行工具。minikube 本身也自带了一个命令行工具，调用的时候形如：`minukube kubectl get service`

但实测这个 `minikube kubectl` 工具的命令语法和 `kubectl` 是有差异的。为了方便后续的测试，我选择安装 K8s 原生的 kubectl 命令行工具。

在 Ubuntu 上通过 apt 安装的大致过程：

```bash
# 安装必要的工具
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
# 下载 GCP 的 GPG key
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
# 添加 Kubernetes 的 apt 源
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
# 安装 kubectl 命令行工具
sudo apt-get update
sudo apt-get install -y kubectl
```

> [kubectl 详细安装文档](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

至此，准备工作已完毕，我们要开始安装 minikube 了。

## 🧪 minikube

[minikube 快速开始文档](https://minikube.sigs.k8s.io/docs/start/)

### 安装 minikube 软件包

Ubuntu 系统上的安装非常简单，下载 deb 软件包后安装即可。

```bash
# 下载软件包
$ curl -LO [https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb](https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb)
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
Dload  Upload   Total   Spent    Left  Speed
100 22.3M  100 22.3M    0     0  13.2M      0  0:00:01  0:00:01 --:--:-- 13.2M

# 安装软件包
$ sudo dpkg -i minikube_latest_amd64.deb
Selecting previously unselected package minikube.
(Reading database ... 89236 files and directories currently installed.)
Preparing to unpack minikube_latest_amd64.deb ...
Unpacking minikube (1.24.0-0) ...
Setting up minikube (1.24.0-0) ...
```

### 启动 minikube

```bash
# 启动 minikube 集群
$ minikube start
😄  minikube v1.24.0 on Ubuntu 20.04 (amd64)
✨  Automatically selected the docker driver. Other choices: none, ssh
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.22.3 preload ...
> preloaded-images-k8s-v13-v1...: 501.73 MiB / 501.73 MiB  100.00% 7.66 MiB
> [gcr.io/k8s-minikube/kicbase:](http://gcr.io/k8s-minikube/kicbase:) 355.77 MiB / 355.78 MiB  100.00% 4.34 MiB p/
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.22.3 on Docker 20.10.8 ...
▪ Generating certificates and keys ...
▪ Booting up control plane ...
▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
▪ Using image [gcr.io/k8s-minikube/storage-provisioner:v5](http://gcr.io/k8s-minikube/storage-provisioner:v5)
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

# 查看默认安装下集群中有哪些 pods
$ kubectl get po -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-78fcd69978-ftf5s           1/1     Running   0             11m
kube-system   etcd-minikube                      1/1     Running   0             11m
kube-system   kube-apiserver-minikube            1/1     Running   0             11m
kube-system   kube-controller-manager-minikube   1/1     Running   0             11m
kube-system   kube-proxy-5n86d                   1/1     Running   0             11m
kube-system   kube-scheduler-minikube            1/1     Running   0             11m
kube-system   storage-provisioner                1/1     Running   1 (11m ago)   11m

# 启动 minikube 仪表板
$ minikube dashboard
🔌  Enabling dashboard ...
▪ Using image kubernetesui/dashboard:v2.3.1
▪ Using image kubernetesui/metrics-scraper:v1.0.7
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:33557/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
👉  http://127.0.0.1:33557/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/

```

这个时候可以通过把上面连接中的 127.0.0.1 替换成你的云服务器的公网 IP 来访问 minikube 仪表板。但我的浏览器中只显示了一个：

```html
Forbidden
```

后续要再研究一下。😅

如果你的浏览器显示的是：

```html
Connect Timeout
[P]Connect Timeout|read tcp4 xxx.xxx.xxx.xxx:xxxx->xxx.xxx.xxx.xxx:xxxx: i/o timeout
```

请确认一下你的公有云防火墙已经设置好了。

### 部署一个示例应用

下面是通过 `kubectl create` 和 `kubectl expose` 命令创建一个服务的过程。另外你也可以编写一个 manifest 文件（yaml 文件），然后使用 `kubectl apply -f` 命令从 manifest 文件来创建服务。
```bash
# 通过命令行创建一个 hello-minikube 的服务
$ kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
deployment.apps/hello-minikube created
$ kubectl expose deployment hello-minikube --type=NodePort --port=8080
service/hello-minikube exposed

# 查看服务的状态
$ kubectl get services hello-minikube
NAME             TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
hello-minikube   NodePort   10.102.150.172   <none>        8080:30275/TCP   2m52s
```

**hello-minikube** 服务已经创建完毕。因为我们没有创建 external load balancer，所以这里 EXTERNAL-IP 显示为空。我们如果要访问这个服务，需要为 *CLUSTER-IP:8080* 做一个端口转发：

```bash
# 为 CLUSTER-IP:8080 创建一个端口转发规则，以下命令会挂起你的终端
$ kubectl port-forward service/hello-minikube 57080:8080
Forwarding from 127.0.0.1:57080 -> 8080
Forwarding from [::1]:57080 -> 8080
```

我们之前已经做了云环境公网地址到云环境内网地址的转发，因此我们现在可以直接通过 http://public_ip:57080/ 来访问 **hello-minikube**。

成功连接的时候，刚才挂起的终端上会显示以下信息：

```bash
Handling connection for 57080
```

你会在浏览器上看到类似这样的信息：

```html
CLIENT VALUES:
client_address=127.0.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://xxx.xxx:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
accept-encoding=gzip, deflate
accept-language=en,zh-CN;q=0.9,zh;q=0.8,en-CN;q=0.7
host=xxx.xxx:57080
proxy-connection=keep-alive
sec-gpc=1
upgrade-insecure-requests=1
user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36
BODY:
-no body in request-
```

## What's next

minikube 的安装、验证工作已经完成。接下来几天，我将尝试部署 Kubeflow 和 KServe。

## 附：一些常用的 minikube 管理命令

- Pause Kubernetes without impacting deployed applications:
     `minikube pause`
- Unpause a paused instance:
    `minikube unpause`
- Halt the cluster:
    `minikube stop`
- Increase the default memory limit (requires a restart):
    `minikube config set memory 16384`
- Browse the catalog of easily installed Kubernetes services:
    `minikube addons list`
- Create a second cluster running an older Kubernetes release:
    `minikube start -p aged --kubernetes-version=v1.16.1`
- Delete all of the minikube clusters:
    `minikube delete --all`
- Get  workstation IP address:
    `minikube ip`
