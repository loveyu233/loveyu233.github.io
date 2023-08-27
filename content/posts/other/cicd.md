---
title: "CICD"
date: 2023-08-14T11:18:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- cicd
---

# Dockerfile

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



# k8s使用

```shell
kubectl [command] [TYPE] [NAME] [flags]
```

>**command**：指定在一个或多个资源上要执行的操作。例如：create、get、describe、delete、apply等
>
>**TYPE**：指定资源类型（如：pod、node、services、deployments等）。资源类型大小写敏感，可以指定单数、复数或缩写形式。
>
>**NAME**：指定资源的名称。名称大小写敏感。**如果省略名称空间，则显示默认名称空间的资源的详细信息**或者提示：No resources found in default namespace.。
>
>**flags**：指定可选的**命令参数**。例如，可以使用 -s 或 –server标识来指定Kubernetes API服务器的地址和端口；-n指定名称空间等。

## 名称空间

```shell
kubectl get ns
kubectl get pod -n ns名称
kubectl create ns hello
kubectl delete ns hello
```



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

```shell
// 使用这个配置文件
kubectl apply -f xxx.yaml

// 查看这个pod
kubectl get pod -n hello

// 查看这个日志
kubectl logs -n hello my-pod-redis
// 查看简略信息
kubectl get pod -n hello -owide
// 查看详细信息
kubectl describe pod -n hello my-pod-redis
```

# deployment

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

>matchLabels:app: 和 labels:app: 的值需要一样
