---
title: "gitlab-ci使用"
date: 2023-08-15T11:18:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- gitlab
- cicd
---

# GItlab预定义变量

```yml
before_script:
  - export SYSTEM_VERSION=$CI_COMMIT_REF_NAME
  - export CI_PIPELINE_ID=$CI_PIPELINE_ID
  - export CI_COMMIT_SHA=$CI_COMMIT_SHA
  - export CI_PROJECT_ID=$CI_PROJECT_ID
  - export CI_PROJECT_NAME=$CI_PROJECT_NAME
  - export CI_JOB_ID=$CI_JOB_ID
  - export CI_RUNNER_DESCRIPTION=$CI_RUNNER_DESCRIPTION
  - export CI_RUNNER_TAGS=$CI_RUNNER_TAGS
  - export CI_COMMIT_MESSAGE=$CI_COMMIT_MESSAGE
  - export CI_PIPELINE_SOURCE=$CI_PIPELINE_SOURCE

stages:
  - build

job:
  stage: build
  script:
    - echo "当前提交的分支或标签名称 $SYSTEM_VERSION"
    - echo "当前流水线的唯一ID $CI_PIPELINE_ID"
    - echo "当前提交的 SHA 标识 $CI_COMMIT_SHA"
    - echo "当前项目的唯一 ID $CI_PROJECT_ID"
    - echo "当前项目的名称 $CI_PROJECT_NAME"
    - echo "当前作业的唯一 ID $CI_JOB_ID"
    - echo "运行作业的 GitLab Runner 的描述 $CI_RUNNER_DESCRIPTION"
    - echo "运行作业的 GitLab Runner 的标签 $CI_RUNNER_TAGS"
    - echo "当前提交的提交信息 $CI_COMMIT_MESSAGE"
    - echo "触发流水线的来源 $CI_PIPELINE_SOURCE"


```



# 执行k8s命令

>项目`设置`>`变量`>`添加变量` key为`KUBECONFIG`value为`k8s配置文件,minikube的在~/.minikube/` 
>
>不配置执行命令会报错没有权限或者连接失败超时等错误

# job运行顺序

>`.pre` 无论在哪个位置都是最先运行,`.post`则是最后运行,然后按照定义的`stages`的顺序运行

```yaml
stages:
    - s1
    - s2


job0:
    stage: .pre
    script:
        - echo "pre"

job1:
    stage: s1
    script:
        - echo "job1"

job2:
    stage: s2
    script:
        - echo "job2"

job:
    stage: .post
    script:
        - echo "post"
```

# script运行顺序

> job内如果有`before_script`则不会运行公共的`before_script`,job内没有才会运行;如果`before_script`失败则整个流水线就会失败

```yml
before_script:
    - echo "b1"

job1:
    before_script:
        - echo "b2"
    script: echo "job1"
    after_script:
        - echo "a1"

job2:
    script:
        - echo "job2"

# 运行结果
echo "b2"
echo "job1"
echo "a1"

echo "b1"
echo "job2"
```
# 是否允许失败

>`allow_failure`默认为`false`当前`job`失败默认下一个job会不执行
>
>`true`则不影响下一个`job`的执行

```yml
job:
    tags:
        - tag1
    script:
        - echo "job"
    allow_failure: true
```



# job执行模式

>`when`当前job的执行模式
>
>- on_success: 默认模式,前面job执行成功才会执行
>- on_failure: 当前面job执行失败才会执行
>- always: 总是执行无论前边job成功与否
>- manual: 手动执行,执行到该job需要自己手动点击运行才会执行
>- delayed: 延时执行,`start_in`单位为秒,经过设置的时间后才会执行
>  - `start_in的值是一个字符串不是数字一定要加引号` 

```yml
job:
    tags:
        - tag1
    script:
        - echo "job"
    when: delayed
    start_in: "10"
```

# 重试

>`retry`当前job运行失败后重试次数,最大重试次数为`2`,设置为`2`如果一直失败的情况下会运行`1+重试次`也就是`三次` 

```yml
job:
  script: echo1 "hello"
  retry: 2
```

# 超时时间

>`timeout`超时时间. 可以用缩写也可以用全拼; 如果`runner超时时间 > 项目设置的超时时间 则按照runner的超时时间为准`
>
>例如: runner的超时时间设置为`1h`,timeout设置为`2h`,此时的最大超时时间按照runner的超时时间为准也就是`1h`
>
>如果runner没有设置超时时间或者超时时间小于timeout时间则按照timeout时间为准  

```yml
job:
  script: echo1 "hello"
  timeout: 1h 1m 1s
```

# 变量

>`variables`:声明变量,`kv形式`,使用变量需要加上`$`符号,在`CI Lint`中会出现无法识别变量把`$变量名`当做字符串的情况,但runner执行没有问题

```yml
variables:
  VAR: abc

job:
  script: echo "Hello, Rules! $VAR"
```



# 并行job

>`parallel` 最小为`2`最大为`50`,创建N个当前job并行运行,jobname从jobname 1/N 到jobname N/N

```yml
job:
  script: echo "hello"
  parallel: 3
```

# 限制分支标签

