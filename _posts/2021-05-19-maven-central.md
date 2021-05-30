---
layout: post
title:  "上传到MavenCenter"
date:   2021-05-20 06:27:01 +0800
categories: jekyll update
---
上传到mavencenter2021/05/15. 
===

主要介绍利用Sonatype将jar或aar提交到Maven的中央仓库。

是不是希望将自己的jar或是aar传到maven官方库中，在The Central Repository中可以被其他人搜索使用呢，是的话，往下看吧。

1、Sonatype简介
Sonatype使用Nexus为开源项目提供托管服务。你可以通过它发布快照(snapshot)或是稳定版(release)到Maven中央仓库。我们只要注册一个Sonatype的JIRA账号、创建一个JIRA ticket，然后对POM文件稍作配置即可。

1. 申请账户

[Log in - Sonatype JIRA](https://issues.sonatype.org/login.jsp)

![f629a383.png](/assets/blog_res/f629a383.png)


打开https://issues.sonatype.org/ 注册Sonatype的JIRA账号，这个账号在后面配置maven server时需要使用。
打开Create a OSSRH ticke 创建一个JIRA ticket，你的一个项目对应着这里的一个JIRA ticket，
其中Summary可以填写项目名，Description填写项目介绍。

Group Id非常重要，必须是你项目pom.xml中的group id的父级，做为你账号和该项目关联的标记。如我项目pom.xml中group id为cn.trinea.android.common，为了我所有项目都可以发布，申请填写的Group Id为cn.trinea
其他按照提示填写即可。完成后大概2个工作日左右，该Issue会变为 _resolved_ 状态表示可用，在可用前下面的过程除了第7步 正式发布外，其他的都没有问题。

![77307b5e.png](/assets/blog_res/77307b5e.png)


![1869cf41.png](/assets/blog_res/1869cf41.png)

-接下来根据提示

When choosing a groupId that reflects your project hosting, in this case, something like io.github.jdpxiaoming would be correct.
com.github groupIds are invalid now. Please read https://central.sonatype.org/changelog/#2021-04-01-comgithub-is-not-supported-anymore-as-a-valid-coordinate for more info.

Please edit this ticket and update the Group Id field with the corrected coordinates.
Also, please create a public repo called OSSRH-68888 so we can verify Github account ownership.

- 更新groupid为`io.github.jdpxiaoming`
- 登录github创建一个公共public的项目叫做`OSSRH-68888`(等了2个小时才明白这句英文- -!,)
https://github.com/jdpxiaoming/OSSRH-68888.git

![984688d6.png](/assets/blog_res/984688d6.png)


然而我又等了一个小时还是不鸟我，那我就直接给这位大哥留言告诉他我已经创建好repo了。
![92960dd7.png](/assets/blog_res/92960dd7.png)


这次小哥很快就给我通过极了，看到这个`RESOLVED`就可以开始下一步愉快地上传aar了。
![a827910a.png](/assets/blog_res/a827910a.png)


开始上传. 
具体脚本我们在上一篇文章中已经提到过了. 
[Gradle配置上传到Nexus私服2020教程 \| poe Blog](https://blog.lxfpoe.work/jekyll/update/2020/05/13/maven-nexus.html)

Gradle/ijkplayer-armv7a/upload/upoadArchive. 执行上传命令. 

![66a668f5.png](/assets/blog_res/66a668f5.png)


登录后台：[Nexus Repository Manager](https://s01.oss.sonatype.org/)可以看到我们的发布包已经上传成功了（上传不需要挂代理2-3分钟就可以了，不可用再考虑使用代理.）


部署完成后，状态会变成Open，点击close会触发对组件的校验，如果校验成功，那么可以点击release按钮将其部署到中央仓库中。

我第一次close失败了，回去又仔细看了下指示。其中有一段是这么说的.

```c
Please comment on this ticket when you've released your first component(s), so we can activate the sync to Maven Central
```

所以我老实的issue页面提交了一个comment.
![2f98ba9f.png](/assets/blog_res/2f98ba9f.png)


然而我发现我并没有发布成功，是因为没有签名授权的asc文件，
需要秘钥设置后重新发布，没问题，马上来搞个秘钥来. 

![a0458a3c.png](/assets/blog_res/a0458a3c.png)

- 创建GPG秘钥

根据官网介绍，sonatype对安全性有要求，所以需要我们注册GPG密钥才可以发布AAR，这也是比jfrog复杂的地方

下载对应环境的pgp安装包(我是win10环境所以是下了一个win的binary)
[Gpg4win - Get Gpg4win](https://gpg4win.org/get-gpg4win.html)

- gpg3.x无法使用因为我的账户是管理员
- 解决办法：安装旧版本`gnupg-w32cli-1.4.23.exe`


```shell

#生成一个新的签名文件(可选0生成无期限签名)

gpg --gen-key

#(1) RSA 和 RSA （默认）

您的选择是？ 1

#RSA 密钥的长度应在 1024 位与 4096 位之间。

您想要使用的密钥长度？(2048) #这里直接回车

密钥的有效期限是？(0) #这里直接回车

些内容正确吗？ (y/N) y

#GnuPG 需要构建用户标识以辨认您的密钥。
真实姓名： aaa
电子邮件地址： bbb@gmail.com
注释：
您选定了此用户标识：
“aaa bbb@gmail.com”
更改姓名（N）、注释（C）、电子邮件地址（E）或确定（O）/退出（Q）？ O


#注意:这时候会有个弹窗要求输入签名密码(后续发布会用到)
>我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘
、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
发生器有更好的机会获得足够的熵。
我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘
、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
发生器有更好的机会获得足够的熵。

gpg: 密钥 15E7C6E7 被标记为绝对信任
公钥和私钥已经生成并被签名。

pub   2048R/15E7C6E7 2021-05-17
密钥指纹 = AFCF 4FCC F030 0406 3093  1ED5 54E7 3820 F28A C537
uid                  jdpxiaoming (maven central gpg sign key.) <poe.caixingming@gmail.com>
sub   2048R/45844949 2021-05-17

```
请注意上面的字符串"F28AC537"，这是"用户ID"的Hash字符串，可以用来替代"用户ID"。

列出目前的签名文件列表
```shell
gpg --list-keys

为这个密钥创建一个吊销证书(可选，最好是创建一个)
gpg --gen-revoke 15E7C6E7

#删除其中一个签名(可选)
#gpg --delete-key 85E38F69046B44C1EC9FB07B76D78F0500D026C4
#导出公钥到public-key.txt
gpg --armor --output public-key.txt --export 15E7C6E7

#导出私钥到public-key.txt
gpg --armor --output private-key.txt --export-secret-keys

#发送到公钥服务器
gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 15E7C6E7
#查询公钥F28AC537是否发布成功
gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 15E7C6E7

#对公钥生成指纹(由于任何人都可以用你的名义上传公钥，我们可以生成公钥指纹，好让他人校验)
gpg --fingerprint 15E7C6E7

#pub   2048R/15E7C6E7 2021-05-17
#密钥指纹 = AFE6 B65A 7459 1606 80F3  C4E6 A698 607A 15E7 C6E7
#uid                  caixingming (poe maven center by #jdpxiaoming.githu.io) <poe.caixingming@gmail.com>
#sub   2048R/860B99ED 2021-05-17

#导出gpg私钥
gpg --export-secret-keys -o \dir\secring.gpg
```

- 继续报错：
`No public key: Key with id: (a698607a15e7c6e7) was not able to be located on <a href="http://keyserver.ubuntu.com:11371/">http://keyserver.ubuntu.com:11371/</a>. Upload your public key and try the operation again`

分析下是因为这个公网的服务器没有找到publick key

- 重新发布公钥到这个服务器:

```c
gpg --keyserver hkp://keyserver.ubuntu.com --send-keys 15E7C6E7
gpg --keyserver hkp://keyserver.ubuntu.com --recv-keys 15E7C6E7
```


配置Gradle
```groovy

# maven centrol accout
NEXUS_USERNAME=jdpxiaoming
NEXUS_PASSWORD=xxxxx

#RSS签名秘钥信息.
signing.keyId=15E7C6E7
signing.password=xxxxx
signing.secretKeyRingFile=D\:\\gpg\\secring.gpg
```

最后登录issue网址close->release. 

如果审核正常可以到下面网址进行搜索。
[Maven Central Repository Search](https://search.maven.org/)



当然开启同步需要外国友人的帮助：
Central sync is activated for io.github.jdpxiaoming. After you successfully release, your component will be published to Central https://repo1.maven.org/maven2/, typically within 10 minutes, though updates to https://search.maven.org can take up to two hours.

Permalink Edit Delete 
jdpxiaoming
Jdpxiaoming Cai added a comment - 9 minutes ago - edited
Please help me activate the sync to Maven Central.

I have release three ijkplayer libray.

![752fa669.png](/assets/blog_res/752fa669.png)

坐等10分钟就可以了. 
![19dba6a5.png](/assets/blog_res/19dba6a5.png)

注意：
- maven sync只需要开启一次，后面发布无需重复提交issue comment.
- 有时候release后同步会比较慢，需要刷新整个浏览器缓存


参考链接：
[Working with PGP Signatures - The Central Repository Documentation](https://central.sonatype.org/publish/requirements/gpg/)

- 友联 & 相关开源项目源码出处. 

- [GitHub - jdpxiaoming/PPlayer: ffmpeg 4.0.2静态库从0开始一个播放器的搭建，支持rtmp、rtsp、hls、本地MP4文件播放，音视频同步，直播推流](https://github.com/jdpxiaoming/PPlayer)

- [GitHub - jdpxiaoming/ijkrtspdemo: ijkplayer open the rtsp & h265 surpport android demo .](https://github.com/jdpxiaoming/ijkrtspdemo)

- [GitHub - jdpxiaoming/FFmpegTools: use ffmpege as the binary file use commands params to run main funcition , picture and videos acitons as you licke.](https://github.com/jdpxiaoming/FFmpegTools/)

