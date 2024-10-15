---
title: 文件分割与合并
description: 文件分割与合并
date: 2024-01-11
slug: split_merge_file
image: 
categories:
    - Default
tags:
    - Split
---

在进行文件传输的过程中，因为网络和文件大小的限制。经常会遇到文件传输失败的情况。针对文件传输失败的情况。大文件由于其传输时间长，一旦传输失败，重新传输费时且不一定能保证再次传输成功。针对这种情况，可以考虑将文件分割成小文件的方式进行传输，减少因传输失败或传输大小限制导致的问题。

# split 文件分割
在Linux中，进行文件分割主要是通过`split`命令进行的操作
```shell
split <optional> <input> <prefix>
```
- -a suffix_length: 指定分割文件的后缀长度来形成文件名后缀，默认为2。
- -d: 指定后缀为数字而不是字母，默认字母
- -b byte_count[K|k|M|m|G|g]: 指定分割文件的字节数，根据字节数进行文件分割
- -l line_count: 根据行数拆分文件，每个文件line_count行
- -n chunk_count: 将文件拆分为chunk_count个文件，前n-1文件具有(input fize size / chunk_count)大小字节，最后一个文件包含剩余字节
- -p pattern: 输入行匹配到对应的pattern时，进行分割

**示例:**
如我们需要将一个大小为拆分为100MB的文件
```shell
split -b 10M example.tar.gz example.tar.gz.
```
执行以上命令将会生成形如以下方式命名的文件
```shell
example.tar.gz.aa
example.tar.gz.ab
...
example.tar.gz.ba
example.tar.gz.bb
...
```

# 文件合并
将分割的文件上传到指定路径后，针对一些特殊的文件，我们需要将其合并后才能正常读取文件内容。文件的读取可以通过cat读取文件内容将输出量覆写到指定文件的方式进行合并。

**如，针对上文将文件分割为example.tar.gz.xx内容的情况下，可以通过如下命令进行合并:**
```shell
cat example.tar.gz.* > example.tar.gz
```
