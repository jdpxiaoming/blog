---
layout: post
title:  "Android新播放器Ijkplayer集成教程"
date:   2020-05-13 23:10:01 +0800
categories: jekyll update
---
Android新播放器Ijkplayer集成教程
===

### ijk编译环境信息 
- Ijkplayer-0.8.8
- 支持rtsp
- 支持http
- 支持hls
- 支持rtmp
- 支持h265 
- 支持arm64/armv7a

## 1. 引入私有库地址. 
```groovy
 repositories {
        maven {
            url 'http://172.16.22.18:8081/repository/maven-public/'
        }
        ...
    }

```

## 2. 在主项目中build.gradle引入以下库
```groovy
    implementation 'tv.danmaku.ijk.media:ijkplayer-view:0.8.8@aar'
    implementation 'tv.danmaku.ijk.media:ijkplayer-java:0.8.8'
    implementation 'tv.danmaku.ijk.media:ijkplayer-armv7a:0.8.8@aar'
    //看情况如果需要64位so则引入. 
    implementation 'tv.danmaku.ijk.media:ijkplayer-arm64:0.8.8@aar'

```

### 3. xml引入播放器view
```java
 <tv.danmaku.ijk.media.ijkplayerview.widget.media.IjkVideoView
            android:id="@+id/video_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="center"
            />
```

### 4. 设置路径和播放类型

```java
// init player
IjkMediaPlayer.loadLibrariesOnce(null);
IjkMediaPlayer.native_profileBegin("libijkplayer.so");

mVideoView.setVideoPath(mVideoPath, IjkVideoView.IJK_TYPE_HTTP_PLAY);
```

- 可供选择的url类型

```java
### 根据播放地址类型设置不同的类型 .

public static final int IJK_TYPE_LIVING_WATCH = 1; //实时监控，要求首开速度,延迟略高一点
public static final int IJK_TYPE_LIVING_LOW_DELAY = 2; //实时直播要求低延迟，不要求首开熟读 .
public static final int IJK_TYPE_HTTP_PLAY = 3;//录播 mp4 /hls/flv...
public static final int IJK_TYPE_FILE_PLAY = 10;//本地文件播放 .
public static final int IJK_TYPE_PLAY_DEFAULT = IJK_TYPE_LIVING_WATCH;//默认播放类型.
```

### 5. 停止播放，销毁
```java
 @Override
protected void onStop() {
        super.onStop();
        Log.i("poe","onStop()");
        if (mBackPressed || !mVideoView.isBackgroundPlayEnabled()) {
            mVideoView.stopPlayback();
            mVideoView.release(true);
            mVideoView.stopBackgroundPlay();
        } else {
            mVideoView.enterBackground();
        }
        IjkMediaPlayer.native_profileEnd();
}
```


- 想知道ijkplayer是如何编译的吗，下篇文章我们来聊聊 如何编译你的ijkplayer打开rtsp开关、h265、pcma等编码支持，敬请期待！


- 友联 & 相关开源项目源码出处. 

[GitHub - jdpxiaoming/PPlayer: ffmpeg 4.0.2静态库从0开始一个播放器的搭建，支持rtmp、rtsp、hls、本地MP4文件播放，音视频同步，直播推流](https://github.com/jdpxiaoming/PPlayer)

[GitHub - jdpxiaoming/ijkrtspdemo: ijkplayer open the rtsp & h265 surpport android demo .](https://github.com/jdpxiaoming/ijkrtspdemo)

