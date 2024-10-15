---
title: 基于OpenCv的视频流处理方法
description: 基于OpenCv的视频流处理方法
date: 2022-06-03
slug: cv/opencv/video_stream
image: 
categories:
    - OpenCv
tags:
    - OpenCv
    - Python
---

# 使用VideoCapture读取视频流

在使用OpenCv处理视频时，无论是视频文件还是摄像头画面，都要使用VideoCapture类来进行每一帧图像的处理。当我们使用视频文件作为参数时，OpenCv则打开视频文件，进行每一帧画面的读取。当我们传递摄像机编号时，OpenCv则打开相机，实时读取相机画面。

**获取VideoCaptrue实例：**

```python
# 读取视频文件
cv2.VideoCapture('video.mp4')
# 打开摄像机
cv2.VideoCapture(0)
```

## 使用VideoCapture读取海康RTSP流

### RTSP流

在使用OpenCv进行计算机视觉处理时，我们很多时候需要连接外部相机，如海康威视。监控相机的常见视频传输协议有：RTSP、RTMP（以RTSP为主）
**RTSP与RTMP比较：**

* RTSP：低时延，实现难度大，适合视频聊天和视频监控
* RTMP：低时延，实现难度大，适合视频聊天和视频监控

目前市面上的相机大多以RTSP流协议为主。
在读取海康相机时，需要使用VideoCapture读取RTSP流协议的内容，而不是通过相机编号直接读取。

```python
"""
海康相机rtsp格式：rtsp://[username]:[password]@[ip]:[port]/[codec]/[channel]/[subtype]/av_stream
        username: 用户名。例如admin。
        password: 密码。例如12345。
        ip: 为设备IP。例如 192.0.0.64。
        port: 端口号默认为554，若为默认可不填写。
        codec：有h264、MPEG-4、mpeg4这几种。
        channel: 通道号，起始为1。例如通道1，则为ch1。
        subtype: 码流类型，主码流为main，辅码流为sub。
"""
```

**使用VideoCapture读取RTSP流示例：**

```python
# 使用rtsp流打开相机
def open_camera(username: str, password: str, ip: str, port: int):
    """
    使用rtsp流打开相机
    rtsp格式：rtsp://[username]:[password]@[ip]:[port]/[codec]/[channel]/[subtype]/av_stream
        username: 用户名。例如admin。
        password: 密码。例如12345。
        ip: 为设备IP。例如 192.0.0.64。
        port: 端口号默认为554，若为默认可不填写。
        codec：有h264、MPEG-4、mpeg4这几种。
        channel: 通道号，起始为1。例如通道1，则为ch1。
        subtype: 码流类型，主码流为main，辅码流为sub。
    :return:相机是否打开，相机
    """
    try:
        # 使用rtsp流打开相机
        cam = cv2.VideoCapture(f'rtsp://{username}:{password}@{ip}:{port}/h264/ch1/main/av_stream')
        return True, cam

    except cv2.error:
        # 捕获cv异常
        # 打开相机失败
        return False, None
```

### ping3

在使用RTSP流读取相机内容时，若IP错误，或相机连接异常，则VideoCapture无法访问相机，VideoCapture实例访问超时且程序异常，在使用PyQt等GUI程序中，程序可能出现异常闪退情况，所以在连接RTSP流前，请先对相机IP进行预校验，减少出错概率。可以在访问相机之前，通过ping 相机IP的方式来对IP进行预校验。
**在Python中可以使用ping3库对IP进行ping测试。**

* ping3.ping成功默认返回秒为单位
* ping3.ping参数错误返回False
* ping3.ping失败时返回None

**ping3.ping函数返回说明：**

> Returns:
> The delay in seconds/milliseconds, False on error and None on timeout.

**ping3.ping示例：**

```python
import ping3

if __name__ == '__main__':
    print(ping3.ping('focus-wind.com'))
    print(ping3.ping('https://focus-wind.com'))
    print(ping3.ping('192.168.1.64'))
```

**ping3.ping运行结果：**

```python
# ping focus-wind.com ping成功
0.03125762939453125
# ping https://focus-wind.com 参数异常
False
# ping 192.168.1.64 无法访问对应IP
None
```

### 完整访问RTSP流示例

