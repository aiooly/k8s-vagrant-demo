
# Kubernetes学习总结：
- 第一部分是自己对Kubernetes的认识做一个记录和总结。
- 第二部分是在本地搭建起Kubernetes集群，并上手体验。

目录
================

* [认识 Kubernetes](#%E8%AE%A4%E8%AF%86-kubernetes)
  * [什么是 Kubernetes？](#%E4%BB%80%E4%B9%88%E6%98%AF-kubernetes)
  * [Kubernetes 架构图](#kubernetes-%E6%9E%B6%E6%9E%84%E5%9B%BE)
  * [Kubernetes 核心概念和组件](#kubernetes-%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5%E5%92%8C%E7%BB%84%E4%BB%B6)
* [体验 Kubernetes](#%E4%BD%93%E9%AA%8C-kubernetes)
  * [1\. 部署的环境准备](#1-%E9%83%A8%E7%BD%B2%E7%9A%84%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
  * [2\. 编写 <a href="https://www\.vagrantup\.com/docs/vagrantfile/" rel="nofollow">Vagrantfile</a>](#2-%E7%BC%96%E5%86%99-vagrantfile)
    * [2\.1 配置1个master，2个node，$script是初始化虚拟机执行的命令（安装，配置软件等）](#21-%E9%85%8D%E7%BD%AE1%E4%B8%AAmaster2%E4%B8%AAnodescript%E6%98%AF%E5%88%9D%E5%A7%8B%E5%8C%96%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%89%A7%E8%A1%8C%E7%9A%84%E5%91%BD%E4%BB%A4%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE%E8%BD%AF%E4%BB%B6%E7%AD%89)
    * [2\.2 虚拟机安装Docker](#22-%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%AE%89%E8%A3%85docker)
    * [2\.3 虚拟机安装kubernetes](#23-%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%AE%89%E8%A3%85kubernetes)
  * [3\.初始化虚拟机](#3%E5%88%9D%E5%A7%8B%E5%8C%96%E8%99%9A%E6%8B%9F%E6%9C%BA)
  * [4\. 配置集群](#4-%E9%85%8D%E7%BD%AE%E9%9B%86%E7%BE%A4)
  * [5\. 安装kubernetes\-dashboard](#5-%E5%AE%89%E8%A3%85kubernetes-dashboard)
  * [6\. 部署 hello\-world](#6-%E9%83%A8%E7%BD%B2-hello-world)

# 认识 Kubernetes

## 什么是 Kubernetes？
Kubernetes是在集群中跨多主机用于**自动部署、扩展和管理容器化（containerized）**应用程序的开源系统，kubernetes提供了诸多机制用来进行应用部署，调度，更新，维护和伸缩，以确保集群的状态是**持续健康**的。

Kubernete能帮我们达成以下目标：
- 跨多台主机进行容器编排。
- 更加充分地利用硬件，最大程度获取运行企业应用所需的资源。
- 有效管控应用部署和更新，并实现自动化操作。
- 挂载和增加存储，用于运行有状态的应用。
- 快速、按需扩展容器化应用及其资源。
- 对服务进行声明式管理，以保证所部署的应用始终按照您部署的方式加以运行。
- 利用自动布局、自动重启、自动复制以及自动扩展功能，对应用实施状况检查和自我修复。

## Kubernetes 架构图
![Kubernetes 架构图](https://kekekeke.sh1a.qingstor.com/k8s-vagrant-demo/k8s-master-node.png)

## Kubernetes 核心概念和组件
![enter image description here](https://kekekeke.sh1a.qingstor.com/k8s-vagrant-demo/k8s-%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5%E5%92%8C%E7%BB%84%E4%BB%B6.png)

# 体验 Kubernetes

想要在本地运行Kubernetes有如下方式

- [Minikube](https://kubernetes.io/docs/setup/minikube/)
- [kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
- 自定义安装：创建虚拟机，安装kubernetes需要的服务组件

**在这我们使用第二种方式**

## 1. 部署的环境准备
我们使用[Vagrant](https://www.vagrantup.com)来虚拟化, [bento/ubuntu-16.04](https://app.vagrantup.com/bento/boxes/ubuntu-16.04) 可以提前下载，由于安装过程需要访问被墙的资源，所以要配置`http_proxy`, 可以使用主机上的Shadowsocks代理。
- 主机环境：Mac OS 
- 虚拟机环境： Vagrant  (bento/ubuntu-16.04)
- 虚拟机provider：VirtualBox
- 代理：Shadowsocks （安装过程需要访问google资源）

## 2. 编写 [Vagrantfile](https://www.vagrantup.com/docs/vagrantfile/)
### 2.1 配置1个master，2个node，`$script`是初始化虚拟机执行的命令（安装，配置软件等）
```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)"

  config.vm.define "master" do |master|
    master.vm.network "private_network", ip: "192.168.10.10"
    master.vm.hostname = "master"
    master.vm.provision "shell", inline: $script
  end

  config.vm.define "node1" do |node1|
    node1.vm.network "private_network", ip: "192.168.10.11"
    node1.vm.hostname = "node1"
    node1.vm.provision "shell", inline: $script
  end

  config.vm.define "node2" do |node2|
    node2.vm.network "private_network", ip: "192.168.10.12"
    node2.vm.hostname = "node2"
    node2.vm.provision "shell", inline: $script
  end
end
```
### 2.2 虚拟机安装Docker
```shell
# Install docker
curl -sSL https://get.daocloud.io/docker | sh

# use Docker as a non-root user, add vagrat to docker group    
sudo usermod -a -G docker vagrant

# docker 配置代理
sudo mkdir /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/http-proxy.conf
sudo echo '
[Service]
Environment="HTTP_PROXY=http://192.168.10.1:1087" "HTTPS_PROXY=http://192.168.10.1:1087"
' > /etc/systemd/system/docker.service.d/http-proxy.conf

# restart docker
sudo systemctl daemon-reload
systemctl restart docker
sudo systemctl show docker --property Environment
```
### 2.3 虚拟机安装kubernetes
```shell
# Install kubernetes
sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# kubelet requires swap off
sudo swapoff -a

# keep swap off after reboot
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
## 3.初始化虚拟机
如果`bento/ubuntu-16.04`没有提前下载，会自动下载该box，然后分别配置master, node1,node2，如果安装过程出现问题，很有可能是http代理的问题
```
➜  k8s-vagrant-demo git:(master) ✗ vagrant up
```
**初始化结束后，可以查看创建的虚拟机状态**
```
➜  k8s-vagrant-demo git:(master) ✗ vagrant status
Current machine states:

master                    running (virtualbox)
node1                     running (virtualbox)
node2                     running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```
**进入master节点验证虚拟机配置的状态**
```
➜  k8s-vagrant-demo git:(master) ✗ vagrant ssh master
...
vagrant@master:~$ docker version
Client:
 Version:           18.06.1-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        e68fc7a
 Built:             Tue Aug 21 17:24:56 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.1-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       e68fc7a
  Built:            Tue Aug 21 17:23:21 2018
  OS/Arch:          linux/amd64
  Experimental:     false
vagrant@master:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.1", GitCommit:"4ed3216f3ec431b140b1d899130a69fc671678f4", GitTreeState:"clean", BuildDate:"2018-10-05T16:43:08Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
```
## 4. 配置集群
**初始化集群**
使用[kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/) init 初始化集群
```
vagrant@master:~$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.10.10  --node-name=master
... 此处略去很多log
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.10.10:6443 --token wu7vfc.1129cjxfenww631q --discovery-token-ca-cert-hash sha256:d48541246b9febd8c1f377996a18f196e7adab7c966406506b52de86a96e4df0
```

**设置配置文件**
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**添加Flannel，pod network**
```
vagrant@master:~$ sudo kubectl apply -f  https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created

vagrant@master:~$ sudo kubectl get pods --all-namespaces
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-576cbf47c7-6x85t         1/1     Running   0          17h
kube-system   coredns-576cbf47c7-j9dzl         1/1     Running   0          17h
kube-system   etcd-master                      1/1     Running   0          3m29s
kube-system   kube-apiserver-master            1/1     Running   0          3m29s
kube-system   kube-controller-manager-master   1/1     Running   0          3m28s
kube-system   kube-flannel-ds-amd64-tf9jz      1/1     Running   0          4m4s
kube-system   kube-proxy-nb2sk                 1/1     Running   0          17h
kube-system   kube-scheduler-master            1/1     Running   0          3m28s
```
**将node1, node2加入到集群中**
```
vagrant ssh node1
vagrant@node1:~$ sudo kubeadm join 192.168.10.10:6443 --token wu7vfc.1129cjxfenww631q --discovery-token-ca-cert-hash sha256:d48541246b9febd8c1f377996a18f196e7adab7c966406506b52de86a96e4df0
```

```
vagrant ssh node2
vagrant@node2:~$ sudo kubeadm join 192.168.10.10:6443 --token wu7vfc.1129cjxfenww631q --discovery-token-ca-cert-hash sha256:d48541246b9febd8c1f377996a18f196e7adab7c966406506b52de86a96e4df0
```
**在master节点上查看所有的nodes**
```
vagrant@master:~$ kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
master   Ready    master   17h     v1.12.1
node1    Ready    <none>   8m27s   v1.12.1
node2    Ready    <none>   8m24s   v1.12.1
```
到此我们的kubernetes集群就搭建完毕，下面我们来部署kubernetes-dashboard，然后就可以通过图形化界面来操作集群。

## 5. 安装kubernetes-dashboard
**安装dashboard**
```
vagrant@master:~$ sudo kubectl create -f https://gist.githubusercontent.com/aiooly/60cb0d150e0884e5f44e654fe3cb1168/raw/feea0964aa692b3b0974f3c7f4e3d5989db2e5a9/kubernetes-dashboard.yaml
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created
```
**查看dashboard service**
```
vagrant@master:~$ kubectl get svc -n kube-system
NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
kube-dns               ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP   18h
kubernetes-dashboard   NodePort    10.96.4.159   <none>        80:30091/TCP    32s
```
**创建clusterrolebinding**
```
vagrant@master:~$ kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
clusterrolebinding.rbac.authorization.k8s.io/add-on-cluster-admin created
```

**浏览器打开dashboard UI**

找到dashboard运行的节点
```
vagrant@master:~$ kubectl get services -n kube-system -o wide
NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE   SELECTOR
kube-dns               ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP   19h   k8s-app=kube-dns
kubernetes-dashboard   NodePort    10.96.4.159   <none>        80:30091/TCP    88m   k8s-app=kubernetes-dashboard
vagrant@master:~$ kubectl get pods --selector="k8s-app=kubernetes-dashboard" -n kube-system -o wide
NAME                                  READY   STATUS    RESTARTS   AGE   IP           NODE    NOMINATED NODE
kubernetes-dashboard-c5b44fc5-8n2gp   1/1     Running   6          88m   10.244.1.2   node1   <none>
```
在主机上打开浏览器，输入：http://192.168.10.11:30091, 如下图：
![](https://kekekeke.sh1a.qingstor.com/k8s-vagrant-demo/kubernetes-dashboard.png)

## 6. 部署 hello-world 
**创建hello-world部署**

```
kubectl run hello-world --replicas=1 --labels="app=hello-world" --image=nginx:1.7.9  --port=80
```

**可以通过下面的命令查看部署情况 **
```
vagrant@master:~$ kubectl get deployments -o wide
NAME          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS    IMAGES        SELECTOR
hello-world   1         1         1            1           111s   hello-world   nginx:1.7.9   app=hello-world
vagrant@master:~$ kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE    IP           NODE    NOMINATED NODE
hello-world-956d45bf6-5k9b5   1/1     Running   0          115s   10.244.2.2   node2   <none>
```
**创建hello-world-service来暴露服务，使集群外可以访问**
这里我们使用NodePort
```
kubectl expose deployment hello-world --type=NodePort --name=hello-world-service
```
查看刚部署的应用的状态
```
vagrant@master:~$ kubectl get services hello-world-service -o wide
NAME                  TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE     SELECTOR
hello-world-service   NodePort   10.104.119.39   <none>        80:30748/TCP   4m37s   app=hello-world
vagrant@master:~$ kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE    NOMINATED NODE
hello-world-956d45bf6-5k9b5   1/1     Running   0          32m   10.244.2.2   node2   <none>
```
或者可以在dashboard上查看
![](https://kekekeke.sh1a.qingstor.com/k8s-vagrant-demo/hello-world-service.png)

**浏览器访问: NodeIP:NodePort**
![](https://kekekeke.sh1a.qingstor.com/k8s-vagrant-demo/hello-world-app.png)




