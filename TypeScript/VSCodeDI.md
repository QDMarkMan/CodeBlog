# VSCode 中的DI

## 写在前面

对于这篇文章的阅读，需要了解的一些前置知识可以帮助你更加顺畅的阅读本文。

- [基本的TypeScript知识](https://www.typescriptlang.org/)
- [什么是Decorators](https://github.com/tc39/proposal-decorators)
- [TS中的Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)
- [DI是什么](https://zh.wikipedia.org/wiki/%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5)
- [IOC是什么](https://zh.wikipedia.org/zh-hans/%E6%8E%A7%E5%88%B6%E5%8F%8D%E8%BD%AC)

## DI

> 在[软件工程](https://zh.wikipedia.org/wiki/軟件工程)中，**依赖注入**（dependency injection）的意思为，给予调用方它所需要的事物。 “依赖”是指可被方法调用的事物。依赖注入形式下，调用方不再直接使用“依赖”，取而代之是“注入” 。“注入”是指将“依赖”传递给调用方的过程。在“注入”之后，调用方才会调用该“依赖”[[1\]](https://zh.wikipedia.org/wiki/依赖注入#cite_note-1)。传递依赖给调用方，而不是让让调用方直接获得依赖，这个是该设计的根本需求。 - wikipedia

依赖注射是[控制反转](https://zh.wikipedia.org/wiki/控制反转)的最为常见的一种技术， 

## IoC

> **控制反转**（英语：Inversion of Control，缩写为**IoC**），是[面向对象编程](https://zh.wikipedia.org/wiki/面向对象编程)中的一种设计原则，可以用来减低计算机代码之间的[耦合度](https://zh.wikipedia.org/wiki/耦合度_(計算機科學))。其中最常见的方式叫做**依赖注入**（Dependency Injection，简称**DI**），还有一种方式叫“依赖查找”（Dependency Lookup）。- wikipedia

在我的理解里面： **控制反转IoC(Inversion of Control)是说创建对象的控制权进行转移，以前\**创建对象的主动权和创建时机是由自己把控的\**，而现在这种权力转移到第三方**。

## 首先来讲一讲`Decorators`



# 总结



## 参考文章

- [依赖注入和控制反转的理解](https://blog.csdn.net/bestone0213/article/details/47424255)