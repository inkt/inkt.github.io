---
layout: post
title: MultiMedia Codec
tags: [Android,FFmpeg]
author: laoYao
---

## 主要参考: [雷霄骅(leixiaohua1020)的专栏](http://blog.csdn.net/leixiaohua1020)

# 我们看到的音视频怎么被播放出来的?

网络层(socket或st)负责传输, 协议层(rtmp或hls)负责网络打包, 封装层(flv、ts)负责编解码数据的封装, 编码层(h.264和aac)负责图像, 音频压缩. 

![](/raw/1.jpg)

* 解协议的作用,就是将流媒体协议的数据,解析为标准的相应的封装格式数据. 

* 解封装的作用,就是将输入的封装格式的数据,分离成为`音频流压缩编码数据`和`视频流压缩编码数据`. 封装格式种类很多,例如MP4,MKV,RMVB,TS,FLV,AVI等等,它的作用就是将已经压缩编码的`视频数据`和`音频数据`按照一定的格式放到一起. 例如,FLV格式的数据,经过解封装操作后,输出H.264编码的视频码流和AAC编码的音频码流. 

* 解码的作用,就是将视频/音频压缩编码数据,解码成为非压缩的视频/音频原始数据. 音频的压缩编码标准包含AAC,MP3,AC-3等等,视频的压缩编码标准则包含H.264,MPEG2,VC-1等等. 

* 视音频同步的作用,就是根据解封装模块处理过程中获取到的参数信息,同步解码出来的视频和音频数据,并将视频音频数据送至系统的显卡和声卡播放出来. 

## 视频编码

视频编码的主要作用是将视频像素数据（RGB,YUV等）压缩成为视频码流,从而降低视频的数据量. 将最真实的画面 用最少的文件 传输.

码率: 平均码率通常是指数字音乐或者视频的平均码率,可以简单的认为等于文件大小除以播放时间. 码率并不是衡量音频/视频质量的唯一标准,格式、图像大小、音频采样率、音频分辨率等因素也是很重要的指标. 

视频码流的数据量占了视音频总数据量的绝大部分. 

### 视频编码基本原理:

> 例如对于像新闻联播这种背景静止,画面主体运动较小的数字视频,每一幅画面之间的区别很小,画面之间的相关性很大. 对于这种情况我们没有必要对每一帧图像单独进行编码,而是可以只对相邻视频帧中变化的部分进行编码,从而进一步减小数据量

* 时间上的冗余信息（temporal redundancy）在视频数据中,相邻的帧（frame）与帧之间通常有很强的关连性,这样的关连性即为时间上的冗余信息. 
* 空间上的冗余信息（spatial redundancy）在同一张帧之中,相邻的像素之间通常有很强的关连性,这样的关连性即为空间上的冗余信息. 
* 感知上的冗余信息（perceptual redundancy）感知上的冗余信息是指在人在观看视频时,人眼无法察觉的信息. 

> I帧: 关键帧: 自带全部信息的独立帧,无需参考其他图像便可独立进行解码

> P帧: 预测帧: 需要参考前面的I帧/P帧才能进行编码

> B帧: 双向预测帧: 要解码B帧,不仅要取得之前的缓存画面,还要解码之后的画面, B帧压缩率高, 但是编解码时会比较耗费CPU, 在直播中可能会增加直播延时, 因此在移动端上一般不使用B帧

> GOP: Group of Picture的缩写,是画面组的意思. 通常意义上的GoP由I帧开始,到下一个I帧之前的帧结束. 一个GoP组内的所有帧独立于其之前的GoP及其后面的GoP. 如果没有 I 帧,P 帧和 B 帧就无法解码. 

![典型 I B P 帧 结构顺序](/raw/2.jpg)

### 常见视频编码标准[压缩算法]

`MPEG` (Moving Picture Experts Group/“动态图像专家组”组织,成立于1988年,致力开发视频、音频的压缩编码技术) 到目前为止已经制定并正在制定以下和视频相关的标准: 

