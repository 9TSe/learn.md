---
title: RTSP服务器
date: 2023-12-05 21:58:26
tags:
- 小项目
categories:
- 音视频
cover: /pic/1.png
---


# 1. RTSP协议简介

## 1.1 RTSP服务器基础框架

>以下代码并没有使用功能,只是通过wireshark进行抓包分析来了解RTSP协议的大致流程
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include <time.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <WinSock2.h>
#include <WS2tcpip.h>
#include <windows.h>
#include <string>
#pragma comment(lib, "ws2_32.lib") //链接阶段将指定的库文件 ws2_32.lib 加入到当前项目中。(套接字编程的文件)
#include <stdint.h>
#pragma warning( disable : 4996 )
#define SERVER_PORT      8554
#define SERVER_RTP_PORT  55532
#define SERVER_RTCP_PORT 55533

static int createTcpSocket()
{
    int sockfd;
    int on = 1;

    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0)
        return -1;

    setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, (const char*)&on, sizeof(on));

    return sockfd;
}

static int bindSocketAddr(int sockfd, const char* ip, int port)
{
    struct sockaddr_in addr;

    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    addr.sin_addr.s_addr = inet_addr(ip);

    if (bind(sockfd, (struct sockaddr*)&addr, sizeof(struct sockaddr)) < 0)
        return -1;

    return 0;
}

static int acceptClient(int sockfd, char* ip, int* port)
{
    int clientfd;
    socklen_t len = 0;
    struct sockaddr_in addr;

    memset(&addr, 0, sizeof(addr));
    len = sizeof(addr);

    clientfd = accept(sockfd, (struct sockaddr*)&addr, &len);
    if (clientfd < 0)
        return -1;

    strcpy(ip, inet_ntoa(addr.sin_addr));
    *port = ntohs(addr.sin_port);

    return clientfd;
}

static int handleCmd_OPTIONS(char* result, int cseq)
{
    sprintf(result, "RTSP/1.0 200 OK\r\n"
        "CSeq: %d\r\n"
        "Public: OPTIONS, DESCRIBE, SETUP, PLAY\r\n"
        "\r\n",
        cseq);
    return 0;
}

static int handleCmd_DESCRIBE(char* result, int cseq, char* url)
{
    char sdp[500];
    char localIp[100];

    sscanf(url, "rtsp://%[^:]:", localIp);

    sprintf(sdp, "v=0\r\n"
        "o=- 9%ld 1 IN IP4 %s\r\n"
        "t=0 0\r\n"
        "a=control:*\r\n"
        "m=video 0 RTP/AVP 96\r\n"
        "a=rtpmap:96 H264/90000\r\n"
        "a=control:track0\r\n",
        time(NULL), localIp);

    sprintf(result, "RTSP/1.0 200 OK\r\nCSeq: %d\r\n"
        "Content-Base: %s\r\n"
        "Content-type: application/sdp\r\n"
        "Content-length: %zu\r\n\r\n"
        "%s",
        cseq,
        url,
        strlen(sdp),
        sdp);

    return 0;
}

static int handleCmd_SETUP(char* result, int cseq, int clientRtpPort)
{
    sprintf(result, "RTSP/1.0 200 OK\r\n"
        "CSeq: %d\r\n"
        "Transport: RTP/AVP;unicast;client_port=%d-%d;server_port=%d-%d\r\n"
        "Session: 66334873\r\n"
        "\r\n",
        cseq,
        clientRtpPort,
        clientRtpPort + 1,
        SERVER_RTP_PORT,
        SERVER_RTCP_PORT);

    return 0;
}

static int handleCmd_PLAY(char* result, int cseq)
{
    sprintf(result, "RTSP/1.0 200 OK\r\n"
        "CSeq: %d\r\n"
        "Range: npt=0.000-\r\n"
        "Session: 66334873; timeout=10\r\n\r\n",
        cseq);

    return 0;
}

static void doClient(int clientSockfd, const char* clientIP, int clientPort) {

    char method[40];
    char url[100];
    char version[40];
    int CSeq;

    int clientRtpPort, clientRtcpPort;
    char* rBuf = (char*)malloc(10000);
    char* sBuf = (char*)malloc(10000);

    while (true) 
    {
        int recvLen;

        recvLen = recv(clientSockfd, rBuf, 2000, 0);
        if (recvLen <= 0)
            break;

        rBuf[recvLen] = '\0';
        std::string recvStr = rBuf;
        printf(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>\n");
        printf("%s rBuf = %s \n", __FUNCTION__, rBuf);

        const char* sep = "\n";
        char* line = strtok(rBuf, sep); //分割字符串, 遇见回车分割一次
        while (line) 
        {
            if (strstr(line, "OPTIONS") ||
                strstr(line, "DESCRIBE") ||
                strstr(line, "SETUP") ||
                strstr(line, "PLAY")) {

                if (sscanf(line, "%s %s %s\r\n", method, url, version) != 3) 
                {
                    // error
                    // OPTIONS rtsp://127.0.0.1:8554 RTSP/1.0
                }
            }
            else if (strstr(line, "CSeq")) 
            {
                if (sscanf(line, "CSeq: %d\r\n", &CSeq) != 1) 
                {
                    // error
                }
            }
            else if (!strncmp(line, "Transport:", strlen("Transport:"))) 
            {
                // Transport: RTP/AVP/UDP;unicast;client_port=13358-13359
                // Transport: RTP/AVP;unicast;client_port=13358-13359

                if (sscanf(line, "Transport: RTP/AVP/UDP;unicast;client_port=%d-%d\r\n",
                    &clientRtpPort, &clientRtcpPort) != 2) 
                {
                    // error
                    printf("parse Transport error \n");
                }
            }
            line = strtok(NULL, sep);
        }

        if (!strcmp(method, "OPTIONS")) 
        {
            if (handleCmd_OPTIONS(sBuf, CSeq))
            {
                printf("failed to handle options\n");
                break;
            }
        }
        else if (!strcmp(method, "DESCRIBE")) 
        {
            if (handleCmd_DESCRIBE(sBuf, CSeq, url))
            {
                printf("failed to handle describe\n");
                break;
            }

        }
        else if (!strcmp(method, "SETUP")) 
        {
            if (handleCmd_SETUP(sBuf, CSeq, clientRtpPort))
            {
                printf("failed to handle setup\n");
                break;
            }
        }
        else if (!strcmp(method, "PLAY")) 
        {
            if (handleCmd_PLAY(sBuf, CSeq))
            {
                printf("failed to handle play\n");
                break;
            }
        }
        else 
        {
            printf("未定义的method = %s \n", method);
            break;
        }
        printf("<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<\n");
        printf("%s sBuf = %s \n", __FUNCTION__, sBuf);

        send(clientSockfd, sBuf, strlen(sBuf), 0);


        //开始播放，发送RTP包
        if (!strcmp(method, "PLAY")) 
        {
            printf("start play\n");
            printf("client ip:%s\n", clientIP);
            printf("client port:%d\n", clientRtpPort);
            while (true) 
            {
                Sleep(40);
                //usleep(40000);//1000/25 * 1000
            }

            break;
        }

        memset(method, 0, sizeof(method) / sizeof(char));
        memset(url, 0, sizeof(url) / sizeof(char));
        CSeq = 0;

    }
    closesocket(clientSockfd);
    free(rBuf);
    free(sBuf);
}


int main(int argc, char* argv[])
{
    // 启动windows socket start
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) //初始化winsock库,使用2.2版本
    {
        printf("PC Server Socket Start Up Error \n");
        return -1;
    }
    // 启动windows socket end

    int serverSockfd;

    serverSockfd = createTcpSocket();
    if (serverSockfd < 0)
    {
        WSACleanup(); //释放winsock库占用的资源
        printf("failed to create tcp socket\n");
        return -1;
    }

    if (bindSocketAddr(serverSockfd, "0.0.0.0", SERVER_PORT) < 0)
    {
        printf("failed to bind addr\n");
        return -1;
    }

    if (listen(serverSockfd, 10) < 0)
    {
        printf("failed to listen\n");
        return -1;
    }

    printf("%s rtsp://127.0.0.1:%d\n", __FILE__, SERVER_PORT);

    while (true) 
    {
        int clientSockfd;
        char clientIp[40];
        int clientPort;

        clientSockfd = acceptClient(serverSockfd, clientIp, &clientPort);
        if (clientSockfd < 0)
        {
            printf("failed to accept client\n");
            return -1;
        }
        printf("accept client;client ip:%s,client port:%d\n", clientIp, clientPort);

        doClient(clientSockfd, clientIp, clientPort); //关键部分
    }
    closesocket(serverSockfd);
    return 0;
}
```
## 1.2 分析抓包数据

![](/img/rtsp1.png)

>客户端第一次发出请求(OPTIONS)
>向服务器询问有那种请求方式

![](/img/rtsp2.png)


>服务器回应OPTIONS
>服务器给出发出请求的方式

![](/img/rtsp3.png)

>客户端第二次发出请求(DESCRIBE)
>请求指定流媒体的描述信息（`SDP`，Session Description Protocol）

![](/img/rtsp4.png)

>服务器回应DESCRIBE
>回应会话描述(`SDP`) 返回流媒体的描述信息，包括媒体类型、编解码器、流地址等。

![](/img/rtsp5.png)

>客户端第三次发出请求(SETUP)
>请求建立媒体流传输的通道
>两个(RTCP和RTP通道)端口号

![](/img/rtsp6.png)

>服务器回应SETUP
>返回确认或者错误消息。
>在这个步骤中，可以指定传输协议（UDP、TCP、RTP 等）和端口号等参数。

![](/img/rtsp7.png)

>客户端第四次发出请求(PLAY)
>请求开始播放流媒体

![](/img/rtsp8.png)

>服务器回应PLAY
>接受请求，并开始向客户端发送媒体数据。

![](/img/rtsp9.png)


# 2. UDP的RTP传输h264的RTSP服务器，能拉流播放

>上一部分已经实现了一个`RTSP`协议交互的案例，客户端播放器能够向我们的RTSP服务端发起连接建立的请求，并且客户端在发起RTSP的`Play`请求以后，RTSP服务端也已经回复了`Play`请求的确认。

这部分需要实现，客户端建立与RTSP服务端的连接后，并且在RTSP服务端回复了客户端的Play请求以后.
服务端需要源源不断的读取一个本地h264视频文件，并将读取到的h264视频流封装到RTP数据包中，再推送至客户端。
这样我们就实现了一个简单的支持RTSP协议流媒体分发服务。


问题的关键就是 `如何将H264封装到RTP数据包`

## 2.1 RTP理解

### 2.1.1 RTP简述

RTP:实时传输协议（Real-time Transport Protocol或简写RTP）是一个网络传输协议.
RTP定义了两种报文：`RTP报文`和`RTCP报文`
- `RTP报文`用于传送媒体数据（如音频和视频,包括自定义类型,如字幕等）
它由 RTP报头和数据两部分组成，RTP数据部分称为有效载荷(payload)；
- `RTCP报文`用于传送控制信息，以实现协议控制功能。
- `RTP报文和RTCP报文`将作为下层协议的数据单元进行传输。
<font color = red>如果使用UDP，则RTP报文和RTCP报文分别使用两个相邻的UDP端口，RTP报文使用低端口，RTCP报文使用高端口。</font>
如果使用其它的下层协议，RTP报文和RTCP报文可以合并，放在一个数据单元中一起传送，控制`信息在前`，媒体`数据在后`。通常，RTP是由应用程序实现的。
eg: TCP作为协议时,RTP和RTCP甚至RTSP都是一个通道进行传输的



![](/img/rtsp10.png)

### 2.1.2 RTP报文格式

- `V`：RTP协议的版本号，占`2位`
当前协议版本号为2。
- `P`：填充标志，占`1位`
如果P=1，则在该报文的尾部填充一个或多个额外的八位组，它们不是有效载荷的一部分。
- `X`：扩展标志，占`1位`
如果X=1，则在RTP报头后跟有一个扩展报头。
- `CC`：CSRC计数器，占`4位`
指示CSRC 标识符的个数。
- `M`: 标记，占`1位`, 不同的有效载荷有不同的含义
对于视频，标记一帧的结束；
对于音频，标记会话的开始。
- `PT`: 有效载荷类型，占`7位`
用于说明RTP报文中有效载荷的类型，如GSM音频、JPEM图像等。
- `序列号`：占`16位`
用于标识发送者所发送的RTP报文的序列号，每发送一个报文，序列号增1。
接收者通过序列号来检测报文丢失情况，重新排序报文，恢复数据。
- `时戳(Timestamp)`：占`32位`，时戳反映了该RTP报文的第一个八位组的采样时刻。
接收者使用时戳来计算延迟和延迟抖动，并进行同步控制。
- `同步信源(SSRC)标识符`：占`32位`，用于标识同步信源。
该标识符是随机选择的，参加同一视频会议的两个同步信源不能有相同的SSRC。
- `特约信源(CSRC)标识符`：每个CSRC标识符占`32位`，可以有0～15个。
每个CSRC标识了包含在该RTP报文有效载荷中的所有特约信源。

>每一行占4字节,32位

![](/img/rtsp11.png)

>视频会议,直播连麦等类似情况时(多路流汇成一路流),CSRC才会存在(多个SSRC汇集为CSRC)

```cpp
#pragma once
#pragma comment(lib, "ws2_32.lib")
#include <stdint.h>

