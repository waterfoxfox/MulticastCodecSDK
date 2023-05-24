

---

# 极简音视频组播(内置编解码) WIN SDK


作为一种与单播（Unicast）和广播（Broadcast）并列的通信方式，组播（Multicast）技术能够有效地解决单点发送、多点接收的问题，从而实现了网络中点到多点的高效数据传送，能够节约大量网络带宽、降低网络负载。常见基于组播的音视频应用有同屏教学、校园广播等。组播基于UDP传输，因此同样面临UDP传输常见的乱序、丢包、重复等挑战，本SDK主要为传统UDP组播增加FEC前向纠错、NACK丢包重传、Qos、JitterBuff等功能，同时内置了音视频编解码能力，为开发音视频组播应用提供易用、可靠的解决方案。

本音视频组播SDK，支持特性：
* 1、	独有的NACK组播重传技术，为每个组播接收端提供个性化重传服务，网络适应性更强。
* 2、	支持基于RS编码的FEC前向纠错，专为音视频类应用量身定制。
* 3、	支持FEC动态Group分组，避免分组跨帧同时尽可能提升抗连续丢包能力。
* 4、	发送端支持Smooth平滑发送，对视频大I帧场景进行优化。
* 5、	支持接收端自适应JitterBuff缓存技术。
* 6、	SDK支持输入YUV420P、NV12、NV21、RGB24、RGBA32格式原始图像，支持Nvidia、Qsv硬件H264编码以及X264软编码、X265软编码。当系统不支持硬编码时自行切换为软编码。
* 7、   SDK支持ffmpeg H264、HEVC软件解码，输出YUV420P。
* 8、   SDK支持输入输出音频PCM，采用Fdk AAC LC编解码。
* 9、   SDK内置视频缩放、音频重采样能力，支持外层输入可变的不同于编码宽高的采集宽高、支持输出外层指定的采样率、声道数。
* 10、	C++开发，性能强劲，无外部库依赖，支持32位、64位系统。
* 11、	API接口简洁易用、完善的文档与DEMO参考。

---



# 一、基本概念
单播、广播、组播：
作为一种与单播（Unicast）和广播（Broadcast）并列的通信方式，组播（Multicast）技术能够有效地解决单点发送、多点接收的问题，从而实现了网络中点到多点的高效数据传送，能够节约大量网络带宽、降低网络负载。利用组播技术可以方便地提供一些新的增值业务，包括在线直播、网络电视、远程教育、远程医疗、网络电台、实时视频会议等对带宽和数据交互的实时性要求较高的信息服务。
三种信息传输方式的比较
* **单播** <br>
如下图所示，在IP网络中若采用单播的方式，信息源（即Source）要为每个需要信息的主机（即Receiver）都发送一份独立的信息拷贝。采用单播方式时，网络中传输的信息量与需要该信息的用户量成正比，因此当需要该信息的用户数量较大时，信息源需要将多份内容相同的信息发送给不同的用户，这对信息源以及网络带宽都将造成巨大的压力。从单播方式的信息传播过程可以看出，该传输方式不利于信息的批量发送。单播主要用于公网场景或无线局域网中，这类场景中广播与组播均无法应用或效果不佳。<br>
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/6acec096dfcb44769a6dc720479bb017.png#pic_center) 
 
											


* **广播** <br>
如下图所示，在一个网段中若采用广播的方式，信息源（即Source）将把信息传送给该网段中的所有主机，而不管其是否需要该信息。假设只有Host B、Host D和Host E需要信息，若将该信息在网段中进行广播，则原本不需要信息的Host A和Host C也将收到该信息，这样不仅信息的安全性得不到保障，而且会造成同一网段中信息的泛滥。因此，广播方式不利于与特定对象进行数据交互，并且还浪费了大量的带宽。<br>

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/7c0ed6cea4f14e00b3a07e413dfe7ab5.png#pic_center)



