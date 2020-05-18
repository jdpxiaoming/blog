---
layout: post
title:  "Android调用FFmpeg4.0.2的main方法"
date:   2020-05-18 17:50:01 +0800
categories: jekyll update
---
Android调用FFmpeg4.0.2的main方法
===


## 1. 将上一章中我们编译的单个so（或者多个so或者静态xx.a都行）拷贝到
```
-app
----libs
--------armeabi-v7a/libffmpeg.so
```


## 2. 拷贝include到`src/main/cpp下`
- 拷贝编译文件下的`include/libav***``

![header.png](https://xucanhui.com/2018/10/26/android-invoke-ffmpeg-cmd/header.png)

- 新建`cpp/ffmpeg`
- 拷贝FFmpeg4.0.2源码的fftools文件（`cmdutils.h.c/ffmpeg.c/ffmpeg.h/ffmpeg_filter.c/ffmpeg_opt.c`）
![ffmpeg-source.png](https://xucanhui.com/2018/10/26/android-invoke-ffmpeg-cmd/ffmpeg-source.png)
- 拷贝编译后跟目录下的`config.h`到ffmpeg下


## 3. `native-lib.cpp`改为`ffmpeg_cmd.c`

```c
JNIEXPORT jint JNICALL
Java_com_example_ffmpegtools_MainActivity_exec(JNIEnv *env, jclass clazz, jint cmdnum, jobjectArray cmdline)
{
    (*env)->GetJavaVM(env, &jvm);
    m_clazz = (*env)->NewGlobalRef(env, clazz);
    //---------------------------------C语言 反射Java 相关----------------------------------------
    //---------------------------------java 数组转C语言数组----------------------------------------
    int i = 0;//满足NDK所需的C99标准
    char **argv = NULL;//命令集 二维指针
    jstring *strr = NULL;

    if (cmdline != NULL) {
        argv = (char **) malloc(sizeof(char *) * cmdnum);
        strr = (jstring *) malloc(sizeof(jstring) * cmdnum);

        for (i = 0; i < cmdnum; ++i) {//转换
            strr[i] = (jstring)(*env)->GetObjectArrayElement(env, cmdline, i);
            argv[i] = (char *) (*env)->GetStringUTFChars(env, strr[i], 0);
        }

    }
    //---------------------------------java 数组转C语言数组----------------------------------------
    //---------------------------------执行FFmpeg命令相关----------------------------------------
    //新建线程 执行ffmpeg 命令
    ffmpeg_thread_run_cmd(cmdnum, argv);
    //注册ffmpeg命令执行完毕时的回调
    ffmpeg_thread_callback(ffmpeg_callback);

    free(strr);
    return 0;
}

