---
title: 经纬度相对坐标转换
description: 将
date: 2025-01-03
slug: latitude_and_longitude_2D_coordinates
image:
categories:

- Default

---

在现实世界中，人们经常通过使用经纬度来表示地理位置信息，但在计算机画面中，我们是通过像素坐标信息来表示坐标信息，所以针对经纬度等坐标信息，我们需要将其转换为对应的二维坐标，方便进行位置计算。

在已知经纬度信息的情况下，求两点之间的相对笛卡尔坐标关系，我们可以通过求两点之间的方位角和距离信息，即可求得两坐标之间的对应关系。

# 经纬度距离计算

在计算经纬度距离时，通常使用的是球面三角法中的Haversine公式。这个公式可以计算地球表面上两点之间的最短距离。

Haversine公式如下所示，其中所有角度都为弧度信息：

$$
a = \sin^2\left(\frac{\Delta \varphi}{2}\right) + \cos \varphi_1 \cdot \cos \varphi_2 \cdot \sin^2\left(\frac{\Delta \lambda}{2}\right)
$$
$$
c = 2 \cdot \text{atan2}\left(\sqrt{a}, \sqrt{1-a}\right)
$$
$$
d = R \cdot c
$$

- $\lambda_1, \varphi_1$: 是第一个点的经度和纬度，
- $\lambda_2, \varphi_2$: 是第二个点的经度和纬度，
- $\Delta \varphi = \varphi_2 - \varphi_1$: 是纬度差，
- $\Delta \lambda = \lambda_2 - \lambda_1$: 是经度差，
- $R$: 是地球半径（平均值为6,371公里），
- $d$: 就是两点之间的距离。

## Haversine公式的Python实现

```Python
import math


def haversine(lat1, lon1, lat2, lon2):
    """
    计算两个经纬度坐标点之间的距离。

    :param lat1: 第一个点的经度 (十进制度数)
    :param lon1: 第一个点的纬度 (十进制度数)
    :param lat2: 第二个点的经度 (十进制度数)
    :param lat2: 第二个点的纬度 (十进制度数)
    :return: 两点之间的距离（km）
    """
    # 将十进制度数转化为弧度
    lat1, lon1, lat2, lon2 = map(math.radians, [lat1, lon1, lat2, lon2])

    # Haversine公式
    dlon = lon2 - lon1
    dlat = lat2 - lat1
    a = math.sin(dlat / 2) ** 2 + math.cos(lat1) * math.cos(lat2) * math.sin(dlon / 2) ** 2
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    r = 6371  # 地球平均半径，单位为公里
    return c * r


distance = haversine(34.259458, 108.947001, 31.23351, 121.505366)
print(f"西安钟楼到上海中心大厦之间的距离是 {distance:.2f} 公里")

# 西安钟楼到上海中心大厦之间的距离是 1220.78 公里
```

## 通过 `geopy` 计算经纬度距离

`geopy` 是 Python 的一个地理信息编码库，该库同样也提供了计算两点之间距离的功能。在 Python 中可以使用该库计算经纬度距离。

```python
from geopy.distance import geodesic

# 定义两个地点的经纬度
location_a = (34.259458, 108.947001)
location_b = (31.23351, 121.505366)

# 计算两个地点之间的距离
distance_km = geodesic(location_a, location_b).kilometers

print(f"西安钟楼到上海中心大厦之间的距离是 {distance_km:.2f} 公里")

# 西安钟楼到上海中心大厦之间的距离是 1222.91 公里
```

# 方位角计算

在计算两个地理坐标点之间的方位角，通过使用球面三角学的公式。方位角表示从一个点出发，沿地球表面到达另一点时的初始前进方向，通常以真北为参考点，顺时针方向测量。

方位角计算公式为：

$$
\Delta \lambda = \lambda_2 - \lambda_1
$$
$$
y = \sin(\Delta \lambda) \cdot \cos(\varphi_2)
$$
$$
x = \cos(\varphi_1) \cdot \sin(\varphi_2) - \sin(\varphi_1) \cdot \cos(\varphi_2) \cdot \cos(\Delta \lambda)
$$
$$
\theta = \text{atan2}(y, x)
$$

- $P_1(\lambda_1, \varphi_1)$: 点1的经度和纬度
- $P_2(\lambda_2, \varphi_2)$: 点2的经度和纬度
- 方位角以正北为参考点
- 方位角沿顺时针测量
- 上述的角度信息均为弧度信息

## 方位角计算Python实现

```Python
import math


def calculate_bearing(lat1, lon1, lat2, lon2):
    """
    计算从点1到点2的初始方位角。

    :param lat1: 第一个点的经度 (十进制度数)
    :param lon1: 第一个点的纬度 (十进制度数)
    :param lat2: 第二个点的经度 (十进制度数)
    :param lat2: 第二个点的纬度 (十进制度数)
    :return: 从点1到点2的初始方位角（度）
    """
    # 将十进制度数转换为弧度
    lat1 = math.radians(lat1)
    lat2 = math.radians(lat2)
    delta_lon = math.radians(lon2 - lon1)

    # 使用公式计算方位角
    y = math.sin(delta_lon) * math.cos(lat2)
    x = math.cos(lat1) * math.sin(lat2) - math.sin(lat1) * math.cos(lat2) * math.cos(delta_lon)
    initial_bearing = math.atan2(y, x)

    # 将弧度转换为度，并调整范围至0-360度
    initial_bearing = math.degrees(initial_bearing)
    compass_bearing = (initial_bearing + 360) % 360

    return compass_bearing


bearing = calculate_bearing(34.259458, 108.947001, 31.23351, 121.505366)
print(f"西安钟楼到上海中心大厦之间的初始方位角是 {bearing:.2f} 度")

# 西安钟楼到上海中心大厦之间的初始方位角是 102.52 度
```

# 相对坐标计算

根据上述公式，我们知道从A到B的距离d以及从A指向B的方位角θ（以正北方向为0度，顺时针为正方向）。我们可以使用三角函数来计算B点相对于A点的坐标。

在标准数学坐标系中，一般是以正东方向为0度，沿逆时针方向增加，所以需要将方位角转换为标准角。

$$
\theta^{'} = 90 - \theta
$$
$$
\Delta x = d \cdot \sin{\theta^{'}}
$$
$$
\Delta y = d \cdot \cos{\theta^{'}}
$$

- 上述公式中，角度皆为弧度信息
- 根据上述公式，计算出B相对A分别在x轴和y轴的增量$\Delta x$, $\Delta y$

然后将增量添加到起点A的坐标中，即可获得点B的相对坐标：

$$
x_2 = x_1 + \Delta x
$$
$$
y_2 = y_1 + \Delta y
$$

## 相对位坐标的Python实现

```Python
def calc_relative_2d_coordinates(distance, theta, origin=(0, 0), is_azimuth: bool = True):
    """
    根据点2相对于点1的距离和角度计算点B的相对坐标。

    :param distance: 距离
    :param theta: 角度(十进制度数)
    :param origin: 相对点1的坐标
    :param is_azimuth: True: 方位角（以正北为参考点，顺时针测量），False: 数学角度（以正东为参考点，逆时针测量）
    :return: (x2, y2): 点2相对于点1（origin）的相对坐标
    """
    # 将方位角转换为数学角度
    if is_azimuth:
        theta = (90 - theta) % 360

    # 将角度转换为弧度
    theta_rad = math.radians(theta)

    # 原点坐标
    x1, y1 = origin

    # 计算点B的坐标
    x2 = x1 + distance * math.cos(theta_rad)
    y2 = y1 + distance * math.sin(theta_rad)

    return x2, y2
```