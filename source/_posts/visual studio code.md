title: visual studio code
date: 2019-12-14 19:02:58
tags: vs code

---

学习 visual studio code

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

## 目录结构

```txt
E:/vscode
├── azure-pipelines.yml  // ci
├── build // build stript
├── cglicenses.json //  licence
├── cgmanifest.json
├── CONTRIBUTING.md // 贡献说明
├── extensions // vs 内置扩展
├── gulpfile.js  // gulp 脚本
├── LICENSE.txt
├── node_modules
├── out // ts 编译输出
├── package.json
├── product.json
├── README.md
├── remote
├── resources // 一些特定平台的资源
├── scripts  // 运行electron的脚本 和 一些测试运行脚本
├── src  // 源代码
├── test // 测试
├── ThirdPartyNotices.txt
├── tsfmt.json
├── tslint.json
└── yarn.lock

```

## VS code 如何更新

通过搜索关键字，可以定位到 `src\vs\platform\update\electron-main\abstractUpdateService.ts` , 然后会发现这是一个 TS 的 `abstract` class 同目录下还有 `updateServeice.[平台名称].ts` 说明具体更新是根据环境不同而基于抽象方法的具体执行，之后可以找到 `updateIpc.ts` 文件，在通过搜索这个文件名称，就可以找到 Code Application 文件 `src\vs\code\electron-main\app.ts`.

```typescript
// src\vs\code\electron-main\app.ts
import { IUpdateService } from "vs/platform/update/common/update";
import { UpdateChannel } from "vs/platform/update/electron-main/updateIpc";
```

```typescript
// src\vs\code\electron-main\app.ts
// IUpdateService 方法初始化
private async createServices(machineId: string, trueMachineId: string | undefined, sharedProcess: SharedProcess, sharedProcessClient: Promise<Client<string>>): Promise<IInstantiationService> {
      	// 省略部分代码
    const services = new ServiceCollection();
		// 省略部分代码
		switch (process.platform) {
			case 'win32':
				services.set(IUpdateService, new SyncDescriptor(Win32UpdateService));
				break;

			case 'linux':
				if (process.env.SNAP && process.env.SNAP_REVISION) {
					services.set(IUpdateService, new SyncDescriptor(SnapUpdateService, [process.env.SNAP, process.env.SNAP_REVISION]));
				} else {
					services.set(IUpdateService, new SyncDescriptor(LinuxUpdateService));
				}
				break;

			case 'darwin':
				services.set(IUpdateService, new SyncDescriptor(DarwinUpdateService));
				break;
		}

	// 省略部分代码
	}
```

通过函数名称可以看到，这个函数被注册到 VS code 的服务中心， 并根据实际平台来注册了对应的方法。

```typescript
// src\vs\code\electron-main\app.ts
// UpdateChannel 方法初始化
private openFirstWindow(accessor: ServicesAccessor, electronIpcServer: ElectronIPCServer, sharedProcessClient: Promise<Client<string>>): ICodeWindow[] {

		// Register more Main IPC services
		const launchMainService = accessor.get(ILaunchMainService);
		const launchChannel = createChannelReceiver(launchMainService, { disableMarshalling: true });
		this.mainIpcServer.registerChannel('launch', launchChannel);

		// Register more Electron IPC services
		const updateService = accessor.get(IUpdateService);
		const updateChannel = new UpdateChannel(updateService);
		electronIpcServer.registerChannel('update', updateChannel);

		const issueService = accessor.get(IIssueService);
		const issueChannel = createChannelReceiver(issueService);
		electronIpcServer.registerChannel('issue', issueChannel);
		// 省略部分代码
	}
```

通过函数名可以得知这是在窗口首次建立的时候所运行的方法（有个好名字真好），函数内注册了一些 IPC （Inter-Process Communication， 进程通信 主进程和渲染进程通信）方法。

createServices 方法会在 openFirstWindow 方法之前被调用， openFirstWindow 方法内通过 accessor.get(IUpdateService) 获得了 createServices 内注册的方法，然后在通过 electronIpcServer 注册在 渲染进程

那么核心代码就是 `src\vs\platform\update` 这个模块了

### update/common

#### update.config.contribution.ts

此文件在 `src\vs\code\electron-main\main.ts` 中直接引用，通过源代码注释可以得知，讲一组对象写入了注册表。一组数据，例如更新模式，允许在后台更新 VS code.

#### update.ts

```javascript
//src\vs\platform\update\common\update.ts
/**
 * Updates are run as a state machine:
 * Updates 作为一个状态机运行:
 *      Uninitialized // 未初始化
 *           ↓
 *          Idle  // 空闲
 *          ↓  ↑
 *   Checking for Updates  →  Available for Download
 *   检查是否有更新             下载可用更新
 *         ↓
 *     Downloading  →   Ready
 *      下载中			  就绪
 *         ↓               ↑
 *     Downloaded   →  Updating
 *     下载完毕         更新
 *
 * Available: There is an update available for download (linux).
 * Available： 有一个可用更新的版本可供下载
 * Ready: Code will be updated as soon as it restarts (win32, darwin).
 * Ready  Code 将在重新启动时立即更新
 * Donwloaded: There is an update ready to be installed in the background (win32).
 *  Donwloaded： 有一个就绪的更新在后台安装
 */
```

从上面的状态机运行上来看，更新在不同系统下行为也是不同的。

`IUpdateService` 是通过 `createDecorator` 函数来创建的一个函数，`createDecorator` 是一个用来创建 VS Code 服务标识的函数，暂时理解`createDecorator` === `updateService` 字符串好了。

然后还创建了一个同名的接口，接下来在 `AbstractUpdateService`实现了这个接口。

#### AbstractUpdateService.ts

### UpdateService 实例化

在`app.ts` 创建 `updateChannel`

```typescript
const updateService = accessor.get(IUpdateService);
// get 方法 内部会调用 _getOrCreateServiceInstance 来实例化
```
