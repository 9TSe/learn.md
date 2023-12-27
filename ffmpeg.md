---
title: ffmpeg命令行简介
date: 2023-12-05 21:37:52
tags:
- ffmpeg
categories:
- 音视频
cover: /pic/1.png
---

# 1. ffmpeg
## 1.1 视频图片转换

>视频生成图片
```shell
ffmpeg -i input.mp4 -r 25 -f image2 data/image%3d.jpg
```
备注:
- `image%3d.jpg` 表示生成的图片序号为3个数字
`image%d.jpg` 表示生成的图片序号依次增加

- `-r` 25 帧数
- `-f image2` 格式化的格式


>图片生成视频

```shell
ffmpeg -r 1 -f image2 -i data/%d.jpg -vcodec libx264 -s 640*480 -g 1 -keyint_min 1 -sc_threshold 0 -pix_fmt yuv420p out.mp4
```

备注：

- `-vcodec libx264`  指定合成视频的编码格式为`h264`
- `-r 1`  fps等于1  (frame rate 帧率)
这个参数需要`写在 -f 之前`，确保FFmpeg能够正确地解释输入文件的每秒图像数，并据此创建视频的时间轴。
- `-s 640*480`   分辨率 (size)
- `-g 1`  GOP长度(关键帧之间的间隔)
- `-keyint_min 1`   keyint表示关键帧（IDR帧）间隔
这个选项表示限制`IDR帧间隔最小为1帧，与设置的GOP等长`
- `-sc_threshold 0`  禁用场景识别，即进制自动添加`IDR帧` (scene threshold（场景阈值）)
- `-pix_fmt  yuv420p`  帧格式  (pixel（像素）)
- `-vf scale=1280:-1`  指定合成视频的分辨率自适应宽为1280，高按照比例计算
(video filter（视频滤镜）) 
`-1` 是一个特殊的值，它告诉 FFmpeg 保持原始高度与新宽度的比例。

ps:
	IDR帧(首个I帧)
eg:

```shell
ffmpeg -r 25 -f image2 -i data/image%3d.jpg -vcodec libx264 -s 1080*606 -g 100 -keyint_min 25 -sc_threshold 0 -pix_fmt yuv420p out.mp4
```

## 1.2 生成m3u8切片

```shell
ffmpeg -i input.mp4 -c:v libx264 -c:a copy -f hls -hls_time 10 -hls_list_size 0
-hls_start_number 0 input/index.m3u8
```

备注：
- `-c:v` codec（编解码器）: video（视频）。 == -vcodec
`-c:a` audio（音频）  == -acodec 
- `-f` (format) 以`hls`格式
- `-hls_time n`: 设置每片的长度，默认值为2。单位为秒
- `-hls_list_size n`:设置播放列表保存的最多条目，设置为0会保存有所片信息，默认值为`5`
- `-hls_start_number n`:设置播放列表中`sequence number`的值为`number`，默认值为`0`
- `-hls_wrap n`:设置多少片之后开始覆盖，如果设置为0则不会覆盖，默认值为`0`.
   这个选项能够避免在磁盘上存储过多的片，而且能够限制写入磁盘的最多的片的数量

>另一种方法

```shell
ffmpeg -i input.mp4 -fflags flush_packets 
-max_delay 2 -flags -global_header 
-hls_time 5 -hls_list_size 0 -vcodec libx264 -acodec aac -r 30 -g 60 index.m3u8
```

备注:
- `-fflags` 设置输入/输出文件或流的标志（flags）
用来激活或修改 FFmpeg 内部的标志，以控制特定的行为
- `-flush_packets` 导致 FFmpeg 立即输出已经在内存缓冲中的数据包（packets）
而不是等待缓冲区满或其他条件触发输出。
- `-max_delay 2`：设置最大延迟时间为 2 秒
* `-flags -global_header`：这是一个设置视频编码器标志的选项。在这个情况下，`-flags` 用于设置特定的编码器标志。`-global_header` 标志指示在视频流的第一个关键帧（I帧）中包含全局头信息（global headers），这对于某些视频流的处理和解码非常重要。

* `-vcodec libx264`：指定视频编码器为 libx264，用于对视频进行 H.264 编码。
* `-acodec aac`：指定音频编码器为 AAC，用于对音频进行 AAC 编码。

