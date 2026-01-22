---
layout: post
title:  "RTSP协议相关"
date:   2026-01-22 19:15:00 +0800
---

<style type="text/css">
  pre code {
    white-space: pre-wrap;
    word-wrap: break-word;
    overflow-wrap: break-word;
  }
</style>

* 目录
{:toc #markdown-toc}

## 参考

- RTSP协议介绍: https://www.cnblogs.com/BreakingY/p/18677365

- RFC 2326: https://www.rfc-editor.org/rfc/rfc2326

- RTP协议介绍: https://www.cnblogs.com/BreakingY/p/18677364

- RFC 3550: https://www.rfc-editor.org/rfc/rfc3550

- RFC 6184: https://www.rfc-editor.org/rfc/rfc6184

- FFMPEG rtsp: https://ffmpeg.org/ffmpeg-protocols.html?spm=5176.28103460.0.0.2a4d6308pzzxjA#rtsp

- ffplay: https://ffmpeg.org/ffplay.html?spm=5176.28103460.0.0.2a4d6308pzzxjA

- MediaMtx: https://mediamtx.org/

- ZLMediaKit: https://docs.zlmediakit.com/

- VLC MediaPlayer: https://www.videolan.org/vlc/

- libVLC: https://www.videolan.org/vlc/libvlc.html

- VLC-Android: https://code.videolan.org/videolan/vlc-android

- VLC-Android Samples: https://code.videolan.org/videolan/libvlc-android-samples

## RTSP 与 RTP 协议

RTSP主要应用在视频监控领域，IPC（Internet Protocol Camera, 网络摄像头）都支持RTSP协议。

从IPC中获取音视频涉主要及到三个协议：RTSP、SDP、RTP。

- RTSP负责媒体控制，协商（确定传输方式，使用TCP还是UDP以及传输通道-TCP和传输端口-UDP）；

- SDP负责描述媒体信息；

- RTP负责传输音视频；

### RTSP

RTSP（Real Time Streaming Protocol），实时流传输协议，是TCP/IP 协议体系中的一个应用层协议。

```
The Real Time Streaming Protocol, or RTSP, is an application-level protocol for control over the delivery of data with real-time properties. RTSP provides an extensible framework to enable controlled, on-demand delivery of real-time data, such as audio and video. Sources of data can include both live data feeds and stored clips. This protocol is intended to control multiple data delivery sessions, provide a means for choosing delivery channels such as UDP, multicast UDP and TCP, and provide a means for choosing delivery mechanisms based upon RTP (RFC 1889).
```

RTSP只用于建立和控制音视频流。

```
The Real-Time Streaming Protocol (RTSP) establishes and controls either a single or several time-synchronized streams of continuous media such as audio and video. It does not typically deliver the continuous streams itself, although interleaving of the continuous media stream with the control stream is possible (see Section 10.12). In other words, RTSP acts as a "network remote control" for multimedia servers.
```

承担具体的音视频数据传输则是RTP协议。

```
The streams controlled by RTSP may use RTP [1], but the operation of RTSP does not depend on the transport mechanism used to carry continuous media.
```

RTSP语法和操作参考了HTTP/1.1，默认端口554。

```
The protocol is intentionally similar in syntax and operation to HTTP/1.1 [2] so that extension mechanisms to HTTP can in most cases also be added to RTSP.
```

RTSP报文的结构与HTTP基本一致：

![img](../../../assets/rtsp/rtsp报文结构.png)

RTSP是C/S模型，RTSP服务端可向RTSP客户端发送音视频(客户端拉流模式)，也可接受来自RTSP客户端发送过来的音视频流(客户端推流模式)。

```
Client: The client requests continuous media data from the media server.
```

```
Media server: The server providing playback or recording services for one or more media streams. Different media streams within a presentation may originate from different media servers. A media server may reside on the same or a different host as the web server the presentation is invoked from.
```

RTSP拉流模式使用的方法包括DESCRIBE、SETUP、PLAY。RTSP推流模式使用的方法包括ANNOUNCE、SETUP、RECORD。

RTSP拉流模式和推流模式，基本类似，推拉流模式只是媒体流向不同，使用的方法不同，其他大同小异。

DESCRIBE: C->S, 向服务器获取URL指定的媒体对象的描述。

```
The DESCRIBE method retrieves the description of a presentation or media object identified by the request URL from a server. It may use the Accept header to specify the description formats that the client understands. The server responds with a description of the requested resource. The DESCRIBE reply-response pair constitutes the media initialization phase of RTSP.
```

SETUP: C->S, 客户端向服务器请求建立会话并准备传输。

```
The SETUP request for a URI specifies the transport mechanism to be used for the streamed media. A client can issue a SETUP request for a stream that is already playing to change transport parameters, which a server MAY allow. If it does not allow this, it MUST respond with error "455 Method Not Valid In This State". For the benefit of any intervening firewalls, a client must indicate the transport parameters even if it has no influence over these parameters, for example, where the server advertises a fixed multicast address.
```

PLAY: C->S, 客户端主动通知服务器以SETUP指定的机制开始发送数据。

```
The PLAY method tells the server to start sending data via the mechanism specified in SETUP. A client MUST NOT issue a PLAY request until any outstanding SETUP requests have been acknowledged as successful.
```

ANNOUNCE

- C->S : 将通过请求 URL 识别的表示描述或者媒体对象提交给服务器。

- S->C : 通知客户端更新会话信息。

```
The ANNOUNCE method serves two purposes:

When sent from client to server, ANNOUNCE posts the description of a presentation or media object identified by the request URL to a server. When sent from server to client, ANNOUNCE updates the session description in real-time.

If a new media stream is added to a presentation (e.g., during a live presentation), the whole presentation description should be sent again, rather than just the additional components, so that components can be deleted.
```

RECORD: C->S, 通知服务器方法客户端将会根据之前的描述开始记录媒体数据。

```
This(RECORD) method initiates recording a range of media data according to the presentation description. The timestamp reflects start and end time (UTC). If no time range is given, use the start or end time provided in the presentation description. If the session has already started, commence recording immediately.
```

![img](../../../assets/rtsp/rtsp拉流过程.png)

### RTP

RTP 协议实际上是由实时传输协议RTP（Real-time Transport Protocol）和实时传输控制协议RTCP（Real-time Transport Control Protocol）两部分组成。

```
This document defines RTP, consisting of two closely-linked parts:

the real-time transport protocol (RTP), to carry data that has real-time properties.

the RTP control protocol (RTCP), to monitor the quality of service and to convey information about the participants in an on-going session.
```

RTP 协议基于多播或单播网络为用户提供连续媒体数据的实时传输服务。

```
This memorandum describes RTP, the real-time transport protocol. RTP provides end-to-end network transport functions suitable for applications transmitting real-time data, such as audio, video or simulation data, over multicast or unicast network services.
```

RTCP 协议是 RTP 协议的控制部分，用于实时监控数据传输质量，为系统提供拥塞控制和流控制。

```
RTP does not address resource reservation and does not guarantee quality-of-service for real-time services. The data transport is augmented by a control protocol (RTCP) to allow monitoring of the data delivery in a manner scalable to large multicast networks, and to provide minimal control and identification functionality.
```

RTP 通常基于 UDP 实现，但也可基于其他底层协议（TCP）。

```
Applications typically run RTP on top of UDP to make use of its multiplexing and checksum services; both protocols contribute parts of the transport protocol functionality. However, RTP may be used with other suitable underlying network or transport protocols (see Section 11)
```

RTP 本身不提供任何机制来保证数据“及时送达”，也不提供其他服务质量（QoS）保障，而是依赖底层协议来实现。

```
Note that RTP itself does not provide any mechanism to ensure timely delivery or provide other quality-of-service guarantees, but relies on lower-layer services to do so. It does not guarantee delivery or prevent out-of-order delivery, nor does it assume that the underlying network is reliable and delivers packets in sequence.
```

注意，RTCP只是监测传输质量，并通知发送端，其自身并不包含自动降低码率/帧率等机制，也没有实现类似 TCP 的拥塞控制算法。实际上，是由应用层基于 RTCP 告知的信息进行相应的调整，以保证传输质量。

![img](../../../assets/rtsp/rtp传输过程.png)

一个RTP报文包含三部分：固定头、贡献源列表（可选）、负载数据；

RTP报文的固定头：

![img](../../../assets/rtsp/rtp报文头.png)

| 字段（位宽）                    | 含义      | 关键说明                                            |
| ------------------------- | ------- | ----------------------------------------------- |
| Version (V, 2 bits)       | RTP 版本号 | 当前必须为 `2`（二进制 `10`）                             |
| Padding (P, 1 bit)        | 填充标志    | 若置 1，表示包末尾有填充字节（用于加密对齐等），最后一个填充字节指示填充长度         |
| Extension (X, 1 bit)      | 扩展头标志   | 若置 1，表示固定头后有一个 RTP 扩展头（格式见 Section 5.3.1）       |
| CSRC count (CC, 4 bits)   | 贡献源计数   | 指明后面 CSRC 列表的个数（0～15），仅在混合器（Mixer）场景使用          |
| Marker (M, 1 bit)         | 标记位     | 由 Profile 定义语义（如 H.264 中表示一帧结束，音频中表示静音→语音切换）    |
| Payload Type (PT, 7 bits) | 负载类型    | 由 Profile 定义（如 RFC 3551：0=PCMU, 96=动态类型如 H.264） |
| Sequence Number (16 bits) | 序列号     | 每发一个 RTP 包 +1，用于检测丢包和乱序（初始值随机）                  |
| Timestamp (32 bits)       | 时间戳     | 基于采样时钟（如音频 8kHz，视频 90kHz），表示第一个字节的采样时刻          |
| SSRC (32 bits)            | 同步源标识符  | 随机生成的唯一 ID，标识发送源（同一会话中不能冲突）                     |

RTP 负载数据不是由 RFC 3550 统一规定的，而是针对于每一种媒体编码格式（如 H.264、Opus、VP8 等）单独制定。

```
RTP payload: The data transported by RTP in a packet, for example audio samples or compressed video data. The payload format and interpretation are beyond the scope of this document.
```

RFC 6184(RTP Payload Format for H.264 Video) 是针对于 H.264 制定的。

```
This memo specifies an RTP payload specification for the video coding standard known as ITU-T Recommendation H.264 [1] and ISO/IEC International Standard 14496-10 [2] (both also known as Advanced Video Coding (AVC)).
```

### RTSP流媒体传输过程

一个完整的RTSP流媒体传输过程是：

（1）RTSP信令交互阶段：

- 客户端发起RTSP连接请求到服务端。

- 服务端响应，建立RTSP连接。

- 客户端与服务端交互，协商媒体传输方式（例如，使用UDP或TCP）。

- 服务端确认传输方式，建立媒体传输通道。

（2）采集音视频阶段：

- 服务端开始采集音频和视频数据。

（3）音视频编码阶段：

- 服务端进行H.264/H.265视频编码和AAC/PCM/PCMA/PACMU音频编码。

（4）RTP封装阶段：

- 服务端将编码后的音视频数据封装成RTP数据包。

- 视频使用H.264/H.265 RTP封装，音频使用AAC/PCM/PCMA/PACMU RTP封装。

（5）RTP数据传输阶段：

- 使用选择的传输方式（UDP或TCP），服务端将RTP数据包发送到客户端。

（6）客户端接收与解包阶段：

- 客户端接收RTP数据包。

- 如果使用UDP传输，客户端直接解包RTP数据。

- 如果使用TCP传输，客户端从RTSP信令中获取通道信息，然后通过TCP连接接收RTP数据。

（7）解码与播放阶段：

- 客户端对接收到的音视频数据进行解码。

- 解码后的数据传递给播放器进行播放。

### RTMP

RTMP（Real-Time Messaging Protocol）和 RTSP 都是用于实时音视频传输的网络协议，但它们由不同公司设计、基于不同传输层、适用于不同场景，彼此之间没有直接继承或依赖关系——它们是“平行”的流媒体协议。

| 协议   | 全称                           | 开发者                                          | 初创时间               |
| ---- | ---------------------------- | -------------------------------------------- | ------------------ |
| RTSP | Real-Time Streaming Protocol | IETF（标准化）<br>（最初由 RealNetworks、Netscape 等提出） | 1998 年（RFC 2326）   |
| RTMP | Real-Time Messaging Protocol | Adobe（原 Macromedia）                          | 2002 年（随 Flash 推出） |

RTSP 是开放标准（IETF RFC），而 RTMP 是 Adobe 私有协议。

| 场景        | RTSP                     | RTMP                          |
| --------- | ------------------------ | ----------------------------- |
| 主要用途      | 控制实时流的播放/暂停/录制（类似“遥控器”）  | 在客户端与服务器之间持续传输音视频+数据（类似“管道”）  |
| 典型应用      | IP 摄像头、视频会议、安防监控         | 直播推流（OBS → 推流平台）、Flash 直播（历史） |
| 方向性       | 支持 拉流（PLAY） 和 推流（RECORD） | 主要用于 推流（Publish），拉流较少见        |
| 是否传输媒体数据？ | 不直接传，用 RTP/RTCP 传数据      | 直接封装音视频在 TCP 流中传输             |

RTSP是控制信令协议（媒体走 RTP），而 RTMP 是信令和媒体合一的传输协议。

RTSP当下广泛运用在安防、IoT等领域，摄像头/IPC普通使用 RTSP。

RTMP被大量运用于直播平台推流。RTMP拉流则基本不再被使用，直播平台拉流则主要使用 HLS（HTTP Live Streaming）。

RTMP 拉流虽已淘汰，但 RTMP 推流仍被几乎所有主流直播平台（如 Twitch、YouTube Live、抖音、B站、快手等）广泛使用，这背后有历史惯性、工程实用性和生态成熟度等方面的原因。

- RTMP 简单、轻量、低开销：RTMP 基于 TCP 长连接，无需复杂握手（相比 WebRTC）

- 超高的工具和软件兼容性：OBS Studio（全球最流行的直播推流软件）默认首选 RTMP

- 稳定可靠，适合“单向推流”场景：TCP 保证有序、不丢包（适合关键帧敏感的视频流），服务器实现简单（Nginx + rtmp-module 几行配置即可接收）；

- 能够与现有转码/分发架构无缝集成：RTMP 作为 统一入口协议，后端可灵活转出多种播放协议。

## 工具支持

### FFMPEG

使用 FFmpeg 推送 RTSP 流（即作为 RTSP 客户端，向 RTSP 服务器发布流），最常用的命令如下：

```
ffmpeg -re -i <输入源> -c copy -f rtsp rtsp://<服务器IP>:<端口>/<路径>
```

| 参数           | 作用                                                      |
| ------------ | ------------------------------------------------------- |
| `-re`        | 以实时速度读取输入（若不指定，则将以最快速度推完）                               |
| `-i <输入>`    | 输入源：文件、设备、网络流等（如 `test.mp4`、`/dev/video0`、`rtmp://...`） |
| `-c copy`    | 直接复用音视频编码（不转码，低 CPU，推荐）                                 |
| `-f rtsp`    | 强制输出格式为 RTSP                                            |
| `rtsp://...` | 目标 RTSP 服务器地址（需支持 RTSP ANNOUNCE / RECORD）               |

例如，推送本地 MP4 文件（模拟直播）：

```
ffmpeg -re -i video.mp4 -c copy -f rtsp rtsp://192.168.1.100:554/live/stream
```

使用 ffplay 拉取(播放) RTSP流：

```
ffplay -rtsp_transport tcp \
       -fflags nobuffer \
       -flags low_delay \
       -framedrop \
       -probesize 32768 \
       -analyzeduration 0 \
       rtsp://192.168.1.64:554/stream
```

参数说明：

- -fflags nobuffer：禁用解复用缓冲

- -flags low_delay：启用低延迟解码

- -framedrop：允许丢帧保持实时

- -probesize / -analyzeduration：减少分析时间，加快启动

### 流媒体服务器

流媒体服务器是RTSP推流和拉流的信令中介和媒体中转站。

常用的流媒体服务器：

- MediaMtx：默认支持 RTSP 推流和多客户端拉流。开箱即用。

- ZLMediaKit：高性能，适合生产环境。功能强大，但需配置。

它们都是开源、跨平台、高性能的流媒体服务器，主要用于接收、转发和分发音视频流，支持多种协议（包括 RTSP、RTMP、HLS、WebRTC 等）。

| 维度    | MediaMTX                                  | ZLMediaKit                                                           |
| ----- | ----------------------------------------- | -------------------------------------------------------------------- |
| 项目起源  | 意大利开发者 aler9（Go 语言）                       | 中国开发者夏曹俊（C++）                                                        |
| 编程语言  | Go（内存安全、并发简单）                             | C++11 + 自研协程（极致性能）                                                   |
| 资源占用  | 极低（<50MB 内存）                              | 较低（100～300MB，取决于流数）                                                  |
| 配置复杂度 | ⭐ 极简（默认配置即用）                              | ⭐⭐ 中等（需理解 `config.ini`）                                              |
| 协议支持  | RTSP, RTSPS, HLS, WebRTC (WHIP/WHEP), SRT | RTSP, RTMP, HLS, HTTP-FLV, WebRTC, GB28181, FMP4                     |
| 特色功能  | - 超轻量<br>- WHIP/WHEP 原生支持<br>- 自动 API 管理  | - 支持国标 GB28181（安防）<br>- 支持 集群部署<br>- 内置 Web 管理界面<br>- 支持 MP4 录制、转码钩子 |
| 适用场景  | - 本地开发测试<br>- IoT 设备推流<br>- 简单直播分发        | - 安防监控平台<br>- 企业级直播系统<br>- 需要 GB28181 或 WebRTC 的场景                   |
| 学习曲线  | ⭐ 非常平缓（5 分钟上手）                            | ⭐⭐ 需要一定音视频基础                                                         |

### VideoLAN 的 VLC media player

使用 VLC media player 可以通过图形界面来拉取(播放) RTSP 流。

![img](../../../assets/rtsp/vlc-media-player.png)

## Android端基于libVLC拉取rtsp流

libVLC 是 VLC Media Player 的底层核心引擎，以 C 语言库 形式提供，可被嵌入到其他应用程序中。

libVLC 继承了 VLC 几乎全部的多媒体处理能力，只要是 VLC Media Player 能播的，libVLC 就能集成到 App 里播。

![img](../../../assets/rtsp/libvlc_stack.png)

[VLC-Android](https://code.videolan.org/videolan/vlc-android) 包含了 libVLC 在 Android 平台的编译版本。

```
You can use our LibVLC module to power your own Android media player. Download the .aar directly from Maven or build from source.
```

libVLC for Android 的 [maven](https://central.sonatype.com/artifact/org.videolan.android/libvlc-all/3.6.5) 坐标：

```
implementation("org.videolan.android:libvlc-all:3.6.5")
```

示例代码如下：

```
package org.videolan.javasample;

import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;

import org.videolan.java_sample.R;
import org.videolan.libvlc.LibVLC;
import org.videolan.libvlc.Media;
import org.videolan.libvlc.MediaPlayer;
import org.videolan.libvlc.util.VLCVideoLayout;

import java.io.IOException;
import java.util.ArrayList;

public class JavaActivity extends AppCompatActivity  {
    private static final boolean USE_TEXTURE_VIEW = false;
    private static final boolean ENABLE_SUBTITLES = true;
    private static final String ASSET_FILENAME = "bbb.m4v";

    private VLCVideoLayout mVideoLayout = null;

    private LibVLC mLibVLC = null;
    private MediaPlayer mMediaPlayer = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_main);

        final ArrayList<String> args = new ArrayList<>();
        args.add("-vvv");
        mLibVLC = new LibVLC(this, args);
        mMediaPlayer = new MediaPlayer(mLibVLC);

        mVideoLayout = findViewById(R.id.video_layout);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mMediaPlayer.release();
        mLibVLC.release();
    }

    @Override
    protected void onStart() {
        super.onStart();

        mMediaPlayer.attachViews(mVideoLayout, null, ENABLE_SUBTITLES, USE_TEXTURE_VIEW);

        try {
            final Media media = new Media(mLibVLC, getAssets().openFd(ASSET_FILENAME));
            mMediaPlayer.setMedia(media);
            media.release();
        } catch (IOException e) {
            throw new RuntimeException("Invalid asset folder");
        }
        mMediaPlayer.play();
    }

    @Override
    protected void onStop() {
        super.onStop();

        mMediaPlayer.stop();
        mMediaPlayer.detachViews();
    }
}
```

其中，Media 用于指定多媒体资源：

```
final Media media = new Media(mLibVLC, getAssets().openFd(ASSET_FILENAME));
```

示例代码中指定的是一个assets目录下的静态文件（bbb.m4v），替换为rtsp流的路径即可实现其拉取(播放)：

```
String rtspUrl = "rtsp://192.168.152.87:8554/mystream";
final Media media = new Media(mLibVLC, Uri.parse(rtspUrl));
```
