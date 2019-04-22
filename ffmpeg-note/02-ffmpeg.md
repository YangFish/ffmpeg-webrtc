# FFmpeg 音视频核心技术

## 1. 学习大纲

**FFmpeg 常用命令**：

- 视频录制命令
- 多媒体文件的分解/复用命令
- 裁剪与合并命令
- 图片/视频互转命令
- 直播相关命令
- 各种滤镜命令

**FFmpeg 基本开发**：

- C 语言回顾
- FFmpeg 核心概念与常用结构体
- 实战 - 多媒体文件的分解与复用
- 实战 - 多媒体格式的互转
- 实战 - 从 MP4 裁剪一段视频
- 作业 - 实现一个简单的小咖秀

**音视频编解码实战**：

- 实战 - H264 解码
- 实战 - H264 编码
- 实战 - 音频 AAC 解码
- 实战 - 音频 AAC 编码
- 实战 - 视频转图片

**音视频渲染实战**：

- SDL 事件处理
- SDL 视频文理渲染
- SDL 音频渲染
- 实战1 - 实现 YUV 视频播放
- 实战2 - YUV 视频倍数播放
- 实战3 - 实现 PCM 播放器

**FFmpeg 开发播放器核心功能**：

- 实战 - 实现 MP4 文件的视频播放
- 实战 - 实现 MP4 文件的音频播放
- 实战 - 实现一个初级播放器
- 实战 - 音视频同步
- 实战 - 实现播放器内核

**Android 中实战 FFmpeg**：

- 编译 Android 端可以使用的 FFmpeg
- Java 与 C 语言相互调用
- 实战 - Android 调用 FFmpeg

**学习建议**：

- 牢牢抓住音视频的处理机制，了解其本质
- 勤加练习，熟能生巧
- 待着问题去学习，事半功倍

**音视频的广泛应用**：

- 直播类：音视频会议、教育直播、娱乐/游戏直播
- 短视频：抖音、快手、小咖秀
- 网络视频：优酷、腾讯视频、爱奇艺等
- 音视频通话：微信、QQ、Skype等
- 视频监控
- 人工智能：人脸识别，智能音箱等，更关注算法

**播放器架构**：

<img src="_asset/播放器架构.png">

**渲染流程**：

<img src="_asset/渲染流程.png">

**FFmpeg 都能做啥**：

- FFmpeg 是一个非常优秀的多媒体框架
- FFmpeg 可以运行在 Linux、Mac、Windows 等平台上
- 能够解码、编码、转码、复用、解复用、过滤音视频数据

**FFmpeg 下载便于与安装**：

```shell
$ git clone https://git.ffmpeg.org/ffmpeg.git
$ config -- help
$ make && make install
```

## 2. FFmpeg 常用命令实战

我们按使用目的可以将 FFMPEG 命令分成以下几类：

- 基本信息查询命令
- 录制
- 分解 / 复用
- 处理原始数据
- 滤镜
- 切割与合并
- 图／视互转
- 直播相关

除了 FFMPEG 的基本信息查询命令外，其它命令都按下图所示的流程处理音视频。

<img src="_asset/FFmpeg处理音视频流程.png">

<img src="_asset/FFmpeg基本信息查询命令.png">

<img src="_asset/FFmpeg录屏命令.png">

```shell
$ ffplay -s 2560x1600 -pix_fmt uyvy422 out.yuv
```

<img src="_asset/分解与复用-01.png">

<img src="_asset/多媒体格式转换.png">

## 3. 初级开发内容

- FFmpeg 日志的使用及目录的操作
- 介绍 FFmpeg 的基本概念及常用的结构体
- 对复用/解复用及流程操作的各种实践

FFmpeg 代码结构：

- libavcodec： 提供了一系列编码器的实现。
- libavformat： 实现在流协议，容器格式及其本IO访问。
- libavutil： 包括了hash器，解码器和各类工具函数。
- libavfilter： 提供了各种音视频过滤器。
- libavdevice： 提供了访问捕获设备和回放设备的接口。
- libswresample： 实现了混音和重采样。
- libswscale： 实现了色彩转换和缩放工能。

### 3.1 FFmpeg 日志系统

```c++
#include <libavutil/log.h>

av_log_set_level(AV_LOG_DEBUG)
    
av_log(NULL, AV_LOG_INFO, "...%s\n", op)
```

- AV_LOG_ERROR
- AV_LOG_WARNING
- AV_LOG_INFO