#define RTP_VESION              2

#define RTP_PAYLOAD_TYPE_H264   96
#define RTP_PAYLOAD_TYPE_AAC    97

#define RTP_HEADER_SIZE         12
#define RTP_MAX_PKT_SIZE        1400

 /*
  *    0                   1                   2                   3
  *    7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *   |V=2|P|X|  CC   |M|     PT      |       sequence number         |
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *   |                           timestamp                           |
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *   |           synchronization source (SSRC) identifier            |
  *   +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
  *   |            contributing source (CSRC) identifiers             |
  *   :                             ....                              :
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *
  */
struct RtpHeader
{
	//位段中,定义的占位数是倒着来的,即在第0个字节中,最后四位是csrcLen
    /* byte 0 */
    uint8_t csrcLen : 4;//CSRC计数器，占4位，指示 CSRC 标识符的个数。
    uint8_t extension : 1;//占1位，如果X=1，则在RTP报头后跟有一个扩展报头。
    					//一般只有传输类型是非音频,非视频的自定义类型或许需要这个X=1
    uint8_t padding : 1;//填充标志，占1位，如果P=1，则在该报文的尾部填充一个或多个额外的八位组
    					//它们不是有效载荷的一部分。
    uint8_t version : 2;//RTP协议的版本号，占2位，当前协议版本号为2。

    /* byte 1 */
    uint8_t payloadType : 7;//有效载荷类型，占7位，用于说明RTP报文中有效载荷的类型，如GSM音频、JPEM图像等。
    uint8_t marker : 1;//标记，占1位，不同的有效载荷有不同的含义，对于视频，标记一帧的结束；
    					//对于音频，标记会话的开始。

    /* bytes 2,3 */
    uint16_t seq;//占16位，用于标识发送者所发送的RTP报文的序列号，每发送一个报文(eg:每发送一帧)，序列号增1。
    			//接收者通过序列号来检测报文丢失情况，重新排序报文，恢复数据。

    /* bytes 4-7 */
    uint32_t timestamp;//占32位，时戳反映了该RTP报文的第一个八位组的采样时刻。
    				//接收者使用时戳来计算延迟和延迟抖动，并进行同步控制。(音视频进行同步)

    /* bytes 8-11 */
    uint32_t ssrc;//占32位，用于标识同步信源。该标识符是随机选择的，参加同一视频会议的两个同步信源不能有相同的SSRC。

   /*标准的RTP Header 还可能存在 0-15个特约信源(CSRC)标识符
   
   每个CSRC标识符占32位，可以有0～15个。每个CSRC标识了包含在该RTP报文有效载荷中的所有特约信源

   */
};

//RTP包的结构体
struct RtpPacket
{
    struct RtpHeader rtpHeader;
    uint8_t payload[0];
};
//包含一个RTP头部和RTP载荷
```


## 2.2 H264理解

### 2.2.1 压缩方式
H264压缩技术主要采用了以下几种方法对视频数据进行压缩:
- 帧内预测压缩，解决的是空域数据冗余问题。(eg:jpg) 不借助于其他数据
- 帧间预测压缩（运动估计与补偿），解决的是时域数据冗徐问题。
- 整数离散余弦变换（DCT），将空间上的相关性变为频域上无关的数据然后进行量化。
- CABAC压缩。

经过`压缩后`的帧分为：I帧，P帧和B帧:

- I帧：关键帧，采用`帧内压缩`技术。
- P帧：向前参考帧，在压缩时，只参考前面已经处理的帧。
采用帧间压缩技术。占I帧的一半大小
- B帧：双向参考帧，在压缩时，它即参考前而的帧，又参考它后面的帧。采用帧间压缩技术。占I帧1/4大小
- 图像序列GOP(GOP组),将一个视频分为若干部分,防止因为部分帧损坏而导致视频崩盘.GOP至少有一个I帧

>虽然B帧的压缩率是最高，但是他占用的CPU以及耗时非常多，既然耗时，在实时直播视频情况就会导致一个延迟的情况。
> 如果只是想减少空间大小，B帧将是一个很好的选择。
> 所以一般在直播中是没有B帧，只有I帧和P帧。

IDR帧和I帧的关系。

IDR(Instantannous Decoder Refresh) 解码器立即刷新 

作用：在解码的过程，一旦有一帧数据出现错误，将是无法恢复的过程，后面数据帧不能使用。
当有了IDR帧，解码器收到IDR帧时，就会将`缓冲区的数据清空`，找到第一个IDR帧，重新解码。
I和IDR帧都使用`帧内预测`，在编码解码中为了方便，首个I帧要和其他I帧区别开，把`第一个I帧叫IDR`，这样方便控制编码和解码流程。
IDR帧必须是一个I帧，但是I帧不一定是IDR帧，这个帧出现的时候，是告诉解码器，可以清除掉所有的参考帧，这是一个全新的序列，新的`GOP`已经开始。I帧有被跨帧参考的可能,IDR不会。
每个GOP中的第一帧就是IDR帧。
`IDR帧是一种特殊的I帧。`

帧与分组的关系：

![](/img/rtsp12.png)

### 2.2.2 h264组成
1. 网络提取层 (Network Abstraction Layer，`NAL`), 编码(压缩)后的流
2. 视讯编码层 (Video Coding Layer，`VCL`), 产生NAL的过程

### 2.2.3 码流结构
H.264的功能分为两层，视频编码层（VCL）和网络提取层（NAL）。
1.VCL数据即被压缩编码后的视频数据序列。
2.在VCL数据要封装到NAL单元中之后，才可以用来传输或存储。

![](/img/rtsp13.png)

- SPS：序列参数集，作用于一系列连续的编码图像；
- PPS：图像参数集，作用于编码视频序列中一个或多个独立的图像；

NALU根据`nal_unit_type`的类型，可以分为：
`VCL的NAL单元 `和 `非VCL的NAL单元`


## 2.3 解封装mp4生成h264文件

```bash
ffmpeg -i test.mp4 -codec copy -bsf: h264_mp4toannexb -f h264 test.h264
```


## 2.4 将H264封装到RTP数据包

H.264由一个一个的NALU组成，每个NALU之间使用`00 00 00 01`或`00 00 01`分隔开，每个NALU的第一次字节都有特殊的含义,

![](/img/rtsp14.png)

![](/img/rtsp15.png)


 
1. `F(forbiden):禁止位`，占用NAL头的第一个位，当禁止位值为1时表示语法错误；
2. `NRI:参考级别`，占用NAL头的第二到第三个位；值越大，该NAL越重要。
3. `Type:Nal单元数据类型`，也就是标识该NAL单元的数据类型是哪种，占用NAL头的第四到第8个位；


常用Nalu_type：

- 0x06 (0 00 00110) `SEI`      type = 6
0x67 (0 11 00111) `SPS`     type = 7
0x68 (0 11 01000) `PPS`      type = 8

- 0x65 (0 11 00101) `IDR`      type = 5
0x65 (0 10 00101) `IDR`      type = 5
0x65 (0 01 00101) `IDR`     type = 5
0x65 (0 00 00101) `IDR`      type = 5

- 0x61 (0 11 00001) `I帧`      type = 1
0x41 (0 10 00001) `P帧`     type = 1
0x01 (0 00 00001) `B帧`     type = 1


![](/img/rtsp16.png)


对于H.264格式了解这些就够了，我们的目的是想从一个H.264的文件中将一个一个的NALU提取出来，然后封装成RTP包，下面介绍如何将NALU封装成RTP包。



 H.264可以由三种RTP打包方式

- `单NALU打包`： 一个RTP包包含一个完整的NALU

- `聚合打包`：对于较小的NALU，一个RTP包可包含多个完整的NALU

- `分片打包`：对于较大的NALU，一个NALU可以分为多个RTP包发送

`注意`：这里要区分好概念，每一个RTP包都包含一个RTP头部和RTP荷载，这是固定的。而H.264发送数据可支持三种RTP打包方式

比较常用的是单NALU打包和分片打包，这里只介绍两种

**单NALU打包**
所谓单NALU打包就是将一整个NALU的数据放入RTP包的载荷中，这是最简单的一种方式。

**分片打包**
每个RTP包都有大小限制的，因为RTP一般都是使用UDP发送，UDP没有流量控制，所以要限制每一次发送的大小，所以如果一个NALU的太大，就需要分成多个RTP包发送，至于如何分成多个RTP包，如下：

首先要明确，RTP包的格式是绝不会变的，永远多是RTP头+RTP载荷

![](/img/rtsp17.png)

 
RTP头部是固定的，那么只能在RTP载荷中去添加额外信息来说明这个RTP包是表示同一个NALU
如果是分片打包的话，那么在RTP载荷开始有两个字节的信息，然后再是NALU的内容

![](/img/rtsp18.png)


第一个字节位`FU Indicator`，其格式如下

![](/img/rtsp19.png)

 
高三位：与NALU第一个字节的高三位相同
Type：`28`，表示该RTP包一个分片，为什么是28？
因为H.264的规范中定义的，此外还有许多其他Type

第二个字节位`FU Header`，其格式如下

![](/img/rtsp20.png)

 
S：标记该分片打包的第一个RTP包
E：比较该分片打包的最后一个RTP包
Type：NALU的Type



## 2.5 代码实现
>rtp.h

```cpp
#pragma once
#pragma comment(lib, "ws2_32.lib")
#include <stdint.h>

#define RTP_VESION              2

#define RTP_PAYLOAD_TYPE_H264   96
#define RTP_PAYLOAD_TYPE_AAC    97

#define RTP_HEADER_SIZE         12
#define RTP_MAX_PKT_SIZE        1400

 /*
  *    0                   1                   2                   3
  *    7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *   |V=2|P|X|  CC   |M|     PT      |       sequence number         |
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *   |                           timestamp                           |
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *   |           synchronization source (SSRC) identifier            |
  *   +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
  *   |            contributing source (CSRC) identifiers             |
  *   :                             ....                              :
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *
  */