```

## 4. 修改`CMakeLists.txt`

```shell
cmake_minimum_required(VERSION 3.4.1)
file(GLOB SOURCE ${CMAKE_SOURCE_DIR}/*.c ${CMAKE_SOURCE_DIR}/*.h)
#add_library(native-lib
#             SHARED
#             native-lib.cpp
#             )

add_library( # Sets the name of the library.
        ffmpeg-cmd
        # Sets the library as a shared library.
        SHARED
        # Provides a relative path to your source file(s).
        ${CMAKE_SOURCE_DIR}/ffmpeg/cmdutils.c
        ${CMAKE_SOURCE_DIR}/ffmpeg/ffmpeg.c
        ${CMAKE_SOURCE_DIR}/ffmpeg/ffmpeg_filter.c
        ${CMAKE_SOURCE_DIR}/ffmpeg/ffmpeg_opt.c
        ${CMAKE_SOURCE_DIR}/ffmpeg_cmd.c
        ${CMAKE_SOURCE_DIR}/ffmpeg_thread.c
        )
#加载动态库，作为本地库使用.
add_library(ffmpeg
        SHARED
        IMPORTED )
set_target_properties( ffmpeg
        PROPERTIES IMPORTED_LOCATION
        ../../../../libs/${CMAKE_ANDROID_ARCH_ABI}/libffmpeg.so )

set(my_lib_path ${CMAKE_SOURCE_DIR}/../../../libs/${CMAKE_ANDROID_ARCH_ABI})
#设置c++编译标记，代表当前项目使用c++编译. -L${my_lib_path}
set(CMAKE_C_FLAGS  "${CMAKE_C_FLAGS}")

# 引入头文件.
include_directories(${CMAKE_SOURCE_DIR})
include_directories(${CMAKE_SOURCE_DIR}/ffmpeg)
include_directories(${CMAKE_SOURCE_DIR}/include)

find_library(
              log-lib
              log )

#链接共享库到目标库libnative-lib中.
target_link_libraries(
                       ffmpeg-cmd
                       ffmpeg
#                        avfilter avformat avcodec avutil swresample swscale
                        android
                        z
                        OpenSLES
                       ${log-lib} )

```


## 5. 修改`ffmpeg.c`源码
- ffmpeg.c

修改main方法：

```c
// 修改前
int main(int argc, char **argv)

// 修改后
int ffmpeg_exec(int argc, char **argv)
```

在ffmpeg_cleanup函数执行结束前重新初始化：

```c
static void ffmpeg_cleanup(int ret) {
	
    // 省略其他代码...
    
    nb_filtergraphs = 0;
    nb_output_files = 0;
    nb_output_streams = 0;
    nb_input_files = 0;
    nb_input_streams = 0;
}
```

在print_report函数中添加代码实现FFmpeg命令执行进度的回调：

```c

static void print_report(int is_last_report, int64_t timer_start, int64_t cur_time) {
    
    // 省略其他代码...
    
    // 定义已处理的时长
    float mss;
    
    secs = FFABS(pts) / AV_TIME_BASE;
    us = FFABS(pts) % AV_TIME_BASE;
    // 获取已处理的时长
    mss = secs + ((float) us / AV_TIME_BASE);
    
    // 调用ffmpeg_progress将进度传到Java层，代码后面定义
    ffmpeg_progress(mss);
    
    // 省略其他代码...
}
```

- ffmpeg.h

添加ffmpeg_exec方法的声明：

```c
int ffmpeg_exec(int argc, char **argv);
```

- cmdutils.c

修改exit_program函数：

```c
void exit_program(int ret) {
    if (program_exit)
        program_exit(ret);

    // 退出线程(该函数后面定义)
    ffmpeg_thread_exit(ret);

    // 删掉下面这行代码，不然执行结束，应用会crash
    //exit(ret);
}
```



## 错误处理
- cpp编译总报错，原因c++和c不能混编，源码也改为c即可.
- hw_device_free_all等几个函数找不到，因为我们的lib没有打开avdevice

```c
打开ffmpeg.c在文件末尾追加几个空方法 . 

HWDevice *hw_device_get_by_name(const char *name) {
}

int hw_device_init_from_string(const char *arg, HWDevice **dev) {
}

void hw_device_free_all(void) {
}

int hw_device_setup_for_decode(InputStream *ist) {
}

int hw_device_setup_for_encode(OutputStream *ost) {
}

int hwaccel_decode_init(AVCodecContext *avctx){

}
```


- 解析文件流错误，明天看 58.flv无法读取文件错误. 
- 打开了ffmpeg的日志输出到anndroid_log发先是因为上面的定义的几个空方法没有正确的值返回,解决办法挨个删除没有用的方法，找到引用的地方Hwdevic改为NUll

```c
1. ffmpeg.c
/*ret = hwaccel_decode_init(s);
            if (ret < 0) {
                if (ist->hwaccel_id == HWACCEL_GENERIC) {
                    av_log(NULL, AV_LOG_FATAL,
                           "%s hwaccel requested for input stream #%d:%d, "
                           "but cannot be initialized.\n",
                           av_hwdevice_get_type_name(config->device_type),
                           ist->file_index, ist->st->index);
                    return AV_PIX_FMT_NONE;
                }
                continue;
            }*/
 /*ret = hw_device_setup_for_decode(ist);
        if (ret < 0) {
            snprintf(error, error_len, "Device setup failed for "
                     "decoder on input stream #%d:%d : %s",
                     ist->file_index, ist->st->index, av_err2str(ret));
            return ret;
        }*/
        
2. ffmpeg_opt.c

static int opt_init_hw_device(void *optctx, const char *opt, const char *arg)
{
    if (!strcmp(arg, "list")) {
        enum AVHWDeviceType type = AV_HWDEVICE_TYPE_NONE;
        printf("Supported hardware device types:\n");
        while ((type = av_hwdevice_iterate_types(type)) !=
               AV_HWDEVICE_TYPE_NONE)
            printf("%s\n", av_hwdevice_get_type_name(type));
        printf("\n");
        exit_program(0);
    } else {
        return 1;//hw_device_init_from_string(arg, NULL);
    }
}

static int opt_filter_hw_device(void *optctx, const char *opt, const char *arg)
{
    if (filter_hw_device) {
        av_log(NULL, AV_LOG_ERROR, "Only one filter device can be used.\n");
        return AVERROR(EINVAL);
    }
    filter_hw_device = NULL;//hw_device_get_by_name(arg);
    if (!filter_hw_device) {
        av_log(NULL, AV_LOG_ERROR, "Invalid filter device %s.\n", arg);
        return AVERROR(EINVAL);
    }
    return 0;
}
```

### 上面搞定了以后你应该就可以正确的执行视频格式转换了 . 


- 那么可以执行是没错，如何停止正在执行命令类似`ctrl+c`
- 查看改造`ffmpeg.c`

```c
[static] void
sigterm_handler(int sig)
{
    int ret;
    received_sigterm = sig;
    received_nb_signals++;
    term_exit_sigsafe();
    if(received_nb_signals > 3) {
        ret = write(2/*STDERR_FILENO*/, "Received > 3 system signals, hard exiting\n",
                    strlen("Received > 3 system signals, hard exiting\n"));
        if (ret < 0) { /* Do nothing */ };
        exit(123);
    }
}

- 去掉sigterm_handler方法的修饰符`static`
- 在ffmpeg.h中定义方法`void sigterm_handler(int sig);`
- 在jni代码中调用 `sigterm_handler(SIGINT);` 如下所示：

/**
 * 取消线程
 */
void ffmpeg_thread_cancel(){
    void *ret=NULL;
    LOGE("ffmpeg_thread_cancel: stop the main looper transcode with code :%d ",SIGINT);
    sigterm_handler(SIGINT);
    pthread_join(ntid, &ret);
}
```

### 完整代码[Github source code ](https://github.com/jdpxiaoming/FFmpegTools/)

- [GitHub - jdpxiaoming/FFmpegTools: use ffmpege as the binary file use commands params to run main funcition , picture and videos acitons as you licke.](https://github.com/jdpxiaoming/FFmpegTools/)
