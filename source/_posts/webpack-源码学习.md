---
title: webpack 源码学习
date: 2022-04-17 10:32:41
tags:
---

webpack 作为最成熟, 使用最广的打包工具, 学习 webpack 的源码

<!-- more -->

# webpack CLI

常用 webpack 的方式是使用命令行例如 `npx webpack` , 但是目前 webpack-cli 是需要单独安装的, 再由 webpack 来进行调用

webpack 的命令行文件也比较简单

```Typescript
#!/usr/bin/env node

/**
 * @param {string} command process to run
 * @param {string[]} args command line arguments
 * @returns {Promise<void>} promise
 */
const runCommand = (command, args) => {
  //... 使用 child_process 来执行命令
};

/**
 * @param {string} packageName name of the package
 * @returns {boolean} is the package installed?
 */
const isInstalled = packageName => {
    // 处理使用 npx webpack, 和支持 pnp 功能的包管理器, 如果包管理器支持pnp那么会自己解决依赖问题, 而不能通过下面简单的查询是否在目录内来判断是否安装对应包
    // pnp see: https://yarnpkg.com/features/pnp
	if (process.versions.pnp) {
		return true;
	}
    // ... 在 node_modules 中循环是否有对应文件夹
	return false;
};

/**
 * 这里直接执行 webpack-cli
 * @param {CliOption} cli options
 * @returns {void}
 */
const runCli = cli => {
	const path = require("path");
	const pkgPath = require.resolve(`${cli.package}/package.json`);
	// eslint-disable-next-line node/no-missing-require
	const pkg = require(pkgPath);
	// eslint-disable-next-line node/no-missing-require
	require(path.resolve(path.dirname(pkgPath), pkg.bin[cli.binName]));
};

/**
 * @typedef {Object} CliOption
 * @property {string} name display name
 * @property {string} package npm package name
 * @property {string} binName name of the executable file
 * @property {boolean} installed currently installed?
 * @property {string} url homepage
 */

/** @type {CliOption} */
const cli = {
	name: "webpack-cli",
	package: "webpack-cli",
	binName: "webpack-cli",
	installed: isInstalled("webpack-cli"),
	url: "https://github.com/webpack/webpack-cli"
};

if (!cli.installed) {
	//...

    // 提示语 在 webpack-cli 不存在的时候提示用户安装
	const notify =
		"CLI for webpack must be installed.\n" + `  ${cli.name} (${cli.url})\n`;

	console.error(notify);

	let packageManager;
    // 推测用户可能使用的包管理工具
	if (fs.existsSync(path.resolve(process.cwd(), "yarn.lock"))) {
		packageManager = "yarn";
	} else if (fs.existsSync(path.resolve(process.cwd(), "pnpm-lock.yaml"))) {
		packageManager = "pnpm";
	} else {
		packageManager = "npm";
	}

	const installOptions = [packageManager === "yarn" ? "add" : "install", "-D"];

	console.error(
		`We will use "${packageManager}" to install the CLI via "${packageManager} ${installOptions.join(
			" "
		)} ${cli.package}".`
	);

	const question = `Do you want to install 'webpack-cli' (yes/no): `;

	const questionInterface = readLine.createInterface({
		input: process.stdin,
		output: process.stderr
	});

	// In certain scenarios (e.g. when STDIN is not in terminal mode), the callback function will not be
	// executed. Setting the exit code here to ensure the script exits correctly in those cases. The callback
	// function is responsible for clearing the exit code if the user wishes to install webpack-cli.
	process.exitCode = 1;
	questionInterface.question(question, answer => {
		questionInterface.close();

		const normalizedAnswer = answer.toLowerCase().startsWith("y");

		if (!normalizedAnswer) {
			console.error(
				"You need to install 'webpack-cli' to use webpack via CLI.\n" +
					"You can also install the CLI manually."
			);

			return;
		}
		process.exitCode = 0;

		console.log(
			`Installing '${
				cli.package
			}' (running '${packageManager} ${installOptions.join(" ")} ${
				cli.package
			}')...`
		);

		runCommand(packageManager, installOptions.concat(cli.package))
			.then(() => {
				runCli(cli);
			})
			.catch(error => {
				console.error(error);
				process.exitCode = 1;
			});
	});
} else {
	runCli(cli);
}
```

可以从上面的文件看到, 实际上执行 webpack 之后, 会调用 webpack-cli 来具体执行

webpack-cli 的 bin

```JavaScript
#!/usr/bin/env node

"use strict";

const importLocal = require("import-local");
const runCLI = require("../lib/bootstrap");

if (!process.env.WEBPACK_CLI_SKIP_IMPORT_LOCAL) {
  // Prefer the local installation of `webpack-cli`

  // 这里使用 import-local 来优先加载本地安装的包
  if (importLocal(__filename)) {
    return;
  }
}

process.title = "webpack";

// 一般调用的都是 webpack build --config ... 什么的 这些参数会给

runCLI(process.argv);
```

bootstrap 内容也很简单

```JavaScript