struct RtpHeader
{
    /* byte 0 */
    uint8_t csrcLen : 4;//CSRC计数器，占4位，指示CSRC 标识符的个数。
    uint8_t extension : 1;//占1位，如果X=1，则在RTP报头后跟有一个扩展报头。
    uint8_t padding : 1;//填充标志，占1位，如果P=1，则在该报文的尾部填充一个或多个额外的八位组，它们不是有效载荷的一部分。
    uint8_t version : 2;//RTP协议的版本号，占2位，当前协议版本号为2。

    /* byte 1 */
    uint8_t payloadType : 7;//有效载荷类型，占7位，用于说明RTP报文中有效载荷的类型，如GSM音频、JPEM图像等。
    uint8_t marker : 1;//标记，占1位，不同的有效载荷有不同的含义，对于视频，标记一帧的结束；对于音频，标记会话的开始。

    /* bytes 2,3 */
    uint16_t seq;//占16位，用于标识发送者所发送的RTP报文的序列号，每发送一个报文，序列号增1。接收者通过序列号来检测报文丢失情况，重新排序报文，恢复数据。

    /* bytes 4-7 */
    uint32_t timestamp;//占32位，时戳反映了该RTP报文的第一个八位组的采样时刻。接收者使用时戳来计算延迟和延迟抖动，并进行同步控制。

    /* bytes 8-11 */
    uint32_t ssrc;//占32位，用于标识同步信源。该标识符是随机选择的，参加同一视频会议的两个同步信源不能有相同的SSRC。

   /*标准的RTP Header 还可能存在 0-15个特约信源(CSRC)标识符
   
   每个CSRC标识符占32位，可以有0～15个。每个CSRC标识了包含在该RTP报文有效载荷中的所有特约信源

   */
};

struct RtpPacket
{
    struct RtpHeader rtpHeader;
    uint8_t payload[0];
};

void rtpHeaderInit(struct RtpPacket* rtpPacket, uint8_t csrcLen, uint8_t extension,
    uint8_t padding, uint8_t version, uint8_t payloadType, uint8_t marker,
    uint16_t seq, uint32_t timestamp, uint32_t ssrc);

int rtpSendPacketOverTcp(int clientSockfd, struct RtpPacket* rtpPacket, uint32_t dataSize);
int rtpSendPacketOverUdp(int serverRtpSockfd, const char* ip, int16_t port, struct RtpPacket* rtpPacket, uint32_t dataSize);
```

>rtp.cpp

```cpp
#include <sys/types.h>
#include <WinSock2.h>
#include <WS2tcpip.h>
#include <windows.h>
#include "rtp.h"

void rtpHeaderInit(struct RtpPacket* rtpPacket, uint8_t csrcLen, uint8_t extension,
    uint8_t padding, uint8_t version, uint8_t payloadType, uint8_t marker,
    uint16_t seq, uint32_t timestamp, uint32_t ssrc)
{
    rtpPacket->rtpHeader.csrcLen = csrcLen;
    rtpPacket->rtpHeader.extension = extension;
    rtpPacket->rtpHeader.padding = padding;
    rtpPacket->rtpHeader.version = version;
    rtpPacket->rtpHeader.payloadType = payloadType;
    rtpPacket->rtpHeader.marker = marker;
    rtpPacket->rtpHeader.seq = seq;
    rtpPacket->rtpHeader.timestamp = timestamp;
    rtpPacket->rtpHeader.ssrc = ssrc;
}
int rtpSendPacketOverTcp(int clientSockfd, struct RtpPacket* rtpPacket, uint32_t dataSize)
{

    rtpPacket->rtpHeader.seq = htons(rtpPacket->rtpHeader.seq);
    rtpPacket->rtpHeader.timestamp = htonl(rtpPacket->rtpHeader.timestamp);
    rtpPacket->rtpHeader.ssrc = htonl(rtpPacket->rtpHeader.ssrc);

    uint32_t rtpSize = RTP_HEADER_SIZE + dataSize;
    char* tempBuf = (char *)malloc(4 + rtpSize);
    tempBuf[0] = 0x24;//$
    tempBuf[1] = 0x00;
    tempBuf[2] = (uint8_t)(((rtpSize) & 0xFF00) >> 8);
    tempBuf[3] = (uint8_t)((rtpSize) & 0xFF);
    memcpy(tempBuf + 4, (char*)rtpPacket, rtpSize);

    int ret = send(clientSockfd, tempBuf, 4 + rtpSize, 0);

    rtpPacket->rtpHeader.seq = ntohs(rtpPacket->rtpHeader.seq);
    rtpPacket->rtpHeader.timestamp = ntohl(rtpPacket->rtpHeader.timestamp);
    rtpPacket->rtpHeader.ssrc = ntohl(rtpPacket->rtpHeader.ssrc);

    free(tempBuf);
    tempBuf = NULL;

    return ret;
}
int rtpSendPacketOverUdp(int serverRtpSockfd, const char* ip, int16_t port, struct RtpPacket* rtpPacket, uint32_t dataSize)
{
    
    struct sockaddr_in addr;
    int ret;

    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    addr.sin_addr.s_addr = inet_addr(ip);

    rtpPacket->rtpHeader.seq = htons(rtpPacket->rtpHeader.seq);//从主机字节顺序转变成网络字节顺序
    rtpPacket->rtpHeader.timestamp = htonl(rtpPacket->rtpHeader.timestamp);
    rtpPacket->rtpHeader.ssrc = htonl(rtpPacket->rtpHeader.ssrc);

    ret = sendto(serverRtpSockfd, (char *)rtpPacket, dataSize + RTP_HEADER_SIZE, 0,
        (struct sockaddr*)&addr, sizeof(addr));

    rtpPacket->rtpHeader.seq = ntohs(rtpPacket->rtpHeader.seq);
    rtpPacket->rtpHeader.timestamp = ntohl(rtpPacket->rtpHeader.timestamp);
    rtpPacket->rtpHeader.ssrc = ntohl(rtpPacket->rtpHeader.ssrc);

    return ret;

}