* `MPEG-1`: 第一个官方的视讯音频压缩标准,随后在Video CD中被采用,其中的音频压缩的第三级（`MPEG-1 Layer 3`）简称`MP3`,成为比较流行的音频压缩格式. 

* `MPEG-2`: 广播质量的视讯、音频和传输协议. 被用于无线数字电视-ATSC、DVB以及ISDB、数字卫星电视（例如DirecTV）、数字有线电视信号,以及DVD视频光盘技术中. 

* `MPEG-3`: 原本目标是为高分辨率电视（HDTV）设计,随后发现MPEG-2已足够HDTV应用,故`MPEG-3`的研发便中止. 

* `MPEG-4`: 2003年发布的视讯压缩标准,主要是扩展`MPEG-1`、`MPEG-2`等标准以支持视频／音频对象（video/audio "objects"）的编码、3D内容、低比特率编码（low bitrate encoding）和数字版权管理（Digital Rights Management）,其中第10部分由ISO/IEC (国际电工委员会) 和ITU-T (国际电信联盟电信标准化部门) 联合发布,称为`H.264/MPEG-4 AVC` (英语: MPEG-4 Part 10, Advanced Video Coding,缩写为MPEG-4 AVC). 

* `MPEG-7`: `MPEG-7`并不是一个视讯压缩标准,它是一个多媒体内容的描述标准. 

* `MPEG-21`: `MPEG-21`是一个正在制定中的标准,它的目标是为未来多媒体的应用提供一个完整的平台. 

现在主流的编码标准为 `H.264`.

## 音频编码

音频编码的主要作用是将音频采样数据（PCM等）压缩成为音频码流,从而降低音频的数据量. 

常见音频编码标准[开发者]有:
`AAC`[MPEG], `AC-3`[Dolby Inc.], `MP3`[MPEG], `WMA`[Microsoft], `FLAC`[Xiph.Org Foundation, Josh Coalson]

## 编解码器

以上只是音视频编码标准, 并不是具体的编解码器. 

