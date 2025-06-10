---
title: "git项目使用"
date: 2023-07-11T11:18:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- git
---

# 代码合并前

## git add xxx

>添加到暂存区

## git commit -m 'xxx'

>添加到本地仓库，必须添加对应的描述信息：``[feat, fix, chore, refactory, update]``
>
>1. feat：表示新增功能（feature）。当你添加新功能或实现一个新的需求时，使用该状态。
>2. fix：表示修复问题（bug fix）。当你修复代码中的错误、缺陷或漏洞时，使用该状态。
>3. chore：表示杂项任务（chore）。当你进行一些琐碎的、非功能性的变更或工作时，比如构建过程的改进、库/依赖项的更新等，使用该状态。
>4. refactor：表示重构代码（refactor）。当你对现有代码进行重构或优化，而不引入新功能或修复问题时，使用该状态。重构旨在改进代码结构、可读性或性能，而不会改变其行为。
>5. update：表示更新内容。当你对现有文档、配置文件、资源文件或其他非代码文件进行更新时，使用该状态。
>
>`` xxx: add xxxx, add xxx``
>
>``git commit -m 'feat: add new grpc method: ListUser' ``
>
>``git commit -m 'fix: #9119 (用户列表没有用户状态)' ``
>
>``git commit -m 'chore: .gitlab-ci.yml use new builder tag' ``
>
>``git commit -m 'refactory: grpc method ListUser' ``
>
>``git commit -m 'update: grpc method ListUser add cache'``

## git push origin HEAD:dev-xxx

>这个 Git 命令 `git push origin HEAD:dev-xxx` 表示将当前分支的最新提交推送到名为 `dev-xxx` 的远程分支。
>
>- `git push`: 这是 Git 命令的一部分，用于将本地的提交推送到远程仓库。
>- `origin`: 这是远程仓库的名称，通常默认为 `origin`。它是你与远程仓库建立连接时设置的别名。
>- `HEAD`: 这是一个特殊的指针，指向当前所在分支的最新提交。
>- `dev-xxx`: 这是远程分支的名称，`dev-xxx` 是一个示例，你可以根据需要替换为实际的远程分支名称。

## 在 gitlab 上发起合并请求

>1. 打开 GitLab 网站并登录到你的账号。
>2. 导航到你想要发起合并请求的项目页面。
>3. 在项目页面上方的菜单栏中，点击 "Merge Requests"（合并请求）选项卡。
>4. 在 "Merge Requests" 页面上，点击 "New merge request"（新建合并请求）按钮。
>5. 在弹出的页面中，选择源分支（通常是你自己的分支）和目标分支（通常是主分支或其他稳定的分支）。
>6. 输入合并请求的标题和描述，提供清晰的说明，使其他人能够理解你的合并请求意图和改动内容。
>7. 确认设置并点击 "Submit merge request"（提交合并请求）按钮。

如果需要修改代码,在修改代码后执行:

### git rebase -i HEAD~2

>这个 Git 命令 `git rebase -i HEAD~2` 表示要进行交互式的变基操作（interactive rebase），将当前分支最近的两个提交合并或编辑。
>
>- `git rebase`: 这是 Git 命令的一部分，用于将提交应用于一个新的基准（base）。
>- `-i`：这是 rebase 命令的选项之一，表示进行交互式的变基操作。
>- `HEAD~2`：这是相对于当前提交的引用。`HEAD` 表示当前提交，`~2` 表示向后回溯两个提交。所以 `HEAD~2` 表示当前分支最近的两个提交。
>
>通过执行这个命令，Git 会打开一个编辑器，并列出最近的两个提交。你可以在编辑器中指定需要进行的操作，如合并提交、编辑提交信息等。

### git push origin HEAD:dev-xxx -f

>``-f`` 强制推送

再次发送合并请求



# 代码合并后

>例如合并到dev分支后

## git branch -D dev-xxx

>``-D`` 强制删除之前的分支

## git fetch origin dev

