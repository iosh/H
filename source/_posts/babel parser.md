---
title: babel parse
date: 2020-05-25 12:48:46
tags: babel
---

learn babel parse how to work

<!-- more -->

# 目标

学习 babel-parser 如何进行工作, 解决一个一个实际问题.

# 入口函数 parse

首先 babel 可以解析集中模块, 在 parse 函数的第二个参数可以进行指定代码所属模块, babel 默认代码为 `script` 即不带模块的方式解析.

## getParser

函数签名

```ts
function getParser(options: ?Options, input: string): Parser {
  // Parser 是一个 class
  let cls = Parser;
  if (options?.plugins) {
    // 插件验证函数
    validatePlugins(options.plugins);

    cls = getParserClass(options.plugins);
  }
  // 返回这个 class 的实例
  return new cls(options, input);
}
```

## Parser

Parser 是一个继承了很多 class 的 class, 它继承的每个 class 都负责 parse 的某个具体方面, 从最原始的 父级 class 开始看

### BaseParser

BaseParser class 比较简单,主要是存储一些信息

```ts
export default class BaseParser {
  // Properties set by constructor in index.js
  options: Options; // 这是 parser 函数的第二个参数
  inModule: boolean; // 是否为模块
  scope: ScopeHandler<*>; // 另一个 class 追踪当前作用域, 以检测重复变量
  classScope: ClassScopeHandler;
  prodParam: ProductionParameterHandler; // 一个有关 参数收集器的 class
  plugins: PluginsMap; // 插件 Map
  filename: ?string; //
  sawUnambiguousESM: boolean = false;
  ambiguousScriptDifferentAst: boolean = false;

  // Initialized by Tokenizer
  state: State; //  由 Tokenizer 初始化
  // input and length are not in state as they are constant and we do
  // not want to ever copy them, which happens if state gets cloned
  input: string;
  length: number;

  hasPlugin(name: string): boolean {
    // 判断插件是否存在
    return this.plugins.has(name);
  }

  getPluginOption(plugin: string, name: string) {
    // 获取插件 options
    // $FlowIssue
    if (this.hasPlugin(plugin)) return this.plugins.get(plugin)[name];
  }
}
```

### CommentsParser

主要是处理和解析`comment(注释的)` 基于 eslint 的 espree 而来的.

### LocationParser

这个 class 主要处理解析错误,从而抛出一个错误和错误位置

### Tokenizer

词法分析

### UtilParser

解析工具,一些解析相关的工具函数

### NodeUtils

node 和 处理 node 的工具函数

### LValParser

处理左值

### ExpressionParser

表达式解析

### StatementParser

语句解析, 组装成 progrm

### Parser

初始化入口

```text
                              Parser class
Parser+-->StatementParser+--> ExpressionParser+--> NodeUtils +----> UtilParser+--+
                                                                                 |
                              CommentsParser<-----+LocationParser<-+Tokenizer<---+

```

# parse

上方调用实例化完成 Parser class 之后调用

```ts
  parse(): File {
    let paramFlags = PARAM;
    if (this.hasPlugin("topLevelAwait") && this.inModule) {
      paramFlags |= PARAM_AWAIT;
    }
    // scope 是一个 stack , 进入一个 scope 就将当前的 scope 添加到 stack 内, 退出 scope 之后随之 pop
    this.scope.enter(SCOPE_PROGRAM);
    //  也是一个相同的 stack 进入添加 exit pop
    this.prodParam.enter(paramFlags);
    // file 和 program 都是调用函数生成了一个 Node
    const file = this.startNode();
    const program = this.startNode();
    this.nextToken();
    file.errors = null;
    this.parseTopLevel(file, program);
    file.errors = this.state.errors;
    return file;
  }
```

函数返回 `File` 对象, 也就是说,整个解析过程是在这个函数周期内完成的.

## 解析 const a = 10

```ts
parses.parse("const a = 10");
```

通过简单的例子来看 babel parse 过程.

## this.nextToken

this.nextToken 位于 Tokenizer 所以很明显就是进行词法分析的. 函数主要做如下工作:

1. 获得当前上下文, 并判断当前字符是否需要跳过(例如空格,tab 之类的不需要解析)
2. 获得当前字符串位置
3. 判断位置是否超出字符
4. 判断 context.overide 是否存在,如果存在就进行调用并将 this 当做参数传递
5. 调用 geTokenFromCode

```ts
this.getTokenFromCode(this.input.codePointAt(this.state.pos));
```

## this.geTokenFromCode

函数是一个大的 switch 语句判断,例如判断当前是否为左括号,右括号之类的, 这里我们第一个字符是 `const` 的 `c`, 并不是一个完整的关键字之类的需要继续读取更多信息

会调用 this.readWrod 函数根据 this.state.pos 位置信息继续向后读取直到读到 const 之后的空格, 确定了 `const` 是一个完整的部分

调用了 type.keywords 来获取 const 匹配的对象, 然后调用 `this.finishToken(type, word);`

之后更新 this.state 对象, 更新上下文

this.nextToken 执行完毕

## this.parseTopLevel

this.parseTopLevel 是 StatementParser 类下的.

parserTopLevel 调用了 this.parseBlockBody 函数, 别的是一些模块 export 出来的变量处理, 和最后的 File 对象组装.

## this.parseBlockBody

函数也很简单, 给 Node 赋值了 body 和 directives 空数组, 之后调用了 this.parseBlockOrModuleBlockBody


## this.parseBlockOrModuleBlockBody