```c
#include <stdio.h>
#include <libavutil/log.h>

int main(int argc, char *argv[])
{
    av_log_set_level(AV_LOG_DEBUG);

    av_log(NULL, AV_LOG_INFO, "hello world: %s!\n", "aaa");

    return 0;
}
```

### 3.2 FFmpeg 文件与目录操作

文件的删除与重命名：

```c++
#include <libavformat/avformat.h>

avpriv_io_delete()
    
avpriv_io_move(src, dst)
```

```c
#include <stdio.h>
#include <libavutil/log.h>
#include <libavformat/avformat.h>

int main(int argc, char *argv[])
{
    int ret;
    ret = avpriv_io_delete("./mytestfile.txt");
    if(ret < 0) {
        av_log(NULL, AV_LOG_ERROR, "Failed to delete file mytestfile.txt\n");
        return -1
    }
    
    ret = avpriv_io_move("111.txt", "222.txt");
    if(ret < 0) {
        av_log(NULL, AV_LOG_ERROR, "Filed to rename\n");
        return -1;
    } 

    return 0;
}
```

```shell
$ clang -g -o ffmpeg_del ffmpeg_file.c `pkg-config --libs libavformat`

# pkg-config --libs libavformat 指令可以搜索libavformat库所在路径

$ pkg-config --libs libavformat
-L/usr/local/ffmpeg/lib -lavformat
```

### 3.3 FFmpeg 操作目录重要函数

```c
# 
avio_open_dir()
avio_read_dir()
avio_close_dir()
```

操作目录重要结构体：

- AVIODirContext

  操作目录的上下文

- AVIODirEntry

  目录项。用于存放文件名，文件大小等信息

```c
#include <stdio.h>
#include <libavutil/log.h>
#include <libavformat/avformat.h>

int main(int argc, char *argv[])
{
    av_log_set_level(AV_LOG_INFO);

    int ret;
    AVIODirContext *ctx = NULL;
    AVIODirEntry *entry = NULL;

    ret = avio_open_dir(&ctx, "./", NULL);
    if (ret < 0) {
        av_log(NULL, AV_LOG_ERROR, "Cant open dir:%s\n", av_err2str(ret));
        return -1;
    }
    while(1) {
        ret = avio_read_dir(ctx, &entry);
        if (ret < 0) {
            av_log(NULL, AV_LOG_ERROR, "Cant read dir: %s\n", av_err2str(ret));
            goto __fail;
        }
        if (!entry) {
            break;
        }

        av_log(NULL, AV_LOG_INFO, "%l2"PRId64" %s\n",
               entry->size,
               entry->name);

        avio_free_directory_entry(&entry);
    }
__fail:
    avio_close_dir(&ctx);
    return 0;
}
```

```shell
$ clang -g -o list ffmpeg_list.c `pkg-config --libs libavformat libavutil`
```

### 3.4 多媒体文件的基本概念

- 多媒体文件其实是个容器
- 在容器里有很多流（Stream/Track)
- 每种流是由不同的编码器编码的
- 从流中读出的数据称为包
- 在一个包中包含着一个或多个帧

几个重要的结构体：

- AVFormatContext
- AVStream
- AVPacket

FFmpeg 操作流数据的基本步骤：

解复用 —> 获取流 —> 读取数据包 —>  释放资源

### 3.5 [实战] 打印音/视频信息

```c
av_register_all()
avformat_open_input() / avformat_close_input()
av_dump_format()
```

```c
#include <stdio.h>
#include <libavutil/log.h>
#include <libavformat/avformat.h>


int main(int argc, char *argv[])
{
    int ret;
    av_log_set_level(AV_LOG_INFO);

    AVFormatContext *fmt_ctx = NULL;

    av_register_all();

    ret = avformat_open_input(&fmt_ctx, "./test.mp4", NULL, NULL);
    if (ret < 0) {
        av_log(NULL, AV_LOG_ERROR, "Can't open file: %s\n", av_err2str(ret));
        return -1;
    }

    av_dump_format(fmt_ctx, 0, "./test.mp4", 0);

    avformat_close_input(&fmt_ctx);

    return 0;
}
```

### 3.6 [实战] 抽取音频数据

```c
av_init_packet()
av_find_best_stream()
av_read_frame() / av_packet_unref()
```

