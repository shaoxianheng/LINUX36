# FFmpeg

## 音视频基本概念

```
编码（encode）

​      通过特定的压缩技术，将某个视频的视频流格式转换成另一种视频格式的视频流方式。

解码（decode)

​		通过特定的解压缩技术，将某个视频格式的视频流转换成另一种视频格式的视频流方式。

转码（transcode）

​		视频转码技术将视频信号从一种格式转换成另一种格式。


```

### 编码

```
视频：

​			YUV420/422->H264

​	         RGB888->H264

​             YUV420->h263



音频

​			PCM（原始) ->AAC

​	        PCM（原始）->G726

​	        PCM （原始) -> G711
```



### 转码

```
视频：

​			改变分辨率（resolution)

​			改变帧率（frame rate）

​	        改变比特率 （bit rate） 等编码参数

音频：

​			改变采样率（sample rate)

​			改变通道数（channels）

​			改变位宽 （sample format）
```



### 封装和解封装

```
封装（mux）：

​						改变分辨率 （resolution）

​						改变帧率 （frame rate）

​						改变比特率 等编码参数

解封装 （demux）：

​						改变采样率 （sample rate）

​						改变通道数	（channels）

​						改变位宽 	（sample format）
```

