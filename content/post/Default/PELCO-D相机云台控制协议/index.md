---
title: PELCO-D相机云台控制协议
description: PELCO-D相机云台控制协议
date: 2024-05-09
slug: pelco-d
image: 
categories:
    - Default
tags:
    - PELCO-D
    - 协议
---

# pelco

pelco D云台控制协议

## 参考手册

- [PELCO-D协议手册](https://www.commfront.com/pages/pelco-d-protocol-tutorial)
- [PELCO-D命令列表](https://www.epiphan.com/userguides/LUMiO12x/Content/UserGuides/PTZ/3-operation/PELCODcommands.htm)

## PELCO-D格式

Pelco-D是由7个十六进制字节组成（除非另有说明，本页中使用的所有字节数据均为十六进制格式）。

|  Byte1   |  Byte2   | Byte3 | Byte4 | Byte5 | Byte6 | Byte7 |
|:--------:|:--------:|:-----:|:-----:|:-----:|:-----:|:-----:|
| Sync同步字节 | 控制相机逻辑地址 |  命令1  |  命令2  | 平移速度  | 倾斜速度  |  校验和  |

- Byte1(Sync): 同步字节，固定为 FF
- Byte2(Camera Address):- 被控相机逻辑地址
- Byte3和Byte4如下所示
- Byte5(平移速度):
    - 范围从00（停止）到3F（高速）
    - FF表示“涡轮”速度（设备可以达到的最大平移速度）
        - 涡轮速度是被单独考虑的，所以从告诉到涡轮速度通常不是一个平稳的步骤
- Byte6(倾斜速度): 范围从00（停止）到3F（最大速度）
- Byte7(校验和): 字节总和（不包括Sync同步字节），然后模0x100（十进制为：256）

## 命令数据

|          |     Bit7      |     Bit6      |     Bit5      |           Bit4            |         Bit3         |       Bit2       |      Bit1       |      Bit0      |
|:--------:|:-------------:|:-------------:|:-------------:|:-------------------------:|:--------------------:|:----------------:|:---------------:|:--------------:|
| Command1 |  Sense(感测位)   | Reserved(保留位) | Reserved(保留位) | Auto/Manual Scan(自动/手动扫描) | Camera On/Off(相机开/关) | Iris Close(关闭光圈) | Iris Open(打开光圈) | Focus Near(近焦) |
| Command2 | Focus Far(远焦) | Zoom Out(缩小)  |  Zoom In(放大)  |       Tilt Down(下移)       |     Tilt Up(上移)      |   Pan Left(左移)   |  Pan Right(右移)  |   Fixed to 0   |

- Sense(感测位): 表示命令1位4和位3的含义
    - 如果Sense为1(Command1 Bit7)，且Auto/Manual Scan(自动/手动扫描)为1(Command1 Bit4)和Camera On/Off(相机开/关)为1(
      Command1 Bit3)，则该命令将启用自动扫描并打开相机
    - 如果Sense为0(Command1 Bit7)，且Auto/Manual Scan(自动/手动扫描)为1(Command1 Bit4)和Camera On/Off(相机开/关)为1(
      Command1 Bit3)，则该命令将启用手动扫描并关闭相机
    - 如果Auto/Manual Scan(自动/手动扫描)为0(Command1 Bit4)和Camera On/Off(相机开/关)为0(Command1 Bit3), 将不会有任何操作
- Reserved(保留位): 保留位应设置为0

## 常用扩展命令

> 在命令数据的基础上，PELCO-D协议还支持扩展命令，可以通过扩展命令实现更多的云台控制需求

|                Command                 | Byte1 |  Byte2  | Byte3 | Byte4 |      Byte5      |     Byte6      | Byte7 |
|:--------------------------------------:|:-----:|:-------:|:-----:|:-----:|:---------------:|:--------------:|:-----:|
|                 Up(上移)                 | 0xFF  | Address | 0x00  | 0x08  |   Pan Speed	    |   Tilt Speed   |  SUM  |  
|                Down(下移)                | 0xFF  | Address | 0x00  | 0x10  |    Pan Speed    |   Tilt Speed   |  SUM  |
|                Left(左移)                | 0xFF  | Address | 0x00  | 0x04  |    Pan Speed    |   Tilt Speed   |  SUM  |
|               Right(右移)                | 0xFF  | Address | 0x00  | 0x02  |    Pan Speed    |   Tilt Speed   |  SUM  |
|               UpLeft(左上)               | 0xFF  | Address | 0x00  | 0x0C  |    Pan Speed    |   Tilt Speed   |  SUM  |
|              UpRight(右上)               | 0xFF  | Address | 0x00  | 0x0A  |    Pan Speed    |   Tilt Speed   |  SUM  |
|              DownLeft(左下)              | 0xFF  | Address | 0x00  | 0x14  |    Pan Speed    |   Tilt Speed   |  SUM  |
|             DownRight(右下)              | 0xFF  | Address | 0x00  | 0x12  |    Pan Speed    |   Tilt Speed   |  SUM  |
|              Zoom In(放大)               | 0xFF  | Address | 0x00  | 0x20  |      0x00       |      0x00      |  SUM  |
|              Zoom Out(缩小)              | 0xFF  | Address | 0x00  | 0x40  |      0x00       |      0x00      |  SUM  |
|             Focus Far(远焦)              | 0xFF  | Address | 0x00  | 0x80  |      0x00       |      0x00      |  SUM  |
|             Focus Near(近焦)             | 0xFF  | Address | 0x01  | 0x00  |      0x00       |      0x00      |  SUM  |
|           Set Preset(设置预置位)            | 0xFF  | Address | 0x00  | 0x03  |      0x00       |   Preset ID    |  SUM  |
|          Clear Preset(删除预置位)           | 0xFF  | Address | 0x00  | 0x05  |      0x00       |   Preset ID    |  SUM  |
|           Call Preset(前往预置位)           | 0xFF  | Address | 0x00  | 0x07  |      0x00       |   Preset ID    |  SUM  |
|       Query Pan Position(查询平移位置)       | 0xFF  | Address | 0x00  | 0x51  |      0x00       |      0x00      |  SUM  |
| Query Pan Position Response(查询平移位置响应)  | 0xFF  | Address | 0x00  | 0x59  | Value High Byte | Value Low Byte |  SUM  |
|      Query Tilt Position(查询倾斜位置)       | 0xFF  | Address | 0x00  | 0x53  |      0x00       |      0x00      |  SUM  |
| Query Tilt Position Response(查询倾斜位置响应) | 0xFF  | Address | 0x00  | 0x5B  | Value High Byte | Value Low Byte |  SUM  |
|      Query Zoom Position(查询缩放位置)       | 0xFF  | Address | 0x00  | 0x55  |      0x00       |      0x00      |  SUM  |
| Query Zoom Position Response(查询缩放位置响应) | 0xFF  | Address | 0x00  | 0x5D  | Value High Byte | Value Low Byte |  SUM  |
