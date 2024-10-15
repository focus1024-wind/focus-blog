---
title: 视频流传输协议
description: 视频流传输协议
date: 2022-06-15
slug: cv/opencv/video_stream_protocol
image: 
categories:
    - OpenCv
tags:
    - OpenCv
    - Python
---

# 常见视频传输协议

| 协议 | httpflv | rtmp | rtsp | hls | dash |
| --- | --- | --- | --- | --- | --- |
| 传输方式 | http流 | tcp流 | tcp流 | http | http |
| 视频封装格式 | flv | flv tag | ts mp4 | Ts文件 | Mp4 3gp webm |
| 延时 | 低 | 低 | 低 | 高 | 高 |
| 数据分段 | 连续流 | 连续流 | 连续流 | 切片文件 | 切片文件 |
| Html5播放 | 可通过html5解封包播放(flv.js) | 不支持 | 不支持 | 可通过html5解封包播放(hls.js) | 如果dash文件列表是mp4webm文件，可直接播放 |

监控行业常见的视频传输协议：RTSP，RTMP（以RTSP流为主）

## RTSP与RTMP比较

* RTSP：低时延，实现难度大，适合视频聊天和视频监控
* RTMP：浏览器支持好，加载flash插件后能直接播放（高版本浏览器目前已禁止flash插件）

## 直播常见协议：RTMP，HTTP

* RTMP：只支持flashplayer，目前已被淘汰
* HTTP：flv，m3u8，ts
* flv：flash video，需要flash支持，使用flv.js可支持播放（B站视频）
* m3u8：切片文件，有延迟，实时性不如RTSP协议，如果压缩过小，可能导致客户端网络原因变卡，如果压缩过大，可能导致视频延迟过高
* ts：切片文件，同m3u8
