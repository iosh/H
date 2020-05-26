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

## this.finishToken

函数更新当前 state 内的位置信息, type value 之类的信息. 然后条件调用 updateContext,

## this.updateContext

updateContext 函数主要是判断 type 是否为关键字以及是否为需要单独处理的一些关键字.

this.nextToken 执行完毕

## this.parseTopLevel

this.parseTopLevel 是 StatementParser 类下的.

parserTopLevel 调用了 this.parseBlockBody 函数, 别的是一些模块 export 出来的变量处理, 和最后的 File 对象组装.

## this.parseBlockBody

函数也很简单, 给 Node 赋值了 body 和 directives 空数组, 之后调用了 this.parseBlockOrModuleBlockBody

## this.parseBlockOrModuleBlockBody

函数调用 this.parseStatement 解析语句,这里需要被解析的语句是 `const`, 这个函数还是个循环,会不断遍历到结尾.

## this.parseStatement

parseStatement 首先会判断是不是 `@` 开头,如果是那么就是 `Directive`, 是不是 `let`

如果不是,那么会 进行 进行一个 switch case 匹配, 这里需要匹配 `const` 就先来看 `const`

```ts
case _types2.types._const:
case _types2.types._var:
  kind = kind || this.state.value;

  if (context && kind !== "var") {
    this.raise(this.state.start, _location.Errors.UnexpectedLexicalDeclaration);
  }
  return this.parseVarStatement(node, kind);
```

## this.parseVarStatement

```ts

  parseVarStatement(node, kind) {
    this.next();
    this.parseVar(node, false, kind);
    this.semicolon();
    return this.finishNode(node, "VariableDeclaration");
  }
```

这里调用 this.next 这个函数调整了 state 的一些位置信息. 例如 lastTokenEnd, 之后调用了 this.nextToken 上面写了 this.nexToken, 之后调用 this.pareVar 来解析声明

## this.parseVar

```ts
  parseVar(node, isFor, kind) {
    const declarations = node.declarations = [];
    // 判断有没有 ts 插件, 因为 ts 的声明 有类型
    const isTypescript = this.hasPlugin("typescript");
    node.kind = kind;

    for (;;) {
      const decl = this.startNode();
      this.parseVarId(decl, kind);
      // 判断是不是 = 入股欧式调用 this.next 读取后面的值
      if (this.eat(_types2.types.eq)) {
        decl.init = this.parseMaybeAssign(isFor);
      } else {
        if (kind === "const" && !(this.match(_types2.types._in) || this.isContextual("of"))) {
          if (!isTypescript) {
            this.unexpected();
          }
        } else if (decl.id.type !== "Identifier" && !(isFor && (this.match(_types2.types._in) || this.isContextual("of")))) {
          this.raise(this.state.lastTokEnd, _location.Errors.DeclarationMissingInitializer, "Complex binding patterns");
        }

        decl.init = null;
      }

      declarations.push(this.finishNode(decl, "VariableDeclarator"));
      if (!this.eat(_types2.types.comma)) break;
    }

    return node;
  }
```

## parseVarId

```ts
  parseVarId(decl, kind) {
    decl.id = this.parseBindingAtom();
    this.checkLVal(decl.id, kind === "var" ? _scopeflags.BIND_VAR : _scopeflags.BIND_LEXICAL, undefined, "variable declaration", kind !== "var");
  }
```

可以看到这里调用了 this.parseBindingAtom , 内部有一些判断, 之后又调用了 this.next,主要是判断是否为 ts ,是否为 async 之类的.

## 解析 "a |> b"

解析 "a |> b", 相同的步骤就不再说了.看看不同之处, 这个管道操作符部署于正是规范,需要插件,所以 Parser 没有插件的时候回报错,主要看看他怎么解决这个问题的.

前面都一样 parseBlockOrModuleBlockBody 开始解析. 它在 parseStatementContent 函数的 大 switch 中匹配不到合适的关键字, 那么因为他可能是个解析式,所以调用 parseExpression 函数进行处理

## parseExpression

```ts
 parseExpression(noIn, refExpressionErrors) {
   // 获得位置信息
    const startPos = this.state.start;
    const startLoc = this.state.startLoc;
    // 猜测这也许是个 赋值
    const expr = this.parseMaybeAssign(noIn, refExpressionErrors);
     //....
    return expr;
  }
```

```ts
parseMaybeAssign(noIn, refExpressionErrors, afterLeftParse, refNeedsArrowPos) {
    // ... 省略
    // 猜测这也许是个分配符
    let left = this.parseMaybeConditional(noIn, refExpressionErrors, refNeedsArrowPos);

    // ...

    return left;
  }

```

```ts
 parseMaybeConditional(noIn, refExpressionErrors, refNeedsArrowPos) {
   // 取到位置
    // 当做操作符解析, |>
    const expr = this.parseExprOps(noIn, refExpressionErrors);
    //...
  }

```

