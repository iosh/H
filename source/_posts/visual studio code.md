title: visual studio code
date: 2019-12-16 19:02:58
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
   yarn # 安装依赖
   # 安装c++模块编译环境，会下载 Python2 和 visual studio 2015 tools 比较大比较慢
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

### UpdateService 实例化

在`app.ts` 创建 `updateChannel`

通过在 `createServices` 函数内注册

```typescript
services.set(IUpdateService, new SyncDescriptor(Win32UpdateService));
```

之后再 `openFirstWindow` 函数内

```typescript
// Register more Electron IPC services
const updateService = accessor.get(IUpdateService);
// accessor.get 在内部会调用 _getOrCreateServiceInstance 函数,从名称来看会获得实例,或者创建实例
// 被实例化的 class 使用了参数装饰器 在 class 上标注了需要的参数
// 然后通过 循环一个一个处理并 Cache 结果,然后实例化  services.set(IUpdateService, class) 里面的class

const updateChannel = new UpdateChannel(updateService);
// 实例化

// 注册通信事件
electronIpcServer.registerChannel("update", updateChannel);
```

UpdateChannel 内部听了两个方法 `listen` `call` 分别是订阅和通知, 让使用变得更加简单.

```typescript
export class UpdateChannel implements IServerChannel {

	// 这里传入的service就是上面实例化出来的与平台有关的实现了  abstract class AbstractUpdateService {} 类的方法
	constructor(private service: IUpdateService) { }

	listen(_: unknown, event: string): Event<any> {
		switch (event) {
			case 'onStateChange': return this.service. ;
		}

		throw new Error(`Event not found: ${event}`);
	}

	call(_: unknown, command: string, arg?: any): Promise<any> {
		switch (command) {
			case 'checkForUpdates': return this.service.checkForUpdates(arg);
			case 'downloadUpdate': return this.service.downloadUpdate();
			case 'applyUpdate': return this.service.applyUpdate();
			case 'quitAndInstall': return this.service.quitAndInstall();
			case '_getInitialState': return Promise.resolve(this.service.state);
			case 'isLatestVersion': return this.service.isLatestVersion();
		}

		throw new Error(`Call not found: ${command}`);
	}
}
```

UpdateChannel 类作为一个通用接口，在实例化的时候接受的参数，是具体执行方法。而具体的执行方法，会根据平台不同传入不同的参数。

### AbstractUpdateService

从名字上来看，这是一个 abstract 类，在 TS 中 abstract 类不允许直接 new 得到实例， 而是需要通过继承，这个函数定义 `IUpdateService` 接口所列出的方法，并定义了一些 abstract 方法需要子类实现。
这种方式抹平了平台差异，调用者无需关心细节

#### update.ts

`AbstractUpdateService` 类继承了 `IUpdateService` 接口，这个接口描述了 `AbstractUpdateService` 需要实现的方法，也是外部调用所调用的方法。

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
 * Donwloaded： 有一个就绪的更新在后台安装
 */
```

从上面的状态机运行上来看，更新在不同系统下行为也是不同的。

`IUpdateService` 是通过 `createDecorator` 函数来创建的一个函数

```typescript
/**
 * A *only* valid way to create a {{ServiceIdentifier}}.
 */
export function createDecorator<T>(serviceId: string): ServiceIdentifier<T> {
  if (_util.serviceIds.has(serviceId)) {
    return _util.serviceIds.get(serviceId)!;
  }

  const id = <any>function(target: Function, key: string, index: number): any {
    if (arguments.length !== 3) {
      throw new Error(
        "@IServiceName-decorator can only be used to decorate a parameter"
      );
    }
    storeServiceDependency(id, target, index, false);
  };

  id.toString = () => serviceId;

  _util.serviceIds.set(serviceId, id);
  return id;
}
```

从代码中看到,它会返回一个函数,而且重写了`toString`方法
从返回的这个方法的签名来看

```typescript
const id = <any>function (target: Function, key: string, index: number): any
```

第三个参数是 index,那么说明这是一个参数装饰器.用来装饰函数参数或者构造函数参数的.

`IUpdateService` 接口是

```typescript
export interface IUpdateService {
  _serviceBrand: undefined; // 这是一个 vs code的约定  https://github.com/microsoft/vscode/issues/79918 表明这个函数会使用装饰器函数。