>这个 Git 命令 `git fetch origin dev` 表示从远程仓库 `origin` 中下载名为 `dev` 的分支的最新提交和引用（包括分支、标签等），并将其存储在本地仓库中。
>
>- `git fetch`: 这是 Git 命令的一部分，用于从远程仓库中获取最新的变动。
>- `origin`: 这是远程仓库的名称，通常默认为 `origin`。它是你与远程仓库建立连接时设置的别名。
>- `dev`: 这是要获取的分支的名称，`dev` 是一个示例，你可以根据需要替换为实际的分支名称。
>
>通过执行这个命令，Git 会向远程仓库 `origin` 发送请求，检查该仓库中名为 `dev` 的分支是否有新的提交。如果有新的提交，Git 将把它们下载到本地仓库，但不会自动合并或更新任何分支。
>
>使用 `git fetch` 可以保持本地仓库与远程仓库同步，同时获取最新的代码变动，但并不会对本地分支进行修改。

## git checkout -b dev-xxx origin/dev

>这个 Git 命令 `git checkout -b dev-xxx origin/dev` 表示创建一个名为 `dev-xxx` 的本地分支，并切换到该分支上，同时将远程仓库 `origin` 中的 `dev` 分支作为起点。
>
>- `git checkout`: 这是 Git 命令的一部分，用于切换到不同的分支或提交。
>- `-b dev-xxx`: 这部分表示创建一个名为 `dev-xxx` 的新分支。如果该分支已经存在，则会产生失败。
>- `origin/dev`: 这是要基于的远程分支的全名。通过指定 `origin` 作为远程仓库，可以从远程仓库中拉取最新的引用。
>
>执行此命令后，Git 将会在本地仓库中创建名为 `dev-xxx` 的新分支，并将工作区切换到该分支上。该新分支的起点将被设置为远程仓库 `origin` 中的 `dev` 分支的最新提交。这样，你就可以在 `dev-xxx` 分支上进行开发和修改，而不会影响到其他分支。
>
>请注意，在切换到新分支之前，请确保当前分支中的修改已经提交或保存，以免数据丢失。

# hotfix热修复

>``热修复（Hotfix）``是指在软件或应用程序中解决紧急问题或漏洞的过程。热修复通常是针对生产环境中出现的紧急问题而进行的，目的是尽快修复问题并将补丁部署到生产环境，以减少对用户的影响。

## git fetch origin --tags

>`git fetch origin --tags` 这个 Git 命令用于从远程仓库 `origin` 下载所有标签（tags）的最新状态。
>
>- `git fetch`: 这是 Git 命令的一部分，用于从远程仓库中获取最新的变动。
>- `origin`: 这是远程仓库的名称，通常默认为 `origin`。它是你与远程仓库建立连接时设置的别名。
>- `--tags`: 这个选项表示下载远程仓库中的所有标签，而不仅仅是分支。
>
>通过执行这个命令，Git 将会向远程仓库 `origin` 发送请求，检查是否有新的标签被创建或已有标签有更新。如果有新的标签或更新的标签，Git 将把它们下载到本地仓库。
>
>通过执行 `git fetch origin --tags`，你可以将本地仓库与远程仓库的标签保持同步。这对于获取最新的标记版本、查看发布信息或跟踪特定的提交非常有用。

## git checkout -b hotfix-v1.3.4 tags/v1.3.4

>`git checkout -b hotfix-v1.3.4 tags/v1.3.4` 是一个 Git 命令，用于基于特定标签创建一个名为 `hotfix-v1.3.4` 的新分支。
>
>- `git checkout`: 这是 Git 命令的一部分，用于切换到不同的分支或提交。
>- `-b hotfix-v1.3.4`: 这部分表示创建一个名为 `hotfix-v1.3.4` 的新分支。如果该分支已经存在，则会产生失败。
>- `tags/v1.3.4`: 这是要基于的特定标签的全名。通过指定 `tags/` 前缀后跟标签名称，可以在该标签的位置创建一个新分支。
>
>执行此命令后，Git 将会在当前提交的位置创建一个名为 `hotfix-v1.3.4` 的新分支，并将工作区切换到该分支上。该新分支的起点将被设置为标签 `v1.3.4` 所指向的提交。这样，你就可以在 `hotfix-v1.3.4` 分支上进行修复工作，而不会影响到其他分支。
>
>请确保你输入的标签名称和分支名称正确无误，并且相应的标签已存在。

