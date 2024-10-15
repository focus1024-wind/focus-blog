---
title: Git文件.gitignore和.gitkeep用法解析
description: Git文件.gitignore和.gitkeep用法解析
date: 2023-03-01
slug: version_management/git/.gitignore_and_.gitkeep
image: 
categories:
    - VersionManagement
tags:
    - gitignore
    - gitkeep
    - VersionManagement
---

- `.gitignore`：告诉 `git` 管理工具，需要忽略那些不需要被跟踪的文件或目录
- `.gitkeep`：无意义，起到占位符的作用，`git` 在进行版本管理时，不会主动跟踪空目录，所以当项目中存在空目录的时候，`git` 会忽略空目录，如果需要将空目录上传到git仓库，一般需要在空目录下创建一个 `.gitkeep` 空文件起到占位的作用。在创建占位文件时，无意义的文件都是可以的，`.gitkeep` 文件只是一种约定俗成的规范名称而已。

# .gitignore

## .gitignore工作原理

在有的项目中，并不是所有的文件都需要上传到Git仓库进行版本管理的，这时我们就需要在项目中添加一个 `.gitignore` 文件告诉 `git` 那些文件或目录不需要被跟踪。

`.gitignore` 文件一般放在项目的根目录下对整个项目的文件进行匹配。也可以放到其他目录下，此时 `.gitignore` 文件以当前所在的路径为根路径进行模式匹配。

在创建 `.gitignore` 文件之前，假如已经把一些需要忽略文件添加到 `git` 版本管理中，那么即使我们在 `.gitignore` 文件中添加了这些文件的匹配规则，此时 `git` 不会匹配这些规则，对添加的文件进行过滤，因为此时这些文件已经被 `git` 进行管理了。所以在项目创建开始时就需要养成创建 `.gitignore` 的习惯。

## .gitignore常用匹配规则

- `#`：注释
- `\`：转义符，使具有特殊意义的字符标识为普通字符
- `!`：否定前缀，表示不忽略（跟踪）指定路径，可以将之前排除过滤的匹配文件再次包含进来。`git` 默认是对空目录不进行跟踪，可以使用 `!` 前缀用来管理跟踪空目录，不过跟推荐使用 `.gitkeep` 占位文件来跟踪管理空目录。
- `/`：目录分隔符
  - `/`位于开头或中间：匹配相对于 `.gitignore`的相对路径
    - `/folder1/file1`：匹配相对于 `.gitignore` 所在路径根目录下单的 `/folder1/file1` 文件
    - `subfolder1/file1`：匹配相对于 `.gitignore` 所在路径的 `subfolder1/file1` 文件，无论是根目录还是某个子目录下的 `subfolder1/file1`。
  - `/`位于结尾：只匹配目录，否则同时匹配目录和文件。
- `*`：通配多个字符，只能匹配到指定路径下的目录或文件，不能匹配子目录的内容。
  - 如 `/folder1/*`：
    - 仅能匹配到 `/folder1/file1`...，`/folder1/subfolder1/`...等文件
    - 不能匹配到 `/folder1/subfolder1/file1`...等文件
- `**`：特殊匹配符，表示匹配所有目录：
  - `**/subfolder1`：匹配任意路径下的文件或目录 `subfolder1`，类似于 `subfolder1`
  - `folder1/**/file1`：匹配 `folder1/file1`, `folder1/subfolder1/file1`, `folder1/subfolder1/subsubfolder1/file1`等文件或目录
  - `/flolder1/**`：匹配 `/flolder1` 路径下所有文件和目录。
- `?`：通配单个字符匹配除了`/`之外的任意单个字符
- `[]`：范围匹配，匹配`[]` 指定范围内的任意单个字符

## 本地 .gitignore 规则

远程代码仓库的 `.gitignore` 文件一般匹配项目中需要过滤的文件。但在实际开发中，项目组的各个成员有可能使用的IDE、编译器，开发环境等的不同，可能会导致每一个人都可能在自己本地需要一套额外的匹配过滤规则，但这套匹配过滤规则是与自己本地绑定的，而不应该上传到远程仓库。

### 本地代码库 .gitignore 规则

可以将自己本地的，不需要上传到仓库的 `.gitignore` 匹配规则写入在 `.git/info/exclude` 文件中

### 全局 .gitignore 规则

全局 `.gitignore` 对本地的所有项目都生效

#### 创建全局 .gitignore 文件来进行全局 .gitignore 匹配

```shell
git config --global core.excludesfile {{file_path}}
```

一般情况下，推荐将全局 `.gitignore` 文件放在用户 `~` 目录下，也命名为 `.gitignore` 文件名，具体可根据实际需要进行修改。
添加全局匹配规则时，到指定文件添加匹配规则即可。

#### 修改 .gitconfig 来进行全局 .gitignore 匹配

`.gitconfig` 文件一般在用户 `~` 目录下，若无此文件，创建空文件即可。

`.gitconfig`中写入：

```shell
[core]
	excludesfile = {{file_path}}
```

通过修改配置文件配置全局 .gitignore 规则，不会自动创建指定文件，需要在指定路径下创建对应的文件。

## 那些文件需要被过滤

- 日志文件
- 缓存或临时文件
- 具有敏感信息的文件，如密码，密钥，身份信息等
- 编译文件，如 `.class`， `.o` 等
- 依赖目录，如 `/venv`, `/node_modules` 等
- 编译输出文件，如 `/dist`, `/public` 等
- 系统文件，如 `.DS_Store` 等
- 项目目录下的压缩打包文件，如 `.zip`, `.tar.gz` 等
- IDE或文本编译器的配置文件，如 `.idea/`, `__pycache__` 等
- 大量数据集

有的人在使用 `git` 时，会将项目打包文件或项目依赖的数据集等大文件放在 `git` 中来进行版本管理，这样做是不对的，在使用 `git` 时，应满足：
- 禁止push提交大文件
- 少量多次commit

# .gitkeep

`git` 版本管理一般不会跟踪空目录，所以在上传到远程仓库时，会发现所有的空目录都不存在。但有的空目录是保持目录结构和进行未来扩展的必要目录，所以也需要将这些空目录上传到远程仓库中。

根据 `.gitignore` 中的匹配规则可知，我们可以使用 `!` 来指定相应的目录进行跟踪来上传远程目录，但在 `.gitignore` 中来指定空目录跟踪，既不方便也有可能造成歧义。所以不推荐使用本方法。

我们可以在空目录下创建空白的占位文件，这样该目录就不再是空目录的，`git` 也会进行相应的跟踪管理。理论上，无意义的占位文件都是可以的，开发人员一般使用 `.gitkeep` 作为空目录的占位文件，仅仅是一种约定俗成的规范。在实际项目中，可以根据项目组的要求使用指定的占位文件，若无要求，推荐使用 `.gitkeep` 保持良好的规范。