---
title: 微信小程序Vant组件配置及使用
description: 微信小程序Vant组件配置及使用
date: 2023-07-05
slug: wechat_miniprogram/ui_with_vant
image: 
categories:
    - MiniProgram
tags:
    - MiniProgram
---

> Vant Weapp 官网文档：[介绍 - Vant Weapp (gitee.io)](https://vant-contrib.gitee.io/vant-weapp/#/home)
Vant Weapp GitHub地址：[youzan/vant-weapp: 轻量、可靠的小程序 UI 组件库 (github.com)](https://github.com/youzan/vant-weapp)


**本教程使用下载代码方式引入vant组件**
## 1. 下载vant组件源码

1.  通过git下载vant源码  
```git
git clone https://github.com/youzan/vant-weapp.git
```

2.  将vant源码路径下的**dist**文件夹复制到微信小程序项目中 

---

## 2. 使用vant组件

1. 将**app.json**下的`"style": "v2"`去除，微信小程序在新版基础组件中强行加了许多样式，若不去除该内容，会造成组件部分样式混乱
2. 在引入vant组件时，只能根据需要，一个组件一个组件往配置项里面添加，无法一次将所有组件全部引入

### 2.1 全局引入

1. 在**app.json**下配置`usingComponents`配置项即可全局引入vant组件。
2. 引入方式：`"van-组件名": "组件所在路径/index"`

```json
"usingComponents": {
    "van-vutton": "/dist/button/index"
}
```


### 2.2 局部引入

1. 对于不太常用的组件，可使用局部引入的方式进行引入
2. 局部引入只需在**wxml**对应的**json**文件中配置`usingComponents`即可
3. 局部引入的方式于全局引入相同