## 1.3 指定码率转换

```shell
ffmpeg -i input.mp4 -b:v 10M -b:a 10M -c:v libx264 -c:v aac out.mp4
```

备注：
<font color = red>切记一点，命令行中涉及编解码时，-c:v copy 不要使用，否则 比如指定的码率参数，分辨率参数等就会失效，而且很难找到原因 </font>
 -  `-b:v 10M` 指定`视频`重新编码的码率为10M/s (`bitrate`(比特率)) == -vb
 - `-b:a 10M` 指定`音频`重新编码的码率为10M/s == -ab
 
`码率: 输出视频每秒的bit数`
## 1.4 录制

>指定时间录制
```shell
#从10:20 录制到30:20
ffmpeg -i input.mp4 -c:v copy -c:a copy -ss 00:10:20 -to 00:30:20 out.mp4
```

>指定录制时长
```shell
#录制30秒
ffmpeg -i input.mp4 -c:v copy -c:a copy -t 30 out.mp4
```

## 1.5 提取裸码流

>提取 `h264 裸码流`
```shell
ffmpeg -i input.mp4 -c:v copy -bsf:v h264_mp4toannexb -an out.h264
```

备注:
- `-bsf:v` Bitstream Filter(比特流过滤器)
- `h264_mp4toannexb` 过滤器名称
将 H.264 编码的 MP4 格式视频转换为 H.264 规范的 `Annex B` 形式。
`Annex B` 是 `H.264` 标准中的一种视频封装格式。它规定了 H.264 视频流的封装方式和数据传输格式。
- ` -an`  audio none(禁用音频)


>提取 `h264 裸码流指定编码质量`
```shell
ffmpeg -i input.mp4 -an -c:v libx264 -crf 18 out.h264
# -crf 18 (固定质量值18)
```


> 提取 `aac` 裸码流

```shell
ffmpeg -i input.mp4 -acodec copy -vn out.aac
```

## 1.6 倒放
>`视频倒放`音频不变
```shell
ffmpeg -i input.mp4 -vf reverse xxx.mp4
ffmpeg -i input.mp4 -vf "reverse" xxx.mp4
```

>`音频倒放`，视频不变
```shell
ffmpeg -i input.mp4 -map 0 -c:v copy -af "areverse" xxx.mp4 
ffmpeg -i input.mp4 -af areverse xxx.mp4 
```

备注:
- `-map` 参数允许你指定要从输入文件中选择的特定流，并将这些流映射到输出文件中。
输入文件可以包含多个音频流、视频流和字幕流等
`-map` 表示传输的第一个文件中的所有流


>视频音频同时`倒放`

```shell
ffmpeg -i input.mp4 -vf reverse -af areverse xxx.mp4
```

## 1.7 转码
>转码->`AVC`（指定转码的部分参数）
```shell
ffmpeg -i input.mp4 -c:v libx264 -preset slow -tune film -profile:v main out.mp4
```
备注：
- `-tune film` (主要配合视频类型和视觉优化的参数) 
- `-preset slow`   编码预设，主要调节 编码速度和质量的平衡
         10个选项如下 从快到慢：ultrafast、superfast、veryfast、faster、fast、medium、slow、slower、veryslow、placebo
- ` -profile:v main` (profile 配置文件) 
         h264有四种画质级别,分别是baseline, extended, main, high：
         1、`Baseline Profile`：基本画质。支持I/P 帧，`只支持无交错`（Progressive）和`CAVLC`；
         2、`Extended profile`：进阶画质。支持I/P/B/SP/SI 帧，`只支持无交错`（Progressive）和`CAVLC`；(用的少)
         3、`Main profile`：主流画质。提供I/P/B 帧，`支持无交错和交错`（Interlaced）， 也支持`CAVLC 和CABAC` 
         4、`High profile`：高级画质。在`main Profile` 的基础上增加了8x8内部预测、自定义量化、 无损视频编码和更多的YUV 格式；


```shell
ffmpeg -i input.mp4 -c:v libx264 -b:v 2048k -vf scale=1280:-1 -y out.mp4
```

>转码->`HEVC` 
```shell
ffmpeg -i input.mp4 -c:v libx265 -c:a copy out.mp4
```