```c
#include <stdio.h>
#include <libavutil/log.h>
#include <libavformat/avformat.h>

int main(int argc, char *argv[])
{
    int ret;
    int len;
    int audio_index;

    char *src = NULL;
    char *dst = NULL;

    av_log_set_level(AV_LOG_INFO);

    AVPacket pkt;
    AVFormatContext *fmt_ctx = NULL;

    av_register_all();

    // 1. read two params form console
    if (argc < 3) {
        av_log(NULL, AV_LOG_ERROR, "eg: %s in_file out_file\n", argv[0]);
        return -1;
    }
    src = argv[1];
    dst = argv[2];
    if (!src || !dst) {
        av_log(NULL, AV_LOG_ERROR, "src or dst is null\n");
        return -1;
    }

    ret = avformat_open_input(&fmt_ctx, src, NULL, NULL);
    if (ret < 0) {
        av_log(NULL, AV_LOG_ERROR, "Can't open file: %s\n", av_err2str(ret));
        return -1;
    }

    FILE *dst_fd = fopen(dst, "wb");
    if (dst_fd) {
        av_log(NULL, AV_LOG_ERROR, "Can't open out file!\n");
        avformat_close_input(&fmt_ctx);
        return -1;
    }
    av_dump_format(fmt_ctx, 0, src, 0);

    // 2. get stream
    ret = av_find_best_stream(fmt_ctx, AVMEDIA_TYPE_AUDIO, -1, -1, NULL, 0);
    if (ret < 0) {
        av_log(NULL, AV_LOG_ERROR, "Can't find the best stream!\n");
        avformat_close_input(&fmt_ctx);
        fclose(dst_fd);
        return -1;
    }

    audio_index = ret;
    av_init_packet(&pkt);
    while(av_read_frame(fmt_ctx, &pkt) >= 0) {
        if (pkt.stream_index == audio_index) {
            // 3. write audio data to aac file.
            len = fwrite(pkt.data, 1, pkt.size, dst_fd);
            if (len != pkt.size) {
                av_log(NULL, AV_LOG_WARNING, "warning, length of data is not equal size of pkt!\n");
            }
        }
        av_packet_unref(&pkt);
    }

    avformat_close_input(&fmt_ctx);
    if (dst_fd) {
        fclose(dst_fd);
    }

    return 0;
}
```

```shell
$ lang -g -o extra_audio extra_audio.c `pkg-config --libs libavutil libavformat`
$ ./extra_audio test.mp4 killer.aa
```

### 3.7 [实战] 抽取视频数据

- Start code
- SPS/PPS
- codec -> extradata

### 3.8 [实战] 将 MP4 转成 FLV 格式

```c
avformat_alloc_output_context2() / avformat_free_context();
avformat_new_stream();
avcodec_parameters_copy();
avformat_write_header();
av_write_frame() / av_interleaved_write_frame();
av_write_trailer()
```

### 3.9 [实战] 从 MP4 截取一段视频

```c
av_seek_frame()
```

### 3.10 [实战] 一个简单的小咖秀

- 将两个媒体文件中分别抽取音频与视频轨
- 将音频与视频轨合并成一个新文件
- 对音频与视频轨进行裁剪

 ## 4. FFmpeg 中级开发内容

- FFmpeg H264 解码
- FFmpeg H264 编码
- FFmpeg AAC 解码
- FFmpeg AAC 编码

### 4.1 FFmpeg H264 解码

```c
#include <libavcodec/avcodec.h>
```

常用数据结构：

- AVCodec 编码器结构体
- AVCodecContext 编码器上下文
- AVFrame 解码后的帧

 结构体内存的分配与释放：

```c
av_frame_alloc / av_frame_free();
avcodec_alloc_context3();
avcodec_free_context();
```

解码步骤：

- 查找解码器（avcodec_find_decoder）
- 打开解码器（avcodec_open2）
- 解码（avcodec_decode_video2）

### 4.2 FFmpeg H264 编码

H264编码流程：

- 查找编码器（avcodec_find_encoder_by_name）
- 设置参数，打开编码器（avcondec_open2）
- 编码（avcondec_encode_video2）

### 4.3 视频转图片

TODO

### 4.4 FFmpeg AAC 编码

- 编码流程与视频相同
- 编码函数 avcodec_encodec_audio2

## 5. SDL 介绍