```python
def open_camera(username: str, password: str, ip: str, port: int = 554):
    """
    使用rtsp流打开相机
    rtsp格式：rtsp://[username]:[password]@[ip]:[port]/[codec]/[channel]/[subtype]/av_stream
        username: 用户名。例如admin。
        password: 密码。例如12345。
        ip: 为设备IP。例如 192.0.0.64。
        port: 端口号默认为554，若为默认可不填写。
        codec：有h264、MPEG-4、mpeg4这几种。
        channel: 通道号，起始为1。例如通道1，则为ch1。
        subtype: 码流类型，主码流为main，辅码流为sub。
    :return:相机是否打开，相机
    """
    try:
        # ping IP，IP预检验
        if ping3.ping(ip) is None or not ping3.ping(ip):
            # ping的结果为None或False，ping失败，不存在该IP
            return False, None
        else:
            # 使用rtsp流打开相机
            cam = cv2.VideoCapture(f'rtsp://{username}:{password}@{ip}:{port}/h264/ch1/main/av_stream')
            return True, cam

    except cv2.error:
        # 捕获cv异常
        # 打开相机失败
        return False, None
```

## 获取视频流信息

一般视频流的信息主要包含画面的宽高以及视频显示帧率，对于视频文件，还包括总共有多少帧画面。针对这些信息，我们可以使用VideoCapture类的get方法获取相关信息。

```python
# 获取视频帧的宽
width = cam.get(cv2.CAP_PROP_FRAME_WIDTH)
# 获取视频帧的高
height = cam.get(cv2.CAP_PROP_FRAME_HEIGHT)
# 获取视频帧的帧率
fps = cam.get(cv2.CAP_PROP_FPS)
# 获取视频文件的总帧数
frame_count = cam.get(cv2.CAP_PROP_FRAME_COUNT)
```

针对获取总帧数函数，若相机不支持获取总帧数，则返回0，获取总帧数仅对视频文件有意义。

## 获取帧画面

直接调用VideoCapture类的read方法，read方法会返回两个参数，一个为是否成功标志，一个为帧画面。
**read方法返回参数：**

* 成功，BGR格式的numpy.ndarray三维数组
* 失败，None

**read使用示例：**

```python
# 获取帧画面
ret, frame = cam.read()
```

在调用read方法时，有时候可能会访问失败，所以为避免访问失败，可以采用while循环确保读取顺利。

```python
# 获取帧画面
ret, frame = cam.read()
# 循环读取，确保读取成功
while not ret:
	ret, frame = cam.read()
```

## 读取帧画面优化

OpenCv底层基于ffmpeg读取视频，在OpenCv读取视频流时，会设置缓存区，将视频流读取到缓存区中，但是使用缓存区的话，会导致页面堆积，页面延迟高过，所以为了避免OpenCv缓存区视频流堆积的情况，可以使用线程实时读取OpenCv画面，将读取的每一帧内容存在队列中，在需要获取帧画面时，获取队列中的数据。

```python
import queue
import threading

import cv2


class CameraThread(threading.Thread):
    # 保存实例化相机，通过实例化相机操作相机
    camera = None
    # 保存每一帧从rtsp流中读取到的画面
    queue_image = queue.Queue(maxsize=10)

    # 线程体是否循环执行标志
    flag_run = False

    # 相机线程调用函数
    def run(self) -> None:
        while self.flag_run:
            try:
                # 捕获异常，避免读取视频操作因异常而退出
                # 相机实例存在，判断相机是否打开
                if self.camera.is_opened():
                    # 相机已打开，读取相机内容
                    ret, frame = self.camera.read()
                    if not ret or frame is None:
                        # 读取相机内容失败
                        break
                    if ret:
                        # 将内容添加到队列中
                        # 判断队列是否满
                        if self.queue_image.full():
                            # 队列满，队头出队
                            self.queue_image.get()
                            # 队尾添加数据
                            self.queue_image.put(frame)
                        else:
                            # 队尾添加数据
                            self.queue_image.put(frame)
            except cv2.error as error:
                # 捕获cv异常
                # 因为子线程会一直调用该程序，可对捕获到的异常不进行处理
                print(error)
                pass

            except Exception as error:
                # 捕获Exception异常
                print(error)
                pass

    # setter：设置相机的camera对象
    def set_camera(self, camera):
        """
        设置相机的camera对象
        """
        # 设置相机的camera
        self.camera = camera

    # 获取队列中的RGB图像
    def get_image(self):
        """
        获取队列中的RGB图像
        :return: img: RGB图像
        """
        # 读取队列中的图片
        img = self.queue_image.get()
        # img为None，读取失败，队列还未获取到图片
        while img is None:
            # 一直读取图片，直到读取图片成功
            img = self.queue_image.get()
        # 返回读取到的内容
        return img

    # 停止运行
    def run_stop(self):
        self.flag_run = False

    # 开始运行
    def run_start(self):
        self.flag_run = True
```