> 使用`cuvid`进行解码和编码实现转码
```shell
ffmpeg -c:v h264_cuvid -i input.mp4 -c:v h264_nvenc -b:v 2048k -vf scale=1280:-1 -y out.mp4
```
备注:
- `-hwaccel cuvid` (指定使用cuvid硬件加速)
- `-c:v h264_cuvid` (使用h264_cuvid进行视频解码)
- `-c:v h264_nvenc` (使用h264_nvenc进行视频编码)
- `-b:v 2048k` (指定输出视频的码率，即输出视频每秒的bit数)
- `-vf scale=1280:-1` (指定输出视频的宽高，高-1代表按照比例自动适应)

>使用`videotoolbox`进行编码实现转码
```shell
ffmpeg -i input.mp4 -vcodec h264_videotoolbox -b:v 2048k -vf scale=1280:-1 -y  out.mp4
```
备注：
- `-vcodec h264_videotoolbox `(使用h264_videotoolbox 进行视频编码)

## 1.8 查看
```shell
#查看当前支持的编码器
ffmpeg -codecs

#查看当前支持的封装格式
ffmpeg -formats

#查看当前支持的滤镜
ffmpeg -filters

#查看指定解码器的相关参数
ffmpeg -h decoder=h264_cuvid

#查看当前支持的硬件加速选项
ffmpeg -hwaccels
#例如：mac核显支持的选项（videotoolbox）英伟达显卡支持的选项（cuvid）

#查看摄像头列表
ffmpeg -list_devices true -f dshow -i dummy

#查看摄像头的分辨率格式
ffmpeg -list_options true -f dshow -i video="FULL HD webcam"
```

## 1.9 推拉流
>`摄像头`推流到`RTMP`服务
```shell
ffmpeg -f dshow -i video="USB webcam" -vcodec libx264 -acodec aac -ar 44100 -ac 1 -r 25 -s 1920*1080 -f flv rtmp://192.168.1.3/live/desktop
```

- `-ac` audio channels（音频通道）


>`摄像头`推流到`RTSP`（rtp over tcp）
```shell
ffmpeg -f dshow -i video="FULL HD webcam" -rtsp_transport tcp -vcodec libx264 -preset ultrafast -acodec libmp3lame -ar 44100 -ac 1 -r 25 -f rtsp rtsp://192.168.0.1/webcam
```

>windows`桌面`推流到`RTMP`服务
```shell
ffmpeg -f gdigrab -i desktop -vcodec libx264 -preset ultrafast -acodec libmp3lame -ar 44100 -ac 1 -r 25 -s 1920*1080 -f flv rtmp://127.0.0.1/live/desktop
```

>windows`桌面`推流到`RTSP`服务（rtp over udp）
```shell
ffmpeg -f gdigrab -i desktop -vcodec libx264 -preset ultrafast -acodec libmp3lame -ar 44100 -ac 1 -r 25 -f rtsp rtsp://127.0.0.1/live/desktop
```

>`RTMP`推流
```shell
ffmpeg -re -i input.flv -f flv -r 25 -s 1920*1080 -an "rtmp://127.0.0.1/live/test"
```

>`RTSP拉流转RTMP推流`
```shell
ffmpeg -rtsp_transport tcp -i "rtsp://admin:12345678@192.168.0.2" -f flv -c:v copy -a:v copy -r 25 -s 1920*1080 "rtmp://127.0.0.1/live/test"
```

>本地视频文件RTSP推流 （tcp）
```shell
ffmpeg -re -i input.mp4 -rtsp_transport tcp -vcodec h264 -acodec copy -f rtsp rtsp://127.0.0.1/live/test
```

>本地视频文件RTSP循环推流（tcp）
```shell
ffmpeg -re -stream_loop -1 -i input.mp4 -rtsp_transport tcp -c copy -f rtsp rtsp://127.0.0.1/live/test
```

>本地视频文件RTSP推流 （udp）
```shell
ffmpeg -re -i input.mp4 -rtsp_transport udp -vcodec h264 -acodec copy -f rtsp rtsp://127.0.0.1/live/test
```

>RTSP拉流并播放 （tcp）
```shell
ffplay -i -rtsp_transport tcp rtsp://127.0.0.1/live/test
```

