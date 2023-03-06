---
title: "K8s命令"
date: 2022-08-14T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- 其他
tags: 
- k8s
- docker
---

# 命令

## kubectl语法

```she
kubectl [command] [TYPE] [NAME] [flags]
```

**command**：指定在一个或多个资源上要执行的操作。例如：create、get、describe、delete、apply等

**TYPE**：指定资源类型（如：pod、node、services、deployments等）。资源类型大小写敏感，可以指定单数、复数或缩写形式。

**NAME**：指定资源的名称。名称大小写敏感。**如果省略名称空间，则显示默认名称空间的资源的详细信息**或者提示：No resources found in default namespace.。

**flags**：指定可选的**命令参数**。例如，可以使用 -s 或 --server标识来指定Kubernetes API服务器的地址和端口；-n指定名称空间等。

## 查看节点详情

必须在主节点执行才可以

```shell
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS   ROLES                  AGE   VERSION
k8s-master   Ready    control-plane,master   14h   v1.20.2
k8s-node1    Ready    <none>                 72m   v1.20.2
k8s-node2    Ready    <none>                 72m   v1.20.2
```



## 查看已有应用的状态

```shell
[root@k8s-master ~]# kubectl get pods -A
NAMESPACE              NAME                                         READY   STATUS             RESTARTS   AGE
kube-system            calico-kube-controllers-56c7cdffc6-ctfpc     1/1     Running            1          1s
kube-system            calico-node-6fb2l                            1/1     Running            1          1s
...
```



## 子节点加入获取令牌

```shell
[root@k8s-master ~]# kubeadm token create --print-join-command
kubeadm join 10.211.55.10:6443 --token 5frj0r.0g2r1z32yddtgxcq     --discovery-token-ca-cert-hash sha256:98448e33982b3684b0870d543eff9c9490e749588caef088f8bfe144d4f2166f
```





## 可视化界面获取令牌

```shell
[root@k8s-master ~]# kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
eyJhbGciOiJSUzI1NiIsImtpZCI6IjRWVGpqaDZLVGdURnljdzE3OXpNa0JJOXZwaTNPR3pQMGpjU09xUmxfMUEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWNydjdnIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmOTgwNWVmNi05ZmI3LTRlMDItOTNiNC00ODllOGJjZWJmYzAiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.qroSYwa6isw4BG7VnoPfS-GH1fR5TjCJKJ0eGTXZUJPJZJjNuxtHMvXSf4EwgXsScHpkgLqX0by6CaMwrkzUwHvqYWdheL2_JSzoO7BnCsNGqA03sjzazPZlI8XoVqiy0NVg8rf561EWxZmFpPlAd4t_yjIM80x3draY-hfWcDqZZeYYQcZPJ8_bJ8OXTLeuKlQOn1WeZ7wifmc7njF5hLYMrFUXlPXBm21kis2zVpYF-K-JG1Y3VZurZDqhNIhv-fIJ-TwiEMonTNiGy-wF_uha_bA7NRg4XmXs6NNw0Ykmn3LdQXvfsO2qByiz1Z0lt8d9TrfzBeC-j8oIy0qxjw[root@k8s-master ~]#
```



## NameSpace

>名称空间，用来对集群资源进行隔离划分，默认只隔离资源不隔离网络

### 查看当前有哪些名称空间

```shell
[root@k8s-master ~]# kubectl get ns
NAME                   STATUS   AGE
default                Active   15h
kube-node-lease        Active   15h
kube-public            Active   15h
kube-system            Active   15h
kubernetes-dashboard   Active   74m
```



### 查看具体某一个名称

kubernetes-dashboard：指定的nameSpace，比如要查看kube-public，则是kubectl get pod -n kube-public

```shell
[root@k8s-master ~]# kubectl get pod -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-66dd8bdd86-m62wl   1/1     Running   0          77m
kubernetes-dashboard-5c4b99db7-99bxt         1/1     Running   0          77m
```



### 创建名称空间

#### 使用命令行的方式：

```shell
[root@k8s-master ~]# kubectl create ns hello
namespace/hello created
```

#### 使用yaml配置文件的方式

```she
vi hello.yaml
写入：
apiVersion: v1
kind: Namespace
metadata:
  name: hello
应用：
[root@k8s-master ~]# kubectl apply -f hello.yaml
namespace/hello created
```



### 删除具体名称空间

>当删除这个名称空间的时候该名称空间内的资源也会被删除

#### 使用命令删除

```shell
[root@k8s-master ~]# kubectl delete ns hello
namespace "hello" deleted
```



#### 删除配置文件创建的名称空间

```shell
[root@k8s-master ~]# kubectl delete -f hello.yaml
namespace "hello" deleted
```



## 修改配置资源

```
kubectl edit 类型 配置文件名称 -n 所在组
```



## Pod

>运行中的一组容器，Pod时k8s中应用的最小单位
>
>一个pod里面可以运行多个容器

### 创建pod

#### 命令行创建

> ContainerCreating：正在创建中；Running：运行中

