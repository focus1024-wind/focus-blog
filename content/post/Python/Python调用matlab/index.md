---
title: Python调用matlab
description: Python调用matlab
date: 2022-10-20
slug: python/use_matlab
image: 
categories:
    - Python
tags:
    - Python
    - matlab
---

> 调用需求：
> 1.需要安装matlab。
> 2.matlab版本需与python版本匹配，[ MATLAB 产品(按版本)所支持的 Python 版本](https://ww2.mathworks.cn/help/matlab/matlab_external/install-the-matlab-engine-for-python.html)。

# 安装调用库
在 "MatLabInstall\extern\engines\python"路径中，执行如下操作：
使用管理员权限打开命令行界面，执行如下命令：
```shell
python setup.py install
```
# python调用matlab
> 1.python文件需与matlab文件同级

## 导入包
```python
import matlab
import matlab.engine
```

1. matlab用于数据转换等工作
2. matlab.engine是用来调用函数的

所有使用matlab调用的数据，需要使用matlab(import matlab)进行数据转换，[matlab数据转换](https://ww2.mathworks.cn/help/matlab/matlab_external/matlab-arrays-as-python-variables.html)。
## 启动和停止matlab引擎
python在调用matlab函数时，需要启动一个matlab.engine，然后通过engine去启动matlab函数，[启动和停止用于 Python 的 MATLAB 引擎](https://ww2.mathworks.cn/help/matlab/matlab_external/start-the-matlab-engine-for-python.html)。

1. 启动引擎：
```python
eng = matlab.engine.start_matlab()
```

2. 停止引擎：
```python
eng.quit()
```
或
```python
eng.exit()
```
## 调用matlab函数
注意：调用matlab函数时，matlab文件必须和调用的python脚本文件处于相同的工作路径下，否则会因找不到文件而报错
使用启动的matlab引擎对象调用matlab对象名.文件名(文件中函数的参数)即可调用matlab文件。
示例：

1. matlab文件内容：
```matlab
% 文件名：count.m
function result = count(a, b)
result  = a + b;
end
```

2. python文件内容：
```python
import matlab
import matlab.engine

eng = matlab.engine.start_matlab()

result = eng.count(1, 2)
print(result)
print(type(result))

eng.quit()
```

3. python文件运行结果：
```python
3
<class 'int'>
```


Python调用matlab