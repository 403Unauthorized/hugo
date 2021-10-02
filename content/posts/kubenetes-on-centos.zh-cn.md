---
title: "Kubenetes on Centos"
date: 2021-10-02T15:21:31+09:00
tags: ["Cloud Native", "Kubernetes"]
categories: ["Kubernetes"]
draft: false
author: "Torres"
toc:
  enable: true
  auto: true
share:
  enable: true
comment:
  enable: true
featuredImagePreview: "/hugo/images/posts/kubernetes-on-centos/kubernetes-on-centos.png"
featuredImage: "/hugo/images/posts/kubernetes-on-centos/kubernetes-on-centos.png"
---

## 前提条件

- 一个 Linux 主机。Kubernetes 项目为基于 Debian 和 Red Hat 的 Linux 发行版提供了通用的说明。
- 每个机器至少 2GB 的内存。
- 至少 2 个 CPU
- 集群中所有机器之间的网络连接，公网或内网都可以。
- 每个节点由唯一的主机名，MAC 地址。
- 指定的端口在机器上是开放的。点击[这里](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)查看具体信息。
- 禁用 Swap。为了使 kubectl 正常工作，必须禁用 swap。

### 让 iptables 能发现桥接网络的流量

确保 `br_netfilter` 模块被装载了。可以通过以下命令来完成：

```bash
lsmod | grep br_netfilter
# br_netfilter           22256  0 
# bridge                151336  2 br_netfilter,ebtable_broute
sudo modprobe br_netfilter
```

为了让 Linux 节点的 iptables 能正确的观测到桥接流量，需要保证 `net.bridge.bridge-nf-call-iptables` 被设置为1：

```bash
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

### 检查需要的端口

### 安装 Docker

参考：[Install Docker Engine on CentOS](https://docs.docker.com/engine/install/centos/) 

## 安装 Kubernetes

### 准备 Kubernetes 服务器

| **服务器类型** | **主机名**   | **具体信息**    |
| :------------- | :----------- | :-------------- |
| Master         | 192.168.2.60 | 2 CPUs, 2GB Ram |
| Worker         | 192.168.2.61 | 2 CPUs, 2GB Ram |
| Worker         | 192.168.2.62 | 2 CPUs, 2GB Ram |

### 安装 kubelet 和 kubeadm

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```

### 禁用 SELinux 和 Swap

```bash
# Set SELinux in permissive mode (effectively disabling it)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# Disable swap
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a
```

## 配置防火墙

**启用 Master 节点端口**:

```bash
sudo firewall-cmd --add-port={6443,2379-2380,10250,10251,10252,5473,179,5473}/tcp --permanent
sudo firewall-cmd --add-port={4789,8285,8472}/udp --permanent
sudo firewall-cmd --reload
```

**启用 Worker 节点端口**:

```bash
sudo firewall-cmd --add-port={10250,30000-32767,5473,179,5473}/tcp --permanent
sudo firewall-cmd --add-port={4789,8285,8472}/udp --permanent
sudo firewall-cmd --reload
```

## 验证安装是否成功

为了验证安装是否成功，我们准备用 **kubeadm** 创建一个 cluster。

### 初始化 Kubernetes Control Plane (K8s 控制平面)

首先，向 `/etc/hosts` 中添加下面的主机名：

> 192.168.2.60 k8s-master01 k8s-master01.torres.com

然后运行以下命令来初始化 master 节点的 control plane：

```bash
# (Optional) You can run this commands to verify the connection with gcr.io
sudo kubeadm config images pull

# Init Control plane
sudo kubeadm init --control-plane-endpoint=k8s-master01.torres.com --node-name=k8s-master01 --upload-certs 
```

然后可以看到以下的日志：

```bash
[init] Using Kubernetes version: v1.21.1
[preflight] Running pre-flight checks
	[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local localhost.localdomain] and IPs [10.96.0.1 192.168.2.60]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost localhost.localdomain] and IPs [192.168.2.60 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost localhost.localdomain] and IPs [192.168.2.60 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 13.004835 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.21" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
b8cb86fb2bd01029d07cd1c67a6ae9ca358655595cef1cc7bec5253b64a81037
[mark-control-plane] Marking the node localhost.localdomain as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node localhost.localdomain as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: tm2wqn.66pko8ldffracpfs
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join k8s-master01:6443 --token tm2wqn.66pko8ldffracpfs \
	--discovery-token-ca-cert-hash sha256:470585a5c3c4706d86d06b23f1960a953960db666798266e528e6faf69981d2e \
	--control-plane --certificate-key b8cb86fb2bd01029d07cd1c67a6ae9ca358655595cef1cc7bec5253b64a81037

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-master01:6443 --token tm2wqn.66pko8ldffracpfs \
	--discovery-token-ca-cert-hash sha256:470585a5c3c4706d86d06b23f1960a953960db666798266e528e6faf69981d2e
```

我们需要执行日志上的命令来启用cluster：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 验证

运行以下指令来验证集群是否初始化成功：

```bash
kubectl cluster-info

Kubernetes control plane is running at https://k8s-master01:6443
CoreDNS is running at https://k8s-master01:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

如果你还想看有哪些节点：

```bash
kubectl get nodes

NAME                    STATUS     ROLES                  AGE     VERSION
localhost.localdomain   NotReady   control-plane,master   3m15s   v1.21.1
```