import { IWebpackCLI } from "./types";

// eslint-disable-next-line @typescript-eslint/no-var-requires
const WebpackCLI = require("./webpack-cli");

const runCLI = async (args: Parameters<IWebpackCLI["run"]>[0]) => {
  // Create a new instance of the CLI object
  const cli: IWebpackCLI = new WebpackCLI();

  try {
    await cli.run(args);
  } catch (error) {
    cli.logger.error(error);
    process.exit(2);
  }
};

module.exports = runCLI;


```

可以从代码中看出, webpack-cli 被实例化之后调用了 run 方法, 如果抛出异常那么打印异常然后 exit 非零.

整个 webpack-cli class 有 2000 多行.

```javascript
class WebpackCLI implements IWebpackCLI {
  // 命令函参数颜色, 使用 colorette 包实现
  colors: WebpackCLIColors;
  createColors(useColors?: boolean): WebpackCLIColors;

  // logger 对象
  logger: WebpackCLILogger;
  getLogger(): WebpackCLILogger {
    return {
      error: (val) => console.error(`[webpack-cli] ${this.colors.red(util.format(val))}`),
      warn: (val) => console.warn(`[webpack-cli] ${this.colors.yellow(val)}`),
      info: (val) => console.info(`[webpack-cli] ${this.colors.cyan(val)}`),
      success: (val) => console.log(`[webpack-cli] ${this.colors.green(val)}`),
      log: (val) => console.log(`[webpack-cli] ${val}`),
      raw: (val) => console.log(val),
    };
  }




  isColorSupportChanged: boolean | undefined;

  // webpack
  webpack: typeof webpack;



  builtInOptionsCache: WebpackCLIBuiltInOption[] | undefined;

  // commander
  program: WebpackCLICommand;

  // 是否为多实例
  isMultipleCompiler(compiler: WebpackCompiler): compiler is MultiCompiler {
       return (compiler as MultiCompiler).compilers as unknown as boolean;
  }

  // 是否为 promise
  isPromise<T>(value: Promise<T>): value is Promise<T> {
       return typeof (value as unknown as Promise<T>).then === "function";
  }

  // 判断是否为 function
  isFunction(value: unknown): value is CallableFunction {
      return typeof value === "function";
  }


  toKebabCase(value: string) => string {
         return str.replace(/([a-z0-9])([A-Z])/g, "$1-$2").toLowerCase();
  }
  capitalizeFirstLetter() {
         if (typeof str !== "string") {
      return "";
    }

    return str.charAt(0).toUpperCase() + str.slice(1);
  }

  // 检查对应的包是否存在
  checkPackageExists(packageName: string): boolean {

      // 如果有 pnp (Plug'n'Play) 则不需要检查
    if (process.versions.pnp) {
      return true;
    }

    let dir = __dirname;

    do {
      try {
        if (fs.statSync(path.join(dir, "node_modules", packageName)).isDirectory()) {
          return true;
        }
      } catch (_error) {
        // Nothing
      }
    } while (dir !== (dir = path.dirname(dir)));

    return false;
  }

  // 通过调用的方式检查可用的包管理工具
  getAvailablePackageManagers(): PackageManager[] {
    const { sync } = require("cross-spawn");
    const installers: PackageManager[] = ["npm", "yarn", "pnpm"];
    const hasPackageManagerInstalled = (packageManager: PackageManager) => {
      try {
        sync(packageManager, ["--version"]);

        return packageManager;
      } catch (err) {
        return false;
      }
    };
    const availableInstallers = installers.filter((installer) =>
      hasPackageManagerInstalled(installer),
    );

    if (!availableInstallers.length) {
      this.logger.error("No package manager found.");

      process.exit(2);
    }

    return availableInstallers;
  }


  getDefaultPackageManager(): PackageManager | undefined {
      // ... 通过检查当前目录是否有锁文件, 来判断项目是用什么包管理工具
  }


  doInstall(packageName: string, options?: PackageInstallOptions): Promise<string> {
      // ...  通过询问用户是否要安装缺失的依赖, 如果 是 用户输入 yes 则调用上面获取到的默认包管理工具进行下载对应依赖 例如 npm install webpack -D
  }

  loadJSONFile<T = unknown>(path: Path, handleError: boolean): Promise<T> {
      // require json 文件
  }

  tryRequireThenImport<T = unknown>(module: ModuleName, handleError: boolean): Promise<T> {
      // 首先尝试使用 require 来加载文件, 如果加载失败那么使用另外一种方法
      // 通过 new Function('packagename', 'return import(packagename)') 的方式动态构造一个 import()
      // 增加这个的目的是为了可以使用 es module 作为配置或者其他

  }


  // 为  commander  增加参数或者方法
  makeCommand(
    commandOptions: WebpackCLIOptions,
    options: WebpackCLICommandOptions,
    action: CommandAction,
  ): Promise<WebpackCLICommand | undefined>;
  makeOption(command: WebpackCLICommand, option: WebpackCLIBuiltInOption): void;



