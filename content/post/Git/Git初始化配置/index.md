---
title: Git初始化配置
description: Git初始化配置
date: 2022-12-01
slug: version_management/git/init
image: 
categories:
    - VersionManagement
tags:
    - GitConfig
    - VersionManagement
---

# Git配置

Git 提供了一个叫做 git config 的工具，专门用来配置或读取相应的工作环境变量。
这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量具体配置在以下三个不同的地方：
- `git/etc/gitconfig` 文件：系统中对所有用户都普遍适用的配置。若使用 `git config` 时用 `--system` 选项，读写的就是这个文件。
- `~/.gitconfig` 文件：用户目录下的配置文件只适用于该用户。若使用 `git config` 时用 `--global` 选项，读写的就是这个文件。
- 当前项目的 Git 目录中的配置文件（也就是工作目录中的 `.git/config` 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 `.git/config` 里的配置会覆盖 `/etc/gitconfig` 中的同名变量。

# 配置用户信息

配置个人的用户名称和电子邮件地址：

```shell
git config --global user.name "$userName$"
git config --global user.email "$userEmail$"
```

如果使用 `--global` 选项，那么就会更改用户主目录下的配置文件，以后所有的项目都会默认使用这里配置的用户信息。
如果要在某个特定的项目中使用其他名字或者邮箱，只要去掉 `--global` 选项重新配置即可，新的设定保存在当前项目的 `.git/config` 文件里。

# 查看配置信息

使用 `git config --list` 命令可以查看当前用户的个人配置信息，若看到重复变量名，则说明它们位于不同的配置文件中。