```



>main.cpp

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include <time.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <WinSock2.h>
#include <WS2tcpip.h>
#include <windows.h>
#include "rtp.h"

#define H264_FILE_NAME   "D:\\edgedown\\test.h264"
#define SERVER_PORT      8554
#define SERVER_RTP_PORT  55532
#define SERVER_RTCP_PORT 55533
#define BUF_MAX_SIZE     (1024*1024)

static int createTcpSocket()
{
    int sockfd;
    int on = 1;

    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0)
        return -1;

    setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, (const char*)&on, sizeof(on));

    return sockfd;
}

static int createUdpSocket()
{
    int sockfd;
    int on = 1;

    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd < 0)
        return -1;

    setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, (const char*)&on, sizeof(on));

    return sockfd;
}

static int bindSocketAddr(int sockfd, const char* ip, int port)
{
    struct sockaddr_in addr;

    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    addr.sin_addr.s_addr = inet_addr(ip);

    if (bind(sockfd, (struct sockaddr*)&addr, sizeof(struct sockaddr)) < 0)
        return -1;

    return 0;
}

static int acceptClient(int sockfd, char* ip, int* port)
{
    int clientfd;
    socklen_t len = 0;
    struct sockaddr_in addr;

    memset(&addr, 0, sizeof(addr));
    len = sizeof(addr);

    clientfd = accept(sockfd, (struct sockaddr*)&addr, &len);
    if (clientfd < 0)
        return -1;

    strcpy(ip, inet_ntoa(addr.sin_addr));
    *port = ntohs(addr.sin_port);

    return clientfd;
}

static inline int startCode3(char* buf)
{
    if (buf[0] == 0 && buf[1] == 0 && buf[2] == 1)
        return 1;
    else
        return 0;
}

static inline int startCode4(char* buf)
{
    if (buf[0] == 0 && buf[1] == 0 && buf[2] == 0 && buf[3] == 1)
        return 1;
    else
        return 0;
}

static char* findNextStartCode(char* buf, int len)
{
    int i;

    if (len < 3)
        return NULL;

    for (i = 0; i < len - 3; ++i)
    {
        if (startCode3(buf) || startCode4(buf))
            return buf;

        ++buf;
    }

    if (startCode3(buf))
        return buf;

    return NULL;
}

static int getFrameFromH264File(FILE* fp, char* frame, int size) {
    int rSize, frameSize;
    char* nextStartCode;

    /*if (!fp < 0)
        return -1;*/

    rSize = fread(frame, 1, size, fp);

    if (!startCode3(frame) && !startCode4(frame))
        return -1;

    nextStartCode = findNextStartCode(frame + 3, rSize - 3);
    if (!nextStartCode)
    {
        //lseek(fd, 0, SEEK_SET);
        //frameSize = rSize;
        return -1;
    }
    else
    {
        frameSize = (nextStartCode - frame);
        fseek(fp, frameSize - rSize, SEEK_CUR);

    }

    return frameSize;
}

static int rtpSendH264Frame(int serverRtpSockfd, const char* ip, int16_t port,
    struct RtpPacket* rtpPacket, char* frame, uint32_t frameSize)
{
 
    uint8_t naluType; // nalu第一个字节
    int sendBytes = 0;
    int ret;

    naluType = frame[0];//& 0x1F; //标准写法应该是与后五位相与获取第一个字节,由于后面有在类型判断上进行了&,所以此处没有&

    printf("frameSize=%d \n", frameSize);

    if (frameSize <= RTP_MAX_PKT_SIZE) // nalu长度小于最大包长：单一NALU单元模式
    {

         //*   0 1 2 3 4 5 6 7 8 9
         //*  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
         //*  |F|NRI|  Type   | a single NAL unit ... |
         //*  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

        memcpy(rtpPacket->payload, frame, frameSize);
        ret = rtpSendPacketOverUdp(serverRtpSockfd, ip, port, rtpPacket, frameSize);
        if(ret < 0)
            return -1;

        rtpPacket->rtpHeader.seq++; //序号加一
        sendBytes += ret;
        if ((naluType & 0x1F) == 7 || (naluType & 0x1F) == 8) // 如果是SPS、PPS就不需要加时间戳
            goto out;
    }
    else // nalu长度小于最大包场：分片模式
    {

         //*  0                   1                   2
         //*  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
         //* +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
         //* | FU indicator  |   FU header   |   FU payload   ...  |
         //* +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+



         //*     FU Indicator
         //*    0 1 2 3 4 5 6 7
         //*   +-+-+-+-+-+-+-+-+
         //*   |F|NRI|  Type   |
         //*   +---------------+



         //*      FU Header
         //*    0 1 2 3 4 5 6 7
         //*   +-+-+-+-+-+-+-+-+
         //*   |S|E|R|  Type   |
         //*   +---------------+


        int pktNum = frameSize / RTP_MAX_PKT_SIZE;       // 有几个完整的包
        int remainPktSize = frameSize % RTP_MAX_PKT_SIZE; // 剩余不完整包的大小
        int i, pos = 1;

        // 发送完整的包
        for (i = 0; i < pktNum; i++)
        {
            rtpPacket->payload[0] = (naluType & 0x60) | 28; //2,3位
            rtpPacket->payload[1] = naluType & 0x1F; //后5位

            if (i == 0) //第一包数据
                rtpPacket->payload[1] |= 0x80; // start
            else if (remainPktSize == 0 && i == pktNum - 1) //最后一包数据
                rtpPacket->payload[1] |= 0x40; // end

            memcpy(rtpPacket->payload+2, frame+pos, RTP_MAX_PKT_SIZE);
            ret = rtpSendPacketOverUdp(serverRtpSockfd, ip, port, rtpPacket, RTP_MAX_PKT_SIZE+2);
            if(ret < 0)
                return -1;

            rtpPacket->rtpHeader.seq++;
            sendBytes += ret;
            pos += RTP_MAX_PKT_SIZE;
        }

        // 发送剩余的数据
        if (remainPktSize > 0)
        {
            rtpPacket->payload[0] = (naluType & 0x60) | 28;
            rtpPacket->payload[1] = naluType & 0x1F;
            rtpPacket->payload[1] |= 0x40; //end

            memcpy(rtpPacket->payload+2, frame+pos, remainPktSize+2);
            ret = rtpSendPacketOverUdp(serverRtpSockfd, ip, port, rtpPacket, remainPktSize+2);
            if(ret < 0)
                return -1;

            rtpPacket->rtpHeader.seq++;
            sendBytes += ret;
        }
    }
    rtpPacket->rtpHeader.timestamp += 90000 / 25; //  一秒定义为90000的单位 / 帧数
    out:

    return sendBytes;

}


static int handleCmd_OPTIONS(char* result, int cseq)
{
    sprintf(result, "RTSP/1.0 200 OK\r\n"
        "CSeq: %d\r\n"
        "Public: OPTIONS, DESCRIBE, SETUP, PLAY\r\n"
        "\r\n",
        cseq);

    return 0;
}

static int handleCmd_DESCRIBE(char* result, int cseq, char* url)
{
    char sdp[500];
    char localIp[100];

    sscanf(url, "rtsp://%[^:]:", localIp);

    sprintf(sdp, "v=0\r\n"
        "o=- 9%ld 1 IN IP4 %s\r\n"
        "t=0 0\r\n"
        "a=control:*\r\n"
        "m=video 0 RTP/AVP 96\r\n"
        "a=rtpmap:96 H264/90000\r\n"
        "a=control:track0\r\n",
        time(NULL), localIp);

    sprintf(result, "RTSP/1.0 200 OK\r\nCSeq: %d\r\n"
        "Content-Base: %s\r\n"
        "Content-type: application/sdp\r\n"
        "Content-length: %zu\r\n\r\n"
        "%s",
        cseq,
        url,
        strlen(sdp),
        sdp);

    return 0;
}

static int handleCmd_SETUP(char* result, int cseq, int clientRtpPort)
{
    sprintf(result, "RTSP/1.0 200 OK\r\n"
        "CSeq: %d\r\n"
        "Transport: RTP/AVP;unicast;client_port=%d-%d;server_port=%d-%d\r\n"
        "Session: 66334873\r\n"
        "\r\n",
        cseq,
        clientRtpPort,
        clientRtpPort + 1,
        SERVER_RTP_PORT,
        SERVER_RTCP_PORT);

    return 0;
}

static int handleCmd_PLAY(char* result, int cseq)
{
    sprintf(result, "RTSP/1.0 200 OK\r\n"
        "CSeq: %d\r\n"
        "Range: npt=0.000-\r\n"
        "Session: 66334873; timeout=10\r\n\r\n",
        cseq);

    return 0;
}

static void doClient(int clientSockfd, const char* clientIP, int clientPort) {

    char method[40];
    char url[100];
    char version[40];
    int CSeq;

    int serverRtpSockfd = -1, serverRtcpSockfd = -1;
    int clientRtpPort, clientRtcpPort;
    char* rBuf = (char*)malloc(BUF_MAX_SIZE);
    char* sBuf = (char*)malloc(BUF_MAX_SIZE);

    while (true) {
        int recvLen;

        recvLen = recv(clientSockfd, rBuf, BUF_MAX_SIZE, 0);
        if (recvLen <= 0) {
            break;
        }

        rBuf[recvLen] = '\0';
        printf("%s rBuf = %s \n",__FUNCTION__,rBuf);

        const char* sep = "\n";
        char* line = strtok(rBuf, sep);
        while (line) {
            if (strstr(line, "OPTIONS") ||
                strstr(line, "DESCRIBE") ||
                strstr(line, "SETUP") ||
                strstr(line, "PLAY")) {

                if (sscanf(line, "%s %s %s\r\n", method, url, version) != 3) {
                    // error
                }
            }
            else if (strstr(line, "CSeq")) {
                if (sscanf(line, "CSeq: %d\r\n", &CSeq) != 1) {
                    // error
                }
            }
            else if (!strncmp(line, "Transport:", strlen("Transport:"))) {
                // Transport: RTP/AVP/UDP;unicast;client_port=13358-13359
                // Transport: RTP/AVP;unicast;client_port=13358-13359

                if (sscanf(line, "Transport: RTP/AVP/UDP;unicast;client_port=%d-%d\r\n",
                    &clientRtpPort, &clientRtcpPort) != 2) {
                    // error
                    printf("parse Transport error \n");
                }
            }
            line = strtok(NULL, sep);
        }

        if (!strcmp(method, "OPTIONS")) {
            if (handleCmd_OPTIONS(sBuf, CSeq))
            {
                printf("failed to handle options\n");
                break;
            }
        }
        else if (!strcmp(method, "DESCRIBE")) {
            if (handleCmd_DESCRIBE(sBuf, CSeq, url))
            {
                printf("failed to handle describe\n");
                break;
            }
        }
        else if (!strcmp(method, "SETUP")) {
            if (handleCmd_SETUP(sBuf, CSeq, clientRtpPort))
            {
                printf("failed to handle setup\n");
                break;
            }

            serverRtpSockfd = createUdpSocket();
            serverRtcpSockfd = createUdpSocket();

            if (serverRtpSockfd < 0 || serverRtcpSockfd < 0)
            {
                printf("failed to create udp socket\n");
                break;
            }

            if (bindSocketAddr(serverRtpSockfd, "0.0.0.0", SERVER_RTP_PORT) < 0 ||
                bindSocketAddr(serverRtcpSockfd, "0.0.0.0", SERVER_RTCP_PORT) < 0)
            {
                printf("failed to bind addr\n");
                break;
            }

        }
        else if (!strcmp(method, "PLAY")) {
            if (handleCmd_PLAY(sBuf, CSeq))
            {
                printf("failed to handle play\n");
                break;
            }
        }
        else {
            printf("未定义的method = %s \n", method);
            break;
        }
        printf("sBuf = %s \n", sBuf);
        printf("%s sBuf = %s \n", __FUNCTION__, sBuf);

        send(clientSockfd, sBuf, strlen(sBuf), 0);


        //开始播放，发送RTP包
        if (!strcmp(method, "PLAY")) {

            int frameSize, startCode;
            char* frame = (char*)malloc(500000);
            struct RtpPacket* rtpPacket = (struct RtpPacket*)malloc(500000);
            FILE* fp = fopen(H264_FILE_NAME, "rb");
            if (!fp) {
                printf("读取 %s 失败\n", H264_FILE_NAME);
                break;
            }
            rtpHeaderInit(rtpPacket, 0, 0, 0, RTP_VESION, RTP_PAYLOAD_TYPE_H264, 0,
                0, 0, 0x88923423);

            printf("start play\n");
            printf("client ip:%s\n", clientIP);
            printf("client port:%d\n", clientRtpPort);

            while (true) {
                frameSize = getFrameFromH264File(fp, frame, 500000);
                if (frameSize < 0)
                {
                    printf("读取%s结束,frameSize=%d \n", H264_FILE_NAME, frameSize);
                    break;
                }

                if (startCode3(frame))
                    startCode = 3;
                else
                    startCode = 4;

                frameSize -= startCode;
                rtpSendH264Frame(serverRtpSockfd, clientIP, clientRtpPort,
                    rtpPacket, frame + startCode, frameSize);

               

                Sleep(40);
                //usleep(40000);//1000/25 * 1000
            }
            free(frame);
            free(rtpPacket);

            break;
        }

        memset(method,0,sizeof(method)/sizeof(char));
        memset(url,0,sizeof(url)/sizeof(char));
        CSeq = 0;

    }

    closesocket(clientSockfd);
    if (serverRtpSockfd) {
        closesocket(serverRtpSockfd);
    }
    if (serverRtcpSockfd > 0) {
        closesocket(serverRtcpSockfd);
    }

    free(rBuf);
    free(sBuf);

}


int main(int argc, char* argv[])
{
    // 启动windows socket start
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)
    {
        printf("PC Server Socket Start Up Error \n");
        return -1;
    }
    // 启动windows socket end

    int rtspServerSockfd;


    rtspServerSockfd = createTcpSocket();
    if (rtspServerSockfd < 0)
    {
        WSACleanup();
        printf("failed to create tcp socket\n");
        return -1;
    }

    if (bindSocketAddr(rtspServerSockfd, "0.0.0.0", SERVER_PORT) < 0)
    {
        printf("failed to bind addr\n");
        return -1;
    }

    if (listen(rtspServerSockfd, 10) < 0)
    {
        printf("failed to listen\n");
        return -1;
    }

    printf("%s rtsp://127.0.0.1:%d\n", __FILE__, SERVER_PORT);

    while (true) {
        int clientSockfd;
        char clientIp[40];
        int clientPort;

        clientSockfd = acceptClient(rtspServerSockfd, clientIp, &clientPort);
        if (clientSockfd < 0)
        {
            printf("failed to accept client\n");
            return -1;
        }

        printf("accept client;client ip:%s,client port:%d\n", clientIp, clientPort);

        doClient(clientSockfd, clientIp, clientPort);
    }
    closesocket(rtspServerSockfd);

    return 0;
}
```



# 3. 音频原理

## 3.1 PCM数据格式

`PCM`(Pulse Code Modulation)也被称为 脉码编码调制。
PCM中的声音数据没有被压缩，如果是单声道的文件，采样数据按时间的先后顺序依次存入。(它的基本组织单位是BYTE(8bit)或WORD(16bit))

一般情况下，一帧PCM是由`2048`次采样组成的

如果是双声道的文件，采样数据按时间先后顺序交叉地存入。如图所示:

![](/img/rtsp21.png)

>对于解析PCM数据有所帮助

麦克风在录音采样后，直接获得的就是音频原始数据pcm。
但由于pcm数据较大，所以通常需要压缩之后才能传输或保存，常见的音频压缩技术有aac，g711，opus，mp3等，最常用的就是`aac`。

1秒钟的pcm的声音大小 = 44100 * 2(声道) * 2（字节） / 1024 * 60(秒) /1024 
一分钟约等于5MB



## 3.2 几个可能会用到的命令行

```bash
//指定时间段录制
ffmpeg -i input.mp4 -c:v copy -c:a copy -ss 00:10:20 -to 00:30:20 out.mp4

//指定录制时长
ffmpeg -i input.mp4 -c:v copy -c:a copy -t 30 out.mp4
备注: -t 30 表示指定30秒的录制时长

//ffmpeg命令行 从mp4视频文件提取aac 音频文件
ffmpeg -i test.mp4  -vn -acodec aac test.aac
备注：-i 表示输入文件 
      -vm disable video / 丢掉视频
      -acodec 设置音频编码格式


//ffmpeg 从aac音频文件解码为pcm音频文件
ffmpeg -i test.aac -f s16le test.pcm
备注：-i 表示输入文件 
      -f 表示输出格式

//ffplay 播放.pcm音频文件
备注：-i 表示指定的输入文件
      -f 表示强制使用的格式
      -ar 表示播放的音频数据的采样率
      -ac 表示播放的音频数据的通道数
```


## 3.3  代码

>Adts.h