>RTSP拉流并播放 （udp）
```shell
ffplay -i rtsp://127.0.0.1/live/test
```

## 1.10 合并

>60s长包含音频的video-60.mp4，和30s长的音频audio-30.mp3 合并。
>audio-30.mp3内的音频会替换到video-60.mp4的音频。
```shell
ffmpeg -i video-60.mp4 -i audio-30.mp3 -c:v copy -c:a aac -strict experimental -map 0:v:0 -map 1:a:0 out.mp4
```

>60s长包含音频的video-60.mp4，和30s长的音频audio-30.mp3 合并。
>合并后的out.mp4包含两路音频。
```shell
ffmpeg -i video-60.mp4 -i audio-30.mp3 -filter_complex "amix=inputs=2:duration=first:dropout_transition=0" -c:v "libx264" -c:a "aac" -y out.mp4
```


---
# 2. ffplay

<font color = red> 一般会使用ffmpeg进行处理后用ffplay直接进行播放,ffplay的功能是有限的</font>

> 播放 `h264` 裸码流
```shell
ffplay -stats -f h264 out.h264
ffplay -i out.h264
```

备注:
- `-stats` 实时显示有关音视频帧的统计信息


>播放 `aac` 裸码流
```shell
ffplay -i out.aac
```

>使用`指定解码器`播放视频
```shell
ffplay -vcodec h264 -i out.mp4
```

>播放`摄像头`
```shell
ffplay -f dshow -i video="FULL HD webcam" 
# FULL HD webcam 是通过查看列表的命令行获得的名称
```

>静音播放
```shell
ffplay -an input.mp4
```

>倍速播放
```shell
#二倍速
ffplay -vf setpts=0.5 input.mp4
```

备注:

- `setpts` play time speed

>升调播放
```shell
#1.5倍速
ffplay -af "atempo=1.5" input.mp4 
```

备注:
- `atempo` Audio Tempo（音频节奏）


---

# 3. ffprobe

>获取视频的总帧数

```shell
ffprobe -v error -count_frames -select_streams v:0 -show_entries stream=nb_read_frames -of default=nokey=1:noprint_wrappers=1 input.mp4
```

- `-v error`：这隐藏了“info”输出 (verbosity(详细程度))

- `-count_frames`：计算每个流的帧数，并在相应的流部分中报告。

- `-select_streams v:0` ：仅选择视频流 (Video stream(视频流) (0 表示索引)

- `-show_entries stream = nb_read_frames` ：只显示读取的帧数。

- `-of default = nokey = 1：noprint_wrappers = 1` ：将输出格式(也称为“writer”)设置为默认值，不打印每个字段的键(`nokey = 1`)，不打印节头和页脚(`noprint_wrappers = 1`)。

执行流程如下：

* `ffprobe` 解析并读取 `input.mp4` 视频文件。
* `-select_streams v:0` 选择了视频流中的第一个流进行分析。
* `-count_frames` 让 `ffprobe` 统计并显示该视频流中的帧数。
* `-show_entries stream=nb_read_frames` 显示了视频流中每个流的 `nb_read_frames` 字段，即已读取的帧数。
* `-of default=nokey=1:noprint_wrappers=1` 指定了输出格式为默认格式，并设置了输出参数，以便在输出时不显示键名并省略外层包装器。
* 最终，输出会显示视频文件中所选视频流的已读取帧数，根据设定的输出格式进行格式化显示。


>基本用法：
```shell
ffprobe input_file
```

* `input_file` 是你要分析的媒体文件的路径。

`ffprobe` 默认会输出媒体文件的详细信息，包括文件格式、编解码器信息、流的详细参数、时长、分辨率、比特率等等。

常见参数：

* `-show_format`：显示媒体文件的格式信息。
* `-show_streams`：显示媒体文件的各个流（视频、音频、字幕等）的详细信息。
* `-select_streams [stream_specifier]`：选择特定类型的流进行分析。
* `-show_frames`：显示每个视频帧的详细信息。

示例：
```shell
ffprobe -show_format -show_streams input.mp4
```


这会显示媒体文件 `input.mp4` 的格式信息以及所有流（视频、音频等）的详细信息。

通过 `ffprobe`，你可以深入了解媒体文件的结构和属性，有助于调试、分析和了解你所处理的音视频文件。