* **组播** <br>
综上所述，传统的单播和广播的通信方式均不能以最小的网络开销实现单点发送、多点接收的问题，IP组播技术的出现及时解决了这个问题。
如下图所示，当IP网络中的某些主机（即Receiver）需要信息时，若采用组播的方式，组播源（即Source）仅需发送一份信息，借助组播路由协议建立组播分发树，被传递的信息在距离组播源尽可能远的网络节点才开始复制和分发。假设只有Host B、Host D和Host E需要信息，采用组播方式时，可以让这些主机加入同一个组播组（Multicast group），组播源向该组播组只需发送一份信息，并由网络中各路由器根据该组播组中各成员的分布情况对该信息进行复制和转发，最后该信息会准确地发送给Host B、Host D和Host E。相比单播来说，组播的优势在于：由于被传递的信息在距信息源尽可能远的网络节点才开始被复制和分发，所以用户的增加不会导致信息源负载的加重以及网络资源消耗的显著增加。相比广播来说，组播的优势在于：由于被传递的信息只会发送给需要该信息的接收者，所以不会造成网络资源的浪费，并能提高信息传输的安全性；另外，广播只能在同一网段中进行，而组播可以实现跨网段的传输。组播的缺点：由于无线组播报文的发送没有相应的确认机制（数据链路层ACK），因此设备无法对丢失的报文进行重传，导致链路质量差的情况下，无线环境下组播报文丢失严重。<br>

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/0a1e5789e3fe4e148509c88013691efa.png#pic_center)





* 音视频组播发送端：
作为组播源将自身音视频组播发送。

* 音视频组播接收端：
接收音视频组播流。

* FEC：
前向纠错技术，通过增加冗余带宽方式，提高网络对于丢包的抵抗力。

* FEC相关参数：
FEC相关参数包括FEC冗余度方法、FEC上行冗余度、FEC min group组大小、FEC max group组大小。FEC冗余度方法包括：固定冗余度（本SDK仅支持固定冗余度）、自动冗余度两种。固定冗余度即全程使用用户指定的冗余度进行FEC编码，自动冗余度则以用户指定的FEC冗余度为基础，根据网络情况进行调整，对于网络较好的场合使用低冗余尽量降低带宽。FEC是分组进行的，即多个拆分包组成一个group，产生其冗余包。group越大，同样冗余度的情况下，产生的冗余包越多，抵抗连续丢包能力越强，同时因丢包产生的抖动也会越大（因为FEC的group可能产生跨多帧的情况，前面包的丢失不得不等到后续包的到来才能恢复）；FEC分组越大消耗CPU资源也越大。建议配置FEC min group大小16；建议根据芯片处理性能设置FEC max group，在性能足够的设备上建议设置为64（PC、主流Android手机均可设置为64，嵌入式平台则根据实际情况设置）。

* NACK：
重传请求机制，与FEC配合，在预估FEC无法恢复时触发接收端到发送端的重传请求，发送端予以重发响应。NACK仅重传一次，不保证数据一定传输成功，优点是可以获得稳定的低延时，缺点是画面仍可能因丢包而卡顿。

* 组播NACK支持：
组播是一种单向的传输技术，正常情况下无法使用NACK重传机制，因为每一个接收端都可能有不同的丢包情况。我们使用单播作为组播的一种补充，从而实现了为不同组播接收端提供个性化的NACK服务。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a462ec49467a482b9dd78bf0e5af6f24.png#pic_center)

* Smooth平滑：
发送端对一帧较大的码流在时间窗口内平滑发送，避免一次性发出给网络带来的压力。

* JitterBuff：
接收端为了抵消网络传输、丢包恢复、NACK重传引入的抖动，引入JitterBuff缓存。缓存时间越大，画面流畅度越高，但延时也同步增大。当需要极低延时时，可设置JitterBuff为0。

* 传输参数：
本SDK中传输相关参数包括：视频通道的上行FEC冗余度、上行FEC Group分组大小、NACK重传支持、接收端JitterBuff缓存时间。对于音视频组播接收端，同样可以设置上行FEC冗余度、上行FEC Group分组大小，只是没有实际意义(因为不会发送数据，当然也不会进行FEC编码)。




---
# 二、API接口
 所有API接口定义均位于SDTerminalSdk.h文件中。

