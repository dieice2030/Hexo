---
title: 使用OpenCV.js进行图像处理
date: 2018-05-22 22:32:13
categories: 伪·技术文章
tags:
---
本来是打算翻译官方文档中OpenCV.js部分，然而实际操作过程中编译没有成功，故水之。
<!--more-->

# 安装`Emscripten`
`Emscripten` 是一个LLVM-JavaScript便与你一起。我们用它来构建OpenCV.js。
安装Emscripten, 参考[Emscripten SDK指南]()。

```bash
git clone https://github.com/juj/emsdk.git
cd emsdk
emsdk install latest
emsdk activate latest
```

添加环境变量`EMSCRIPTEN`，值为`Emscripten`的路径

# 获得OpenCV源码
可以从官网下载`ZIP`，也可以直接`git clone`


# 使用源码构建OpenCV.js
需要安装`Python`和`CMake`
