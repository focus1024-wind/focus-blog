---
title: Git配置SSH公钥
description: Git配置SSH公钥
date: 2023-02-01
slug: version_management/git/ssh_key
image: 
categories:
    - VersionManagement
tags:
    - GitConfig
    - SSH
    - VersionManagement
---

# 生成添加ssh公匙

- 在GitBash界面按照如下命令生成公匙

```shell
ssh-keygen -t rsa -C "xxxxx@xxxxx.com"
```

> - `-t`： Type，指定要创建密钥的类型
> 	- `rsa`：通过rsa加密算法生成的密钥
>	- `dsa`：通过dsa加密算法生成的密钥
>	- `ecdsa`：通过带椭圆曲线的dsa加密算法生成的密钥
> - `-C`：Commit，更改密钥文件的注释


> 注意:
> 1. 这里的 xxxxx@xxxxx.com 只是生成的 sshkey 的注释名称，并不约束或要求具体命名为某个邮箱。
> 2. 现网的大部分教程均讲解的使用邮箱生成，其一开始的初衷仅仅是为了便于辨识所以使用了邮箱。
> 3. 根据项目组规范要求，使用个人邮箱命名sshkey的名称。

- 按照提示完成三次回车，即可生成 ssh key。
- 通过查看 ~/.ssh/id_rsa.pub文件内容，获取到你的 public key。
> ~/：当前用户文件夹
> - Linux：/home/{userName}
> - Windows：C:\Users\{userName}
- 复制id_rsa.pub文件中中生成的sshkey
- 在Git仓库的[ssh公匙]页面添加sshkey
- 将复制的sshkey添加到公匙输入框中，设置标题，确定即可添加公匙

# 验证ssh公匙

- 打开 shell 界面
- 输入`ssh -T git@gitee.com`

> 首次使用需要确认并添加主机到本机SSH可信列表。若返回 Hi XXX! You've successfully authenticated, but Gitee.com does not provide shell access. 内容，则证明添加成功。
