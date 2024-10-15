---
title: OpenCv读取中文路径
description: opencv-python读取中文路径
date: 2022-03-06
slug: cv/opencv/read_chinese_path
image: 
categories:
    - OpenCv
tags:
    - OpenCv
    - Python
---

**问题：** 在使用`cv2.imread()`读取含有中文路径的图片时，无法正常读取图片，读取到的内容为None。

**原因：** opencv在使用`cv2.imread()`读取图片时，只能接收ASCII码的路径参数，导致cv2在读取含有中文路径的图片时为None，无法正常读取图片。

**解决方法：** 使用`np.fromfile()`读取路径为`np.uint8`	格式，然后使用`cv2.imdecode()`读取数据，并将数据转换为图片格式。

**cv2.imdecode()：** 从指定的内存缓存中读取数据，并把数据转换(解码)成图像格式。

**示例：**

```python
import cv2
import numpy as np

if __name__ == '__main__':
    img = cv2.imdecode(np.fromfile(r'C:\Users\focus\Pictures\背景图片\希望之鲸.jpg', dtype=np.uint8), cv2.IMREAD_COLOR)
    cv2.namedWindow('img', cv2.WINDOW_NORMAL)
    cv2.imshow('img', img)
    cv2.waitKey(0)
```

<div style="text-align: center;">
    <img src="./OpenCv读取中文路径.png" title="" alt="OpenCv读取中文路径">
</div>