  /**
   *
   * run 函数主要是解析用户输入参数, 运行指定参数所匹配的程序或者方法 ,和提供命令函参数帮助的函数
   *
  */
  run(
    args: Parameters<WebpackCLICommand["parseOptions"]>[0],
    parseOptions?: ParseOptions,
  ): Promise<void>; {

        // 第一部分处理命令行提示
    // 使用 https://www.npmjs.com/package/commander 来解析和提示用户输入的命令行参数

    const loadCommandByName = async (
      commandName: WebpackCLIExternalCommandInfo["name"],
      allowToInstall = false,
    ) => {
      const isBuildCommandUsed = isCommand(commandName, buildCommandOptions);
      const isWatchCommandUsed = isCommand(commandName, watchCommandOptions);

      if (isBuildCommandUsed || isWatchCommandUsed) {

          //  这里 makeCommand 会向 commander 添加对应指令, 比如这里就是 build 参数 和对应执行的函数
        await this.makeCommand(
          isBuildCommandUsed ? buildCommandOptions : watchCommandOptions,

          // 这里执行初始化
          async () => {
            this.webpack = await this.loadWebpack(); // 引入 webpack

            return isWatchCommandUsed
              ? this.getBuiltInOptions().filter((option) => option.name !== "watch")
              : this.getBuiltInOptions();
          },

          // 这里是 build 参数对应的命令, 也就是 run webpack
          async (entries, options) => {
            if (entries.length > 0) {
              options.entry = [...entries, ...(options.entry || [])];
            }

            await this.runWebpack(options, isWatchCommandUsed);
          },
        );
      }
      //...
  }



  getBuiltInOptions(): WebpackCLIBuiltInOption[]{
      //... build 的一些配置选项
  }


  loadWebpack(handleError?: boolean): Promise<typeof webpack>{
      //... 加载 webpack
  }
  loadConfig(options: Partial<WebpackDevServerOptions>): Promise<WebpackCLIConfig> {
      //... 加载 配置文件
  }

  buildConfig(
    config: WebpackCLIConfig,
    options: WebpackDevServerOptions,
  ): Promise<WebpackCLIConfig> {
      // 处理 build 参数
  }
  isValidationError(error: Error): error is WebpackError{
      // 判断错误
  }
  createCompiler(
    options: Partial<WebpackDevServerOptions>,
    callback?: Callback<[Error | undefined, WebpackCLIStats | undefined]>,
  ){
        if (typeof options.nodeEnv === "string") {
      process.env.NODE_ENV = options.nodeEnv;
    }
    //  处理配置文件
    let config = await this.loadConfig(options);
    // 处理build 相关参数
    config = await this.buildConfig(config, options);

    let compiler: WebpackCompiler;
    try {
      compiler = this.webpack(
        config.options as WebpackConfiguration,
        callback
          ? (error, stats) => {
              if (error && this.isValidationError(error)) {
                this.logger.error(error.message);
                process.exit(2);
              }

              callback(error, stats);
            }
          : callback,
      );
      // @ts-expect-error error type assertion
    } catch (error: Error) {
      if (this.isValidationError(error)) {
        this.logger.error(error.message);
      } else {
        this.logger.error(error);
      }

      process.exit(2);
    }

    // TODO webpack@4 return Watching and MultiWatching instead Compiler and MultiCompiler, remove this after drop webpack@4
    if (compiler && (compiler as WebpackV4Compiler).compiler) {
      compiler = (compiler as WebpackV4Compiler).compiler;
    }

    return compiler;


  }



  needWatchStdin(compiler: Compiler | MultiCompiler): boolea



  runWebpack(options: WebpackRunOptions, isWatchCommand: boolean){
          // eslint-disable-next-line prefer-const

    // webpack 返回值
    let compiler: Compiler | MultiCompiler;
    let createJsonStringifyStream: typeof stringifyStream;

    if (options.json) {
      const jsonExt = await this.tryRequireThenImport<JsonExt>("@discoveryjs/json-ext");

      createJsonStringifyStream = jsonExt.stringifyStream;
    }

    const callback = (error: Error | undefined, stats: WebpackCLIStats | undefined): void => {
      if (error) {
        this.logger.error(error);
        process.exit(2);
      }
      // ...
      //  输出错误 监听错误 退出等
      } // ...
    };

    //...
    compiler = await this.createCompiler(options as WebpackDevServerOptions, callback);

    if (!compiler) {
      return;
    }


    // 处理  watch 情况
    const isWatch = (compiler: WebpackCompiler): boolean =>
      Boolean(
        this.isMultipleCompiler(compiler)
          ? compiler.compilers.some((compiler) => compiler.options.watch)
          : compiler.options.watch,
      );

    if (isWatch(compiler) && this.needWatchStdin(compiler)) {
      process.stdin.on("end", () => {
        process.exit(0);
      });
      process.stdin.resume();
    }
  }

}
```
