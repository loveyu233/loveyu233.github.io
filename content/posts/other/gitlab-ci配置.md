---
title: "gitlab-ci配置"
date: 2023-08-14T11:18:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- gitlab
- cicd
---

# Dockerfile报错

```yml
FROM golang:1.20.6-alpine3.18 AS build

WORKDIR /app

COPY . .

ENV GOPROXY=https://goproxy.cn

RUN go mod tidy && go build main.go

FROM alpine:3.18

WORKDIR /app

COPY --from=build /app/main .

EXPOSE 9999

CMD ["./main"]
```



> 当使用多阶段构建时且每个阶段的alpine版本不同会产生错误:``"xxx no such file or directory"``
>
> 这里的``"no such file or directory"`` 是指``/lib64/ld-linux-x86-64.so.2``找不到而不是``main``文件找不到
>
> 原因是go的``net``包关于域名解析有两种选项:
>
> 1. 使用纯Go解析器直接发送DNS请求给/etc/resolv.conf文件中的服务器
> 2. 使用基于CGo的解析器调用C库程序，例如getaddrinfo和getnameinfo
>
> 默认使用纯Go解析器，因为对于调用一个阻塞的DNS请求，Go仅需要消耗一个goroutine，而C程序需要消耗一个操作系统线程。但当CGo可用时（CGO_ENABLE=1），则会使用基于CGo的解析器，除非有如下情况：
>
> 1. 系统不允许程序直接发起DNS请求（OS X）
> 2. LOCALDOMAIN 环境变量存在，即使值为空
> 3. RES_OPTIONS 或者 HOSTALIASES 环境变量不为空
> 4. ASR_CONFIG 环境变量不为空（仅OpenBSD）
> 5. Go解析器没有实现 /etc/resolv.conf 或 /etc/nsswitch.conf 中指定的特性
> 6. 名称以 .local 结尾，或是一个 mDNS 名称
>
> 但有些镜像默认是用cgo所以会导致找不到文件
>
> 在go build的时候可以通过-tags netgo或-tags netcgo来指定net包使用纯Go还是CGo。于是在Dockerfile中的go build指令中添加-tags netgo参数即可解决

解决

>1. 每个阶段使用的alpine版本一致
>2. 在go build 命令添加参数``-tags netgo``即``RUN go build -tags netgo main.go``

为什么会找不到c库

>golang镜像使用的C标准库是gnu-libc。再在alpine镜像中执行ldd命令发现使用的C标准库是musl-libc。
>
>所以Go编译的可执行程序在动态链接了C库后在不同的libc库上编译和运行，自然会出现文件找不到的问题。

# docker-compose

>### `创建网络:`  
>
>```yml
>networks:
>mynet:
>driver: bridge
>```
>
>### `在每一个服务中指定使用该网络:` 
>
>``` yml
>networks:
>     - mynet
>```
>
>### 不同容器间访问直接使用`容器名称即可` 
>
>```go
>// 例如在main中连接pgsql和redis
>gorm.Open(postgres.Open("host=pgsql user=pgsql password=pgsql dbname=pgsql port=5432 TimeZone=Asia/Shanghai"))
>
>redis.NewClient(&redis.Options{
>		Addr: "redis:6379",
>	})
>```
>

```yaml
version: "1.0"

services:
  redis:
    image: "redis:latest"
    container_name: "redis"
    networks:
      - mynet
    ports:
      - "6379:6379"

  pgsql:
    image: "postgres:12"
    container_name: "pgsql"
    networks:
      - mynet
    environment:
      POSTGRES_USER: pgsql
      POSTGRES_PASSWORD: pgsql
      POSTGRES_DB: pgsql
    ports:
      - "5432:5432"

  main:
    image: test
    container_name: main
    networks:
      - mynet
    links:
      - pgsql
      - redis
    depends_on:
      - pgsql
      - redis
    ports:
      - "9001:9001"

networks:
  mynet:
    driver: bridge
```