## 修改代码

### git push origin HEAD:hotfix-v1.3.4

>`git push origin HEAD:hotfix-v1.3.4` 这个 Git 命令用于将当前分支的提交推送到远程仓库 `origin` 的 `hotfix-v1.3.4` 分支上。
>
>- `git push`: 这是 Git 命令的一部分，用于将本地提交推送到远程仓库。
>- `origin`: 这是远程仓库的名称，通常默认为 `origin`。它是你与远程仓库建立连接时设置的别名。
>- `HEAD`: 这是一个指向当前分支最新提交的引用。它表示将当前分支的最新提交推送到远程仓库。
>- `hotfix-v1.3.4`: 这是远程仓库中的目标分支，你想要将当前分支的提交推送到这个分支上。
>
>执行此命令后，Git 将会把当前分支最新的提交推送到远程仓库 `origin` 的 `hotfix-v1.3.4` 分支上。如果该分支不存在，则会在远程仓库中创建一个新的分支。如果该分支已存在，则会覆盖远程分支上的提交。
>
>请确保你在执行此命令之前已经完成了必要的提交并检查了你要推送的目标分支。

# 过滤大文件

>一旦提交（commit）了大文件, rebase 也无法从 git 中剔出提交

## git filter repo

>



# git项目迁移

## 新建git仓库

>mkdir test
>cd test
>git init 
>touch README.md
>git add README.md
>git commit -m "first commit"
>git remote add origin https://gitee.com/debby-echo/test.git
>git push -u origin "master"



## git迁移现有项目到其他项目

>export file=xxx
>git clone http://xxxyinhao/$file.git
>cd $file
>git remote rename origin old-origin
>git remote add origin http://xxx/frontend/$file.git
>git push -u origin --all
>git push -u origin --tags
>cd ../
>
>1. `export file=xxx`：这是一个 Shell 命令，将一个名为 `file` 的环境变量设置为 `xxx`。
>2. `git clone http://xxxyinhao/$file.git`：使用 Git 克隆命令从远程仓库克隆一个代码库。URL `http://xxxyinhao/$file.git` 中的 `$file` 将被先前设置的环境变量所替代。
>3. `cd $file`：进入刚刚克隆的代码库的根目录。
>4. `git remote rename origin old-origin`：将默认远程仓库的名称 `origin` 重命名为 `old-origin`。这样做是为了方便后续添加新的远程仓库。
>5. `git remote add origin http://xxx/frontend/$file.git`：添加一个新的远程仓库，并将它命名为 `origin`。URL `http://xxx/frontend/$file.git` 是你要推送到的新远程仓库的地址。
>6. `git push -u origin --all`：推送本地所有分支到新的远程仓库 `origin`。选项 `-u` 用于将本地分支与远程分支关联起来。
>7. `git push -u origin --tags`：推送本地所有标签到新的远程仓库 `origin`。

## git修改当前项目的git地址

>git remote set-url origin http://xxx/frontend/xxx.git



# gitignore

>.gitignore可以忽略你不想上传的文件，比如doc,target,classes等等
>
>只需要在.git同目录下新增.gitignore文件，然后添加不需要上次的目录即可



## add后添加了.gitignore解决办法

### 如果已经add

>清除记录的所有文件管理信息，注意行末的点表示当前目录，一定要写 
>
>``git rm -r --cached . ``
>
>添加该仓库的所有文件，注意行末的点表示当前目录，一定要写 
>
>``git add . ``
>
>提交注释
>
> ``git commit -m "....." ``
>
>推送到仓库 
>
>``git push -u origin master``

### 已经push

>删除mydir目录
>
>``git rm -r --cached mydir ``
>
>删除mydir目录下的hello.txt文件
>
>``git rm -r --cached mydir/hello.txt ``
>
>``git commit -m "删除说明"  ``
>
>推送
>
>``git push``

