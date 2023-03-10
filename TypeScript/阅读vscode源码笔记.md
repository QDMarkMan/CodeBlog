
# 阅读vscode源码笔记

工作原因最近要阅读`vscode`的源码。这里做个记录

## 写在前面

> 在看源码时的一些笔记。
***

## 文件结构

## 一些小技巧

- 数组复制方式

  ```js
  const configurations = configRegistry.getConfigurations().slice(); // 返回一个新的数组
  ```

- `document.documentElement!`

  后面的感叹号**是非null和非undefined的类型断言**
