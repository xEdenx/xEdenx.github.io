---
title: 使用GitBook发布技术文档
description: 使用 GitBook 编写、构建和发布技术文档。
publishDate: 2016-05-25 18:50:18
tags:
  - GitBook
  - 技术文档
  - tools
---

`GitBook`是一个用于发布个人书籍的平台，类似于国外著名的LeanPub. 其中一个很大的特点是它利用git作为版本管理和发布工具, 加上是在线形式，你可以很方便的进行作为的快速更新.
gitbook提供了一个简单的命令行工具gitbook用来编译和预览的书籍.

## 安装

可以直接通过npm安装gitbook到全局

```
npm install gitbook-cli -g
```

gitbook只提供了如下四个命令

```
$ gitbook -h
Usage: gitbook [options] [command]
Commands:
build [options] [source_dir] 编译指定目录，输出Web格式(_book文件夹中)
serve [options] [source_dir] 监听文件变化并编译指定目录，同时会创建一个服务器用于预览Web
pdf [options] [source_dir] 编译指定目录，输出PDF
epub [options] [source_dir] 编译指定目录，输出epub
mobi [options] [source_dir] 编译指定目录，输出mobi
init [source_dir]   通过SUMMARY.md生成作品目录
Options:
-h, --help     output usage information
-V, --version  output the version number
```

## 书写

- `README.md`
  REAME相当于书籍的前言部分, 可以忽略

- `cover_small.png` 和 `cover.png`
  书籍的封面图

- `SUMMARY.md`
  SUMMARY是最重要的一个部分, 它创建的是整书的索引, 你也可以通过gitbook init读取SUMMARY.md来生成目录结构. 格式如下

```
# Summary
* [版本信息](README.md)
* [简介](简介/简介.md)
* [流程介绍](流程介绍/流程介绍.md)
* [使用指南](使用指南/使用指南.md)
   * [集成SDK](使用指南/集成SDK.md)
   * [添加权限](使用指南/添加权限.md)
   * [调用方式](使用指南/调用方式.md)
* [示例Demo](示例Demo/示例Demo.md)
* [意见反馈](意见反馈/意见反馈.md)
```

接下来就是依次完成你每个章节的书写了, 你需要开启`gitbook serve .`来进行实时的web预览(服务器默认为`localhost:4000`)。
同时会生成预览html。

## 发布

gitbook的命令行工具不提供对发布操作的支持，你可以直接使用git发布，首先你需要添加gitbook的仓库作为你的一个远程库. 比如sdk的路径为

```
git remote add gitbook https://push.gitbook.io/tneciv/sdk.git
git push gitbook master
```

## NOTICE

Gitbook v3.x+ 生成的本地html会出现`gitbook.js:3 xmlhttprequest cannot load...`跨域问题，如需本地打开html，需要`gitbook build --gitbook=2.6.7`
