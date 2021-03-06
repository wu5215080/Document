## 前言
本文档重点说明的是可能和标准协议不一样的部分，其他的具体细节可以对应本目录下的抓包数据 qiniu-rtmp.pcap 和 rtmp_specification_1.0.pdf。
## 握手阶段
握手分为简单握手和复杂握手。先判断是否能复杂握手，如果能复杂握手，就复杂握手，否则使用简单握手。
## 交互阶段
命令顺序严格按照抓包。
其中 `connect()` 命令的 `flashVer` 字段，格式为 `FMLE/3.0 (compatible; obs-studio/0.12.0; FMSc/1.0)`，需要表明设备名称和软件名称。
## metadata 字段
必填字段为：
 - 播放：`cdn_ip` 边缘节点 IP
 - 推流：`framerate` 帧率
 - 推流：`videodatarate` 视频码率(kbps), 方便CDN判断是否进行转码
 - 推流：`audiodatarate` 音频码率
 
其他字段选填。

## Latency Benchmark 

关于延迟测量方法(Latency Benchmark)，时间戳同步和延迟测量。

### Time Sync

推流和播放器会调用API同步校准时间，熊猫提供API返回当前绝对时间。

推流时，将绝对时间，放到`onMetaData`数据包中，供后续时间戳作为基准参考（比如使用相对时间戳）。

推流端校准时间后，使用校准的时间打Packet的时间戳。

> 备注：在APP启动时，做一次时间校准。在推流前做一次校准。

> 备注：需要使用UTC时间。

例如：

```
推流端启动时，调用API：http://foo.me/api/timestamp
返回：TBD

在推流时，发送onMetaData：
Key：cu_basetime
Value: 绝对时间，格式TBD #转换为毫秒是1492137596654
```

### Timestamp Packet

推流端可以选择相对时间，将Timestamp在RTMP/FLV标准协议的Timestamp中传输。

CDN收到包后，不做时间戳改变，透传给播放器。

播放器根据校准时的绝对时间，和流中的时间比较得出延迟。

例如：

```
假设onMetaData中指定的绝对时间1492137596654
推流时发送的包序列：

Type: Video
Timestamp: 0
Data: xxx
#包的绝对时间: 1492137596654+0

Type: Audio
Timestamp: 0
Data: xxx
#包的绝对时间: 1492137596654+0

Type: Video
Timestamp: 40
Data: xxx
#包的绝对时间: 1492137596654+40

Type: Audio
Timestamp: 50
Data: xxx
#包的绝对时间: 1492137596654+50
```

在播放器根据当前的时间，可以计算出延迟。

## 数据阶段
音频 csid 为 6，视频/AMF0/AMF3消息 csid 为 4，其他 csid 为 5