### 1、系统环境初始化，仅需调用一次
```c
void  SDTerminal_Enviroment_Init(const char * outputPath,  int outputLevel)
参数：
@param: outputPath：日志文件输出的目录，若目录不存在，SDK将自动创建，支持相对路径或绝对路径。日志对于问题定位非常重要，建议开启。
@param: outputLevel：日志输出的级别，只有等于或者高于该级别的日志会输出到文件，日志级别有： DEBUG、INFO、WARNING、ERROR、ALARM、FATAL、NONE，当指定为NONE时，将不会生成日志文件。具体见TERMINAL_LOG_OUTPUT_LEVEL定义。
```

### 2、系统退出时调用一次反初始化
```c
void  SDTerminal_Enviroment_Free ()
```

### 3、创建SDK对象
```c
void*  SDTerminal_New(TerminalEncodeParams* ptEncodeParams, RemoteVideoYuvDataCallback pfOutputVideo, RemoteAudioPcmDataCallback pfOutputAudio, void* pObject);
参数：
@ ptEncodeParams，音视频编码相关参数，包括编码标准类型、是否采用硬编码、编码码率、编码帧率、编码分辨率、音频采样率、声道数等。
@ pfOutputVideo，视频接收解码输出回调函数。
@ pfOutputAudio，音频接收解码输出回调函数。
@ pObject，输出回调函数透传数据。
返回值：对象指针，返回NULL表示失败。
```

### 4、销毁SDK对象
```c
void  SDTerminal_Delete(void** ppTerminal);
参数：
@ ppTerminal，模块指针的指针
说明：使用者应该做好与其他API之间的互斥保护
返回值：无

```

### 5、准备会话
```c
BOOL  SDTerminal_Online(
void* pTerminal,
TerminalLogonParams* ptLogonParams, 
TerminalTransParams* ptTransParams);
参数：
@ pTerminal，模块指针
@ ptLogonParams，建立连接相关参数，包括客户端类型、组播IP、组播端口、本地IP、组播NACK端口。本地IP的指定用于避免在多网卡时系统默认IP不可控的问题。组播NACK端口用于单播方式传输NACK信令与媒体包，需配置为与组播源一致端口。
@ ptTransParams，音视频传输相关参数，包括FEC相关、接收缓存相关等。
返回值：
返回TRUE表示成功，返回FALSE则为失败。

```

### 6、结束会话
```c
void  SDTerminal_Offline(void* pTerminal);
参数：
@ pTerminal，模块指针
返回值：无

```
### 7、发送视频数据
```c
void  SDTerminal_SendVideoData(void* pTerminal, unsigned char* buf, unsigned int unWidth, unsigned int unHeight, TERMINAL_VIDEO_INPUT_FORMAT eFormat);
参数：
@ pTerminal，模块指针
@ buf，输入图像存放区。
@ unWidth，输入图像宽度，当与编码宽高不一致时，内部自行缩放。建议与编码宽高一致。
@ unHeight，输入图像高度，当与编码宽高不一致时，内部自行缩放。建议与编码宽高一致。
@ eFormat, 输入图像格式，建议与编码器当前使用的格式一致。
说明：若采用硬编码，外层可先通过SDTerminal_GetHardwareEncInfo接口获得当前硬编码支持的输入格式，以此作为外层采集格式。这样避免内部再进行色度空间转换、缩放处理，提高性能。
返回值：无

```

### 8、发送音频数据
```c
void  SDTerminal_SendAudioData(void* pTerminal, unsigned char* buf, unsigned int unLen);
参数：
@pTerminal，模块指针
@ buf，输入音频PCM存放区。
@ unLen，输入音频PCM字节数。
返回值：无

```


### 9、获取当前SDK版本信息
```c
unsigned int SDTerminal_GetVersion (void* pTerminal);
参数：
@ pTerminal，模块指针
返回值：获得当前SDK的版本信息

```


---

# DEMO源码
https://github.com/waterfoxfox/SDT-Multicast-FileSendDemo <br>
https://github.com/waterfoxfox/SDT-Multicast-FileRecvDemo <br>



# 相关资源
组播纯传输SDK（无内置编解码）:  [https://github.com/waterfoxfox/MulticastSDK/](https://github.com/waterfoxfox/MulticastSDK/)  <br>
DEMOS: [https://github.com/waterfoxfox/MulticastDemos](https://github.com/waterfoxfox/MulticastDemos) <br>
SDK 商用及定制化、技术支持服务可联系：[http://www.mediapro.cc/](http://www.mediapro.cc/)