[常见的编解码器](https://en.wikipedia.org/wiki/Comparison_of_video_codecs) 有: `x264`, `Xvid`, `FFmpeg`, `DivX`, `VP9`. 不同的编解码器支持的标准不尽相同.

|  编解码器  |  开发者  |  支持的标准  |  简介  |
| ---- | ---- | ---- | ---- |
|x264|x264 team|MPEG-4 AVC/H.264|x264是一个采用GPL授权的视频编码自由软件[1]. x264的主要功能在于进行H.264/MPEG-4 AVC的视频编码,而不是作为解码器（decoder）之用. |
|Xvid|Xvid team|MPEG-4 ASP|Xvid（旧称为XviD）是一个开放源代码的MPEG-4视频编解码器,是由一群原OpenDivX开发者在OpenDivX于2001年7月停止开发后自行开发的. |
|FFmpeg|FFmpeg team|MPEG-1, MPEG-2, MPEG-4 ASP, H.261, H.263, VC-3, WMV7, WMV8, MJPEG, MS-MPEG-4v3, DV, Sorenson codec etc.|FFmpeg是一个自由软件,可以运行音频和视频多种格式的录影、转换、流功能,包含了libavcodec—这是一个用于多个项目中音频和视频的解码器库,以及libavformat—一个音频与视频格式转换库. “FFmpeg”这个单词中的“FF”指的是“Fast Forward”. |

### 软硬解

* 软解(S/W decoding): CPU解码: 加大CPU负担, 耗电增加、没有硬解码流畅, 解码速度相对慢, 兼容性好

* 硬解(H/W decoding): 调用GPU的专门模块来编解码, 减少CPU运算: 播放流畅、低功耗, 解码速度快, 兼容性差

![SoC](/raw/3.png)

## 常见的封装格式

[封装格式](https://zh.wikipedia.org/wiki/%E8%A7%86%E9%A2%91%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F) 种类很多,例如MP4,MKV,RMVB,TS,FLV,AVI等等,它的作用就是将已经压缩编码的`视频数据`和`音频数据`按照一定的格式放到一起. 例如,FLV格式的数据,经过解封装操作后,输出H.264编码的视频码流和AAC编码的音频码流. 也封装字幕(一个或多个字幕文件).

| 扩展名 | 开发者 | 支持的 视频编码 | 支持的 音频编码 | 
| ---- | ---- | ---- | ---- |
| .MP4 | MPEG |MPEG-2, MPEG-4, H.264, H.263等 | AAC, MPEG-1 Layers I, II, III, AC-3等 |
| .MKV | CoreCodec, Inc. | Virtually anything | Virtually anything |
| .AVI | Microsoft Inc. | Almost anything through VFW | Almost anything through ACM |
| .FLV | Adobe Inc. | Sorenson, VP6, H.264 | MP3, ADPCM, Linear PCM, AAC等 |

### 转码

* 输入视频的封装格式是FLV,视频编码标准是H.264,音频编码标准是AAC; 输出视频的封装格式是AVI,视频编码标准是MPEG2,音频编码标准是MP3. 从流程中可以看出,首先从输入视频中分离出视频码流和音频压缩码流,然后分别将视频码流和音频码流进行解码,获取到非压缩的像素数据/音频采样数据,接着将非压缩的像素数据/音频采样数据重新进行编码,获得重新编码后的视频码流和音频码流,最后将视频码流和音频码流重新封装成一个文件. 

![](/raw/4.jpeg)

* 封装格式转换, 并不进行编解码. 处理速度极快, 视音频质量无损. 

![](/raw/5.jpeg)

## `FFmpeg` 的使用

```
|-- FFmpeg Components
    |-- FFmpeg Tools
        |-- ffmpeg              A command line tool to convert multimedia files between formats
        |-- FFplay              一个简单的播放器,基于SDL与FFmpeg库
        |-- FFprobe             ffprobe gathers information from multimedia streams and prints it in human- and machine-readable fashion.
        |-- FFserver            多媒体即时广播流服务器
    |-- FFmpeg Libraries for developers
        |-- libavutil           包含一些共用的函数,如随机数生成,数据结构,数学运算等
        |-- libswscale          可以转换像素数据的格式,同时可以拉伸图像的大小. 
        |-- libavcodec          包含全部FFmpeg音频/视频编解码库
        |-- libavformat         包含demuxers和muxer库, 对视频进行封装/解封装
        |-- libavdevice         可以读取电脑的多媒体设备的数据,或者输出数据到指定的多媒体设备上
        |-- libavfilter         可以给视音频添加各种滤镜效果
        |-- libswresample       可以对音频进行重采样, rematrixing 以及转换采样格式等操作
        |-- Libpostproc         用于进行视频的一些后期处理
```

![Easy!!!](/raw/6.jpg)

### 常用命令

```
ffmpeg -i input.mp4 output.avi
```

#### 主要参数
```
-i 设定输入流
-f 设定输出格式
-ss 开始时间
```
#### 视频参数
```
-b 设定视频流量,默认是200Kbit/s
-s 设定画面的宽和高
-aspect 设定画面的比例
-vn 不处理视频
-vcoder 设定视频的编码器,未设定时则使用与输入流相同的编解码器
```
#### 音频参数
```
-ar 设定采样率
-ac 设定声音的Channel数
-acodec 设定沈阳的Channel数
-an 不处理音频
```

截取一张352x240尺寸大小,格式为jpg的图片

```
ffmpeg -i input_file -y -f image2 -t 0.001 -s 352x240 output.jpg
```

在视频的第8.01秒出截取230x240的缩略图

```
ffmpeg -i input_file -y -f mjpeg -ss 8 -t 0.001 -s 320x240 output.jpg
```

每隔一秒截一张图

```
ffmpeg -i out.mp4 -f image2 -vf fps=fps=1 out%d.png
```

每隔20秒截一张图

```
ffmpeg -i out.mp4 -f image2 -vf fps=fps=1/20 out%d.png
```

分离视频音频流

```
ffmpeg -i input_file -vcodec copy -an output_file_video    //分离视频流
ffmpeg -i input_file -acodec copy -vn output_file_audio    //分离音频流
```

视频转码

```
ffmpeg -i test.mp4 -vcoder h264 -s 352*278 -an -f m4v test.264    //转码为码流原始文件
ffmpeg -i test.mp4 -vcoder h264 -bf 0 -g 25 -s 352-278 -an -f m4v test.264    //转码为码流原始文件
ffmpeg -i test.avi -vcoder mpeg4 -vtag xvid -qsame test_xvid.avi    //转码为封装文件 -bf B帧数目控制, -g 关键帧间隔控制, -s 分辨率控制
```

视频封装

```
ffmpeg -i video_file -i audio_file -vcoder copy -acodec copy output_file
```

视频剪切

```
ffmpeg -i test.avi -r 1 -f image2 image.jpeg    //视频截图
ffmpeg -i input.avi -ss 0:1:30 -t 0:0:20 -vcoder copy -acoder copy output.avi   //剪切视频 -r 提取图像频率, -ss 开始时间, -t 持续时间
```

视频定点剪切

```
ffmpeg -i input.avi -strict -2 -vf crop=1080:1080:0:420 output.avi  // 其中 width 和 height 表示裁剪后的尺寸,x:y 表示裁剪区域的左上角坐标. 
```

图片定点裁剪
```
ffmpeg -i in.png -vf crop=70:70:100:100 out.png
```

![](/raw/7.png)

使用ffmpeg合并MP4文件

```
ffmpeg -i "Apache Sqoop Tutorial Part 1.mp4" -c copy -bsf:v h264_mp4toannexb -f mpegts intermediate1.ts
ffmpeg -i "Apache Sqoop Tutorial Part 2.mp4" -c copy -bsf:v h264_mp4toannexb -f mpegts intermediate2.ts
ffmpeg -i "Apache Sqoop Tutorial Part 3.mp4" -c copy -bsf:v h264_mp4toannexb -f mpegts intermediate3.ts
ffmpeg -i "Apache Sqoop Tutorial Part 4.mp4" -c copy -bsf:v h264_mp4toannexb -f mpegts intermediate4.ts
ffmpeg -i "concat:intermediate1.ts|intermediate2.ts|intermediate3.ts|intermediate4.ts" -c copy -bsf:a aac_adtstoasc "Apache Sqoop Tutorial.mp4"
```

使用ffmpeg转换flv到mp4

```
ffmpeg -i out.flv -vcodec copy -acodec copy out.mp4
```

视频添加水印: 水印局中

```
ffmpeg -i out.mp4 -i input_file.gif -filter_complex overlay="(main_w/2)-(overlay_w/2):(main_h/2)-(overlay_h)/2" output.mp4

// -i out.mp4(视频源)
// -i input_file.gif(水印图片)
// overlay 水印的位置
// output.mp4 输出文件
```

水平翻转语法: -vf hflip

```
ffplay -i out.mp4 -vf hflip
```

垂直翻转语法: -vf vflip

```
ffplay -i out.mp4 -vf vflip
```

视频加速

```
ffmpeg -i input.mkv -an -filter:v "setpts=0.5*PTS" output.mkv
```

### 在Android中 为什么要用FFmpeg

Android原生支持的[编码标准,容器格式,网络协议](https://developer.android.com/guide/topics/media/media-formats.html)

主要的 相关类有 [`android.media.MediaCodec`](https://developer.android.com/reference/android/media/MediaCodec.html)

> `MediaCodec`(硬编码) 用着麻烦, 比如: [裁剪视频源码](https://github.com/LineageOS/android_packages_apps_Gallery2/blob/cm-14.1/src/com/android/gallery3d/app/VideoUtils.java)

> `FFmpeg`(软编码) 虽然打包后体积大 [待解决: 禁用无用组件] - -! , 但是功能强大, 用着方便

> 如果仅仅为了裁剪视频, 也可用`mp4parser`, [see Best Video Transcoding Libraries](http://leaks.wanari.com/2016/07/13/android-video-transcoding-libraries/)

### 怎么在Android中使用FFmpeg

具体参考: [最简单的基于FFmpeg的移动端例子: Android HelloWorld](http://blog.csdn.net/leixiaohua1020/article/details/47008825)

![](/raw/8.jpg)

比较方便的库 [FFmpeg Android Java](http://writingminds.github.io/ffmpeg-android-java/), 打包后20M

```java
FFmpeg ffmpeg = FFmpeg.getInstance(context);
try {
    String[] command = "-i /sdcard/vid.mp4 /sdcard/out.mkv".split(" ");
    ffmpeg.execute(command, new ExecuteBinaryResponseHandler() {

        @Override
        public void onStart() {}

        @Override
        public void onProgress(String message) {}

        @Override
        public void onFailure(String message) {}

        @Override
        public void onSuccess(String message) {}

        @Override
        public void onFinish() {}
  });
} catch (FFmpegCommandAlreadyRunningException e) {
    // Handle if FFmpeg is already running
}
```

![](/raw/9.gif)

### 直播

```
|-- 完整的直播
    |-- 采集端
        |-- 音视频采集
        |-- 视频处理: 滤镜,美颜,水印
            |-- 美颜: 本质就是让像素点模糊
            |-- 美白: 提高亮度
        |-- 音视频编码压缩[软硬解]
        |-- MUXING: 封装成FLV,TS
            |-- FLV: 一种流媒体封装格式,由于它形成的文件极小、加载速度极快
            |-- TS: 两个TS片段可以无缝拼接, 播放器能连续播放
        |-- 通过流媒体协议 推流至服务端
            |-- rtmp: 延迟小,跨平台性较差: 从采集推流端到流媒体服务器再到播放端是一条数据流, 基于 TCP 长连接
            |-- hls: 延迟较大,跨平台好: 基本原理就是当采集推流端将视频流推送到流媒体服务器时, 服务器将收到的流信息每缓存一段时间就封包成一个新的 ts 文件(切片), 同时服务器会建立一个 m3u8 的索引文件来维护最新几个 ts 片段的索引. 当播放端获取直播时, 它是从 m3u8 索引文件获取最新的 ts 视频文件片段来播放, 从而保证用户在任何时候连接进来时都会看到较新的内容, 实现近似直播的体验. 
    |-- 服务端
        |-- 数据分发CDN
            |-- 内容分发网络,将网站的内容发布到最接近用户的网络”边缘”, 使用户可以就近取得所需的内容, 解决 Internet网络拥挤的状况, 提高用户访问网站的响应速度.
        |-- 处理: 转码,鉴黄,录制
    |-- 播放端
        |-- 拉流
        |-- DEMUXING: 从FLV,TS中分离出音视频数据
        |-- 音视频解码[软硬解]
        |-- 解码,渲染播放
    |-- 聊天室[信令/礼物等]
        |-- 即时通信: 消息发送,接收,解析
            |-- 消息过多
        |-- 异常处理: 心跳
```

## 三方SDK: [腾讯 `ILiveSdk` 的使用](https://github.com/zhaoyang21cn/ILiveSDK_Android_Suixinbo)

add dependencies

```groovy
compile 'com.tencent.livesdk:livesdk:1.1.1'
compile 'com.tencent.ilivesdk:ilivesdk:1.5.2'
```

add AVRootView

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
             android:layout_width="match_parent"
             android:layout_height="match_parent">

    <com.tencent.ilivesdk.view.AVRootView
        android:id="@+id/live_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
    
    ...
</FrameLayout>
```

init sdk & register message listener

```java
ILiveSDK.getInstance().initSdk(this.getApplicationContext(), Constant.SDK_APPID, Constant.ACCOUNT_TYPE);

ILVLiveConfig liveConfig = new ILVLiveConfig();
liveConfig.setLiveMsgListener(new ILVLiveConfig.ILVLiveMsgListener {

    // do something according to text type
    @Override
    public void onNewTextMsg(ILVText text, String senderId, TIMUserProfile userProfile) {}

    @Override
    public void onNewCustomMsg(ILVCustomCmd cmd, String id, TIMUserProfile userProfile) {}

    @Override
    public void onNewOtherMsg(TIMMessage message) {}
});

ILVLiveManager.getInstance().init(liveConfig);
```

LiveRoom LifeCycle

```java
public class LiveLifeFragment extends RxFragment {

    private AVRootView avRootView;

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        avRootView = (AVRootView) view.findViewById(R.id.live_view);
        ILVLiveManager.getInstance().setAvVideoView(avRootView);

        if (isHost) {
            createLive();
        }
        if (!isHost) {
            joinLive();
        }
    }

    @Override
    public void onResume() {
        super.onResume();
        ILVLiveManager.getInstance().onResume();
    }

    @Override
    public void onPause() {
        super.onPause();
        ILVLiveManager.getInstance().onPause();
    }

    @Override
    public void onDestroy() {
        ILVLiveManager.getInstance().onDestory();
        super.onDestroy();
    }

    private void createLive() {
        ILVLiveRoomOption hostOption = new ILVLiveRoomOption(userId)
                .controlRole(Constant.HOST_ROLE)
                .authBits(AVRoomMulti.AUTH_BITS_DEFAULT)
                .videoRecvMode(AVRoomMulti.VIDEO_RECV_MODE_SEMI_AUTO_RECV_CAMERA_VIDEO)
                .cameraId(liveEntity.isBackCamera ? ILiveConstants.BACK_CAMERA : ILiveConstants.FRONT_CAMERA)
                .autoFocus(true);

        ILVLiveManager.getInstance()
                .createRoom(Integer.parseInt(liveEntity.roomId), hostOption,
                        new ILiveCallBack() {
                            @Override
                            public void onSuccess(Object data) {}
                        });
    }

    private void joinLive() {
        ILVLiveRoomOption memberOption = new ILVLiveRoomOption(userId)
                .autoCamera(false)
                .controlRole(Constant.NORMAL_MEMBER_ROLE)
                .authBits(AVRoomMulti.AUTH_BITS_JOIN_ROOM
                        | AVRoomMulti.AUTH_BITS_RECV_AUDIO
                        | AVRoomMulti.AUTH_BITS_RECV_CAMERA_VIDEO
                        | AVRoomMulti.AUTH_BITS_RECV_SCREEN_VIDEO)
                .videoRecvMode(AVRoomMulti.VIDEO_RECV_MODE_SEMI_AUTO_RECV_CAMERA_VIDEO)
                .autoMic(false);

        ILVLiveManager.getInstance().joinRoom(Integer.parseInt(liveEntity.roomId), memberOption, 
                        new ILiveCallBack() {
                            @Override
                            public void onSuccess(Object data) {}
                        });
    }

    private void stopLive() {
        if (avRootView != null) {
            avRootView.clearUserView();
        }

        ILVLiveManager.getInstance().quitRoom(new ILiveCallBack() {
            @Override
            public void onSuccess(Object data) {}
        });
    }

    private void sendMessage(int type, String textToSend) {
        ILVCustomCmd cmd = new ILVCustomCmd();
        cmd.setCmd(ILVLiveConstants.ILVLIVE_CMD_CUSTOM_LOW_LIMIT);
        cmd.setType(ILVText.ILVTextType.eGroupMsg);
        cmd.setDestId(ILiveRoomManager.getInstance().getIMGroupId());
        cmd.setParam(textToSend);

        ILVLiveManager.getInstance().sendCustomCmd(cmd,                        
                        new ILiveCallBack() {
                            @Override
                            public void onSuccess(Object data) {}
                        });
    }
}
```

直播间 流程图

![](/raw/10.jpg)

额外参考: [视频直播的技术原理和实现思路方案整理](http://www.jianshu.com/p/bd42bacbe4cc), [如何快速的开发一个完整的iOS直播app](https://github.com/f2e-journey/xueqianban/issues/61), [直播总结@IOS](https://github.com/tiantianlan/LiveExplanation)