```ts
  parseExprOps(noIn, refExpressionErrors) {
    //.. 尝试解析为一元运算符
    const expr = this.parseMaybeUnary(refExpressionErrors);
    // ...


   return this.parseExprOp(expr, startPos, startLoc, -1, noIn);

```

```ts
parseMaybeUnary(refExpressionErrors) {
    // 省略一些判断,主要是判断  await throw  之类的

    const startPos = this.state.start;
    const startLoc = this.state.startLoc;

    let expr = this.parseExprSubscripts(refExpressionErrors);
    return expr;
  }


```

```ts
parseExprAtom(refExpressionErrors) {
   parseExprAtom(refExpressionErrors) {
    debugger;
    if (this.state.type === _types.types.slash) this.readRegexp();
    const canBeArrow = this.state.potentialArrowAt === this.state.start;
    let node;

    switch (this.state.type) {
      // 省略一些匹配
      case _types.types.name:
        {
          node = this.startNode();
          const containsEsc = this.state.containsEsc;
          const id = this.parseIdentifier();

          if (!containsEsc && id.name === "async" && this.match(_types.types._function) && !this.canInsertSemicolon()) {
            const last = this.state.context.length - 1;

            if (this.state.context[last] !== _context.types.functionStatement) {
              throw new Error("Internal error");
            }

            this.state.context[last] = _context.types.functionExpression;
            this.next();
            return this.parseFunction(node, undefined, true);
          } else if (canBeArrow && !containsEsc && id.name === "async" && this.match(_types.types.name) && !this.canInsertSemicolon()) {
            const oldMaybeInArrowParameters = this.state.maybeInArrowParameters;
            const oldMaybeInAsyncArrowHead = this.state.maybeInAsyncArrowHead;
            const oldYieldPos = this.state.yieldPos;
            const oldAwaitPos = this.state.awaitPos;
            this.state.maybeInArrowParameters = true;
            this.state.maybeInAsyncArrowHead = true;
            this.state.yieldPos = -1;
            this.state.awaitPos = -1;
            const params = [this.parseIdentifier()];
            this.expect(_types.types.arrow);
            this.checkYieldAwaitInDefaultParams();
            this.state.maybeInArrowParameters = oldMaybeInArrowParameters;
            this.state.maybeInAsyncArrowHead = oldMaybeInAsyncArrowHead;
            this.state.yieldPos = oldYieldPos;
            this.state.awaitPos = oldAwaitPos;
            this.parseArrowExpression(node, params, true);
            return node;
          }

          if (canBeArrow && this.match(_types.types.arrow) && !this.canInsertSemicolon()) {
            this.next();
            this.parseArrowExpression(node, [id], false);
            return node;
          }
          // 这里都不匹配 , 返回了 一个 node 对象
          return id;
        }

      // 省略一些匹配
      // 最后匹配不到就抛出错误
      default:
        throw this.unexpected();
    }
  }
}

```

```ts
 parseExprSubscripts(refExpressionErrors) {
    const startPos = this.state.start;
    const startLoc = this.state.startLoc;
    const potentialArrowAt = this.state.potentialArrowAt;
    // atom 解析
    const expr = this.parseExprAtom(refExpressionErrors);

    if (expr.type === "ArrowFunctionExpression" && expr.start === potentialArrowAt) {
      return expr;
    }

    return this.parseSubscripts(expr, startPos, startLoc);
  }
```

```ts


  parseExprOp(left, leftStartPos, leftStartLoc, minPrec, noIn) {
    let prec = this.state.type.binop;

    if (prec != null && (!noIn || !this.match(_types.types._in))) {
      if (prec > minPrec) {
        const operator = this.state.value;
        // 判断操作符是否为 |>
        if (operator === "|>" && this.state.inFSharpPipelineDirectBody) {
          return left;
        }
       // 生成一个节点
        const node = this.startNodeAt(leftStartPos, leftStartLoc);
        node.left = left;
        node.operator = operator;

        if (operator === "**" && left.type === "UnaryExpression" && (this.options.createParenthesizedExpressions || !(left.extra && left.extra.parenthesized))) {
          this.raise(left.argument.start, _location.Errors.UnexpectedTokenUnaryExponentiation);
        }

        const op = this.state.type;
        const logical = op === _types.types.logicalOR || op === _types.types.logicalAND;
        const coalesce = op === _types.types.nullishCoalescing;

        if (op === _types.types.pipeline) {
          // 这里判断存在不存在处理 |> 的插件, 不存在就抛出错误, 到这里就截止了
          this.expectPlugin("pipelineOperator");
          this.state.inPipeline = true;
          this.checkPipelineAtInfixOperator(left, leftStartPos);
        } else if (coalesce) {
          prec = _types.types.logicalAND.binop;
        }
        //...
  }
```