  readonly onStateChange: Event<State>; // 事件通知
  readonly state: State; // 当前状态

  checkForUpdates(context: any): Promise<void>; // 检查更新事件
  downloadUpdate(): Promise<void>; // 下载更新
  applyUpdate(): Promise<void>; // 申请更新
  quitAndInstall(): Promise<void>; // 退出程序以及安装更新

  isLatestVersion(): Promise<boolean | undefined>; // 检查是否最新版本
}
```

#### checkForUpdates

```typescript
	async checkForUpdates(context: any): Promise<void> {
		// 日志系统，在构造函数中使用了函数参数装饰器
		// 表明了这个类需要的构造函数参数，在通过server.get 的时候会被收集和赋值
		this.logService.trace('update#checkForUpdates, state = ', this.state.type);
		// 判断当前状态是否为闲置的
		if (this.state.type !== StateType.Idle) return
		// 执行 doCheckForUpdates
		this.doCheckForUpdates(context);
	}
```

这里这个 `doCheckForUpdates` 是一个 abstract 函数即需要子类继承的

##### win doCheckForUpdates

1. 判断 url 是否存在，这个 url 是一个类似于:

```text
https://update.code.visualstudio.com/api/update/win32-x64-user/stable/26076a4de974ead31f97692a0d32f90d735645c0
```

由下面这个函数生成,可以看到由平台,版本,commit 组成.

```typescript
export function createUpdateURL(platform: string, quality: string): string {
  return `${product.updateUrl}/api/update/${platform}/${quality}/${product.commit}`;
}
```

请求这个 url,如果没有新版本就会返回一个 204 ,而且不包含 body,如果有新版本那么就会包含一个类似于下面这样的结果

```json
{
  "url": "https://vscode.cdn.azure.cn/stable/26076a4de974ead31f97692a0d32f90d735645c0/VSCodeSetup-ia32-1.41.1.exe",
  "name": "1.41.1",
  "version": "26076a4de974ead31f97692a0d32f90d735645c0",
  "productVersion": "1.41.1",
  "hash": "089b1c71b884ef42a74e0dc0a5eb107eaa19d608",
  "timestamp": 1576680705440,
  "sha256hash": "d72dfa1e4644e0bbe7ebe7243a2ac59f51b2d2ca261bb24936f2879ad1d6aac3",
  "supportsFastUpdate": true
}
```

2. 修改当前状态为的 `checking for updates`

```typescript
this.setState(State.CheckingForUpdates(context));
```

3. 请求上面那个地址
   ```typescript
   this.requestService.request({ url: this.url }, CancellationToken.None)
   ``

   1. 如果没有更新修改状态值为 `Idle` 然后结束
   2. 这里还会根据程序目录有没有 `unins000.exe` 这个程序来决定是否进行下载
   3. 如果没有 `unins000.exe` 标记状态为 `available for download`
   4. 如果有那么先清理系统中可能存在的临时文件,之后判断有本地有没有下载好的安装程序,有就终止下载,没有就下载安装包
   5. 下载完成之后会判断包是否支持快速更新,以及用户是否允许后台更新.如果都允许那么进行更新,并且会判断当前 product.target 的值是否为 user
   6. 如果是 user 就调用 `doApplyUpdate` 函数进行更新否则修改状态值为 `downloaded`

###### doApplyUpdate

这个函数是 win 下才有的
1. 判断状态不等于 `downloaded` 和 `downloaded`
2. 判断本地安装包存在
3. 然后调用 child_process.spawn 执行静默安装
4. 安装完毕之后将状态修改为 Idle 

##### linux doCheckForUpdates

1. 和 win 一样
2. 和 win 一样
3. 请求更新地址
   1. 和 win 一样
   2. 如果有更新就将状态标记为 `available for download`

##### darwin(macOS)

1. 修改状态为
   vs code 在 macOS 下直接使用 electron.autoUpdater.checkForUpdate
