---
title: "linux命令"
date: 2022-09-24T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- 其他
tags: 
- linux
---

# 后台运行相关命令

> nohup: 不受当前终端关闭的影响,就算当前终端关闭命令也一样会运行不会被迫终止
```shell
### 只输出正常内容
nohup command > filePath &

### 错误也输出
nohup command > filePath 2>1& &

### 实例
nohup minikube dashboard > dashboard.log 2>&1 &
```

## 查看后台运行任务
```shell
# jobs只能查看当次的
jobs

# 列出用户名下所有的进程，包括后台任务
ps -ef | grep username

### 输出实例,其中的中括号即为job_id
[1]-  Running                 nohup minikube dashboard > dashboard.log 2>&1 &
[2]+  Running                 nohup kubectl proxy --port=9898 --address='10.220.10.50' --accept-hosts='.*' > proxy.log 2>&1 &
```

## 关闭任务
```shell
kill %job_id
# 强制关闭
kill -9 %job_id 
```

## 后台任务调度到前台
> 下 Ctrl + Z 组合键将任务暂停并放入后台
```shell
fg %job_id
```

## 运行后台Stopped任务
> Ctrl + Z后任务会进入Stopped状态
```shell
bg %job_id
```

# 查找

## 查找命令或可执行的文件名
```shell
whereis <command/file>

### 例如
whereis ls
```

## 查找文件位置
> 是在指定目录及其子目录中进行递归搜索，可以根据多个条件进行搜索，功能比较强大。
```shell
find <path> <options> <expression>

### 例如
find . -name "file.txt"
```

## 查找文件内信息
> 是在文本文件中搜索指定模式，可以使用正则表达式进行高级匹配，适合于对文件内容进行查找。
> 参数
> -i：忽略大小写。
> -r：递归地搜索目录中的文件。
> -v：反向匹配，显示不包含指定字符串的行。
> -n：显示匹配行的行号
```shell
grep <pattern> <file>

### 例如,会显示这一行并把查找信息加红
grep "jlk" asd.txt 
```

# 传输文件
> 目录则加 -r, scp -r
```shell
### 将本地文件复制到远程主机
scp local_file remote_username@remote_ip:remote_folder
scp /home/user/file.txt user@192.168.1.100:/data


### 将远程文件复制到本地主机
scp remote_username@remote_ip:remote_file local_folder
scp user@192.168.1.100:/data/file.txt /home/user
```

# systemctl
```shell
### 启动一个服务
systemctl start <service>
### 停止一个服务
systemctl stop <service>
### 重启一个服务
systemctl restart <service>
### 查看服务状态
systemctl status <service>

### 查看服务列表
systemctl list-units --type=service
### 查看服务日志
journalctl -u <service>

### 启用一个服务,用于设置指定的服务在系统启动时自动启动
systemctl enable <service>
### 禁用一个服务
systemctl disable <service>
### 重新加载 systemd 并应用最新配置
systemctl daemon-reload

```


# vim
```shell
h   左移一个字符
j   下移一行
k   上移一行
l   右移一个字符
0   到行首
$   到行尾
G   跳转到文件末尾
gg  跳转到文件开头

i   在光标前插入文本
a   在光标后插入文本
o   在当前行下面插入一行并进入插入模式
A   在行尾插入文本
s   删除当前字符并进入插入模式
cw  删除从光标位置到单词结尾的内容并进入插入模式
dd  删除当前行
yy  复制当前行
p   将复制的内容粘贴到光标所在行的下一行
p   在光标下方粘贴复制或剪切的内容
u   撤销上一次操作
Ctrl+r 重做上一次撤销的操作

/word   查找字符串 "word"
n       查找下一个匹配项
N       查找上一个匹配项
:%s/old/new/g  全局替换 old 字符串为 new 字符串

:n      跳转到第n行
:w      保存文件
:q      退出 Vim 编辑器
:q!     强制退出 Vim 编辑器，放弃更改
:wq     保存并退出 Vim 编辑器
```

# 系统信息
```shell
### 显示系统信息。
uname -a

### 显示当前用户的用户名。
whoami

### 显示系统资源使用情况和运行中的进程。
top

### 显示文件系统的磁盘空间使用情况。
df

### 显示系统内存使用情况。
free

###  显示cpu信息
lscpu

### 显示全部cpu的详细信息
cat /proc/cpuinfo 

```

# 文件权限
> 命令 权限/用户/用户组 文件/目录
```shell
### 修改文件或目录的权限
chmod

### 修改文件或目录的所有者
chown

### 修改文件或目录的所属组
chgrp
```

# 用户管理
```shell
### 创建新用户,--comment 添加说明; --create-home 同时创建/home/username目录; --shell指定新用户的默认shell选项; -g 指定用户组
useradd username

### 更改用户密码
passwd username

### 显示username的用户组
groups username

### 修改username的用户组为new_group
usermod -g new_group username

usermod -g root hzyy

### 删除username用户
### -r 不删除username的主目录
userdel username

### 实例
useradd --comment '说明信息' -g group_name --create-home username --shell /bin/bash


sudo usermod -aG docker $USER
newgrp docker
```

# 防火墙

```shell
# 查看防火墙
sudo iptables -L

# -n 可以显示端口
sudo iptables -L IN_public_allow -n

# 开放端口
iptables -A IN_public_allow -p tcp --dport 端口 -j ACCEPT

# 删除端口
iptables -A IN_public_allow -p tcp --dport 端口 -j DROP

# 系统重新启动后仍然生效
iptables-save > /etc/sysconfig/iptables
```

