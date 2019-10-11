# 阅读vscode源码所得

工作原因最近要阅读`vscode`的源码。这里做个记录

## 写在前面

**现在不建议阅读，我自己都没有读利索**

## 文件结构

## 骚操作

- 数组复制方式
  ```js
  const configurations = configRegistry.getConfigurations().slice(); // 返回一个新的数组
  ```

- `document.documentElement!`

  后面的感叹号是非null和非undefined的类型断言
