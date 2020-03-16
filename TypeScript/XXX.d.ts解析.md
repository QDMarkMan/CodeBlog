# XXX.d.ts解析

> TS（TypeScript）近两年增长趋势极为火爆，很多开发者都转向了TS的怀抱，。了解TS语法的的同时也必须知道如何编写高质量的.d.ts(TypeScript声明文件)，今天我们来聊一聊 .d.ts是干什么用的。

## `.d.ts`是什么

`d.ts`全称是`TypeScript Declaration File`，用来存放一些声明。大部分基于ts编写的库都会有这个文件。下面看一个模块定义文件简单示例

`module.d.ts`
```ts
// Type definitions formyLib

/*~ 这是模块模板文件。 使用的时候将其重命名为index.d.ts 并将其放在与模块同名的文件夹中。
 *~ 
 *~ 例如，如果您正在为“super-greeter”编写文件，文件应该是'super-greeter / index.d.ts'
 */

/*~ 如果此模块是在模块加载器环境外部加载时公开全局变量“myLib”的UMD模块 ，
 *~ 请在此处声明全局变量。 否则，删除此声明。
 */
export as namespace myLib;

/*~ 模块内方法的定义
 */
export function myMethod(a: string): string;
export function myOtherMethod(a: number): number;

/*~ 通过导入模块声明可用的类型 */
export interface someType {
    name: string;
    length: number;
    extras?: string[];
}

/*~ 使用const，let或var声明模块的属性 */
export const myField: number;

/*~ 
 *~  如果模块的虚线名称中有类型，属性或方法，请在“命名空间”中声明它们。
 */
export namespace subProp {
    /*~ For example, given this definition, someone could write:
     *~   import { subProp } from 'yourModule';
     *~   subProp.foo();
     *~   or
     *~   import * as yourMod from 'yourModule';
     *~   yourMod.subProp.foo();
     */
    export function foo(): void
}
```

## `.d.ts`是干什么的

我们大家知道，`ts`是有类型区分的。那么`.d.ts`就是是类型的声明文件，这些类型声明帮助编译器识别类型，从而达到`TS`中对类型的控制。目前我感觉他的作用在我看来就是编辑器的智能提示。

## 如何编写一个高质量的声明文件

在开始编写之前，有一些`TS`相关的内容是不得不了解的。
- `declare`
一般的`.d.ts`中会大量出现这个关键字，`declare` 可以创建 `*.d.ts` 文件中的变量，**`declare` 只能作用于最外层**,基本上顶层的定义都需要使用 `declare`， `class` 也不例外：
```ts
declare const name: number;
declare function greet(greeting: string): void;
declare namespace tasaid {
    // 这里不能 declare
    interface blog {
        website: 'http://tasaid.com'
    } 
}
declare class Demo {
    demo: string
}
```
- `namespace`
为防止类型重复，使用 `namespace` 用于划分区域块，分离重复的类型，顶层的 `namespace` 需要 `declare` 输出到外部环境，子命名空间不需要 `declare`。在之前的版本中命名空间被称作“内部模块”。所以通过这个也能知道`namespace`的作用是什么：

`namespace`其实是一种组织代码的手段，以便于在记录它们类型的同时还不用担心与其它对象产生命名冲突。比如下面:
```ts
// 使用命名空间的验证器
namespace Validation {
    // 命名空间内代码不会被外部污染产生变量重复的情况
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    const lettersRegexp = /^[A-Za-z]+$/;
    const numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}
// 验证器对象
let validators: { [s: string]: Validation.StringValidator; } = {};
// 验证器属性
let validators["ZIP code"] = new Validation.ZipCodeValidator()
```

### 确定你要写的声明文件的类型

一般来讲，你组织声明文件的方式取决于库是如何被使用的。`.d.ts`针对不同的需求我们有不同的组织方式。
1. **全局库**

全局库是指能在全局命名空间下访问的，许多库都是简单的暴露出一个或多个全局变量。 比如，如果你使用过 `jQuery，$`变量可以被够简单的引用：