```cpp
#pragma once

#include <cstdint>

struct AdtsHeader {
    unsigned int syncword;  //12 bit 同步字 '1111 1111 1111'，说明一个ADTS帧的开始
    unsigned int id;        //1 bit MPEG 标示符， 0 for MPEG-4，1 for MPEG-2
    unsigned int layer;     //2 bit 总是'00'
    unsigned int protectionAbsent;  //1 bit 1表示没有crc，0表示有crc
    unsigned int profile;           //1 bit 表示使用哪个级别的AAC
    unsigned int samplingFreqIndex; //4 bit 表示使用的采样频率
    unsigned int privateBit;        //1 bit
    unsigned int channelCfg; //3 bit 表示声道数
    unsigned int originalCopy;         //1 bit
    unsigned int home;                  //1 bit

    /*下面的为改变的参数即每一帧都不同*/
    unsigned int copyrightIdentificationBit;   //1 bit
    unsigned int copyrightIdentificationStart; //1 bit
    unsigned int aacFrameLength;               //13 bit 一个ADTS帧的长度包括ADTS头和AAC原始流
    unsigned int adtsBufferFullness;           //11 bit 0x7FF 说明是码率可变的码流

    /* number_of_raw_data_blocks_in_frame
     * 表示ADTS帧中有number_of_raw_data_blocks_in_frame + 1个AAC原始帧
     * 所以说number_of_raw_data_blocks_in_frame == 0
     * 表示说ADTS帧中有一个AAC数据块并不是说没有。(一个AAC原始帧包含一段时间内1024个采样及相关数据)
     */
    unsigned int numberOfRawDataBlockInFrame; //2 bit
};

int parseAdtsHeader(uint8_t* headerBuf, struct AdtsHeader* adtsHeader);
int convertAdtsHeaderToBuf(struct AdtsHeader* adtsHeader, uint8_t* adtsHeaderBuf);
```


>Adts.cpp

```cpp
#include "Adts.h"
#include <cstring>
#include <stdio.h>

int parseAdtsHeader(uint8_t* headerBuf, struct AdtsHeader* adtsHeader) {
    //    static int frame_number = 0;
    memset(adtsHeader, 0, sizeof(*adtsHeader));

    if ((headerBuf[0] == 0xFF) && ((headerBuf[1] & 0xF0) == 0xF0)) {

        adtsHeader->id = ((unsigned int)headerBuf[1] & 0x08) >> 3;
        adtsHeader->layer = ((unsigned int)headerBuf[1] & 0x06) >> 1;
        adtsHeader->protectionAbsent = (unsigned int)headerBuf[1] & 0x01;
        adtsHeader->profile = ((unsigned int)headerBuf[2] & 0xc0) >> 6;
        adtsHeader->samplingFreqIndex = ((unsigned int)headerBuf[2] & 0x3c) >> 2;
        adtsHeader->privateBit = ((unsigned int)headerBuf[2] & 0x02) >> 1;
        adtsHeader->channelCfg = ((((unsigned int)headerBuf[2] & 0x01) << 2) | (((unsigned int)headerBuf[3] & 0xc0) >> 6));
        adtsHeader->originalCopy = ((unsigned int)headerBuf[3] & 0x20) >> 5;
        adtsHeader->home = ((unsigned int)headerBuf[3] & 0x10) >> 4;
        adtsHeader->copyrightIdentificationBit = ((unsigned int)headerBuf[3] & 0x08) >> 3;
        adtsHeader->copyrightIdentificationStart = (unsigned int)headerBuf[3] & 0x04 >> 2;
        adtsHeader->aacFrameLength = (((((unsigned int)headerBuf[3]) & 0x03) << 11) |
            (((unsigned int)headerBuf[4] & 0xFF) << 3) |
            ((unsigned int)headerBuf[5] & 0xE0) >> 5);
        adtsHeader->adtsBufferFullness = (((unsigned int)headerBuf[5] & 0x1f) << 6 |
            ((unsigned int)headerBuf[6] & 0xfc) >> 2);
        adtsHeader->numberOfRawDataBlockInFrame = ((unsigned int)headerBuf[6] & 0x03);

        return 0;
    }
    else {
        printf("failed to parse adts header\n");
        return -1;
    }
}
int convertAdtsHeaderToBuf(struct AdtsHeader* adtsHeader, uint8_t* adtsHeaderBuf) {

    adtsHeaderBuf[0] = 0xFF;
    adtsHeaderBuf[1] = 0xF1;
    adtsHeaderBuf[2] = ((adtsHeader->profile) << 6) + (adtsHeader->samplingFreqIndex << 2) + (adtsHeader->channelCfg >> 2);
    adtsHeaderBuf[3] = (((adtsHeader->channelCfg & 3) << 6) + (adtsHeader->aacFrameLength >> 11));
    adtsHeaderBuf[4] = ((adtsHeader->aacFrameLength & 0x7FF) >> 3);
    adtsHeaderBuf[5] = (((adtsHeader->aacFrameLength & 7) << 5) + 0x1F);
    adtsHeaderBuf[6] = 0xFC;

    return 0;
}
```


>decode_aac, aac格式转pcm格式

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "fdk-aac/aacdecoder_lib.h"

//aac转pcm格式

int getADTSframe(unsigned char* buffer, int buf_size, unsigned char* data, unsigned int* data_size) {
    int size = 0;

    if (!buffer || !data || !data_size) {
        return -1;
    }

    while (1) {
        if (buf_size < 7) {
            return -1;
        }
        //Sync words
        if ((buffer[0] == 0xff) && ((buffer[1] & 0xf0) == 0xf0)) {
            size |= ((buffer[3] & 0x03) << 11);     //high 2 bit
            size |= buffer[4] << 3;                //middle 8 bit
            size |= ((buffer[5] & 0xe0) >> 5);        //low 3bit
            break;
        }
        --buf_size;
        ++buffer;
    }

    if (buf_size < size) {
        return 1;
    }

    memcpy(data, buffer, size);
    *data_size = size;

    return 0;
}

int main() {
    const char* in_name = "data/xhjcz1_test.aac";

    int data_size = 0;
    int cnt = 0;
    int offset = 0;

    //FILE *myout=fopen("output_log.txt","wb+");
    FILE* myout = stdout;

    unsigned char* aacframe = (unsigned char*)malloc(1024 * 5);
    unsigned char* aacbuffer = (unsigned char*)malloc(1024 * 1024);

    FILE* infile = fopen(in_name, "rb");
    if (!infile) {
        printf("failed to open %s\n", in_name);
        return -1;
    }

    printf("-----+- ADTS Frame Table -+------+\n");
    printf(" NUM | Profile | Frequency| Size | Decode Size |\n");
    printf("-----+---------+----------+------+\n");
    HANDLE_AACDECODER aacCoder;
    aacCoder = aacDecoder_Open(TT_MP4_ADTS, 1);
    //    aacCoder = aacDecoder_Open(TT_MP4_RAW , 1);
    AAC_DECODER_ERROR aacError;
    /*
    int conceal_method=2;//0 muting 1 noise 2 interpolation
    UCHAR eld_conf[] = { 0xF8, 0xE8, 0x50, 0x00 };//ste eld 44.1k
//    UCHAR ld_conf[] = { 0xBA, 0x88, 0x00, 0x00 };//mono ld 32k

    UCHAR *conf[] = { eld_conf };
    static UINT conf_len = sizeof(eld_conf);
    AAC_DECODER_ERROR err = aacDecoder_ConfigRaw(aacCoder, conf, &conf_len);
    if(err>0){
        fprintf(myout,"conf err\n");
    }
    aacDecoder_SetParam(aacCoder, AAC_CONCEAL_METHOD,conceal_method);
    aacDecoder_SetParam(aacCoder,  AAC_PCM_MAX_OUTPUT_CHANNELS,1); //MONO:1
    */
    unsigned int valid = 0;
    unsigned int size = 0;
    unsigned int decsize = 8 * 1024 * sizeof(INT_PCM);
    unsigned char* decdata = (unsigned char*)malloc(decsize);


    while (!feof(infile)) {
        data_size = fread(aacbuffer + offset, 1, 1024 * 1024 - offset, infile);
        unsigned char* input_data = aacbuffer;

        while (true)
        {
            int ret = getADTSframe(input_data, data_size, aacframe, &size);
            if (ret == -1) {
                printf("adts帧结束\n");
                break;
            }
            else if (ret == 1) {
                memcpy(aacbuffer, input_data, data_size);
                offset = data_size;
                printf("当前adts帧损坏\n");
                break;
            }
            valid = size;
            aacError = aacDecoder_Fill(aacCoder, &aacframe, &size, &valid);
            if (aacError > 0) {
                fprintf(myout, "fill err\n");
            }
            aacError = aacDecoder_DecodeFrame(aacCoder, (INT_PCM*)decdata, decsize / sizeof(INT_PCM), 0);
            if (aacError > 0) {
                fprintf(myout, "decode err\n");
            }

            CStreamInfo* pcmFrame = aacDecoder_GetStreamInfo(aacCoder);

            fprintf(myout, "pcmFrame: channels=%d,simmpleRate=%d,frameSize=%d\n",
                pcmFrame->numChannels, pcmFrame->sampleRate, pcmFrame->frameSize);


            char profile_str[10] = { 0 };
            char frequence_str[10] = { 0 };

            unsigned char profile = aacframe[2] & 0xC0;
            profile = profile >> 6;
            switch (profile) {
            case 0: sprintf(profile_str, "Main"); break;
            case 1: sprintf(profile_str, "LC"); break;
            case 2: sprintf(profile_str, "SSR"); break;
            default:sprintf(profile_str, "unknown"); break;
            }

            unsigned char sampling_frequency_index = aacframe[2] & 0x3C;
            sampling_frequency_index = sampling_frequency_index >> 2;

            switch (sampling_frequency_index) {
            case 0: sprintf(frequence_str, "96000Hz"); break;
            case 1: sprintf(frequence_str, "88200Hz"); break;
            case 2: sprintf(frequence_str, "64000Hz"); break;
            case 3: sprintf(frequence_str, "48000Hz"); break;
            case 4: sprintf(frequence_str, "44100Hz"); break;
            case 5: sprintf(frequence_str, "32000Hz"); break;
            case 6: sprintf(frequence_str, "24000Hz"); break;
            case 7: sprintf(frequence_str, "22050Hz"); break;
            case 8: sprintf(frequence_str, "16000Hz"); break;
            case 9: sprintf(frequence_str, "12000Hz"); break;
            case 10: sprintf(frequence_str, "11025Hz"); break;
            case 11: sprintf(frequence_str, "8000Hz"); break;
            default:sprintf(frequence_str, "unknown"); break;
            }


            fprintf(myout, "%5d| %8s|  %8s| %5d| %5d |\n", cnt, profile_str, frequence_str, size, decsize);
            data_size -= size;
            input_data += size;
            cnt++;
        }
        printf("--------------------------\n");
    }
    free(decdata);
    free(aacbuffer);
    free(aacframe);
    fclose(infile);

    return 0;
}
```


>sdl2_play_aac.cpp播放aac音频

```cpp
//
// Created by sun on 2022/12/10.
//

//播放一个aac的音频文件
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include "Adts.h"
#include "fdk-aac/aacdecoder_lib.h"
#define SDL_MAIN_HANDLED
#include <SDL2/SDL.h>


//Buffer:
//|-----------|-------------|
//chunk-------pos---len-----|
static  Uint8* audio_chunk;
static  Uint32  audio_len;
static  Uint8* audio_pos;

/* Audio Callback
 * The audio function callback takes the following parameters:
 * stream: A pointer to the audio buffer to be filled
 * len: The length (in bytes) of the audio buffer
 *
*/
void fill_audio(void* data, Uint8* stream, int len) {
    //SDL 2.0
    //printf("fill_audio\n");
    SDL_memset(stream, 0, len);
    if (audio_len == 0)
        return;
    len = (len > audio_len ? audio_len : len);

    SDL_MixAudio(stream, audio_pos, len, SDL_MIX_MAXVOLUME);
    audio_pos += len;
    audio_len -= len;
}