# docker image上传

```shell
# 1.登录dockerhub
docker login

# 2. 给本地镜像打标签
docker tag local_image:tag_name username/repository:tag_name

# 3. 上传镜像
docker push username/repository:tag_name
```



# Gitlab-runner

>安装: https://docs.gitlab.cn/runner/install/linux-repository.html
>
>注册: sudo gitlab-runner register
>
>启动: sudo gitlab-runner run
>
>启动后如果gitlab显示未灰色三角形则删除gitlab-runner配置文件然后重新注册



# Gitlab CICD

没有docker权限解决办法:

```shell
usermod -a -G docker gitlab-runner
```



重新初始化现存的 Git 版本库于xxx,解决办法更新git

```shell
sudo yum -y remove git
sudo yum -y remove git-*
sudo yum -y install https://packages.endpointdev.com/rhel/7/os/x86_64/endpoint-repo.x86_64.rpm
sudo yum -y install git
git version
```

``.gitlab-ci.yml`` 

```yml
# 声明任务且按照顺序执行,相同类型的任务如果runner有剩余则并行执行
stages:
  - pre
  - build
  - run

# 声明一个工作流
workflow:
#  配置什么时候会触发流水线
  rules:
#    如果当前的 CI 合并请求目标分支名称是 "main"，则运行
    - if: '$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "main"'
      when: always
#    如果 CI 流水线源是通过推送触发的，则不运行
    - if: '$CI_PIPELINE_SOURCE  == "push"'
      when: never
#    如果 CI 流水线源是通过拉取请求触发的，则运行。
    - if: '$CI_PIPELINE_SOURCE == "pull_request"'
      when: always

pre-job:
  stage: pre
#  如果这个任务失败不会停止,而是继续下一个任务执行
  allow_failure: true
  script:
    - docker stop t1
    - docker rm t1
    - docker rmi cicd-test
#    - docker stop $(docker ps -q)
#    - docker rm -f $(docker ps -aq)
#    - docker rmi -f $(docker images -aq)

build-job:
  stage: build
  before_script:
    - echo "开始build镜像."
  after_script:
    - echo "镜像build完成."
  script:
    - docker build . -t cicd-test
#   保存镜像cicd-test 为  cicd-test.tar.gz
    - docker save cicd-test -o cicd-test.tar.gz
#   保存流水线所产出的文件，可以再流水线中下载这个文件
  artifacts:
#   位置
    paths:
#     docker load -i image.tar 使用这个镜像
      - cicd-test.tar.gz

run-job:
  stage: run
  script:
    - docker run -d -p 9999:9999 --name t1 cicd-test

push-job:
  stage: run
  script:
#   给镜像添加tag
    - docker tag cicd-test 123/cicd-test
#   上传镜像备份
    - docker push 123/cicd-test
```



# minikube

## 安装minikube

>前提安装完成docker

```shell
adduser newUser
usermod -aG sudo newUser
su - newUser
sudo groupadd docker
sudo usermod -aG docker $USER

=== 如果报错 xxx不在 sudoers 文件中。此事将被报告。
vim /etc/sudoers
=== 在root    ALL=(ALL:ALL) ALL 下添加 newUser    ALL=(ALL:ALL) ALL

=== 使用root用户执行
curl -Lo minikube https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.23.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

=== 使用创建的普通用户执行
minikube start --image-mirror-country='cn'
=== 如果失败则使用:
minikube start --registry-mirror=https://registry.docker-cn.com --image-mirror-country='cn' --kubernetes-version=v1.23.3

=== 验证
minikube kubectl -- get po -A

=== 普通用户
// 启动
minikube start
// 开启网页显示
minikube dashboard
//开启代理不然无法打开网页
kubectl proxy --address='0.0.0.0' --disable-filter=true
执行结果:
W0720 10:18:55.891119   52250 proxy.go:175] Request filter disabled, your proxy is vulnerable to XSRF attacks, please be cautious
Starting to serve on [::]:8001
打开(端口为执行后输出的端口):
http://服务器ip:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads?namespace=default
(必须添加/http:kubernetes-dashboard:/proxy/#/workloads?namespace=default,不然返回的是json数据而不是网页可视化)

=== 执行minikube 命令需要在普通用户执行
```