```shell
[root@k8s-master ~]# kubectl run mynginx --image=nginx
pod/mynginx created
```

#### 配置文件创建

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mynginx
  name: mynginx
#  namespace: default
spec:
  containers:
  - image: nginx
    name: mynginx
```



### 删除pod

>如果这个pod不是在默认的名称空间则：kubectl delete pod mynginx -n 名称空间

```shell
[root@k8s-master ~]# kubectl delete pod mynginx
pod "mynginx" deleted
```



### 查看pod

```shell
[root@k8s-master ~]# kubectl get pod
NAME      READY   STATUS              RESTARTS   AGE
mynginx   0/1     ContainerCreating   0          8s
```



### 查看这个pod的详情

#### 详细信息

>mynginx：pod名字

```shell
[root@k8s-master ~]# kubectl describe pod mynginx
Name:         mynginx
Namespace:    default
Priority:     0
Node:         k8s-node2/10.211.55.13
Start Time:   Fri, 05 Aug 2022 12:32:59 +0800
Labels:       run=mynginx
Annotations:  cni.projectcalico.org/containerID: a57d93ceef8261e45572be09ecea789f976904a2d658a01a3c25f934be8eb1e1
              cni.projectcalico.org/podIP: 10.244.169.130/32
              cni.projectcalico.org/podIPs: 10.244.169.130/32
Status:       Running
IP:           10.244.169.130
IPs:
  IP:  10.244.169.130