int main() {
    const char* in_name = "data/test.aac";

    //SDL2 start
    if (SDL_Init(SDL_INIT_AUDIO | SDL_INIT_TIMER)) {
        printf("Could not initialize SDL - %s\n", SDL_GetError());
        return -1;
    }
    SDL_AudioSpec wanted_spec;
    wanted_spec.freq = 44100;
    wanted_spec.format = AUDIO_S16SYS;
    wanted_spec.channels = 2;
    wanted_spec.silence = 0;
    wanted_spec.samples = 1024;
    wanted_spec.callback = fill_audio;

    if (SDL_OpenAudio(&wanted_spec, NULL) < 0) {
        printf("SDL_OpenAudio err \n");
        return -1;
    }
    SDL_PauseAudio(0);
    //SDL2 end
    FILE* infile = fopen(in_name, "rb");
    if (!infile) {
        printf("failed to open %s\n", in_name);
        return -1;
    }
    printf("-----+- ADTS Frame Table -+------+\n");
    printf(" NUM | Profile | Frequency| Size | Decode Size |\n");
    printf("-----+---------+----------+------+\n");
    HANDLE_AACDECODER aacCoder;
    aacCoder = aacDecoder_Open(TT_MP4_ADTS, 1);
    AAC_DECODER_ERROR aacError;

    int cnt = 0;


    struct AdtsHeader adtsHeader;
    int adtsHeaderLen;
    int adtsContentLen;

    unsigned int valid = 0;

    uint8_t* frame = (uint8_t*)malloc(2000);

    unsigned int decFrameSize = 2 * 1024 * sizeof(INT_PCM);
    unsigned char* decFrame = (unsigned char*)malloc(decFrameSize);


    while (true) {
        adtsHeaderLen = fread(frame, 1, 7, infile);
        //        printf("adtsHeaderLen = %d \n",adtsHeaderLen);
        if (adtsHeaderLen <= 0) {
            printf("read adtsHeaderLen err\n");
            break;
        }
        if (parseAdtsHeader(frame, &adtsHeader) < 0) {
            printf("parseAdtsHeader err\n");
            break;
        }
        adtsContentLen = fread(frame + 7, 1, adtsHeader.aacFrameLength - 7, infile);
        //        printf("adtsContentLen = %d \n",adtsContentLen);
        if (adtsContentLen < 0) {
            printf("read adtsContentLen err\n");
            break;
        }

        valid = adtsHeader.aacFrameLength;
        aacError = aacDecoder_Fill(aacCoder, &frame, &adtsHeader.aacFrameLength, &valid);
        if (aacError > 0) {
            printf("aacDecoder_Fill err\n");
            break;
        }
        aacError = aacDecoder_DecodeFrame(aacCoder, (INT_PCM*)decFrame, decFrameSize / sizeof(INT_PCM), 0);
        if (aacError > 0) {
            printf("aacDecoder_DecodeFrame err\n");
            break;
        }

        //Set audio buffer (PCM data)
        audio_chunk = (Uint8*)decFrame;
        //Audio buffer length
        audio_len = decFrameSize;
        audio_pos = audio_chunk;

        while (audio_len > 0) {
            //            SDL_Delay(23);
            SDL_Delay(1);
        }

        printf("cnt=%d,aacFrameLength=%d,decFrameSize=%d \n",cnt, adtsHeader.aacFrameLength, decFrameSize);

        cnt++;
    }

    free(frame);
    free(decFrame);
    fclose(infile);

    SDL_Quit();

    printf("end\n");
    return 0;
}
```

>sdl2_play_pcm.cpp, 播放pcm音频

```cpp
#include <stdio.h>
#define SDL_MAIN_HANDLED
#include <SDL2/SDL.h>

//播放一个pcm格式的音频

//Buffer:
//|-----------|-------------|
//chunk-------pos---len-----|
static  Uint8* audio_chunk;
static  Uint32  audio_len;
static  Uint8* audio_pos;

/* Audio Callback
 * The audio function callback takes the following parameters:
 * stream: A pointer to the audio buffer to be filled
 * len: The length (in bytes) of the audio buffer
 *
*/
void fill_audio(void* udata, Uint8* stream, int len) {
    //SDL 2.0
    //printf("fill_audio\n");
    SDL_memset(stream, 0, len);
    if (audio_len == 0)
        return;
    len = (len > audio_len ? audio_len : len);

    SDL_MixAudio(stream, audio_pos, len, SDL_MIX_MAXVOLUME);
    audio_pos += len;
    audio_len -= len;
}

int main() {
    const char* filename = "data/xhjcz1_test.aac.pcm";

    //Init
    if (SDL_Init(SDL_INIT_AUDIO | SDL_INIT_TIMER)) {
        printf("Could not initialize SDL - %s\n", SDL_GetError());
        return -1;
    }
    //SDL_AudioSpec
    SDL_AudioSpec wanted_spec;
    wanted_spec.freq = 44100;
    wanted_spec.format = AUDIO_S16SYS;
    wanted_spec.channels = 2;
    wanted_spec.silence = 0;
    wanted_spec.samples = 1024;
    wanted_spec.callback = fill_audio;

    if (SDL_OpenAudio(&wanted_spec, NULL) < 0) {
        printf("SDL_OpenAudio err: %s \n", SDL_GetError());
        return -1;
    }

    FILE* fp = fopen(filename, "rb+");
    if (fp == NULL) {
        printf("cannot open this file\n");
        return -1;
    }
    int pcm_buffer_size = 409600;// 一次性从pcm文件中读取的数据大小
    char* pcm_buffer = (char*)malloc(pcm_buffer_size);
    int data_count = 0;

    //Play
    SDL_PauseAudio(0);
    size_t nr;
    while (true) {
        nr = fread(pcm_buffer, 1, pcm_buffer_size, fp);

        if (nr != pcm_buffer_size) {
            printf("读取pcm结束 nr=%zu \n", nr);
            // Loop
            fseek(fp, 0, SEEK_SET);
            fread(pcm_buffer, 1, pcm_buffer_size, fp);
            data_count = 0;
            break;
        }
        printf("Now Playing %10d Bytes data.\n", data_count);
        data_count += pcm_buffer_size;
        //Set audio buffer (PCM data)
        audio_chunk = (Uint8*)pcm_buffer;
        //Audio buffer length
        audio_len = pcm_buffer_size;
        audio_pos = audio_chunk;

        while (audio_len > 0)//Wait until finish
        {
            SDL_Delay(23);
        }

    }
    free(pcm_buffer);
    SDL_Quit();

    return 0;
}
```

# 4. UDP的RTP传输音频aac的RTSP服务器，能拉流播放



## 4.1 aac码流格式


- `ADIF(Audio Data Interchage Format)`，音频数据交换格式：
只有一个统一的头，必须得到所有数据后解码，适用于本地文件。
- `ADTS(Audio Data Transport Stream)`，音视数据传输流：
每一帧都有头信息，任意帧解码，适用于传输流。



AAC共有9种规格，以适应不同的场合的需要：

- `MPEG-2 AAC LC` 低复杂度规格（Low Complexity） 注：比较简单，没有增益控制，但提高了编码效率，在中等码率的编码效率以及音质方面，都能找到平衡点

- `MPEG-2 AAC Main` 主规格

- `MPEG-2 AAC SSR` 可变采样率规格（Scaleable Sample Rate）

- `MPEG-4 AAC LC` 低复杂度规格（Low Complexity）---现在的手机比较常见的MP4文件中的音频部份就包括了该规格音频文件

- `MPEG-4 AAC Main` 主规格 注：包含了除增益控制之外的全部功能，其音质最好

- `MPEG-4 AAC SSR` 可变采样率规格（Scaleable Sample Rate）

- `MPEG-4 AAC LTP` 长时期预测规格（Long Term Predicition）

- `MPEG-4 AAC LD` 低延迟规格（Low Delay）

- `MPEG-4 AAC HE` 高效率规格（High Efficiency）---这种规格适合用于低码率编码，有Nero ACC 编码器支持


| 序号 | 字段 | 长度 | 说明 |
|------|------|------|-----|
| 1    | synword | 12bit | 同步头，总是0xFFF，代表着⼀个ADTS帧的开始。 |
| 2    | id | 1bit | 设置MPEG标识符，1bit，0标识MPEG-4，1标识MPEG-2。 |
| 3    | layer | 2bit | 总是00。 |
| 4    | protection_absent | 1bit | 误码校验，标识是否进行误码校验。0表示有CRC校验，1表示没有CRC校验。为0时头部7bytes后面+2bytesCRC检验位。 |
| 5    | profile | 2bit | AAC级别，比如AAC LC=1。profile的值等于Audio Object Type的值减1。 |
| 6    | sampling_frequency_index | 4bit | 采样率下标，下标对应的采样率如下：<br>0: 96000 Hz<br>1: 88200 Hz<br>2 : 64000 Hz<br>3 : 48000 Hz<br>4 : 44100 Hz<br>5 : 32000 Hz<br>6 : 24000 Hz<br>7 : 22050 Hz<br>8 : 16000 Hz<br>9 : 12000 Hz<br>10 : 11025 Hz<br>11 : 8000 Hz<br>12 : 7350 Hz<br>13 : Reserved<br>14 : Reserved<br>15 : frequency is written explictly |
| 7    | private_bit | 1bit | 私有位，编码时设置为0，解码时忽略。 |
| 8    | channel_configuration | 3bit | 声道数。<br>0: Defined in AOT Specifc Config<br>1: 1 channel : front - center<br>2 : 2 channels : front - left, front - right<br>3 : 3 channels : front - center, front - left, front - right<br>4 : 4 channels : front - center, front - left, front - right, back - center<br>5 : 5 channels : front - center, front - left, front - right, back - left, back - right<br>6 : 6 channels : front - center, front - left, front - right, back - left, back - right, LFE - channel<br>7 : 8 channels : front - center, front - left, front - right, side - left, side - right, back - left, back - right, LFE - channel<br>8 - 15 : Reserved |
| 9    | orininal_copy | 1bit | 编码时设置为0，解码时忽略。 |
| 10   | home | 1bit | 编码时设置为0，解码时忽略。 |
| 11   | copyrigth_identification_bit | 1bit | 编码时设置为0，解码时忽略。 |
| 12   | copyrigth_identification_stat | 1bit | 编码时设置为0，解码时忽略。 |
| 13   | aac_frame_length | 13bit | 一个ADTS帧的⻓度，包括ADTS头和AAC原始流。 |
| 14   | adts_bufferfullness | 11bit | 缓冲区充满度，0x7FF说明是码率可变的码流，不需要此字段。CBR可能需要此字段，不同编码器使用情况不同。具体查看附录。 |
| 15   | number_of_raw_data_blocks_in_frame | 2bit | 表示ADTS帧中有number_of_raw_data_blocks_in_frame + 1个AAC原始帧，为0表示说ADTS帧中只有一个AAC数据. |


| 序号 | 字段 | 长度 | 值              |
|------|------|------|----------------|
| 1    | synword | 12bit | 111111111111   |
| 2    | id | 1bit | 0                |
| 3    | layer | 2bit | 00              |
| 4    | protection_absent | 1bit | 1    |
| 5    | profile | 2bit | 01             |
| 6    | sampling_frequency_index | 4bit | 0100（十进制4） |
| 7    | private_bit | 1bit | 0          |
| 8    | channel_configuration | 3bit | 010 （十进制2）  |
| 9    | orininal_copy | 1bit | 0        |
| 10   | home | 1bit | 0                 |
| 11   | copyrigth_identification_bit | 1bit | 0  |
| 12   | copyrigth_identification_stat | 1bit | 0 |
| 13   | aac_frame_length | 13bit | 0000101111100（十进制380） |
| 14   | adts_bufferfullness | 11bit | 00010010000 |
| 15   | number_of_raw_data_blocks_in_frame | 2bit | 00 |



>aac的RTP打包方式

  AAC的RTP打包方式并没有向H.264那样丰富，一种方式，主要AAC一帧数据大小都是几百个字节，不会向H.264那么少则几个字节，多则几千。
     AAC的RTP打包方式就是将ADTS帧取出ADTS头部，取出AAC数据，每帧数据封装成一个RTP包
     需要注意的是，并不是将AAC数据直接拷贝到RTP的载荷中
     。AAC封装成RTP包，在RTP载荷中的前四个字节是有特殊含义的，然后再是AAC数据，如下图所示。


![](/img/rtsp22.png)


 其中RTP载荷的一个字节为0x00，第二个字节为0x10
 第三个字节和第四个字节保存AAC Data的大小，最多只能保存13bit
 第三个字节保存数据大小的高八位
 第四个字节的高5位保存数据大小的低5位.

```cpp
rtpPacket->payload[0] = 0x00;
rtpPacket->payload[1] = 0x10;
rtpPacket->payload[2] = (frameSize & 0x1FE0) >> 5; //高8位
rtpPacket->payload[3] = (frameSize & 0x1F) << 3; //低5位
```

>RTP包时间戳的计算

  假设音频的采样率位44100，即每秒钟采样44100次。
AAC一般将1024次采样编码成一帧，所以一秒就有44100/1024=43帧。
RTP包发送的每一帧数据的时间增量为44100/43=1025。
每一帧数据的时间间隔为1000/43=23ms。


>可能会用到的ffmpeg命令行

```cpp
//ffmpeg命令行 从mp4视频文件提取aac 音频文件
ffmpeg -i test.mp4  -vn -acodec aac test.aac
备注：-i 表示输入文件 
      -vm disable video / 丢掉视频
      -acodec 设置音频编码格式