## 安装kubectl

>不安装则需要在使用kubectl前加上minikube

```shell
=== root用户执行
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubectl 

=== 需要在普通用户上执行
kubectl get pod
```

## 网络问题安装方式

1. 下载minikube安装包

```shell
https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
下载到本地然后scp到服务器
服务器执行
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```



2. 下载镜像

```shel
docker pull anjone/kicbase
```

3. 指定镜像启动

```shell
minikube start --vm-driver=docker --base-image="anjone/kicbase" --image-mirror-country='cn' --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' --kubernetes-version='v1.22.0' --force
```

参数	说明	值
–base-image	指定镜像，忽略SHA校验	anjone/kicbase
–vm-driver	使用docker	docker
–image-mirror-country	镜像所在的国家是中国	cn
–image-repository	镜像仓库地址为阿里云	registry.cn-hangzhou.aliyuncs.com/google_containers
–logtostderr	输出日志到stderr	
–kubernetes-version	指定kubernetes版本	v1.22.0
–force	强制使用root账户	

## 外网服务访问

> 直接访问nodeport类型的pod是无法访问成功的,因为minikube是用docker安装的,需要指定下边命令进入到minikube里面才可以访问到`minikube ssh` 
>
> 开启代理`--address 10.220.10.50`不指定默认为本机只能通过localhost或者127.0.0.1进行访问,必须指定本机ip后才可以使用ip访问到`9001:8000` 外网访问的端口:这个service的端口
>
> ```shell
> kubectl port-forward --address 10.10.10.10 service/nginx-demo 9001:8000
> ```
>
> 1. **kubectl port-forward**:
>
>   - **功能**：`kubectl port-forward` 允许您将本地端口映射到 Kubernetes 集群中运行的 Pod 的端口，从而可以直接与这些 Pod 进行交互，而无需通过公共网络。
>   - **用途**：通常用于调试和测试，以便在本地主机上访问运行在 Kubernetes 中的应用程序或服务。
>
> 2. **kubectl proxy**:
>
>   - **功能**：`kubectl proxy` 创建一个代理服务器，通过该代理服务器可以访问 Kubernetes API 服务器。它将 API 服务器的 REST API 映射到本地端口，从而允许您在不直接暴露 API 服务器的情况下与其交互。
>   - **用途**：通常用于访问 Kubernetes API 服务器，执行诸如查看集群状态、执行操作、监视事件等任务。

## minikube 外部访问可视化

>注意开放端口:  `sudo iptables -A IN_public_allow -p tcp -w --dport 9898 -j ACCEPT` 

```shell
kubectl proxy --port=9898 --address='10.220.10.50' --accept-hosts='^.*' &
```



## 镜像

```shell
gcr.dockerproxy.com/kaniko-project/executor:debug
```

# kaniko

>在本地登录dockerhub,命令:`docker login`,登录成功后会显示docker的config.json文件位置,一般为`~/.docker/config.json` 
>
>`kubectl create secret generic kaniko-secret --from-file=~/.docker/config.json` 
>
>kubectl apply -f 下边yaml的文件,这种方式的每次都是需要执行不方便,下边的runner+kaniko是完全体
>
>`k8s中配置使用效果更好` 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.dockerproxy.com/kaniko-project/executor:latest
    args: [ "--context=git://github.com/xxx/cicd.git",
            "--destination=xxx/cicd:1.0"]
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  volumes:
    - name: kaniko-secret
      secret:
        secretName: kaniko-secret
        items:
          - key: config.json
            path: config.json

