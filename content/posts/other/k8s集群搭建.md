---
title: "K8s集群搭建"
date: 2022-07-24T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- 其他
tags: 
- k8s
---

## 安装k8s

### 一：系统设置

#### 1.1： 关闭防火墙

>systemctl stop firewalld
>
>systemctl disable firewalld



#### 1.2：关闭selinux

>sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
>
>setenforce 0  # 临时



#### 1.3：关闭swap

>swapoff -a 
>
>sed -ri 's/.*swap.*/#&/' /etc/fstab



#### 1.4：设置主机名

>hostnamectl set-hostname <hostname>



#### 1.5：添加hosts

>cat >> /etc/hosts << EOF
>192.168.178.171 k8s-master
>192.168.178.172 k8s-node1
>192.168.178.173 k8s-node2
>EOF



#### 1.6：将桥接的IPv4流量传递到iptables的链

>cat > /etc/sysctl.d/k8s.conf << EOF
>net.bridge.bridge-nf-call-ip6tables = 1
>net.bridge.bridge-nf-call-iptables = 1
>EOF
>
>sysctl --system  # 生效



#### 1.7：设置时间同步

>yum install ntpdate -y
>
>ntpdate time.windows.com



### 二：安装docker

>yum install -y docker
>
>配置镜像：
>
>vim /etc/docker/daemon.json
>
>{
>"registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"],
>"exec-opts": ["native.cgroupdriver=systemd"]
>}
>
>systemctl restart docker



### 三：安装k8s组件

#### 配置阿里云软件源

>cat > /etc/yum.repos.d/kubernetes.repo << EOF
>[kubernetes]
>name=Kubernetes
>baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
>enabled=1
>gpgcheck=0
>repo_gpgcheck=0
>gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
>EOF

#### 安装组件

>dnf install -y kubernetes-kubeadm kubernetes-kubelet kubernetes-master
>
>systemctl enable kubelet



#### 3.1：初始化

>kubeadm init \
>--apiserver-advertise-address=masterIp \
>--image-repository registry.aliyuncs.com/google_containers \
>--kubernetes-version v1.20.2 \
>--service-cidr=10.96.0.0/12 \
>--pod-network-cidr=10.244.0.0/16
>
>1. --apiserver-advertise-address 集群通告地址
>2. --image-repository 由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址
>3. --kubernetes-version K8s版本，与上面安装的一致 kubeadm  version：查看版本
>4. --service-cidr 集群内部虚拟网络，Pod统一访问入口
>5. --pod-network-cidr Pod网络，与下面部署的CNI网络组件yaml中保持一致



#### 3.2：安装网络组

>curl https://docs.projectcalico.org/v3.18/manifests/calico.yaml -O
>
>启用网路组：kubectl apply -f calico.yaml

### 四：子节点

#### 一：子节点执行上述一二三操作

#### 二：主节点执行：

##### 2.1: 获取token

> 命令：kubeadm token list
>
> 结果： ff1wdt.gaop6xb7n159ob68 

##### 2.2: 获取sha256

> 命令：openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
>
> 结果： 98448e33982b3684b0870d543eff9c9490e749588caef088f8bfe144d4f2166f 



#### 三: 子节点添加到主节点

>kubeadm join 主节点ip:6443 --token 上述获得的token --discovery-token-ca-cert-hash sha256:上述获得的sha256

#### 四：主节点验证

子节点现实不是Ready稍等再次运行即可

>[root@k8s-master ~]# kubectl get nodes
>NAME         STATUS   ROLES                  AGE     VERSION
>k8s-master   Ready    control-plane,master   3m1s     v1.20.2
>k8s-node1    Ready    <none>                 3m14s   v1.20.2
>k8s-node2    Ready    <none>                 3m21s   v1.20.2



### 五：可视化界面安装

#### 6.1: 应用可视化配置

>kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml

#### 6.2: 设置访问端口

>kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard

#### 6.3: 修改配置提供外部访问

>kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
>
>把type: ClusterIP 改为 type: NodePort

#### 6.4: 创建账号获取token

>vi dash.yaml
>
>```yaml
>#创建访问账号，准备一个yaml文件； vi dash.yaml
>apiVersion: v1
>kind: ServiceAccount
>metadata:
>name: admin-user
>namespace: kubernetes-dashboard
>---
>apiVersion: rbac.authorization.k8s.io/v1
>kind: ClusterRoleBinding
>metadata:
>name: admin-user
>roleRef:
>apiGroup: rbac.authorization.k8s.io
>kind: ClusterRole
>name: cluster-admin
>subjects:
>- kind: ServiceAccount
>name: admin-user
>namespace: kubernetes-dashboard
>```
>
>应用：kubectl apply -f dash.yaml

#### 6.5: 获取令牌

>```shell
>kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
>```

#### 6.6 查看端口

>```shell
>kubectl get svc kubernetes-dashboard -n kubernetes-dashboard
>```
>
>443:30120/TCP：30120为对外访问端口

 必须使用https://访问；访问时会提示有问题链接，如果是谷歌浏览器鼠标点击该页面然后键盘输入 thisisunsafe 即可访问 





## 安装问题解决

每次解决问题都可以执行``kubeadm reset``进行重置配置

主节点内存配置不小于4g不然会警告

一：The connection to the server localhost:8080 was refused - did you specify the right host or port?

>原因：
>
>kubernetes master没有与本机绑定集群初始化的时候没有绑定，此时设置在本机的环境变量即可解决问题。
>
>解决：
>
>echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
>
>source /etc/profile



二：	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/

>cat <<EOF> /etc/docker/daemon.json
>{
>"exec-opts": ["native.cgroupdriver=systemd"]
>}
>EOF
>
>systemctl restart docker



三：** not found path

>使用yum install **
>
>下载相对应缺失组件即可