//ffmpeg 从aac音频文件解码为pcm音频文件
ffmpeg -i test.aac -f s16le test.pcm
备注：-i 表示输入文件 
      -f 表示输出格式

//ffplay 播放.pcm音频文件
ffplay -ar 44100 -ac 2 -f s16le -i test.pcm
备注：-i 表示指定的输入文件
      -f 表示强制使用的格式
      -ar 表示播放的音频数据的采样率
      -ac 表示播放的音频数据的通道数
```

## 4.2 代码实现

>rtp.h

```cpp
#pragma once
#pragma comment(lib, "ws2_32.lib")
#include <stdint.h>

#define RTP_VESION              2

#define RTP_PAYLOAD_TYPE_H264   96
#define RTP_PAYLOAD_TYPE_AAC    97

#define RTP_HEADER_SIZE         12
#define RTP_MAX_PKT_SIZE        1400

 /*
  *    0                   1                   2                   3
  *    7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0|7 6 5 4 3 2 1 0
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *   |V=2|P|X|  CC   |M|     PT      |       sequence number         |
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *   |                           timestamp                           |
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *   |           synchronization source (SSRC) identifier            |
  *   +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
  *   |            contributing source (CSRC) identifiers             |
  *   :                             ....                              :
  *   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  *
  */
struct RtpHeader
{
    /* byte 0 */
    uint8_t csrcLen : 4;
    uint8_t extension : 1;
    uint8_t padding : 1;
    uint8_t version : 2;

    /* byte 1 */
    uint8_t payloadType : 7;
    uint8_t marker : 1;

    /* bytes 2,3 */
    uint16_t seq;

    /* bytes 4-7 */
    uint32_t timestamp;

    /* bytes 8-11 */
    uint32_t ssrc;
};

struct RtpPacket
{
    struct RtpHeader rtpHeader;
    uint8_t payload[0];
};

void rtpHeaderInit(struct RtpPacket* rtpPacket, uint8_t csrcLen, uint8_t extension,
    uint8_t padding, uint8_t version, uint8_t payloadType, uint8_t marker,
    uint16_t seq, uint32_t timestamp, uint32_t ssrc);

int rtpSendPacketOverTcp(int clientSockfd, struct RtpPacket* rtpPacket, uint32_t dataSize);
int rtpSendPacketOverUdp(int serverRtpSockfd, const char* ip, int16_t port, struct RtpPacket* rtpPacket, uint32_t dataSize);
```

>rtp.cpp

```cpp
#include <sys/types.h>
#include <WinSock2.h>
#include <WS2tcpip.h>
#include <windows.h>
#include "rtp.h"

void rtpHeaderInit(struct RtpPacket* rtpPacket, uint8_t csrcLen, uint8_t extension,
    uint8_t padding, uint8_t version, uint8_t payloadType, uint8_t marker,
    uint16_t seq, uint32_t timestamp, uint32_t ssrc)
{
    rtpPacket->rtpHeader.csrcLen = csrcLen;
    rtpPacket->rtpHeader.extension = extension;
    rtpPacket->rtpHeader.padding = padding;
    rtpPacket->rtpHeader.version = version;
    rtpPacket->rtpHeader.payloadType = payloadType;
    rtpPacket->rtpHeader.marker = marker;
    rtpPacket->rtpHeader.seq = seq;
    rtpPacket->rtpHeader.timestamp = timestamp;
    rtpPacket->rtpHeader.ssrc = ssrc;
}
int rtpSendPacketOverTcp(int clientSockfd, struct RtpPacket* rtpPacket, uint32_t dataSize)
{

    rtpPacket->rtpHeader.seq = htons(rtpPacket->rtpHeader.seq);
    rtpPacket->rtpHeader.timestamp = htonl(rtpPacket->rtpHeader.timestamp);
    rtpPacket->rtpHeader.ssrc = htonl(rtpPacket->rtpHeader.ssrc);

    uint32_t rtpSize = RTP_HEADER_SIZE + dataSize;
    char* tempBuf = (char *)malloc(4 + rtpSize);
    tempBuf[0] = 0x24;//$
    tempBuf[1] = 0x00;
    tempBuf[2] = (uint8_t)(((rtpSize) & 0xFF00) >> 8);
    tempBuf[3] = (uint8_t)((rtpSize) & 0xFF);
    memcpy(tempBuf + 4, (char*)rtpPacket, rtpSize);

    int ret = send(clientSockfd, tempBuf, 4 + rtpSize, 0);

    rtpPacket->rtpHeader.seq = ntohs(rtpPacket->rtpHeader.seq);
    rtpPacket->rtpHeader.timestamp = ntohl(rtpPacket->rtpHeader.timestamp);
    rtpPacket->rtpHeader.ssrc = ntohl(rtpPacket->rtpHeader.ssrc);

    free(tempBuf);
    tempBuf = NULL;

    return ret;
}
int rtpSendPacketOverUdp(int serverRtpSockfd, const char* ip, int16_t port, struct RtpPacket* rtpPacket, uint32_t dataSize)
{
    
    struct sockaddr_in addr;
    int ret;

    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    addr.sin_addr.s_addr = inet_addr(ip);

    rtpPacket->rtpHeader.seq = htons(rtpPacket->rtpHeader.seq);
    rtpPacket->rtpHeader.timestamp = htonl(rtpPacket->rtpHeader.timestamp);
    rtpPacket->rtpHeader.ssrc = htonl(rtpPacket->rtpHeader.ssrc);

    ret = sendto(serverRtpSockfd, (char *)rtpPacket, dataSize + RTP_HEADER_SIZE, 0,
        (struct sockaddr*)&addr, sizeof(addr));

    rtpPacket->rtpHeader.seq = ntohs(rtpPacket->rtpHeader.seq);
    rtpPacket->rtpHeader.timestamp = ntohl(rtpPacket->rtpHeader.timestamp);
    rtpPacket->rtpHeader.ssrc = ntohl(rtpPacket->rtpHeader.ssrc);

    return ret;
}
```


>main.cpp

```cpp
//
// Created by sun on 2022/12/9.
//

#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include <time.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <WinSock2.h>
#include <WS2tcpip.h>
#include <windows.h>
#include "rtp.h"

#define SERVER_PORT     8554
#define SERVER_RTP_PORT  55532
#define SERVER_RTCP_PORT 55533
#define BUF_MAX_SIZE    (1024*1024)
#define AAC_FILE_NAME   "../data/test-long.aac"

static int createTcpSocket() {
    int sockfd;
    int on = 1;

    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0)
        return -1;

    setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, (const char*)&on, sizeof(on));

    return sockfd;
}

static int createUdpSocket() {
    int sockfd;
    int on = 1;

    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd < 0)
        return -1;

    setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, (const char*)&on, sizeof(on));

    return sockfd;
}

static int bindSocketAddr(int sockfd, const char* ip, int port) {
    struct sockaddr_in addr;

    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    addr.sin_addr.s_addr = inet_addr(ip);

    if (bind(sockfd, (struct sockaddr*)&addr, sizeof(struct sockaddr)) < 0)
        return -1;

    return 0;
}

struct AdtsHeader {
    unsigned int syncword;  //12 bit 同步字 '1111 1111 1111'，一个ADTS帧的开始
    uint8_t id;        //1 bit 0代表MPEG-4, 1代表MPEG-2。
    uint8_t layer;     //2 bit 必须为0
    uint8_t protectionAbsent;  //1 bit 1代表没有CRC，0代表有CRC
    uint8_t profile;           //1 bit AAC级别（MPEG-2 AAC中定义了3种profile，MPEG-4 AAC中定义了6种profile）
    uint8_t samplingFreqIndex; //4 bit 采样率
    uint8_t privateBit;        //1bit 编码时设置为0，解码时忽略
    uint8_t channelCfg;        //3 bit 声道数量
    uint8_t originalCopy;      //1bit 编码时设置为0，解码时忽略
    uint8_t home;               //1 bit 编码时设置为0，解码时忽略

    uint8_t copyrightIdentificationBit;   //1 bit 编码时设置为0，解码时忽略
    uint8_t copyrightIdentificationStart; //1 bit 编码时设置为0，解码时忽略
    unsigned int aacFrameLength;               //13 bit 一个ADTS帧的长度包括ADTS头和AAC原始流
    unsigned int adtsBufferFullness;           //11 bit 缓冲区充满度，0x7FF说明是码率可变的码流，不需要此字段。CBR可能需要此字段，不同编码器使用情况不同。这个在使用音频编码的时候需要注意。

    /* number_of_raw_data_blocks_in_frame
     * 表示ADTS帧中有number_of_raw_data_blocks_in_frame + 1个AAC原始帧
     * 所以说number_of_raw_data_blocks_in_frame == 0
     * 表示说ADTS帧中有一个AAC数据块并不是说没有。(一个AAC原始帧包含一段时间内1024个采样及相关数据)
     */
    uint8_t numberOfRawDataBlockInFrame; //2 bit
};

//in是码流
static int parseAdtsHeader(uint8_t* in, struct AdtsHeader* res) {
    static int frame_number = 0;
    memset(res, 0, sizeof(*res));

    if ((in[0] == 0xFF) && ((in[1] & 0xF0) == 0xF0))
    {
        res->id = ((uint8_t)in[1] & 0x08) >> 3;//第二个字节与0x08与运算之后，获得第13位bit对应的值
        res->layer = ((uint8_t)in[1] & 0x06) >> 1;//第二个字节与0x06与运算之后，右移1位，获得第14,15位两个bit对应的值
        res->protectionAbsent = (uint8_t)in[1] & 0x01;
        res->profile = ((uint8_t)in[2] & 0xc0) >> 6;
        res->samplingFreqIndex = ((uint8_t)in[2] & 0x3c) >> 2;
        res->privateBit = ((uint8_t)in[2] & 0x02) >> 1;
        res->channelCfg = ((((uint8_t)in[2] & 0x01) << 2) | (((unsigned int)in[3] & 0xc0) >> 6));
        res->originalCopy = ((uint8_t)in[3] & 0x20) >> 5;
        res->home = ((uint8_t)in[3] & 0x10) >> 4;
        res->copyrightIdentificationBit = ((uint8_t)in[3] & 0x08) >> 3;
        res->copyrightIdentificationStart = (uint8_t)in[3] & 0x04 >> 2;
        
        res->aacFrameLength = (((((unsigned int)in[3]) & 0x03) << 11) |
            (((unsigned int)in[4] & 0xFF) << 3) |
            ((unsigned int)in[5] & 0xE0) >> 5);

        res->adtsBufferFullness = (((unsigned int)in[5] & 0x1f) << 6 |
            ((unsigned int)in[6] & 0xfc) >> 2);
        res->numberOfRawDataBlockInFrame = ((uint8_t)in[6] & 0x03);

        return 0;
    }
    else
    {
        printf("failed to parse adts header\n");
        return -1;
    }
}

