---
title: "Kubenetes on Centos"
date: 2021-10-02T15:21:26+09:00
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

## Prerequisite

- A compatible Linux host. The Kubernetes project provides generic instructions for Linux distributions based on Debian and Red Hat, and those distributions without a package manager.
- 2 GB or more of RAM per machine (any less will leave little room for your apps).
- 2 CPUs or more.
- Full network connectivity between all machines in the cluster (public or private network is fine).
- Unique hostname, MAC address, and product_uuid for every node. See [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#verify-mac-address) for more details.
- Certain ports are open on your machines. See [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports) for more details.
- Swap disabled. You **MUST** disable swap in order for the kubelet to work properly.

### Letting iptables see bridged traffic

Make sure that the `br_netfilter` module is loaded. This can be done by running `lsmod | grep br_netfilter`. To load it explicitly call `sudo modprobe br_netfilter`.

```bash
lsmod | grep br_netfilter
# br_netfilter           22256  0 
# bridge                151336  2 br_netfilter,ebtable_broute
sudo modprobe br_netfilter
```

As a requirement for your Linux Node's iptables to correctly see bridged traffic, you should ensure `net.bridge.bridge-nf-call-iptables` is set to 1 in your `sysctl` config.

```bash
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

### Check Required Ports



### Install Docker

Refer: [Install Docker Engine on CentOS](https://docs.docker.com/engine/install/centos/) 

## Install Kubernetes

### Prepare Kubernetes Servers

| **Server Type** | **Host Name** | **Specs**       |
| :-------------- | :------------ | :-------------- |
| Master          | 192.168.2.60  | 2 CPUs, 2GB Ram |
| Worker          | 192.168.2.61  | 2 CPUs, 2GB Ram |
| Worker          | 192.168.2.62  | 2 CPUs, 2GB Ram |

Here I used a VM which has **2 CPUs** and **4GB Ram** for master node.

### Install kubeadm and kubectl

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

### Disable SELinux and Swap

```bash
# Set SELinux in permissive mode (effectively disabling it)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# Disable swap
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a
```

## Configure Firewall

**Enable Master Node Ports**:

```bash
sudo firewall-cmd --add-port={6443,2379-2380,10250,10251,10252,5473,179,5473}/tcp --permanent
sudo firewall-cmd --add-port={4789,8285,8472}/udp --permanent
sudo firewall-cmd --reload
```

**Enable Worker Node Ports**:

```bash
sudo firewall-cmd --add-port={10250,30000-32767,5473,179,5473}/tcp --permanent
sudo firewall-cmd --add-port={4789,8285,8472}/udp --permanent
sudo firewall-cmd --reload
```

## Verify the installation

In order to verify whether the installation is successful, we are gonna create a cluster using ***kubeadm***.

### Initialize Kubernetes Control Plane

First of all, I added a host name to `/etc/hosts` , as below:

> 192.168.2.60 k8s-master01 k8s-master01.torres.com

Then I run below commands to initialize the control plane on master node:

```bash
# (Optional) You can run this commands to verify the connection with gcr.io
sudo kubeadm config images pull

# Init Control plane
sudo kubeadm init --control-plane-endpoint=k8s-master01.torres.com --node-name=k8s-master01 --upload-certs 
```

Then you can see logs as follows:

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

As the logs says, we need to run these commands after the initialization:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Verification

We can run this command to verify the cluster we just created:

```bash
kubectl cluster-info

Kubernetes control plane is running at https://k8s-master01:6443
CoreDNS is running at https://k8s-master01:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Also if you want to see the nodes:

```bash
kubectl get nodes

NAME                    STATUS     ROLES                  AGE     VERSION
localhost.localdomain   NotReady   control-plane,master   3m15s   v1.21.1
```

If you see the status of the nodes you created is not ready, you can check if all the pods in the node is **RUNNING**:

```bash
kubectl get pods --all-namespaces

NAMESPACE     NAME                                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-558bd4d5db-9szx5                        0/1     Pending   0          7m31s
kube-system   coredns-558bd4d5db-lx7jv                        0/1     Pending   0          7m31s
kube-system   etcd-localhost.localdomain                      1/1     Running   0          7m36s
kube-system   kube-apiserver-localhost.localdomain            1/1     Running   0          7m36s
kube-system   kube-controller-manager-localhost.localdomain   1/1     Running   0          7m36s
kube-system   kube-proxy-2skgn                                1/1     Running   0          7m31s
kube-system   kube-scheduler-localhost.localdomain            1/1     Running   0          7m38s
```

The **coredns** is still Pending, so we have to install a network plugin to make the coredns work. 

## Install network plugin - Calico

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

onfigmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
Warning: policy/v1beta1 PodDisruptionBudget is deprecated in v1.21+, unavailable in v1.25+; use policy/v1 PodDisruptionBudget
poddisruptionbudget.policy/calico-kube-controllers created
```

Then run following command to see the service:

```bash
kubectl get pods --all-namespaces

NAMESPACE     NAME                                            READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-78d6f96c7b-9kd5b        1/1     Running   0          6m57s
kube-system   calico-node-f8fsw                               1/1     Running   0          6m57s
kube-system   coredns-558bd4d5db-9szx5                        1/1     Running   0          14m
kube-system   coredns-558bd4d5db-lx7jv                        1/1     Running   0          14m
kube-system   etcd-localhost.localdomain                      1/1     Running   0          14m
kube-system   kube-apiserver-localhost.localdomain            1/1     Running   0          14m
kube-system   kube-controller-manager-localhost.localdomain   1/1     Running   0          14m
kube-system   kube-proxy-2skgn                                1/1     Running   0          14m
kube-system   kube-scheduler-localhost.localdomain            1/1     Running   0          14m
```

After the installation of Network Plugin (Calico), we can see **coredns** is Running, and there are another two pods for calico network plugin.



Now, we are going to install Kubernetes Dashboard!

## Install Kubernetes Dashboard

By default, the dashboar ui is not deployed, so we need to do it manually with following command:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml

# Then you can use this command to see the dashboard is running
kubectl get pods --all-namespaces

NAMESPACE              NAME                                            READY   STATUS    RESTARTS   AGE
kube-system            calico-kube-controllers-78d6f96c7b-9kd5b        1/1     Running   0          9m41s
kube-system            calico-node-f8fsw                               1/1     Running   0          9m41s
kube-system            coredns-558bd4d5db-9szx5                        1/1     Running   0          17m
kube-system            coredns-558bd4d5db-lx7jv                        1/1     Running   0          17m
kube-system            etcd-localhost.localdomain                      1/1     Running   0          17m
kube-system            kube-apiserver-localhost.localdomain            1/1     Running   0          17m
kube-system            kube-controller-manager-localhost.localdomain   1/1     Running   0          17m
kube-system            kube-proxy-2skgn                                1/1     Running   0          17m
kube-system            kube-scheduler-localhost.localdomain            1/1     Running   0          17m
kubernetes-dashboard   dashboard-metrics-scraper-856586f554-p7gxx      1/1     Running   0          44s
kubernetes-dashboard   kubernetes-dashboard-78c79f97b4-qs57f           1/1     Running   0          45s
```

The yaml file above is a recommended config for production environment, it helps us to create service, service account, cluster role and deployment for kubernetes dashboard.

You can access Dashboard using the kubectl command-line tool by running the following command:

```bash
kubectl proxy
```

This will start serving at `localhost:8001` by default. Then you can access http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/ to see your dashboard.

But there are some optional parameters for the command:

```bash
kubectl proxy --address='192.168.2.60' --port=9001 --accept-hosts='^*$'
```

`--address`: The IP address on which to serve on.

`-p --port`: The port on which to run the proxy. Set to 0 to pick a random port.

`--accept-hosts`: Regular expression for hosts that the proxy should accept. (使用正则表达式指定proxy应该接收的hosts)

`--api-prefix='/'`: Prefix to serve the proxied API under.

For more details of the command, please use `kubectl proxy --help`.

### Kubernetes Dashboard

When every pod of kubenetes-dashboard is running, then you can access the dashboard.

![Kubernetes Dashboard](/images/kubernetes-on-centos_1.png)

Now you can see the dashboar login screen, but it requires authentication at login.

So you need to find the **Token** or **Kubeconfig** to sign in with following commands:

```bash
# See secret in kubernetes-dashboard namespace
kubectl -n kubernetes-dashboard get secret

NAME                               TYPE                                  DATA   AGE
default-token-p7rgf                kubernetes.io/service-account-token   3      27m
kubernetes-dashboard-certs         Opaque                                0      27m
kubernetes-dashboard-csrf          Opaque                                1      27m
kubernetes-dashboard-key-holder    Opaque                                2      27m
kubernetes-dashboard-token-8kjgr   kubernetes.io/service-account-token   3      27m

# Get details of token
kubectl -n kubernetes-dashboard describe secret kubernetes-dashboard-token-8kjgr

Name:         kubernetes-dashboard-token-8kjgr
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard
              kubernetes.io/service-account.uid: d54e2160-e9b0-444d-93fc-5d38f8fa61ef

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImNKX2pGTG9fUXE0cmk1cE5oUkpPYzVoald2TXFWN2QySHBEV1RlclNlMWcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi04a2pnciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImQ1NGUyMTYwLWU5YjAtNDQ0ZC05M2ZjLTVkMzhmOGZhNjFlZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.YbuWfZ3Qu9hjQTEVpuR7FxGrjboO8OpjqY4tE1O86v9UD9OilCbb330QRCNbTPcobLfXvFHRsJ4MVY2LHaCM1s2y2fpGE3dGPpPs2XcLJj2Aw3e_mJKHy9sZHtjfAG0cFgWKhyr_LTEuxk3_0pEVMKtuk2WeSqoo37ADhUjR92G7dK0TGkahkNzOH0-I_Yn40oZn9wA9w0r4DCGd5q8s2c5piHd6jGOuRX-7_UKgVfc6GYkRngAsrVZTnqfZkjv0LoH8Egkmu2X3CRvMz4JrlyhScTPZ77Uck0PXVklU57tK1PdgPIcczlvVJtr4avyBZsS5Y9j5zjm7sSDvPMeuJQ
```

Then you can use the token to sign into Kubernetes Dashboard.

> But this account do not have any permissions to all the content on dashboard, we need to create an admin account.

![Kubernetes Dashboard 2](/images/kubernetes-on-centos_2.png)

But there is no deployments, no pods… (Because of the permission of serviceaccount:kubernetes-dashboard: **"system:serviceaccount:kubernetes-dashboard:kubernetes-dashboard" cannot list resource "replicationcontrollers" in API group "" in the namespace "default"**)

I will add tutorial for adding cluster role and service account into kubernetes.

## Conclusion

If you want to reload the certs afterward,  run this command:

```bash
kubeadm init phase upload-certs --upload-certs
```

Join cluster with master node:

```bash
kubeadm join k8s-master01:6443 --token 0yj7hj.u6i2pv98o522rt99 \
	--discovery-token-ca-cert-hash sha256:a4507c9a70a7c36543f4913b8a636c2b8a24a73ff4aefec6dd43144a40188437 \
	--control-plane --certificate-key 82441af4e2a70d7de93a15dbdde52a165e1cf6184d98c9ccd4b2d64bc01d55c8
```

Join cluster with worker node:

```bash
kubeadm join k8s-master01:6443 --token 0yj7hj.u6i2pv98o522rt99 \
	--discovery-token-ca-cert-hash sha256:a4507c9a70a7c36543f4913b8a636c2b8a24a73ff4aefec6dd43144a40188437
```

How to access kubernetes dashboard remotely: (Kubernetes Dashboard is in your virtual machine, but you want to access it on local machine)

```bash
# command
ssh -L <port whatever you like>:127.0.0.1:8001 -N -f -l <username of kubernetes server> <kubernetes server ip> -P 22
# Example
ssh -L 8001:127.0.0.1:8001 -N -f -l leiyongqi 192.168.2.60 -P 22
```

**Tips**

If you specify an address for proxy, if you want to access on local machine rather than virtual machine, you need to change the command above:

```bash
ssh -L <port whatever you like>:<the address you specified in kubectl proxy command>:8001 -N -f -l <username of kubernetes server> <kubernetes server ip> -P 22
```

## Reference

[Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) 

[Creating Highly Available clusters with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/) 

[Install Docker Engine on CentOS](https://docs.docker.com/engine/install/centos/) 

[How can I remotely access kubernetes dashboard with token](https://stackoverflow.com/questions/59006501/how-can-i-remotely-access-kubernetes-dashboard-with-token) 

[no endpoints available for service \"kubernetes-dashboard\"](https://stackoverflow.com/questions/52893111/no-endpoints-available-for-service-kubernetes-dashboard) 