>`only`: 定义哪些分支的项目会被job执行
>
>`except`:定制哪些分支和标签的项目不会被job执行
>
>`only和except将会慢慢被淘汰,rules成为主流使用方式`
>
>#### `rules`: 规则按顺序评估，直到第一次匹配
>
>`文档:` https://docs.gitlab.cn/jh/ci/jobs/job_control.html
>
>- `if` 如果条件匹配
>
> - `if: $CI_PIPELINE_SOURCE == "merge_request_event"`
>
>   - | merge_request_event |       对于在创建或更新合并请求时创建的流水线       |
>     | :-----------------: | :------------------------------------------------: |
>     |        push         | 对于由 `git push` 事件触发的流水线，包括分支和标签 |
>
> - `if: $CI_COMMIT_TAG`：如果为标签推送更改
>
> - `if: $CI_COMMIT_BRANCH`：如果更改被推送到任何分支。
>
> - `if: $CI_COMMIT_BRANCH == "main"` ：如果更改被推送到 `main` 
>
>- `changes` 指定文件发生变化
>
>- `exists` 指定文件存在

```yml
job:
  script: echo "hello"
  only:
    - main
  except:
    - dev
```

```yml
job:
  script: echo "Hello, Rules!"
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual
    - changes:
      - Dockerfile
      when: manual
    - exists:
      - main.go
      when: manual
```

>`workflow`: 用来检测`管道`是否运行,`when只有always执行和never不执行两个值` 

```yml
variables:
  VAR: abc

workflow:
  rules:
    - if: '$VAR == "abc"'
      when: always
    - when: never

job:
  script: echo "Hello, Rules!"
```

# 限制runner

>`tags` 只运行在有该标签的runner上

```yml
stages:
  - build

job1:
  stage: build
  tags:
    - main-dev
  script:
    - ls
```



# 缓存

>缓存目录为home/gitlab-runner/cache

```yml
stages:
  - build
  - run

cache:
  paths:
    - target/*

job-build:
  stage: build
  script: 
    - mkdir target
    - echo "build" > target/build.txt
    - ls

job-run:
  stage: run
  script: 
    - echo "run" > target/run.txt
    - ls
    - tar -cvf target.tar target
  artifacts:
    paths:
      - target

```

# 制品

>指定文件可以在运行后下载
>
>`expose_as`: 用于在合并请求UI中公开显示
>
>`name`:制品命令
>
>`when`:什么时候会有这个制品,`on_success 默认,只有成功才会有,on_failure:失败,always:总是`
>
>`expire_in`:默认单位为秒,可以指定单位,默认为30天
>
>`paths`:提供下载的文件位置

```yml
job:
  script: echo "adsad"
  artifacts:
    expose_as: "name_1"
    name: "main"
    when: always
    expire_in: 1 week
    paths:
      - main.go
```

>`dependencies`: 获取前边job的制品下载

```yml
job:
  script: echo "adsad" >> a.txt
  artifacts:
    expose_as: "name_1"
    name: "a"
    when: always
    expire_in: 1 week
    paths:
      - a.txt

job2:
  dependencies:
    - job
  script: ls
```



# needs多阶段依赖运行

>`needs`: 指定的job运行后当前job就会运行`最多指定10个`,而不是等上一个阶段的job全部运行完才运行

```yml
stages:
  - build
  - test
  - deploy

module-a-build:
  stage: build
  script:
    - echo "hello module-a-build"
    - sleep 10

module-b-build:
  stage: build
  script:
    - echo "hello module-b-build"
    - sleep 10

module-a-test:
  stage: test
  script:
    - echo "hello module-a-test"
    - sleep 10
  needs: ["module-a-build"]

module-b-test:
  stage: test
  script: 
    - echo "ehllo module-b-test"
    - sleep 10
  needs: ["module-b-build"]
```

>`job`指定要下载制品的job名称
>
>`artifacts`指定是否下载,默认为true

```yml
module-b-test:
  stage: test
  script: 
    - echo "ehllo module-b-test"
    - sleep 10
  needs: 
    - job: "module-b-build"
      artifacts: true
```

# 引用外部ci文件

>如果引入有同名job,会被.gitlab-ci.yml覆盖

```yml
### ci/config1-ci.yml
config1-ci-job:
  stage: run
  script: echo "config1-ci"
  

### .gitlab-ci.yml
stages:
  - build
  - run

### 引用本仓库的内容
include:
  local: ci/config1-ci.yml
### 引用其他仓库内容
include:
  project: HuangZhenyu/config  // 项目位置
  ref: main					   // 分支
  file: ".gitlab-ci.yml"       // yml文件名称
### 引入官方配置
include:
  - tmplate: yml文件名称        // yml文件在官方仓库里面找,只需要文件名称就行
### 引入远程配置
include:
  - remote: http://xxxx/.gitlab-ci.yml
job:
  stage: build
  script: echo "build"
```

# 继承

>`job1继承了.jobjob1和.job都有的以job1为主,.job有的job1没得则按照.job` 

```yml
stages:
  - build
  - run

.job:
  script: echo "...job"
  stage: build

job1:
  extends: .job
  script: echo "job1"

### 合并后的job1: 
job1:
  stage: build
  script: echo "job1"

job2:
  stage: run
  script: echo "job2"
```



# docker实例

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

```

