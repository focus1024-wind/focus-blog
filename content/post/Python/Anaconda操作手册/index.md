---
title: Anaconda操作手册
description: Anaconda操作手册
date: 2023-07-05
slug: python/anaconda_operating_manual
image: 
categories:
    - Python
tags:
    - Python
    - Conda
---

# 1. Anaconda
[Anaconda](https://www.anaconda.com/) 是一个主要用于科学计算的开源的 [Python](https://www.python.org/) 发行版本，其中包含 conda，python 等多个科学包及其依赖项。Anaconda 提供了包管理和环境管理的功能，可以很方便的解决 python 的版本控制以及第三方依赖包问题。Anaconda 向使用者提供了 conda 工具，通过 conda 可以很方便的进行包管理和环境管理。

## 1.1 Anaconda下载
[Anaconda官网：https://www.anaconda.com/](https://www.anaconda.com/)
	[Anaconda官网下载地址：https://www.anaconda.com/products/individual#Downloads](https://www.anaconda.com/products/individual#Downloads)
	[Anaconda清华源下载地址：https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)
	Anaconda 支持 Windows，Linux，Mac，可以根据自己的需要下载自己需要的版本。
	注意，在安装 Anaconda 的时候，需要选择将 Anaconda 添加到 PATH 路径中。

## 1.2 Linux安装Anaconda

1. 下载Anaconda安装文件
2. 进入到下载文件目录，使用 `bash` 命令安装Anaconda
```bash
bash {{Anaconda.sh}}
```

3. 键入回车阅读Anaconda协议
4. 协议内容较多，可使用回车逐行读取，也可以使用空格直接下一页
5. 阅读完协议后，输入 yes 同意协议并进行下一步，输入 no 不同意协议并退出安装，默认为 no
6. Anaconda会以当前用户下的anaconda3文件夹作为默认安装位置(~/anaconda3) 
   1. 回车在默认位置进行安装
   2. `CTRL + C` 退出安装
   3. 直接输入路径，回车后则在输入的路径下进行安装 
      1. 安装位置的父目录不能存在，否则会报`ERROR: File or directory already exists`，如你安装在 `~/anaconda3` 这个路径的时候，需要保证 `~` 用户路径下没有 anaconda3 这个文件夹
7. 配置PATH路径：

### Linux 配置 Anaconda 环境变量

1. 正常操作
```bash
# 1.打开.bashrc/.zashrc配置文件(根据自己的终端，打开对应的配置文件即可，这里以.bashrc为例)
nano .bashrc

# 2.在文件中写入Anaconda环境配置
export PATH="{{Anaconda}}/bin:$PATH"

# 3.刷新配置文件
source .bashrc
```

2. 快捷操作
```bash
# 1.将 Anaconda 的 bin 目录添加到 PATH 路径中
# 通过 echo 命令 可以将配置直接写入到配置文件中，不再需要打开，进行打开写入保存退出等一系列操作
echo 'export PATH="{{Anaconda}}/bin:$PATH"' >> ~/.bashrc

# 2.更新 bashrc 以立即生效
source ~/.bashrc
```

**在写入配置时，一定要注意在上方加入注释信息，之后可以快速定位到相关配置。**
**检查是否安装成功：**
	安装 Anaconda 后，会自动安装 conda 工具，以及一个 python 的 base 环境，通过控制台输出 conda 以及 base 环境 的版本号，就能够检查 Anaconda 是否安装成功。
```bash
>> conda --version
<< conda 4.10.1

---

>> python --version
<< Python 3.8.8
# 在使用Linux版的Anaconda时，因为Linux系统安装时一般会默认自带python版本，所以直接使用 python -- version 命令显示的不一定是Anaconda的base环境中的版本号
# 使用 source activate 激活Anaconda base环境
```

# 1.3 Linux下操作Anaconda环境

1. 因为Linux环境一般都默认安装python，所以打开终端的环境不一定是Anaconda的base环境，使用 `source activate` 命令可以激活base环境
2. 在linux系统中，直接使用 `activate` 命令无法激活或切换环境，需使用`source activate {{envName}}` 激活或切换环境

## 1.4 配置 conda 镜像
直接使用 conda 命令新建虚拟环境速度较慢，所以需配置 conda 镜像
[清华镜像站 Anaconda镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)

1. 在用户路径(~)下新建 `.condarc` 文件
2. 将镜像站的 channels 写入 .condarc 文件中
```bash
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

3. 使用 `conda clean -i` 命令清除索引缓存，保证使用的为镜像站提高的索引

# 2 Conda
安装 Anaconda 后，系统会自带 conda 工具，使用 conda 能够更好的对 python 进行版本管理。

## 2.1 查看 conda 帮助
一般的命令行工具都会自带帮助工具，在使用 conda 工具时，可以通过查看 conda 帮助来获取帮助信息。
**获取 conda 帮助信息：**`conda -h/--help`
```bash
>> conda -h/--help
<< 
usage: conda-script.py [-h] [-V] command ...

conda is a tool for managing and deploying applications, environments and packages.

Options:
positional arguments:
  command
    clean        Remove unused packages and caches.
    ...

optional arguments:
  -h, --help     Show this help message and exit.
  ...

conda commands available from other packages:
  build
 	...
```

**获取具体命令的帮助信息：**`conda {{command}} -h/--help`
```bash
>> conda list -h/--help
<<
usage: conda-script.py list [-h] [-n ENVIRONMENT | -p PATH] [--json] [-v] [-q] [--show-channel-urls] [-c] [-f]
                            [--explicit] [--md5] [-e] [-r] [--no-pip]
                            [regex]

List linked packages in a conda environment.

Options:
positional arguments:
  regex                 List only packages matching this regular expression.

optional arguments:
  -h, --help            Show this help message and exit.
  ...

Target Environment Specification:
  -n ENVIRONMENT, --name ENVIRONMENT
                        Name of environment.
  ...
  
Examples:
List all packages in the current environment:
    conda list
		...
```

## 2.2 Conda 环境管理
conda 的环境管理允许我们同时安装若干个不同版本的python，并且我们能够根据项目的需要，选择不同的python版本。

### 2.2.1 创建 python 环境
```bash
# 该方法创建的是基础的python环境，不自带其他额外的科学工具包
conda create -n {{projectName}} python={{pythonVersion}} 

# 使用该方法创建环境，会额外安装 miniconda 中携带的科学工具包
conda create -n {{projectName}} python={{pythonVersion}} miniconda

# 使用该方法创建环境，会安装 anaconda 中的所有科学工具包
conda create -n {{projectName}} python={{pythonVersion}} anaconda
```

### 2.2.2 查看所有Anaconda环境
```bash
# 使用该命令，可以查看anaconda的所有环境
conda env list
```

### 2.2.3 删除Anaconda环境
```bash
conda remove -n {{projectName}}
```

### 2.2.4 更新conda
```bash
conda update conda
```
