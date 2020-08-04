---
layout: post
title:  "Git常用语法"
date:   2020-06-11 21:10:01 +0800
categories: jekyll update
---
Git常用语法
===

# 修改远程地址库
```shell
方法一
进入git_test根目录
git remote 查看所有远程仓库， git remote xxx 查看指定远程仓库地址
git remote set-url origin http://192.168.100.235:9797/john/git_test.git

方法二 通过命令先删除再添加远程仓库
进入git_test根目录
git remote 查看所有远程仓库， git remote xxx 查看指定远程仓库地址
git remote rm origin
git remote add origin http://192.168.100.235:9797/john/git_test.git
```


# 切换到指定提交版本
```shell
# 查看提交的tag标记 `commit 79713684715cb22ee261aef61e910018ade6112d`
git log 

commit 79713684715cb22ee261aef61e910018ade6112d
...
#切换分支

git checkout 79713684715cb22ee261aef61e910018ade6112d
```


# 使用git同时提交代码到两个source
- ex:ijkplayer源码同时修改后提交到github和公司的gitserver

```shell
# 1. 增加多个仓库地址，都是git即可. 
git remote set-url origin --add https://github.com/jdpxiaoming/ijkrtspdemo.git
git remote set-url origin --add http://172.30.17.99/Bonobo.Git.Server/IjkPlayer_wdz.git

在 .git/config 里得到

…
[remote “origin”]
url = https://github.com/jdpxiaoming/ijkrtspdemo.git
url = http://172.30.17.99/Bonobo.Git.Server/IjkPlayer_wdz.git

# 2. 强制推送到两个地址
git push -f origin master

# 3. 同时拉取多个源的更新
git pull origin master


# 另外一种 use case，你想从 repo1 pull，但是 push 的时候要推送到 repo1 和另一个  repo2，
git remote set-url origin --add https://xxx/git
git remote set-url origin --push --add https://xxx/boke

推拉同上.
```

# 使用 git stash save "2020/05/08 15:44"保存草稿

```shell
git stash save "test-cmd-stash"
#查看stash 列表 .
git stash list
#恢复草稿.  这个指令将缓存堆栈中的第一个stash删除，并将对应修改应用到当前的工作目录下。 
git stash pop
#将缓存堆栈中的stash多次应用到工作目录中，但并不删除stash拷贝
git stash apply
#移除stash
git stash list
git stash drop stash@{0}

#删除所有缓存的stash。
git stash clear

```

# 基于master创建分支并推送到服务器
A、查看本地分支
--
>使用 git branch命令，如下：

```shell
$ git branch
* master
*标识的是你当前所在的分支。

```

B、查看远程分支
---
>命令如下：

```shell
git branch -r
```
C、查看所有分支
---
>命令如下：

```shell
git branch -a
```

2、本地创建新的分支
---
>命令如下：

```shell
git branch [branch name]
```

>例如：

```shell
git branch gh-dev
```

3、切换到新的分支
---
>命令如下：

```shell
git checkout [branch name]
```

>例如：

```shell
$ git checkout gh-dev
Switched to branch 'gh-dev'
```

4、创建+切换分支
---
>创建分支的同时切换到该分支上，命令如下：

```shell
git checkout -b [branch name]
```
>git checkout -b [branch name] 的效果相当于以下两步操作：

```shell
git branch [branch name]
git checkout [branch name]

```

5、将新分支推送到github
---

> 命令如下：

```shell
git push origin [branch name]
```

>例如：

```shell
git push origin gh-dev
```
6、删除本地分支
---

>命令如下：

```shell
git branch -d [branch name]
```

>例如：

```shell
git branch -d gh-dev
```

7、删除github远程分支
---

>命令如下：

```shell
git push origin :[branch name]
```

分支名前的冒号代表删除。
----
>例如：

```shell
git push origin :gh-dev
```

三、git提交本地代码到新分支
---
1、切换到新的分支
---
>命令如下：

```shell
git checkout [branch name]
```

>例如：

```shell
$ git checkout gh-dev

Switched to branch 'gh-dev'
```

2、添加本地需要提交代码
---
>命令如下：

```shell
git add .
```

3、提交本地代码
---
>命令如下：

```shell
git commit -m "add my code to new branchB"
```

4、push 到git仓库
---
>命令如下：

```shell
git push origin [branch name]
```

>例如：

```shell
git push origin gh-dev
```

四、 合并分支ijkplayer到主线版本modulize

```shell
#查看分支
git branch -a
#切换到主线版本
git checkout modulize
#拉取代码修改的dev分支修改到本地
git pull origin/ijkplayer
#提交修改，解决冲突
git commit xxx
#合并好的本地代码，推送上去 
git push origin/modulize 
```




- 友联 & 相关开源项目源码出处. 

- [GitHub - jdpxiaoming/PPlayer: ffmpeg 4.0.2静态库从0开始一个播放器的搭建，支持rtmp、rtsp、hls、本地MP4文件播放，音视频同步，直播推流](https://github.com/jdpxiaoming/PPlayer)

- [GitHub - jdpxiaoming/ijkrtspdemo: ijkplayer open the rtsp & h265 surpport android demo .](https://github.com/jdpxiaoming/ijkrtspdemo)

- [GitHub - jdpxiaoming/FFmpegTools: use ffmpege as the binary file use commands params to run main funcition , picture and videos acitons as you licke.](https://github.com/jdpxiaoming/FFmpegTools/)

