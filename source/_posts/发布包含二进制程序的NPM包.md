---
title: 发布包含二进制程序的 npm 包
date: 2024-09-08 19:01:09
tags: npm
---

介绍如何发布包含二进制可执行文件的 npm 包

<!-- more -->

通常情况下 npm 包都是 JavaScript 代码，但有时候我们需要发布包含二进制程序的 npm 包。越来越多的包通过使用其他语言编写然后编译为二进制文件进行发布，例如赫赫有名的 `esbuild` `biomejs` 他们本身是通过其他语言例如 Go 或者 Rust 编写的，然后将这些二进制文件发布到 npm 包中。

通常情况下二进制可执行文件都是无法跨平台的，所以需要为每个目标系统编译一份二进制可执行文件。例如 linux macos windows 系统不同，并且cpu架构也不同，例如 x86 或者 arm，这些都是无法进行通用的。

如果我们将这些二进制文件包含在一个包内，这会导致包的体积变得十分庞大，并且包含了非常多无用的二进制文件，这对某些网络的用户带来了巨大的挑战。

所以我们需要一种方案，这个方案可以让你在指定平台下载对应平台相匹配的二进制文件。从而实现按需下载，减少包体积。

目前比较常用的方法是为不同平台的二进制文件分别发一个包，例如 biomejs 。

Biome 是一个用于 Web 项目的高性能工具链，旨在为开发者提供维护项目的工具，它本身是使用 Rust 进行编写的，可以将它视为 Eslint 和 Prettier 的直接替代品。

用户使用 `npm install @biomejs/biome` 来安装 biome ，他会自动根据平台安装对应的二进制包， 可以通过 biome package.json 来查看 biome 一共提供了三个平台 8 种包来适配不同的平台和环境。

- @biomejs/cli-win32-x64
- @biomejs/cli-win32-arm64
- @biomejs/cli-darwin-x64
- @biomejs/cli-darwin-arm64
- @biomejs/cli-linux-x64
- @biomejs/cli-linux-arm64
- @biomejs/cli-linux-x64-musl
- @biomejs/cli-linux-arm64-musl

我们来了解一下实现细节。

# optionalDependencies