```



# runner+kaniko

## 部署gitlab-runner

>官方链接:https://docs.gitlab.com/runner/install/kubernetes.html#prerequisites
>
>helm安装:https://helm.sh/zh/docs/intro/install/

```shell
helm repo add gitlab https://charts.gitlab.io

helm show values gitlab/gitlab-runner > values.yaml

# 去掉values.yaml里面的gitlabUrl 和 runnerRegistrationToken注释
# 在项目设置cicd runner注册页面找到对应的值写入即可,修改rbac:create为true
# 修改values.yaml中image镜像指定为: image = "gcr.dockerproxy.com/kaniko-project/executor:debug"

helm install --namespace default gitlab-runner -f ./values.yaml gitlab/gitlab-runner

# 删除使用
helm delete --namespace default gitlab-runner
```

## 添加权限

>runner添加权限,创建yaml文件写入下边内容,执行`kubectl apply -f 文件位置` 

```shell
apiVersion: rbac.authorization.k8s.io/v1
kind: "ClusterRole"
metadata:
  name: gitlab-runner
  labels:
    app: gitlab-runner
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["*"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingresses"]
    verbs: ["*"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["*"]
```

## 示例

```yaml
before_script:
  - export SYSTEM_VERSION=$CI_COMMIT_REF_NAME-v4.0

stages:
  - build
  - deploy

build:
  stage: build
  tags:
    - k8s
# 不需要指定image,默认就是拉取kaniko在kaniko里面执行
#  image:
#    name: gcr.dockerproxy.com/kaniko-project/executor:v1.9.0-debug
#    entrypoint: [""]
  script:
    - export CI_REGISTRY_AUTH_API_IMAGE="loveyu233/cicd:${SYSTEM_VERSION}"
    - export CI_DOCKERFILE="$CI_PROJECT_DIR/Dockerfile"
    - export CI_REGISTRY_AUTH_API="https://index.docker.io/v1/"
    - export CI_REGISTRY_USERNAME="xxx@gmail.com"
    - export CI_REGISTRY_PASSWORD="xxxxxx"
    - mkdir -p /kaniko/.docker
    - echo "{\"auths\":{\"${CI_REGISTRY_AUTH_API}\":{\"auth\":\"$(printf "%s:%s" "${CI_REGISTRY_USERNAME}" "${CI_REGISTRY_PASSWORD}" | base64 | tr -d '\n')\"}}}" > /kaniko/.docker/config.json
    - /kaniko/executor --insecure-pull --insecure --context "${CI_PROJECT_DIR}" --dockerfile "${CI_DOCKERFILE}" --destination "${CI_REGISTRY_AUTH_API_IMAGE}"

```



# k8s Yaml文件

## Pod

```yaml
# 版本
apiVersion: v1
# 资源类型
kind: Pod
# 数据
metadata:
	# 在那个名称空间下创建这个pod
    namespace: hello
    # pod的名称是什么
    name: my-pod-redis
# 信息
spec:
    containers:
    	# 容器名称
        - name: abcredis
        # 那个容器
          image: redis
```

## deployment

>apiVersion: apps/v1 表示使用的 Kubernetes API 版本是 apps/v1。
>kind: Deployment 表示定义了一个 Deployment 资源对象。
>metadata 用于定义资源对象的元数据，如标签和名称等。
>labels 是一个键值对集合，用于标识资源对象的属性。这里的 app: app1 表示将该 Deployment 标记为属于 "app1" 应用程序。
>name: app1 指定了该 Deployment 的名称为 "app1"。
>spec 用于定义资源对象的规格，即 Deployment 的具体配置。
>replicas: 3 表示要创建 3 个副本（Pod）。
>selector 是用于选择要管理的 Pod 的标签。
>matchLabels 是匹配 Pod 标签的条件，这里的 app: app1 表示只选择具有 "app: app1" 标签的 Pod。
>template 定义了要创建的 Pod 的模板。
>metadata 定义了 Pod 元数据，包括标签。
>labels 定义了 Pod 的标签，这里同样是 app: app1。
>spec 定义了 Pod 的规格。
>containers 定义了要在 Pod 中运行的容器列表。每个容器包含以下信息：
>name 定义了容器的名称，这里有两个容器，一个是名为 "app1" 的容器，另一个是名为 "nginx" 的容器。
>image 指定了容器使用的镜像。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    namespace: hello
    name: my-deploy
spec:
    replicas: 1
    selector:
        matchLabels:
            app: hello-deploy
    template:
        metadata:
            labels:
                app: hello-deploy
        spec:
            containers:
                - image: redis
                  name: h-d-p-redis
```



## Service

>apiVersion: v1 表示使用的 Kubernetes API 版本是 v1。
>kind: Service 表示定义了一个 Service 资源对象。
>metadata 用于定义资源对象的元数据，包括标签和名称等。
>labels 是一个键值对集合，用于标识资源对象的属性。这里的 app: app1 表示将该 Service 标记为属于 "app1" 应用程序。
>name: app1 指定了该 Service 的名称为 "app1"。
>spec 用于定义资源对象的规格，即 Service 的具体配置。
>selector 定义了要与之关联的 Pod 的选择器，这里的 app: app1 表示只选择具有 "app: app1" 标签的 Pod。
>ports 定义了 Service 的端口转发规则。
>port: 8000 表示 Service 所监听的端口号是 8000。
>protocol: TCP 表示使用 TCP 协议。
>targetPort: 80 表示将请求转发到 Pod 内部的 80 端口上。

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: app1
  name: app1
spec:
  selector:
    app: app1
  ports:
    - port: 8000
      protocol: TCP
      targetPort: 80
```

# deployment记录

>#### `--record 新版本被弃用`,取而代之的是使用` annotate 加 kubernetes.io/change-cause`
>
>`如果使用annotate则每次更新后都需要指定一次,不然会沿用上一次的annotate的变更描述` 

```shell
[user@practice ~]$ kubectl rollout history deployment/nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
4         kubectl set image deployment/nginx nginx=nginx:1.16.1 --cluster=minikube --record=true

[user@practice ~]$ kubectl set image deployment/nginx nginx=nginx:stable-alpine3.17
deployment.apps/nginx image updated
[user@practice ~]$ kubectl annotate deployment/nginx kubernetes.io/change-cause="image updated to 3.17"
deployment.apps/nginx annotate
[user@practice ~]$ kubectl rollout history deployment/nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
3         <none>
4         kubectl set image deployment/nginx nginx=nginx:1.16.1 --cluster=minikube --record=true
5         image updated to 3.17

[user@practice ~]$ kubectl set image deployment/nginx nginx=nginx:alpine3.18
deployment.apps/nginx image updated
[user@practice ~]$ kubectl rollout history deployment/nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
3         <none>
4         kubectl set image deployment/nginx nginx=nginx:1.16.1 --cluster=minikube --record=true
5         image updated to 3.17
6         image updated to 3.17

[user@practice ~]$ kubectl annotate deployment/nginx kubernetes.io/change-cause="image updated to 3.18"
deployment.apps/nginx annotate
[user@practice ~]$ kubectl rollout history deployment/nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
3         <none>
4         kubectl set image deployment/nginx nginx=nginx:1.16.1 --cluster=minikube --record=true
5         image updated to 3.17
6         image updated to 3.18

```

```shell
# 修改镜像
kubectl set image deployment/nginx nginx=nginx:1.16.1

# 添加修改说明,每次修改都需要执行一次
kubectl annotate deployment/nginx kubernetes.io/change-cause="image updated to 1.16.1"

# --record 被弃用不推荐,建议使用上一条来添加变更说明
kubectl set image deployment/nginx nginx=nginx:1.16.1 --record

# 查看某一次的变更详情
kubectl rollout history deployment/nginx --revision=5

# 回滚到上次
kubectl rollout undo deployment/nginx

# 回滚到指定的版本
kubectl rollout undo deployment/nginx --to-revision=2
```