static int rtpSendAACFrame(int socket, const char* ip, int16_t port,
    struct RtpPacket* rtpPacket, uint8_t* frame, uint32_t frameSize) {
    //打包文档：https://blog.csdn.net/yangguoyu8023/article/details/106517251/
    int ret;

    rtpPacket->payload[0] = 0x00;
    rtpPacket->payload[1] = 0x10;
    rtpPacket->payload[2] = (frameSize & 0x1FE0) >> 5; //高8位
    rtpPacket->payload[3] = (frameSize & 0x1F) << 3; //低5位

    memcpy(rtpPacket->payload + 4, frame, frameSize);

    ret = rtpSendPacketOverUdp(socket, ip, port, rtpPacket, frameSize + 4);
    if (ret < 0)
    {
        printf("failed to send rtp packet\n");
        return -1;
    }

    rtpPacket->rtpHeader.seq++;

    /*
     * 如果采样频率是44100
     * 一般AAC每个1024个采样为一帧
     * 所以一秒就有 44100 / 1024 = 43帧
     * 时间增量就是 44100 / 43 = 1025
     * 一帧的时间为 1 / 43 = 23ms
     */
    rtpPacket->rtpHeader.timestamp += 1025;

    return 0;
}

static int acceptClient(int sockfd, char* ip, int* port) {
    int clientfd;
    socklen_t len = 0;
    struct sockaddr_in addr;

    memset(&addr, 0, sizeof(addr));
    len = sizeof(addr);

    clientfd = accept(sockfd, (struct sockaddr*)&addr, &len);
    if (clientfd < 0)
        return -1;

    strcpy(ip, inet_ntoa(addr.sin_addr));
    *port = ntohs(addr.sin_port);

    return clientfd;
}

static char* getLineFromBuf(char* buf, char* line) {
    while (*buf != '\n')
    {
        *line = *buf;
        line++;
        buf++;
    }

    *line = '\n';
    ++line;
    *line = '\0';

    ++buf;
    return buf;
}

static int handleCmd_OPTIONS(char* result, int cseq) {
    sprintf(result, "RTSP/1.0 200 OK\r\n"
        "CSeq: %d\r\n"
        "Public: OPTIONS, DESCRIBE, SETUP, PLAY\r\n"
        "\r\n",
        cseq);

    return 0;
}

static int handleCmd_DESCRIBE(char* result, int cseq, char* url) {
    char sdp[500];
    char localIp[100];

    sscanf(url, "rtsp://%[^:]:", localIp);

    sprintf(sdp, "v=0\r\n"
        "o=- 9%ld 1 IN IP4 %s\r\n"
        "t=0 0\r\n"
        "a=control:*\r\n"
        "m=audio 0 RTP/AVP 97\r\n"
        "a=rtpmap:97 mpeg4-generic/44100/2\r\n"
        "a=fmtp:97 profile-level-id=1;mode=AAC-hbr;sizelength=13;indexlength=3;indexdeltalength=3;config=1210;\r\n"
   
        //"a=fmtp:97 SizeLength=13;\r\n"
        "a=control:track0\r\n",
        time(NULL), localIp);

    sprintf(result, "RTSP/1.0 200 OK\r\nCSeq: %d\r\n"
        "Content-Base: %s\r\n"
        "Content-type: application/sdp\r\n"
        "Content-length: %d\r\n\r\n"
        "%s",
        cseq,
        url,
        strlen(sdp),
        sdp);

    return 0;
}

static int handleCmd_SETUP(char* result, int cseq, int clientRtpPort) {
    sprintf(result, "RTSP/1.0 200 OK\r\n"
        "CSeq: %d\r\n"
        "Transport: RTP/AVP;unicast;client_port=%d-%d;server_port=%d-%d\r\n"
        "Session: 66334873\r\n"
        "\r\n",
        cseq,
        clientRtpPort,
        clientRtpPort + 1,
        SERVER_RTP_PORT,
        SERVER_RTCP_PORT
    );

    return 0;
}

static int handleCmd_PLAY(char* result, int cseq) {
    sprintf(result, "RTSP/1.0 200 OK\r\n"
        "CSeq: %d\r\n"
        "Range: npt=0.000-\r\n"
        "Session: 66334873; timeout=10\r\n\r\n",
        cseq);

    return 0;
}


static void doClient(int clientSockfd, const char* clientIP, int clientPort) {

    int serverRtpSockfd = -1, serverRtcpSockfd = -1;

    char method[40];
    char url[100];
    char version[40];
    int CSeq;

    int clientRtpPort, clientRtcpPort;
    char* rBuf = (char*)malloc(BUF_MAX_SIZE);
    char* sBuf = (char*)malloc(BUF_MAX_SIZE);

    while (true) {
        int recvLen;

        recvLen = recv(clientSockfd, rBuf, BUF_MAX_SIZE, 0);
        if (recvLen <= 0) {
            break;
        }

        rBuf[recvLen] = '\0';
        printf("%s rBuf = %s \n", __FUNCTION__, rBuf);

        const char* sep = "\n";
        char* line = strtok(rBuf, sep);
        while (line) {
            if (strstr(line, "OPTIONS") ||
                strstr(line, "DESCRIBE") ||
                strstr(line, "SETUP") ||
                strstr(line, "PLAY")) {

                if (sscanf(line, "%s %s %s\r\n", method, url, version) != 3) {
                    // error
                }
            }
            else if (strstr(line, "CSeq")) {
                if (sscanf(line, "CSeq: %d\r\n", &CSeq) != 1) {
                    // error
                }
            }
            else if (!strncmp(line, "Transport:", strlen("Transport:"))) {
                // Transport: RTP/AVP/UDP;unicast;client_port=13358-13359
                // Transport: RTP/AVP;unicast;client_port=13358-13359

                if (sscanf(line, "Transport: RTP/AVP/UDP;unicast;client_port=%d-%d\r\n",
                    &clientRtpPort, &clientRtcpPort) != 2) {
                    // error
                    printf("parse Transport error \n");
                }
            }
            line = strtok(NULL, sep);
        }

        if (!strcmp(method, "OPTIONS")) {
            if (handleCmd_OPTIONS(sBuf, CSeq))
            {
                printf("failed to handle options\n");
                break;
            }
        }
        else if (!strcmp(method, "DESCRIBE")) {
            if (handleCmd_DESCRIBE(sBuf, CSeq, url))
            {
                printf("failed to handle describe\n");
                break;
            }
        }
        else if (!strcmp(method, "SETUP")) {
            if (handleCmd_SETUP(sBuf, CSeq, clientRtpPort))
            {
                printf("failed to handle setup\n");
                break;
            }

            serverRtpSockfd = createUdpSocket();
            serverRtcpSockfd = createUdpSocket();
            if (serverRtpSockfd < 0 || serverRtcpSockfd < 0)
            {
                printf("failed to create udp socket\n");
                break;
            }

            if (bindSocketAddr(serverRtpSockfd, "0.0.0.0", SERVER_RTP_PORT) < 0 ||
                bindSocketAddr(serverRtcpSockfd, "0.0.0.0", SERVER_RTCP_PORT) < 0)
            {
                printf("failed to bind addr\n");
                break;
            }

        }
        else if (!strcmp(method, "PLAY")) {
            if (handleCmd_PLAY(sBuf, CSeq))
            {
                printf("failed to handle play\n");
                break;
            }
        }
        else {
            printf("未定义的method = %s \n", method);
            break;
        }

        printf("%s sBuf = %s \n", __FUNCTION__, sBuf);

        send(clientSockfd, sBuf, strlen(sBuf), 0);


        //开始播放，发送RTP包
        if (!strcmp(method, "PLAY")) {

            struct AdtsHeader adtsHeader;
            struct RtpPacket* rtpPacket;
            uint8_t* frame;
            int ret;

            FILE* fp = fopen(AAC_FILE_NAME, "rb");
            if (!fp) {
                printf("读取 %s 失败\n", AAC_FILE_NAME);
                break;
            }

            frame = (uint8_t*)malloc(5000);
            rtpPacket = (struct RtpPacket*)malloc(5000);

            rtpHeaderInit(rtpPacket, 0, 0, 0, RTP_VESION, RTP_PAYLOAD_TYPE_AAC, 1, 0, 0, 0x32411);

            while (true)
            {
                ret = fread(frame, 1, 7, fp);
                if (ret <= 0)
                {
                    printf("fread err\n");
                    break;
                }
                printf("fread ret=%d \n",ret);

                if (parseAdtsHeader(frame, &adtsHeader) < 0)
                {
                    printf("parseAdtsHeader err\n");
                    break;
                }
                ret = fread(frame, 1, adtsHeader.aacFrameLength - 7, fp);
                if (ret <= 0)
                {
                    printf("fread err\n");
                    break;
                }

                rtpSendAACFrame(serverRtpSockfd, clientIP, clientRtpPort,
                    rtpPacket, frame, adtsHeader.aacFrameLength - 7);

                Sleep(1);
                //usleep(23223);//1000/43.06 * 1000
            }

            free(frame);
            free(rtpPacket);

            break;

        }

        memset(method, 0, sizeof(method) / sizeof(char));
        memset(url, 0, sizeof(url) / sizeof(char));
        CSeq = 0;
    }

    closesocket(clientSockfd);
    if (serverRtpSockfd) {
        closesocket(serverRtpSockfd);
    }
    if (serverRtcpSockfd > 0) {
        closesocket(serverRtcpSockfd);
    }

    free(rBuf);
    free(sBuf);

}

int main() {
    // 启动windows socket start
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)
    {
        printf("PC Server Socket Start Up Error \n");
        return -1;
    }
    // 启动windows socket end

    int rtspServerSockfd;

    int ret;

    rtspServerSockfd = createTcpSocket();
    if (rtspServerSockfd < 0)
    {
        printf("failed to create tcp socket\n");
        return -1;
    }

    ret = bindSocketAddr(rtspServerSockfd, "0.0.0.0", SERVER_PORT);
    if (ret < 0)
    {
        printf("failed to bind addr\n");
        return -1;
    }

    ret = listen(rtspServerSockfd, 10);
    if (ret < 0)
    {
        printf("failed to listen\n");
        return -1;
    }

    printf("%s rtsp://127.0.0.1:%d\n", __FILE__, SERVER_PORT);

    while (1)
    {
        int clientSockfd;
        char clientIp[40];
        int clientPort;

        clientSockfd = acceptClient(rtspServerSockfd, clientIp, &clientPort);
        if (clientSockfd < 0)
        {
            printf("failed to accept client\n");
            return -1;
        }

        printf("accept client;client ip:%s,client port:%d\n", clientIp, clientPort);

        doClient(clientSockfd, clientIp, clientPort);
    }

    closesocket(rtspServerSockfd);

    return 0;
}
```

 
  