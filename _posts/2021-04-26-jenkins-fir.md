---
layout: post
title:  "jenkins自动打包命令行方式上传fir"
date:   2021-05-19 06:27:01 +0800
categories: jekyll update
---
jenkins自动打包命令行方式上传fir并通知企业微信群2021/04/23
===

1. 登录[传送门->fir](https://www.betaqr.com/apps)获取api token `xxxx`

先说结论：fir自从换了域名以后老的jenkins插件就失效了，pass!后面更精彩
--

下载插件
Jenkins 插件下载地址 http://7xju1s.com1.z0.glb.clouddn.com/fir-plugin-1.9.5.hpi

安装插件
进入 Jenkins 管理界面后，点击左侧进入 系统管理
![38768cf7.png](/assets/blog_res/38768cf7.png)


然后找到 管理插件 并点击进入
![83410998.png](/assets/blog_res/83410998.png)

进入插件管理后，点击 高级 选项卡
![56dfe837.png](/assets/blog_res/56dfe837.png)


然后在页面找到 上传插件，选择已下载好的 fir.im jenkins 插件文件路径，并点击 上传 等待安装成功。
![2a74d522.png](/assets/blog_res/2a74d522.png)

安装成功后，如果没有创建 Jenkins 项目，请先创建项目。如果需要配置已存在的项目，请进入在 配置 中找到 增加构建后操作步骤 ，并选择 Upload to fir.im 添加到 Jenkins 项目中。
![c91a88e3.png](/assets/blog_res/c91a88e3.png)

添加成功后开始配置各种参数，如图显示：
![f3266936.png](/assets/blog_res/f3266936.png)

配置插件
fir.im Token（必填）
fir.im Token 查看方法：直接点击 API token (http://fir.im/apps/apitoken) 进行查看.
![cef17a16.png](/assets/blog_res/cef17a16.png)


## but，因为fir更换域名所以插件无效了,采用命令行编译自动化部署方式. 

fastlane
=== 

[fastlane docs](https://docs.fastlane.tools/)
>fastlane is the easiest way to automate beta deployments and releases for your iOS and Android apps. 🚀 It handles all tedious tasks, like generating screenshots, dealing with code signing, and releasing your application.


但是fastlane和jenkins的编译重合了，pass!抬走，下一位
--

`fir-cli`命令行工具
===

[[教程] 如何在 jenkins 里使用 fir-cli 工具](http://blog.betaqr.com/use-fir-cli-in-jenkins/)

官方Github项目地址：
[GitHub - FIRHQ/fir-cli: fir.im(betaqr.com) command-line interface](https://github.com/FIRHQ/fir-cli)
[ubuntu安装rvm_坚持两年的博客-CSDN博客_ubuntu安装rvm](https://blog.csdn.net/qq_21794917/article/details/106941086)

1.ubuntu上安装rvm

```shell
1. curl -L get.rvm.io | bash -s stable
2. gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
3. curl -L get.rvm.io | bash -s stable
4. #source /usr/local/rvm/scripts/rvm
5. source /home/wdz/.rvm/scripts/rvm
6. rvm list known
```

安装`gpg2`

```shell
#下面的命令安装失败：404  Not Found [IP: 91.189.91.38 80] 就执行update。 
sudo apt-update
sudo apt install gnupg2
# 公钥问题配置. 
gpg2 --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB


```

![3515906a.png](/assets/blog_res/3515906a.png)

安装 ruby(这个是rvm专用)

继续运行下列指令 安装 ruby 2.6.2 版本 (>= 2.4.2 版本均可)

```shell
# 国内用户如果想加速运行, 则可以执行下列语句使用rubychina 的镜像
# echo "ruby_url=https://cache.ruby-china.com/pub/ruby" > ~/.rvm/user/db  

rvm install 2.6.2

rvm use 2.6.2 --default

# 经过一阵子等待, 会安装完毕. 国内会比较慢
#如果过程中提示需要一些依赖软件, 则需要sudo yum install 那些需要依赖的软件, 或者 再开一个terminal 用root 身份运行yum install
```

![0ef22d65.png](/assets/blog_res/550c80a0.png)


安装`fir-cli`

```shell
gem install fir-cli

fir help  
```

![b872c573.png](/assets/blog_res/b872c573.png)


配置`jenkins.`


host内登录fir

```shell
fir login xxxx(token)
```

![5e36b7c6.png](/assets/blog_res/5e36b7c6.png)


编辑`jenkins.`

```shell
#!/bin/bash --login
fir help  
fir publish /home/wdz/.jenkins/workspace/ovopark/apk/*.apk -c "jenkins自动打包上传"
```


创建企业微信机器人：
选择企业微信群-》右键-》添加机器人获取hook地址：

```shell
https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxx
```

登录fir并进入应用的集成界面->添加企业微信机器人:

![1529f117.png](/assets/blog_res/1529f117.png)


最后开始编译：
查看编译控制台，如下则表示成功！
![1576e226.png](/assets/blog_res/1576e226.png)
![5de7366f.png](/assets/blog_res/5de7366f.png)


最后看下企业微信群成功收到一条消息，形成闭环，恭喜你！
![78230699.png](/assets/blog_res/78230699.png)




- 友联 & 相关开源项目源码出处. 

- [GitHub - jdpxiaoming/PPlayer: ffmpeg 4.0.2静态库从0开始一个播放器的搭建，支持rtmp、rtsp、hls、本地MP4文件播放，音视频同步，直播推流](https://github.com/jdpxiaoming/PPlayer)

- [GitHub - jdpxiaoming/ijkrtspdemo: ijkplayer open the rtsp & h265 surpport android demo .](https://github.com/jdpxiaoming/ijkrtspdemo)

- [GitHub - jdpxiaoming/FFmpegTools: use ffmpege as the binary file use commands params to run main funcition , picture and videos acitons as you licke.](https://github.com/jdpxiaoming/FFmpegTools/)