>用-r参数删除目录, git rm --cached hello.txt 删除的是仓库中的文件，且本地工作区的文件会保留且不再与远程仓库发生跟踪关系，如果本地仓库中的文件也要删除则用git rm hello.txt
>
>``git update-index --assume-unchanged  .\components.d.ts``



# tag

## git tag -a v1.0 -m "版本介绍" 

>创建新的tag并添加描述信息

## git tag

>查看所有的标签

## git tag -l "v1.*"

>模糊匹配

## 为早期的提交打标签

### git log --pretty=oneline

>查看id后边需要使用

### 如果遇到中文乱码解决办法

>git config --global i18n.commitencoding utf-8  --该命令表示：提交命令的时候使用utf-8编码集提交 
>
>git config --global i18n.logoutputencoding utf-8 --该命令表示：日志输出时使用utf-8编码集显示 
>
>set LESSCHARSET=utf-8  --设置LESS字符集为utf-8

### git tag -a v1.0 -m "版本介绍" dele264

>指定提交id打标签

## git tag -d <tagname> 

>删除本地的tag

## git push origin --delete <tagname> 

>删除远程仓库的tag



# 示例

```shell                           
~/desktop/git > mkdir gittest                                                                  
~/desktop/git > cd gittest                                                                    
~/desktop/git/gittest > git init                                                               
Initialized empty Git repository in C:/Users/user/Desktop/git/gittest/.git/
~/desktop/git/gittest master > git branch -M main                                               
~/desktop/git/gittest main > git remote add origin https://github.com/loveyu233/gittest.git     
~/desktop/git/gittest main > git checkout -b dev-t                     
Switched to a new branch 'dev-t'
~/desktop/git/gittest dev-t > vim readmd                                                      
~/desktop/git/gittest dev-t ?1 > git add .                                                     
warning: in the working copy of 'readmd', LF will be replaced by CRLF the next time Git touches it
~/desktop/git/gittest dev-t +1 > git commit -m "update: readme"                                 
[dev-t (root-commit) 73fe335] update: readme
 1 file changed, 1 insertion(+)
 create mode 100644 readmd
~/desktop/git/gittest dev-t > git push origin HEAD:dev-t                                      
fatal: unable to access 'https://github.com/loveyu233/gittest.git/': Recv failure: Connection was reset
~/desktop/git/gittest dev-t > git commit -m "update: readme"                                   
On branch dev-t
nothing to commit, working tree clean
~/desktop/git/gittest dev-t > git push origin HEAD:dev-t                                       
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 216 bytes | 216.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/loveyu233/gittest.git
 * [new branch]      HEAD -> dev-t
~/desktop/git/gittest dev-t > vim readmd                                                       
~/desktop/git/gittest dev-t !1 > git add .                                                     
warning: in the working copy of 'readmd', LF will be replaced by CRLF the next time Git touches it
~/desktop/git/gittest dev-t +1 > git commit -m "update: readme1"                               
[dev-t adecc28] update: readme1
 1 file changed, 1 insertion(+), 1 deletion(-)
~/desktop/git/gittest dev-t > git push origin HEAD:dev-t                                       
fatal: unable to access 'https://github.com/loveyu233/gittest.git/': Failed to connect to github.com port 443 after 21089 ms: Couldn't connect to server
~/desktop/git/gittest dev-t > git push origin HEAD:dev-t                                       
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Writing objects: 100% (3/3), 249 bytes | 249.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/loveyu233/gittest.git
   73fe335..adecc28  HEAD -> dev-t
~/desktop/git/gittest dev-t > git branch -D dev-t                                               
error: Cannot delete branch 'dev-t' checked out at 'C:/Users/user/Desktop/git/gittest'
~/desktop/git/gittest dev-t > git checkout main                                                 
error: pathspec 'main' did not match any file(s) known to git
~/desktop/git/gittest dev-t > git fetch origin main                                             
remote: Enumerating objects: 1, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (1/1), 625 bytes | 208.00 KiB/s, done.
From https://github.com/loveyu233/gittest
 * branch            main       -> FETCH_HEAD
 * [new branch]      main       -> origin/main
~/desktop/git/gittest dev-t > git checkout main                                                 
Switched to a new branch 'main'
branch 'main' set up to track 'origin/main'.
~/desktop/git/gittest main > git branch -D dev-t                                               
Deleted branch dev-t (was adecc28).
~/desktop/git/gittest main > git checkout -b dev-t2 origin/main                                 
Switched to a new branch 'dev-t2'
branch 'dev-t2' set up to track 'origin/main'.
~/desktop/git/gittest dev-t2:main > vim readmd                                                 
~/desktop/git/gittest dev-t2:main !1 > git add .                                               
warning: in the working copy of 'readmd', LF will be replaced by CRLF the next time Git touches it
~/desktop/git/gittest dev-t2:main +1 > git commit -m "update: readmet2"                         
[dev-t2 9f5b39d] update: readmet2
 1 file changed, 1 insertion(+), 1 deletion(-)
~/desktop/git/gittest dev-t2:main >1 > git push origin HEAD:dev-t2                             
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Writing objects: 100% (3/3), 277 bytes | 277.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
remote:
remote: Create a pull request for 'dev-t2' on GitHub by visiting:
remote:      https://github.com/loveyu233/gittest/pull/new/dev-t2
remote:
To https://github.com/loveyu233/gittest.git
 * [new branch]      HEAD -> dev-t2
~/desktop/git/gittest dev-t2:main >1 > git status                                               
On branch dev-t2
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)
nothing to commit, working tree clean
~/desktop/git/gittest dev-t2:main >1 > vim readmd                                             
~/desktop/git/gittest dev-t2:main >1 !1 > git add .                                             
warning: in the working copy of 'readmd', LF will be replaced by CRLF the next time Git touches it
~/desktop/git/gittest dev-t2:main >1 +1 > git commit -m "update: readmet2abc"                   
[dev-t2 93eb93f] update: readmet2abc
 1 file changed, 2 insertions(+), 1 deletion(-)
~/desktop/git/gittest dev-t2:main >2 > git push origin HEAD:dev-t2                             
~/desktop/git/gittest dev-t2:main >2 > git rebase -i HEAD~2                                     
Successfully rebased and updated refs/heads/dev-t2.
~/desktop/git/gittest dev-t2:main >2 > git push origin HEAD:dev-t2 -f                           
fatal: unable to access 'https://github.com/loveyu233/gittest.git/': Recv failure: Connection was reset
~/desktop/git/gittest dev-t2:main >2 > git push origin HEAD:dev-t2 -f                           
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 12 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 291 bytes | 291.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/loveyu233/gittest.git
   9f5b39d..93eb93f  HEAD -> dev-t2
~/desktop/git/gittest dev-t2:main >2 > git tag                                                 
~/desktop/git/gittest dev-t2:main >2 > git tag -a v1.0 -m "msg"                                 
~/desktop/git/gittest dev-t2:main >2 > git push origin v1.0                                     
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 155 bytes | 155.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/loveyu233/gittest.git
 * [new tag]         v1.0 -> v1.0
~/desktop/git/gittest dev-t2:main >2 > git tag                                                 
~/desktop/git/gittest dev-t2:main >2 > git log --pretty=oneline                                
~/desktop/git/gittest dev-t2:main >2 > git tag -a v1.1 -m "新内容" 93eb93                       
~/desktop/git/gittest dev-t2:main >2 > git tag -d v1.0                                        
Deleted tag 'v1.0' (was 32d1adc)
~/desktop/git/gittest dev-t2:main >2 > git push origin --delete tag                             
fatal: tag shorthand without <tag>
~/desktop/git/gittest dev-t2:main >2 > git push origin --delete v1.0                           
To https://github.com/loveyu233/gittest.git
 - [deleted]         v1.0
~/desktop/git/gittest dev-t2:main >2 > git push origin v1.1                                     
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 171 bytes | 171.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/loveyu233/gittest.git
 * [new tag]         v1.1 -> v1.1  
```