> [SDL 官网]([http://www.libsdl.org](http://www.libsdl.org/))

- SDL（Simple DirectMedia Layer） 是一套[开放源代码](https://zh.wikipedia.org/wiki/%E9%96%8B%E6%94%BE%E5%8E%9F%E5%A7%8B%E7%A2%BC)的[跨平台](https://zh.wikipedia.org/wiki/%E8%B7%A8%E5%B9%B3%E5%8F%B0)[多媒体](https://zh.wikipedia.org/wiki/%E5%A4%9A%E5%AA%92%E9%AB%94)开发[库](https://zh.wikipedia.org/wiki/%E5%87%BD%E5%BC%8F%E5%BA%AB)
- 由 C 语言实现的跨平台的媒体开源库
- 多用于开发游戏、模拟器、媒体播放器等多媒体应用领域

语法与子系统：

SDL将功能分成下列数个子系统（subsystem）：

- **Video（图像）**—图像控制以及线程（thread）和事件管理（event）。
- **Audio（声音）**—声音控制
- **Joystick（摇杆）**—游戏摇杆控制
- **CD-ROM（光盘驱动器）**—光盘媒体控制
- **Window Management（视窗管理）**－与视窗程序设计集成
- **Event（事件驱动）**－处理事件驱动

以下是一支用C语言写成、非常简单的SDL示例：

```c
// Headers
#include "SDL.h"

// Main function
int main(int argc, char* argv[])
{
    // Initialize SDL
    if(SDL_Init(SDL_INIT_EVERYTHING) == -1)
        return(1);

    // Delay 2 seconds
    SDL_Delay(2000);

    // Quit SDL
    SDL_Quit();

    // Return
    return 0;
}
```

上述程序会加载所有SDL子系统（出错则退出程序），然后暂停两秒，最后关闭SDL并退出程序。

### 5.1 SDL 编译与安装

- 下载 SDL 源码
- 生成Makefile configure --prefix=/usr/local
- 安装 sudo make -j 8 && make install

### 5.2 使用 SDL 基本步骤

- 添加头文件 #include <SDL.h>
- 初始化 SDL
- 退出 SDL

SDL 渲染窗口：

```c
SDL_Init() / SDL_Quit();
SDL_CreateWindow() / SDL_DestoryWindow();
SDL_CreateRender();  // 创建渲染器
```

```shell
$ clang -g -o first_sdl first_sdl.c `pkg-config --libs sdl2`
```

SDL 渲染窗口：

```c
SDL_CreateRender() / SDL_DestoryRenderer();
SDL_RenderClear();
SDL_RenderPresent();
```

### 5.3 SDL 事件基本原理

- SDL 将所有的事件都存放在一个队列中
- 所有对事件的操作，其实就是队列的操作

SDL 事件种类：

- SDL_WindowEvent：窗口事件
- SDL_KeyboardEvent：键盘事件
- SDL_MouseMotionEvent：鼠标事件
- 自定义事件

SDL 事件处理：

```c
SDL_PollEvent(); // 轮询检测
SDL_WaitEvent(); // 常用的方式
SDL_WaitEventTimeout();
```

### 5.4 文理渲染

SDL 渲染基本原理：

<img src="_asset/SDL渲染基本原理.png">

SDL 文理相关 API：

```c
SDL_CreateTexture();
- format: YUV, RGB
- access: Texture 类型， Target， Stream

SDL_DestroyTexture();
```

SDL 渲染相关 API：

```c
SDL_SetRenderTarget();
SDL_RenderClear();
SDL_RenderCopy();
SDL_RenderPresent();
```

### 5.5 [实战] YUV 视频播放器

创建线程：

```c
SDL_CreateThread();
- fn: 线程执行函数
- name: 线程名
- data: 执行函数参数
```

SDL 更新文理：

```c 
SDL_UpdateTexture();
SDL_UpdateYUVTexture();
```

### 5.6 SDL 播放音频

播放音频基本流程：

<img src="_asset/播放音频基本流程.png">

播放音频的基本原则：

- 声卡向你要数据而不是你主动推给声卡
- 数据的多少由音频参数决定的

SDL 音频 API：

```c
SDL_OpenAudio() / SDL_CloseAudio();
SDL_PauseAudio();
SDL_MixAudio();
```

### 5.7 实现 PCM 播放器



























<img src="_asset/">

<img src="_asset/">

<img src="_asset/">

<img src="_asset/">

<img src="_asset/">

<img src="_asset/">

<img src="_asset/">

<img src="_asset/">




