Containers:
  mynginx:
    Container ID:   docker://3fd9c2a38e9442cc869f246348e6113e5ad60bf04403d7d25476603d6776571d
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:ecc068890de55a75f1a32cc8063e79f90f0b043d70c5fcf28f1713395a4b3d49
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 05 Aug 2022 12:33:17 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-gtnwl (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-gtnwl:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-gtnwl
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  102s       default-scheduler  Successfully assigned default/mynginx to k8s-node2
  Normal  Pulling    <invalid>  kubelet            Pulling image "nginx"
  Normal  Pulled     <invalid>  kubelet            Successfully pulled image "nginx" in 16.394112095s
  Normal  Created    <invalid>  kubelet            Created container mynginx
  Normal  Started    <invalid>  kubelet            Started container mynginx
```

>只看下面这些就可以
>
>```shell
>Events:
>Type    Reason     Age        From               Message
>----    ------     ----       ----               -------
>Normal  Scheduled  102s       default-scheduler  Successfully assigned default/mynginx to k8s-node2
>Normal  Pulling    <invalid>  kubelet            Pulling image "nginx"
>Normal  Pulled     <invalid>  kubelet            Successfully pulled image "nginx" in 16.394112095s
>Normal  Created    <invalid>  kubelet            Created container mynginx
>Normal  Started    <invalid>  kubelet            Started container mynginx
>```
>
>default/mynginx to k8s-node2：创建pod在k8s-node2这个节点中
>
>```shell
>[root@k8s-node2 ~]# docker ps|grep mynginx
>3fd9c2a38e94        nginx                                                "/docker-entrypoint.…"   4 minutes ago       Up 4 minutes                            k8s_mynginx_mynginx_default_d14fec40-728e-4247-82a2-57af1d8fc3c5_0
>a57d93ceef82        registry.aliyuncs.com/google_containers/pause:3.2    "/pause"                 5 minutes ago       Up 5 minutes                            k8s_POD_mynginx_default_d14fec40-728e-4247-82a2-57af1d8fc3c5_0
>```
>
>



#### 简略信息

```shell
[root@k8s-master ~]# kubectl get pod -owide
NAME      READY   STATUS    RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
mynginx   1/1     Running   0          5m11s   10.244.169.131   k8s-node2   <none>           <none>
```

>[root@k8s-master ~]# curl 10.244.169.131
>hello world

### 查看pod的日志

```shell
[root@k8s-master ~]# kubectl logs mynginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/08/05 04:53:52 [notice] 1#1: using the "epoll" event method
2022/08/05 04:53:52 [notice] 1#1: nginx/1.23.1
2022/08/05 04:53:52 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/08/05 04:53:52 [notice] 1#1: OS: Linux 5.10.0-60.18.0.50.oe2203.aarch64
2022/08/05 04:53:52 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/08/05 04:53:52 [notice] 1#1: start worker processes
2022/08/05 04:53:52 [notice] 1#1: start worker process 31
2022/08/05 04:53:52 [notice] 1#1: start worker process 32
```



### 进入到pod容器里面

```shell
[root@k8s-master ~]# kubectl exec -it mynginx -- /bin/bash
root@mynginx:/#
```





### 创建多资源的pod

>一个pod里面如果部署两个一样的资源的话会报错，端口占用
>
>例如：一个pod里面部署两个nginx，第一个nginx可以成功但第二个nginx就会报错，因为第一个nginx给80端口占用了所以第二个nginx无法成功部署就会报错

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: myapp
  name: myapp
spec:
  containers:
  - image: nginx
    name: nginx
  - image: tomcat:8.5.68
    name: tomcat
```

```shell
[root@k8s-master ~]# kubectl get pod
NAME      READY   STATUS    RESTARTS   AGE
myapp     2/2     Running   0          2m27s
mynginx   1/1     Running   0          21m
[root@k8s-master ~]# kubectl get pod -owide
NAME      READY   STATUS    RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
myapp     2/2     Running   0          3m55s   10.244.36.66     k8s-node1   <none>           <none>
mynginx   1/1     Running   0          23m     10.244.169.131   k8s-node2   <none>           <none>
```



>一个pod里面的多个资源是共享网络的，比如 nginx 访问 tomcat 可以用 127.0.0.1:8080来访问

```shell
oot@myapp:/usr/local/tomcat# curl 127.0.0.1:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```







## Deployment

>控制Pod，使Pod拥有多副本，自愈，扩缩容等能力
>
>使用deployment声明创建的pod正常删除后还会自动再启动一个相同的



例如：

>删除kubectl delete pod mytomcat-6f5f895f4f-zclpj后再次查看发现又出现一个mytomcat-6f5f895f4f-tr9lq

```shell
[root@k8s-master ~]# kubectl create deployment mytomcat --image=tomcat:8.5.68
deployment.apps/mytomcat created
[root@k8s-master ~]# kubectl get pod
NAME                        READY   STATUS    RESTARTS   AGE
mytomcat-6f5f895f4f-zclpj   1/1     Running   0          7s
[root@k8s-master ~]# kubectl delete pod mytomcat-6f5f895f4f-zclpj
pod "mytomcat-6f5f895f4f-zclpj" deleted
[root@k8s-master ~]# kubectl get pod
NAME                        READY   STATUS              RESTARTS   AGE
mytomcat-6f5f895f4f-tr9lq   1/1     ContainerCreating   0          11s
```



### 单副本创建

```shell
[root@k8s-master ~]# kubectl create deployment mytomcat --image=tomcat:8.5.68
deployment.apps/mytomcat created
```

### 多副本创建

>image:	镜像
>
>replicas:	副本数

命令行：

```shell
[root@k8s-master ~]# kubectl create deploy my-dep --image=nginx --replicas=3
deployment.apps/my-dep created
[root@k8s-master ~]# kubectl get deploy
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
my-dep   3/3     3            3           13s
[root@k8s-master ~]# kubectl get pod
NAME                      READY   STATUS    RESTARTS   AGE
my-dep-5b7868d854-5gdql   1/1     Running   0          79s
my-dep-5b7868d854-66rv2   1/1     Running   0          79s
my-dep-5b7868d854-bfxzf   1/1     Running   0          79s
```

配置文件：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-dep
  template:
    metadata:
      labels:
        app: my-dep
    spec:
      containers:
      - image: nginx
        name: nginx
```



### 查找

```shell
[root@k8s-master ~]# kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
mytomcat   1/1     1            1           3m21s
```



### 删除

```shell
[root@k8s-master ~]# kubectl delete deploy mytomcat
deployment.apps "mytomcat" deleted

[root@k8s-master ~]# kubectl get pod
No resources found in default namespace.
[root@k8s-master ~]# kubectl get deploy
No resources found in default namespace.
```







### 扩缩容

>replicas：值大于原本资源数呢就是扩容，小于就是缩容
>
>也可以使用命令：kubectl edit deploy my-dep；该命令会打开my-dep的yaml文件，找到replicas把值改为想要的值即可

```shell
[root@k8s-master ~]# kubectl get pod
NAME                      READY   STATUS    RESTARTS   AGE
my-dep-5b7868d854-5gdql   1/1     Running   0          79s
my-dep-5b7868d854-66rv2   1/1     Running   0          79s
my-dep-5b7868d854-bfxzf   1/1     Running   0          79s
[root@k8s-master ~]# kubectl scale deploy/my-dep --replicas=5
deployment.apps/my-dep scaled
[root@k8s-master ~]# kubectl get deploy
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
my-dep   5/5     5            5           11m
[root@k8s-master ~]# kubectl get pod
NAME                      READY   STATUS    RESTARTS   AGE
my-dep-5b7868d854-5gdql   1/1     Running   0          11m
my-dep-5b7868d854-66rv2   1/1     Running   0          11m
my-dep-5b7868d854-bfxzf   1/1     Running   0          11m
my-dep-5b7868d854-g7465   1/1     Running   0          15s
my-dep-5b7868d854-tqjlf   1/1     Running   0          15s
```





## 自愈后故障转移

>自愈：当一个pod出问题后自动重启尝试解决问题
>
>故障转移：例如当一个pod的物理机直接死掉了k8s就会在存在的别的node上再次创建对应pod

```shell
[root@k8s-master ~]# kubectl get pod -w
NAME                      READY   STATUS   RESTARTS   AGE
my-dep-5b7868d854-66rv2   0/1     Error    0          3d21h
my-dep-5b7868d854-g7465   0/1     Error    0          3d21h
my-dep-5b7868d854-jv5xh   0/1     Error    2          3d21h
my-dep-5b7868d854-g7465   0/1     Error    0          3d21h
my-dep-5b7868d854-g7465   0/1     Error    0          3d21h
my-dep-5b7868d854-66rv2   0/1     Error    0          3d22h
my-dep-5b7868d854-66rv2   0/1     Error    0          3d22h
my-dep-5b7868d854-g7465   1/1     Running   1          3d21h
my-dep-5b7868d854-66rv2   1/1     Running   1          3d22h
my-dep-5b7868d854-jv5xh   0/1     Error     2          3d21h
my-dep-5b7868d854-jv5xh   0/1     Error     2          3d21h
my-dep-5b7868d854-jv5xh   0/1     Error     2          3d21h
my-dep-5b7868d854-jv5xh   1/1     Running   3          3d21h
```



## 滚动更新

>启动一个新的杀死一个老的
>
>record: 记录更新版本

```shell
[root@k8s-master ~]# kubectl set image deployment/my-dep nginx=nginx:1.16.1 --record
deployment.apps/my-dep image updated
```



```shell
[root@k8s-master ~]# kubectl get pod -w
NAME                      READY   STATUS    RESTARTS   AGE
my-dep-5b7868d854-66rv2   1/1     Running   1          3d22h
my-dep-5b7868d854-g7465   1/1     Running   1          3d21h
my-dep-5b7868d854-jv5xh   1/1     Running   3          3d21h
my-dep-6b48cbf4f9-9fkjj   0/1     Pending   0          0s
my-dep-6b48cbf4f9-9fkjj   0/1     Pending   0          0s
my-dep-6b48cbf4f9-9fkjj   0/1     ContainerCreating   0          0s
my-dep-6b48cbf4f9-9fkjj   0/1     ContainerCreating   0          1s
my-dep-6b48cbf4f9-9fkjj   1/1     Running             0          29s
my-dep-5b7868d854-jv5xh   1/1     Terminating         3          3d21h
my-dep-6b48cbf4f9-dmsgs   0/1     Pending             0          0s
my-dep-6b48cbf4f9-dmsgs   0/1     Pending             0          0s
my-dep-6b48cbf4f9-dmsgs   0/1     ContainerCreating   0          0s
my-dep-5b7868d854-jv5xh   1/1     Terminating         3          3d21h
my-dep-6b48cbf4f9-dmsgs   0/1     ContainerCreating   0          0s
my-dep-5b7868d854-jv5xh   0/1     Terminating         3          3d21h
my-dep-5b7868d854-jv5xh   0/1     Terminating         3          3d21h
my-dep-5b7868d854-jv5xh   0/1     Terminating         3          3d21h
my-dep-6b48cbf4f9-dmsgs   1/1     Running             0          22s
my-dep-5b7868d854-66rv2   1/1     Terminating         1          3d22h
my-dep-6b48cbf4f9-nlbz2   0/1     Pending             0          0s
my-dep-6b48cbf4f9-nlbz2   0/1     Pending             0          0s
my-dep-6b48cbf4f9-nlbz2   0/1     ContainerCreating   0          0s
my-dep-5b7868d854-66rv2   1/1     Terminating         1          3d22h
my-dep-6b48cbf4f9-nlbz2   0/1     ContainerCreating   0          0s
my-dep-5b7868d854-66rv2   0/1     Terminating         1          3d22h
my-dep-5b7868d854-66rv2   0/1     Terminating         1          3d22h
my-dep-5b7868d854-66rv2   0/1     Terminating         1          3d22h
my-dep-6b48cbf4f9-nlbz2   1/1     Running             0          7s
my-dep-5b7868d854-g7465   1/1     Terminating         1          3d21h
my-dep-5b7868d854-g7465   1/1     Terminating         1          3d21h
my-dep-5b7868d854-g7465   0/1     Terminating         1          3d21h
my-dep-5b7868d854-g7465   0/1     Terminating         1          3d21h
my-dep-5b7868d854-g7465   0/1     Terminating         1          3d21h
```



## 查看历史记录

### 查看历史记录

```shell
[root@k8s-master ~]# kubectl rollout history deployment/my-dep
deployment.apps/my-dep
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/my-dep nginx=nginx:1.16.1 --record=true
```

### 查看某一个的详细记录

```shell
[root@k8s-master ~]# kubectl rollout history deployment/my-dep --revision=2
deployment.apps/my-dep with revision #2
Pod Template:
  Labels:	app=my-dep
	pod-template-hash=6b48cbf4f9
  Annotations:	kubernetes.io/change-cause: kubectl set image deployment/my-dep nginx=nginx:1.16.1 --record=true
  Containers:
   nginx:
    Image:	nginx:1.16.1
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```



## 版本回退

```shell
[root@k8s-master ~]# kubectl rollout undo deployment/my-dep --to-revision=1
deployment.apps/my-dep rolled back
```



```shell
[root@k8s-master ~]# kubectl get pod -w
NAME                      READY   STATUS    RESTARTS   AGE
my-dep-6b48cbf4f9-9fkjj   1/1     Running   0          6m28s
my-dep-6b48cbf4f9-dmsgs   1/1     Running   0          5m59s
my-dep-6b48cbf4f9-nlbz2   1/1     Running   0          5m37s
my-dep-5b7868d854-dpl85   0/1     Pending   0          0s
my-dep-5b7868d854-dpl85   0/1     Pending   0          0s
my-dep-5b7868d854-dpl85   0/1     ContainerCreating   0          0s
my-dep-5b7868d854-dpl85   0/1     ContainerCreating   0          1s
my-dep-5b7868d854-dpl85   1/1     Running             0          9s
my-dep-6b48cbf4f9-nlbz2   1/1     Terminating         0          6m19s
my-dep-5b7868d854-mchtm   0/1     Pending             0          0s
my-dep-5b7868d854-mchtm   0/1     Pending             0          0s
my-dep-5b7868d854-mchtm   0/1     ContainerCreating   0          0s
my-dep-6b48cbf4f9-nlbz2   1/1     Terminating         0          6m19s
my-dep-5b7868d854-mchtm   0/1     ContainerCreating   0          1s
my-dep-6b48cbf4f9-nlbz2   0/1     Terminating         0          6m20s
my-dep-6b48cbf4f9-nlbz2   0/1     Terminating         0          6m25s
my-dep-6b48cbf4f9-nlbz2   0/1     Terminating         0          6m25s
my-dep-5b7868d854-mchtm   1/1     Running             0          7s
my-dep-6b48cbf4f9-dmsgs   1/1     Terminating         0          6m48s
my-dep-5b7868d854-2t5pk   0/1     Pending             0          0s
my-dep-5b7868d854-2t5pk   0/1     Pending             0          0s
my-dep-5b7868d854-2t5pk   0/1     ContainerCreating   0          0s
my-dep-6b48cbf4f9-dmsgs   1/1     Terminating         0          6m48s
my-dep-5b7868d854-2t5pk   0/1     ContainerCreating   0          1s
my-dep-6b48cbf4f9-dmsgs   0/1     Terminating         0          6m49s
my-dep-5b7868d854-2t5pk   1/1     Running             0          7s
my-dep-6b48cbf4f9-9fkjj   1/1     Terminating         0          7m24s
my-dep-6b48cbf4f9-9fkjj   1/1     Terminating         0          7m24s
my-dep-6b48cbf4f9-9fkjj   0/1     Terminating         0          7m25s
my-dep-6b48cbf4f9-dmsgs   0/1     Terminating         0          6m57s
my-dep-6b48cbf4f9-dmsgs   0/1     Terminating         0          6m57s
my-dep-6b48cbf4f9-9fkjj   0/1     Terminating         0          7m36s
my-dep-6b48cbf4f9-9fkjj   0/1     Terminating         0          7m36s
```





## 工作负载

Deployment：无状态应用部署，比如微服务，用来提供多副本等功能

StatefulSet：有状态应用部署，比如redis，提供稳定的储存、网络等功能

DaemonSet：守护型应用部署，比如日志收集组件，在每个机器上都运行一份

Job/CronJob：定时任务部署，比如垃圾清理组件，可以在指定时间运行



## Service

>pod的流量入口
>
>将一组 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 公开为网络服务的抽象方法。pod的服务发现与负载均衡
>
>kubectl expose deploy my-dep --port=8000 --target-port=80
>service/my-dep exposed：用本机的8000端口来负载均衡my-dep里面所有pod的80端口
>
>port：暴露的端口；target-port：要代理的端口

### ClusterIP

>这种方式只能在集群内部访问,如果是这种模式的可以不指定type，直接写kubectl expose deploy my-dep --port=8000 --target-port=80

```shell
[root@k8s-master ~]# curl 10.244.169.144
111
[root@k8s-master ~]# curl 10.244.36.74
333
[root@k8s-master ~]# curl 10.244.36.75
222
[root@k8s-master ~]# kubectl expose deploy my-dep --port=8000 --target-port=80 --type=ClusterIP
service/my-dep exposed
[root@k8s-master ~]# kubectl get service
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP    4d16h
my-dep       ClusterIP   10.100.217.119   <none>        8000/TCP   16s
[root@k8s-master ~]# curl 10.100.217.119:8000
333
[root@k8s-master ~]# curl 10.100.217.119:8000
111
[root@k8s-master ~]# curl 10.100.217.119:8000
222
[root@k8s-master ~]# curl 10.100.217.119:8000
222
```



命令对应的yaml文件：

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  selector:
    app: my-dep
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
```

```shell
[root@k8s-master ~]# kubectl get pod --show-labels
NAME                      READY   STATUS    RESTARTS   AGE   LABELS
my-dep-5b7868d854-2t5pk   1/1     Running   0          25m   app=my-dep,pod-template-hash=5b7868d854
my-dep-5b7868d854-dpl85   1/1     Running   0          25m   app=my-dep,pod-template-hash=5b7868d854
my-dep-5b7868d854-mchtm   1/1     Running   0          25m   app=my-dep,pod-template-hash=5b7868d854
```



### NodePort

>可以让外部网络访问
>
>8000：为集群内部访问端口；
>
>30387：为外部访问端口，在所有的pod上都会开放这个端口

```shell
[root@k8s-master ~]# kubectl expose deployment my-dep --port=8000 --target-port=80 --type=NodePort
service/my-dep exposed
[root@k8s-master ~]# kubectl get service
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          4d17h
my-dep       NodePort    10.109.215.217   <none>        8000:30387/TCP   11s
[root@k8s-master ~]# curl 10.109.215.217:8000
333
[root@k8s-master ~]# curl 10.109.215.217:8000
222
[root@k8s-master ~]# curl 10.109.215.217:8000
333
[root@k8s-master ~]# curl 10.109.215.217:8000
333
[root@k8s-master ~]# curl 10.211.55.10:30387
333
[root@k8s-master ~]# curl 10.211.55.10:30387
111
[root@k8s-master ~]# curl 10.211.55.10:30387
222
```



## Ingress

>service的统一网关入口
>
>官网地址：https://kubernetes.github.io/ingress-nginx/
>
>Ingress就是nginx做的



```shell
[root@k8s-master ~]# wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/baremetal/deploy.yaml
[root@k8s-master ~]# kubectl apply -f deploy.yaml
```

>等待：ingress-nginx          ingress-nginx-controller-6cb6fdd64b-d7rwl    1/1     Running         这pod变为running即可

```shell
[root@k8s-master ~]# kubectl get svc -A
ingress-nginx          ingress-nginx-controller             NodePort    10.96.39.63      <none>        80:32067/TCP,443:30795/TCP   9m43s
```

>32067：这个端口为http端口；30795：这个端口为https端口



测试环境：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-server
  template:
    metadata:
      labels:
        app: hello-server
    spec:
      containers:
      - name: hello-server
        image: tomcat
        ports:
        - containerPort: 9000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - image: nginx
        name: nginx
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
spec:
  selector:
    app: nginx-demo
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-server
  name: hello-server
spec:
  selector:
    app: hello-server
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 9000
```



### 创建ingress规则

>必须有hello-server和nginx-demo这两个service

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  name: ingress-host-bar
spec:
  ingressClassName: nginx
  rules:
  - host: "hello.atguigu.com"
    http:
      paths:
      - pathType: Prefix	# 前缀模式：hello.atguigu.com/***
        path: "/"	# 这里也可以写成/abc但是如果不是/而是比的例如/abc那么hello-server这个资源下必须有这个/abc的路由才可以要不然在ingress层就直接报404，因为没有/abc这个路径所以根本到不了nginx-demo这个资源中，在ingress就直接报错了；如果有一个微服务而这个微服务都是/order/**这种的那么这里的path就可以指定为/order这样外部访问的时候就不用/order/**访问而是可以直接/**来访问了
        backend:
          service:
            name: hello-server
            port:
              number: 8000
  - host: "demo.atguigu.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: nginx-demo
            port:
              number: 8000
```





### 查看ingress规则

```shell
[root@k8s-master ~]# kubectl get ingress
NAME               CLASS   HOSTS                                ADDRESS   PORTS   AGE
ingress-host-bar   nginx   hello.atguigu.com,demo.atguigu.com             80      12s
```

>当外部网络访问hello.atguigu.com会转发到hello-server这个service上同理访问demo.atguigu.com会转发到nginx-demo这个service上



### 路径重写

>重写后：
>
>- `rewrite.bar.com/something` rewrites to `rewrite.bar.com/`
>- `rewrite.bar.com/something/` rewrites to `rewrite.bar.com/`
>- `rewrite.bar.com/something/new` rewrites to `rewrite.bar.com/new`
>
>下面例子则是将demo.atguigu.com/nginx/转为demo.atguigu.com/
>
>使用方法为：
>
>1.   annotations: nginx.ingress.kubernetes.io/rewrite-target: /$2
>2.   path: "/nginx(/|$)(.*)"

```shell
apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: ingress-host-bar
spec:
  ingressClassName: nginx
  rules:
  - host: "hello.atguigu.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: my-dep
            port:
              number: 8000
  - host: "demo.atguigu.com"
    http:
      paths:
      - pathType: Prefix
        path: "/nginx(/|$)(.*)"  # 把请求会转给下面的服务，下面的服务一定要能处理这个路径，不能处理就是404
        backend:
          service:
            name: nginx-demo
            port:
              number: 8000
```



### 流量限制

>annotations添加一下即可

```yaml
nginx.ingress.kubernetes.io/limit-rps: "1"
```



## 存储

### NFS

#### master

```shell
[root@k8s-master ~]# yum install -y nfs-utils
Last metadata expiration check: 0:33:18 ago on 2022年08月09日 星期二 17时40分07秒.
Package nfs-utils-1:2.5.4-4.oe2203.aarch64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
[root@k8s-master ~]# echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports
[root@k8s-master ~]# mkdir -p /nfs/data
[root@k8s-master ~]# systemctl enable rpcbind --now
[root@k8s-master ~]# systemctl enable nfs-server --now
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
[root@k8s-master ~]# exportfs -r
exportfs: /etc/exports [1]: Neither 'subtree_check' or 'no_subtree_check' specified for export "*:/nfs/data/".
  Assuming default behaviour ('no_subtree_check').
  NOTE: this default has changed since nfs-utils version 1.0.x

[root@k8s-master ~]# exportfs
/nfs/data     	<world>
[root@k8s-master ~]# cd /nfs/data/
[root@k8s-master data]# echo 111 > a.txt
[root@k8s-master data]# ls
a.txt
[root@k8s-master data]# cat a.txt
111
[root@k8s-master data]# cat /nfs/data/a.txt
111
222
```



#### node

```shell
[root@k8s-node1 ~]# yum install -y nfs-utils
Last metadata expiration check: 0:41:04 ago on 2022年08月09日 星期二 17时28分40秒.
Package nfs-utils-1:2.5.4-4.oe2203.aarch64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
[root@k8s-node1 ~]# showmount -e 10.211.55.10
Export list for 10.211.55.10:
/nfs/data *
[root@k8s-node1 ~]# mkdir -p /nfs/data
[root@k8s-node1 ~]# mount -t nfs 10.211.55.10:/nfs/data /nfs/data
[root@k8s-node1 ~]# cat /nfs/data/a.txt
111
[root@k8s-node1 ~]# echo 222 >> /nfs/data/a.txt
[root@k8s-node1 ~]# cat /nfs/data/a.txt
111
222
```





### 原生挂载

>挂载设置的目录本机是必须有的不然会报错挂载失败，例如挂载的为/nfs/data/nginx-pv，那么本机就必须有/nfs/data/nginx-pv这个目录

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-pv-demo
  name: nginx-pv-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-pv-demo
  template:
    metadata:
      labels:
        app: nginx-pv-demo
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
        - name: html
          nfs:
            server: 10.211.55.10
            path: /nfs/data/nginx-pv
```





## PV&PVC

>PV：持久卷，将应用需要持久化的数据保存到指定位置
>
>PVC：持久卷声明，声明需要使用的持久卷规格



```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv01-10m
spec:
  capacity:
    storage: 10M
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  nfs:
    path: /nfs/data/01
    server: 10.211.55.10
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv02-1gi
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  nfs:
    path: /nfs/data/02
    server: 10.211.55.10
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv03-3gi
spec:
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  nfs:
    path: /nfs/data/03
    server: 10.211.55.10
```



```shell
[root@k8s-master ~]# vi pv.yaml
[root@k8s-master ~]# kubectl apply -f pv.yaml
persistentvolume/pv01-10m created
persistentvolume/pv02-1gi created
persistentvolume/pv03-3gi created
[root@k8s-master ~]# kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv01-10m   10M        RWX            Retain           Available           nfs                     64s
pv02-1gi   1Gi        RWX            Retain           Available           nfs                     64s
pv03-3gi   3Gi        RWX            Retain           Available           nfs                     64s
```



```yaml
# pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
  storageClassName: nfs
```

```shell
[root@k8s-master ~]# vi pvc.yaml
[root@k8s-master ~]# kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv01-10m   10M        RWX            Retain           Available           nfs                     3m32s
pv02-1gi   1Gi        RWX            Retain           Available           nfs                     3m32s
pv03-3gi   3Gi        RWX            Retain           Available           nfs                     3m32s
[root@k8s-master ~]# kubectl apply -f pvc.yaml
persistentvolumeclaim/nginx-pvc created
[root@k8s-master ~]# kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   REASON   AGE
pv01-10m   10M        RWX            Retain           Available                       nfs                     4m
pv02-1gi   1Gi        RWX            Retain           Bound       default/nginx-pvc   nfs                     4m
pv03-3gi   3Gi        RWX            Retain           Available                       nfs                     4m
[root@k8s-master ~]# kubectl get pvc
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-pvc   Bound    pv02-1gi   1Gi        RWX            nfs            2m46s

[root@k8s-master ~]# kubectl get pvc,pv
NAME                              STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/nginx-pvc   Bound    pv02-1gi   1Gi        RWX            nfs            6m39s

NAME                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   REASON   AGE
persistentvolume/pv01-10m   10M        RWX            Retain           Available                       nfs                     10m
persistentvolume/pv02-1gi   1Gi        RWX            Retain           Bound       default/nginx-pvc   nfs                     10m
persistentvolume/pv03-3gi   3Gi        RWX            Retain           Available                       nfs                     10m
[root@k8s-master ~]#
```



## ConfigMap

>挂载配置文件用，配置文件会被保存到etcd中



```shell
[root@k8s-master ~]# vi redis.conf	# 内容为：appendonly yes
[root@k8s-master ~]# kubectl create cm redis-conf --from-file=redis.conf
configmap/redis-conf created
[root@k8s-master ~]# kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      5d
redis-conf         1      5s
[root@k8s-master ~]# rm -rf redis.conf
[root@k8s-master ~]# kubectl get cm redis-conf -oyaml
apiVersion: v1
data:	#真正的数据所在
  redis.conf: |
    appendonly yes	#这里的内容就是之前的redis.conf内容
kind: ConfigMap
metadata:
  creationTimestamp: "2022-08-09T13:11:49Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:redis.conf: {}
    manager: kubectl-create
    operation: Update
    time: "2022-08-09T13:11:49Z"
  name: redis-conf
  namespace: default
  resourceVersion: "81822"
  uid: 16d9cd57-65f2-40bb-a0c4-6440f4d8ba38
```



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    command:
      - redis-server
      - "/redis-master/redis.conf"  #指的是redis容器内部的位置
    ports:
    - containerPort: 6379
    volumeMounts:
    - mountPath: /data
      name: data
    - mountPath: /redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: redis-conf
        items:
        - key: redis.conf
          path: redis.conf
```



进入redis容器查看配置文件：

```shell
root@redis:/data# cd /redis-master/
root@redis:/redis-master# ls
redis.conf
root@redis:/redis-master# cat redis.conf 
appendonly yes
root@redis:/redis-master# 
```



>修改cm中的redis.conf文件，对应使用该配置的容器的配置文件也会修改

```shell
[root@k8s-master ~]# kubectl edit cm redis-conf
[root@k8s-master ~]# kubectl get cm redis-conf -oyaml
apiVersion: v1
data:
  redis.conf: |
    appendonly yes
    requirepass 12345
kind: ConfigMap
metadata:
  creationTimestamp: "2022-08-09T13:31:11Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data: {}
    manager: kubectl-create
    operation: Update
    time: "2022-08-09T13:31:11Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        f:redis.conf: {}
    manager: kubectl-edit
    operation: Update
    time: "2022-08-09T13:35:36Z"
  name: redis-conf
  namespace: default
  resourceVersion: "84144"
  uid: 1cdb4a39-95c2-4366-8e9b-cdecc92814c9
  
  
#进入容器查看：
root@redis:/redis-master# cat redis.conf 
appendonly yes
requirepass 12345

# requirepass没有对应值是因为没有重启redis
root@redis:/redis-master# redis-cli
127.0.0.1:6379> CONFIG GET appendonly
1) "appendonly"
2) "yes"
127.0.0.1:6379> CONFIG GET requirepass
1) "requirepass"
2) ""
127.0.0.1:6379> 
```



## Secret

>Secret 对象类型用来保存敏感信息，例如密码、OAuth 令牌和 SSH 密钥。 将这些信息放在 secret 中比放在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 的定义或者 [容器镜像](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-image) 中来说更加安全和灵活



> 创建pod，这里image指定的为dockerhub的私密image肯定是运行失败的
>
> 报错信息：Failed to pull image "hzyy/test": rpc error: code = Unknown desc = Error response from daemon: pull access denied for hzyy/test, repository does not exist or may require 'docker login'

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-nginx
spec:
  containers:
  - name: private-nginx
    image: leifengyang/guignginx:v1.0
```



### 创建Secret

```shell
##命令格式
kubectl create secret docker-registry regcred \
  --docker-server=<你的镜像仓库服务器> \
  --docker-username=<你的用户名> \
  --docker-password=<你的密码> \
  --docker-email=<你的邮箱地址>
```



```shell
[root@k8s-master ~]# kubectl create secret docker-registry hzyy-docker \
--docker-username=123 \
--docker-password=123.. \
--docker-email=123@qq.com
secret/leifengyang-docker created
[root@k8s-master ~]# kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-gtnwl   kubernetes.io/service-account-token   3      5d1h
hzyy-docker           kubernetes.io/dockerconfigjson        1      3s
[root@k8s-master ~]# kubectl get secret hzyy-docker -oyaml
apiVersion: v1
data:	#	数据已经被加密了
  .dockerconfigjson: eyJhdXRocyI6eyJod1HRwczovL2luZGV4LmRammvY2tlci5pby92MS8iOnsidX1Nl1cm5hbWUiOiJoenl5Ii7wicGFzc3dvacmQiOiJoe5nl5NjY0022Li34ihhiLCJlbWFpbCI6IjE5NTgA2zODczMjFAcX3EuY29tIiwiYXV0aCI62Im6FIcD9VlVHB3vZW5sNU5qWTJMaT3Q9In19fQ 
kind: Secret
metadata:
  creationTimestamp: "2022-08-09T14:06:46Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:.dockerconfigjson: {}
      f:type: {}
    manager: kubectl-create
    operation: Update
    time: "2022-08-09T14:06:46Z"
  name: hzyy-docker
  namespace: default
  resourceVersion: "87085"
  uid: f2e49813-3344-45d2-8f77-90de3917824a
type: kubernetes.io/dockerconfigjson
```



> 添加信息重新创建pod；添加：  imagePullSecrets: -name: hzyy-docker
>
> 再次kubectl apply 即可成功创建

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-nginx
spec:
  containers:
  - name: private-nginx
    image: hzyy/test
  imagePullSecrets:
  - name: hzyy-docker
```
