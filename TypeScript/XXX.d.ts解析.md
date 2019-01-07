# XXX.d.ts解析

> TS（TypeScript）近两年增长趋势极为火爆，很多开发者都转向了TS的怀抱，。了解TS语法的的同时也必须知道如何编写高质量的.d.ts(TypeScript声明文件)，今天我们来聊一聊 .d.ts是干什么用的。

## `.d.ts`是什么

`d.ts`全称是`TypeScript Declaration File`，用来存放一些声明。大部分基于ts编写的库都会有这个文件。下面看一个模块定义文件简单示例

`module.d.ts`
```ts
// Type definitions formyLib

/*~ 这是模块模板文件。 您应该将其重命名为index.d.ts 并将其放在与模块同名的文件夹中。
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

## 如何编写一个高质量的生命文件

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

###