biome 将这些包作为依赖添加到了 package.json 的 optionalDependencies 中，可以查看这里来了解 [optionalDependencies](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#optionaldependencies) 的说明。

简而言之就是在 optionalDependencies 中列出的包，是可选的，根据一些标准可以选择不安装或者安装失败也并且不会报错。

那么标准是什么

# os 和 cpu

package.json 中有两个字段 `os` 和 `cpu` 可以查看[文档](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#os)

包管理器可以识别这些字段，并且当这些字段与执行环境相匹配的时候才会安装对应的包。例如：

```
{
    "name": "my-package",
    ...
    "os": ["linux"],
    "cpu": ["x64"]

}
```

上面这个包就指定了自己的 os 和 cpu 字段,它将自己声明为只能在 linux x64 ，那么包管理器也只有在 x64 的 Linux 系统才会安装这个包。

如此我们就可以为每个平台的包分别填写这些字段来让包管理器下载对应平台的包。

# 示例

假设我们要发布一个 simple-package， 并且分别为 windows 和 linux 以及 macos 系统提供对应的二进制文件。

那么我们一共要发四个包，分别是 `simple-package` `simple-package-win32-x64` `simple-package-linux-x64` `simple-package-darwin-arm`。

这个例子只列出 x64 的 linux windows 以及 arm 的 macos，当然你可以添加更多例如 arm 的linux windows 等。

他们的 package.json 应该是这样的

```json
{
  "name": "simple-package-win32-x64",
  "version": "1.0.0",
  "os": ["win32"],
  "cpu": ["x64"]
}
```

```json
{
  "name": "simple-package-linux-x64",
  "version": "1.0.0",
  "os": ["linux"],
  "cpu": ["x64"]
}
```

```json
{
  "name": "simple-package-darwin-arm",
  "version": "1.0.0",
  "os": ["darwin"],
  "cpu": ["arm64"]
}
```

文件夹内应该是这样的

```
simple-package-linux-x64 或者 simple-package-darwin-arm

├── package.json
└── simple-binary

simple-binary-win32-x64

├── package.json
└── simple-binary.exe

```

有一个 `package.json` 和 一个二进制文件。

同时要记得，需要为这些二进制文件赋予可执行权限, 如 `chmod +x simple-binary`

接下来就可以将这些包通过 npm publish 发布了。

在 simple-package 这个包中将上面的三个包添加到 `optionalDependencies` 中

```json
{
  "name": "simple-package",
  "version": "1.0.0",
  "optionalDependencies": {
    "simple-package-win32-x64": "1.0.0",
    "simple-package-linux-x64": "1.0.0",
    "simple-package-darwin-arm": "1.0.0"
  }
}
```

## postinstall 脚本

可选的我们可以为 simple-package 提供一个 postinstall 脚本， 可以在这个脚本中检查并提示用户，当前系统是否受到支持。

postinstall.js

```javascript
import fs from "node:fs";
import path from "node:path";

// platform 会告诉我们当前是什么平台， arch 会告诉我们当前按是什么架构
const { platform, arch } = process;

const ALL_SUPPORTED_PACKAGES = {
  linux: {
    x64: "simple-package-linux-x64",
  },
  darwin: {
    arm64: "simple-package-darwin-arm",
  },
  win32: {
    x64: "simple-package-win32-x64",
  },
};

if (!ALL_SUPPORTED_PACKAGES[platform]) {
  return console.waning(`当前系统 ${platform} 不受支持`);
}

if (!ALL_SUPPORTED_PACKAGES[platform][arch]) {
  return console.waning(`当前系统 ${platform} ${arch} 架构不受支持`);
}

const binaryPackage = ALL_SUPPORTED_PACKAGES[platform][arch];

try {
  require.resolve(
    `${binaryPackage}/${platform === "win32" ? "simple-binary.exe" : "simple-binary"}`
  );
} catch () {
  console.warning(`找不到二进制文件 ${binaryPackage}/${platform === "win32" ? "simple-binary.exe" : "simple-binary"} 请尝试重新安装依赖包`);
}
```

更新一下 package.json

```json
{
  "name": "simple-package",
  "version": "1.0.0",
  "scripts": {
    "postinstall": "node postinstall.js"
  },
  "optionalDependencies": {
    "simple-package-win32-x64": "1.0.0",
    "simple-package-linux-x64": "1.0.0",
    "simple-package-darwin-arm": "1.0.0"
  }
}
```

在 `simple-package` 中运行可执行文件可以使用 `child_process` 的 `spawn` 或者 `exec` 方法, 下面是一个例子

```javascript
import { spawn，exec } from "node:child_process";

const { platform, arch } = process;

const ALL_SUPPORTED_PACKAGES = {
  linux: {
    x64: "simple-package-linux-x64",
  },
  darwin: {
    arm64: "simple-package-darwin-arm",
  },
  win32: {
    x64: "simple-package-win32-x64",
  },
};


const binaryPackage = ALL_SUPPORTED_PACKAGES[platform][arch];

const child = spawn(
  `${binaryPackage}/${platform === "win32" ? "simple-binary.exe" : "simple-binary"}`
);

// 或者

// const child = exec(`${binaryPackage}/${platform === "win32" ? "simple-binary.exe" : "simple-binary"}`， (error, stdout, stderr) => {});
```

这两者的区别是 spawn 返回一个 Streams 而 exec 是回调函数，它内部有一个 buffer 用来存储输出的内容，简而言之，当你的程序有量大输出选择 spawn ，否则可以使用 exec

当然还有 `fork` `execFile` `execFileSync` `execSync` 等方法根据自己的选择进行使用。