下面是一个全局库的模板
```ts
/*
 *~ 如果这个库是可调用的（例如可以作为myLib（3）调用）,在这里包括那些呼叫签名，否则，删除此部分
 *  定义顶级函数调用
 */
declare function myLib(a: string): string;
declare function myLib(a: number): number;

/*~ 如果您希望此库的名称是有效的类型名称，则可以在此处执行此操作。
 *~定义库的类型接口
 *~你可以这样来使用类型'var x: myLib'
 */
interface myLib {
    name: string;
    length: number;
    extras?: string[];
}

/*~ 如果您的库具有在全局变量上公开的属性，就可以把他们包裹在命名空间,
 *~ 接口和类型也应该放在这里
 */
declare namespace myLib {
    // 变量（可以对库中的修改）
    let timeout: number;
    // 常量
    const version: string;
    // class 类
    class Cat {
        constructor(n: number);
        //~ 只读
        readonly age: number;
        //~ 方法
        purr(): void;
    }
    // 接口
    interface CatSettings {
        weight: number;
        name: string;
        tailLength?: number;
    }
    //类型
    type VetID = string | number;
    // 函数
    function checkCat(c: Cat, s?: VetID);
}
```
2. **模块化库**

一些库只能工作在模块加载器的环境下。 比如，像 `express`只能在`Node.js`里工作所以必须使用`CommonJS`的`require`函数加载。模块化的库一般很少对`window`或`global`进行赋值。这个一般用的不是特别的多。这种库一般不暴露全局变量，需要通过特定加载机制（如`require/define/import`）引用的模块形式的类库

3. `UMD(plugin)`

`UMD`的库会检查是当前模块加载器环境，至于`UMD`是什么，可以看[这篇文章](https://github.com/QDMarkMan/CodeBlog/blob/master/Javascript/JavaScript%E4%B8%AD%E7%9A%84%E6%A8%A1%E5%9D%97.md),大多数流行的库比如 `Moment.js,lodash`现在都能够被当成`UMD`包, `UMD`的写法其实全局库基本上是一样的。

所以我们各种类库分为3类(这里非原创，在公众号看的我忘记是哪个了)
- `global`：暴露出全局变量的类库
- `module`：不暴露全局变量，需要通过特定加载机制（如require/define/import）引用的模块形式的类库
- `plugin`：会影响其它类库功能的类库（当然，也可能会影响原声明，比如添个新`API`）

### TS给的模板

TS的[官网](https://www.tslang.cn/docs/handbook/declaration-files/templates.html)给出了7种官方模板文件

- `global-modifying-module.d.ts`: 全局修改模块文件
- `global-plugin.d.ts`：适用于module plugin类库
- `global.d.ts`： 适用于全局库
- `module-class.d.ts`：暴露一个`class`的类库
- `module-function.d.ts`： 暴露一个`function`的类库
- `module-plugin.d.ts`：适用于module plugin类库
- `module.d.ts`：适用于模块库

### 具体语法

这里来看一下具体的语法是什么。

- **全局变量**
```ts
// const/let 都可以用在这里
declare var foo: number;
```
- **全局函数**
```ts
declare function demo(a: string): void;
```
- **全局对象**
```ts
// declare namespace声明了一个对象myLib
declare namespace myLib {
    function a(s: string): string;
    let s: number;
}
```
- **函数重载**
我感觉这个用的好像不是很多
```ts
// 根据不同的入参执行不同的操作
declare function getWidget(n: number): Widget;
declare function getWidget(s: string): Widget[];
```
- **接口**
接口基本上算是比较常用的可复用类型了
```ts
interface MyInterface {
  a: string;
  b?: number;
  c?: string;
}
declare function demo(demo: MyInterface): void;
```
- **类型别名**
`type`也是一种可复用类型。
```ts
//　生命一种类型
type TypeDemo = string | (() => string) | MyInterface;
declare function typeDemo(g: TypeDemo): void;
```
- **类型“模块”**
类似于`namespace`能够组织代码模块（把一组相关代码放在一起），`declare namespace`能用来组织类型“模块”（把一组相关类型声明放在一起）
```ts
declare namespace NameModule {
    interface Space1 {
        verbose?: boolean;
    }
    interface Space2 {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
```


- **类**
定义一个类然后直接抛出
```ts
declare class Demo {
    constructor(demo: string);

    demo: string;
    showDemo(): void;
}
```
- **命名空间**
有时候声明文件也存在全局声明冲突的问题，那么我们通过`namespace`解决
```ts
// 使用命名空间
namespace MyNameSpace {
    // 命名空间内代码不会被外部污染产生变量重复的情况
    export interface DemoInterface {
        demo(s: string): boolean;
    }
}
```

### 发布一个.d.ts

