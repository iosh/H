title: visual studio code
date: 2019-12-14 19:02:58
tags: vs code

---

<!-- more -->



## 安装与编译

1. 下载源码

   ```bash
   git clone https://github.com/microsoft/vscode
   ```

2. 安装环境

   ```bash
   cd vscode
   npm install --global --production windows-build-tools # 安装c++模块编译环境，会下载 Python2 和 visual studio 2017 tools 比较大比较慢
   yarn # 安装依赖
   
   # 如果在安装依赖途中发生错误，例如找不到 visual studio 2015 可以尝试 
   npm install --global --production windows-build-tools  --vs2015
   ```

3. 运行

   ```bash
   yarn watch # 编译ts
   
   # 运行 electron
   # windows
   .\scripts\code.bat
   
   # macOS and Linux
   ./scripts/code.sh 
   
   ```

