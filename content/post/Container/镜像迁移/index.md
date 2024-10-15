---
title: 镜像迁移
description: 导出打包镜像、迁移镜像、导入镜像、修改镜像标签、推送镜像
date: 2023-12-13
slug: container/image_migrate
image: 
categories:
    - Container
    - Docker
tags:
    - Container
    - Docker
    - Tool
    - Image
---

在日常的服务部署开发中，我们有时需要迁移环境，将服务器上的私有镜像从一个服务器迁移到另一个服务器中。在以微服务为架构的项目中，我们的一个项目可能存在大量的镜像，对每一个镜像单独进行导出打包迁移即重复又麻烦，针对这种情况，我们可以通过开发脚本来实现快速的迁移部署，减少重复工作。

# 镜像迁移流程

- 导出镜像为打包文件
- 迁移镜像到其他服务器
- 导入镜像
- [可选]根据需要修改镜像标签
- [可选]推送镜像到镜像中心

# Python脚本处理

## 获取镜像列表

### table格式显示镜像列表

在 `bash`环境中，我们一般通过 `[docker] image ls`命令会以 `table`格式显示服务器中的镜像。

![[docker] image ls](image_ls.png)

### json格式显示镜像列表

以 `table`格式显示的镜像虽然直观，但是脚本以流的方式读取内容为 `str`类型数据，`str`类型针对 `table`格式转化出的数据处理相对比较麻烦。所以针对镜像列表的获取，我们可以采用 `json`格式进行读取。

![[docker] image ls --help](image_ls_help.png)

调用 `docker image ls --help`命令可知，docker 输出镜像列表支持通过 `--format`参数指定输出格式内容。

#### json格式显示镜像列表

调用 `docker image ls --format json`以json格式显示镜像列表

![[docker] image ls --format json](image_ls_json.png)

#### 兼容旧版Docker显示json格式镜像列表

在旧版的docker中，没有默认的json模板，此时调用 `docker image ls --format json`命令将无法正确显示镜像列表，阅读 `docker image ls --help`的输出和[docker go format官网文档](https://docs.docker.com/go/formatting/)。可以看到，docker是通过go template进行的格式化输出，针对这种情况，可以在输出时通过手动设置json模板 `{{json .}}`来显示json格式内容。

调用 `docker image ls --format "{{json .}}"`以json格式显示镜像列表

![[docker] image ls --format "{{json .}}"](image_ls_json_template.png)

注意 `--format string`这里，以json模板进行输出时，`""`是可选项，但是在设置模板时，`""`是必选项，否则会导致输出错误。

## Python获取shell输入流

在 `bash`环境中，我们一般通过管道 `|`获取上一步操作的输入流进行操作。在Python中，我们可以通过 `os`或者 `subprocess`库来执行shell命令。

### subprocess获取输入流

在Python的subprocess依赖库中，可以通过 `subprocess.getoutput(cmd)`命令来获取输入流内容，将输入流转化为 `str`类型数据。

### os获取输入流

在Python的os依赖库中，可以通过 `os.popen(cmd).read()`命令来获取输入流内容，将输入流转化为 `str`类型数据。

**推荐使用os来获取输入流，具有更好的兼容性，即使在Python2版本上也支持使用**

## str转jsonl

以json格式输出的镜像列表，是每个镜像都以json格式进行输出，整体是以jsonl的格式，所以不可以通过 `json.load()`来解析转化json数据，针对该情况，可以编写函数来实现将输入流转化为列表进行处理。

```python
def str2jsonl(string):
    """
    str类型数据转换为jsonl(列表)
    """
    jsonl = []
    lines = string.split('\n')
    for line in lines:
        if line.strip() != '':
            jsonl.append(json.loads(line))
    return jsonl
```

## Python执行shell命令

在python中，可以通过 `os.system(cmd)`命令来执行shell命令。

# 导出镜像为打包文件

导出镜像并打包的命令为: `docker save ${image_id} | gzip -c > ${filename}.tar.gz`

```python
def save_images():
    """
    导出镜像为.tar.gz文件到本地
    """
    # 执行shell命令，获取输入流
    images = os.popen('docker image ls --format "{{json .}}"').read()

    images = str2jsonl(images)

    for image in images:
        tag = ""
        for _tag in image['Tag'].split('.'):
            tag = tag + "_" + _tag
        # 导出镜像 | 将镜像打包为tar.gz文件
        os.system(f"docker save {image['Repository']}:{image['Tag']} | gzip -c > {image['Repository'].split('/')[-1]}_{tag}.tar.gz")
```

# 迁移镜像（网络互通）

导出镜像为 `.tar.gz`后，可以根据网络环境通过传输文件的方式将镜像传输到其他服务器中。

但是在网络互通的环境中，可以直接通过Python启动文件服务器的方式快速传输：

python3：`python3 -m http.server 8000`

python2: `python -m SimpleHTTPServer 8080`

在导出镜像服务器通过python命令快速启动一个文件服务器，其他服务器即可通过 `IP:Port`的方式访问获取文件。

**在传输完文件后，尽快将文件服务器关闭，避免网络内其他人无授权访问主机内容。**

# 导入镜像

导入镜像命令为: `docker load -i ${filename}`

```python
def str2list(string):
    return string.split('\n')


def load_images():
    """
    导入镜像
    """
    # 执行shell命令，获取输入流
    images = os.popen('ls').read()

    images = str2list(images)

    for image in images:
        if '' != image:
            os.system(f"docker load -i {image}")
```

# [可选]修改镜像标签

在进行服务迁移时，我们可能根据需要，需要修改镜像的标签，以便后续将镜像推送到指定镜像中心。一般这种情况下，修改镜像标签主要是修改镜像中心前缀和项目名，修改相对固定，可以通过脚本来实现统一修改。

修改镜像标签命令: `docker tag ${old_image_tag} ${new_image_tag}`

```python
def fix_tags(image_center: str, project_name: str):
    # 执行shell命令，获取输入流
    images = os.popen('docker image ls --format "{{json .}}"').read()

    images = str2jsonl(images)

    for image in images:
        if len(image['Repository'].split("/")) >= 2:
            repository = f"{image['Repository'].split('/')[-2]}/{image['Repository'].split('/')[-1]}"
        else:
            repository = image['Repository'].split("/")[-1]
        os.system(f"docker tag {image['Repository']}:{image['Tag']}\t\t{image_center}/{project_name}/{repository}:{image['Tag']}")
```

# [可选]推送镜像到镜像中心

登陆镜像中心命令: `docker login -u ${username} ${image_center}`

推送镜像标签命令: `docker push ${image}`

Docker会根据镜像名前缀推送到指定镜像中心。

```python
def push_images():
    """
    推送镜像到镜像中心
    """
    # 执行shell命令，获取输入流
    images = os.popen('docker image ls --format "{{json .}}"').read()
   
    images = str2jsonl(images)

    for image in images:
        os.system(f"docker push -i {image['Repository']}:{image['Tag']}")
```
