> RFC 全称 （Request for Comments），征求意见稿 [维基百科](https://en.wikipedia.org/wiki/Memorandum)。

### 整体结构 [](https://www.rectcircle.cn/series/software-project-management/rust-rfc/#%E6%95%B4%E4%BD%93%E7%BB%93%E6%9E%84)

- 头部元信息
- 摘要
- 动机
- 指南级别的解释
- 参考级别解释
- 缺点
- 理由和替代方案
- 现有技术
- 未解决的问题
- 未来的可能性

### 文件名 [](https://www.rectcircle.cn/series/software-project-management/rust-rfc/#%E6%96%87%E4%BB%B6%E5%90%8D)

`0000-feature-name.md`

### 头部元信息 [](https://www.rectcircle.cn/series/software-project-management/rust-rfc/#%E5%A4%B4%E9%83%A8%E5%85%83%E4%BF%A1%E6%81%AF)

```md
Feature Name: (fill me in with a unique ident, my_awesome_feature)
Start Date: (fill me in with today's date, YYYY-MM-DD)
RFC PR: rust-lang/rfcs#0000
Rust Issue: rust-lang/rust#0000
```

- Feature Name: 特性唯一标识（和文件名没有关系）
- Start Date: 启动时间
- RFC PR: 当前 PR 的链接，PR 的 id 将作为 RFC 的编号

### 摘要 [](https://www.rectcircle.cn/series/software-project-management/rust-rfc/#%E6%91%98%E8%A6%81)

我们为什么要这样做？它支持哪些用例？预期的结果是什么？

### 指南级别的解释 [](https://www.rectcircle.cn/series/software-project-management/rust-rfc/#%E6%8C%87%E5%8D%97%E7%BA%A7%E5%88%AB%E7%9A%84%E8%A7%A3%E9%87%8A)

解释该提案，好像它已被包含在语言中，并且您正在向另一个 Rust 程序员传授它。这通常意味着：

- 引入新的命名概念
- 主要用例子来解释这个特性
- 解释 Rust 程序员应该如何思考这个特性，以及它应该如何影响他们使用 Rust 的方式。它应该尽可能具体地解释影响。
- 如果适用，提供错误信息、废弃警告或迁移指导的示例。
- 如果适用，描述向现有的 Rust 程序员和新的 Rust 程序员教授这个功能的区别。

对于以实现为导向的RFC（例如，编译器内部），本节应专注于编译贡献者如何考虑变革，并举例说明其具体影响。对于政策RFC，本节应提供对政策的示例驱动的介绍，并在具体方面解释其影响。

### 参考级别解释

这是RFC的技术部分。以足够的细节解释设计：

- 该功能与其他功能的相互作用是明确的。
- 合理地明确了如何实现该功能。
- 通过示例解剖视角案例。

本节应回到上一节所举的例子，并更充分地解释详细提案如何使这些例子发挥作用。

### 缺点 [](https://www.rectcircle.cn/series/software-project-management/rust-rfc/#%E7%BC%BA%E7%82%B9)

存在缺点

### 理由和替代方案 [](https://www.rectcircle.cn/series/software-project-management/rust-rfc/#%E7%90%86%E7%94%B1%E5%92%8C%E6%9B%BF%E4%BB%A3%E6%96%B9%E6%A1%88)

- 为什么在可能的设计中，这种设计是最好的？
- 还考虑过哪些其他设计，不选择这些设计的理由是什么？
- 不这样做的影响是什么？

### 现有技术 [](https://www.rectcircle.cn/series/software-project-management/rust-rfc/#%E7%8E%B0%E6%9C%89%E6%8A%80%E6%9C%AF)

讨论与本提案有关的现有技术，包括好的和坏的。其中可包括以下几个例子：

- 对于语言、库、crate、工具和编译器的建议。这个功能在其他编程语言中是否存在？ 他们的社区有什么经验？
- 对于社区的建议。是否有其他社区做了这个功能，他们有什么经验？
- 对于其他团队来说。我们可以从其他社区在这里做的事情中学到什么经验？
- 论文。有没有发表过的论文或者是讨论这个问题的好帖子？如果你有一些相关的论文可以参考，这可以作为一个更详细的理论背景。

本节旨在鼓励您作为作者思考其他语言的

经验教训，为您的RFC读者提供更全面的信息。如果没有先例，那也没关系–无论你的想法是全新的，还是从其他语言改编而来，我们都会感兴趣。

请注意，虽然其他语言创造的先例是一些动力，但它本身并不能成为RFC的动力。也请考虑到rust有时会故意偏离通用语言的特性。

### 未解决的问题 [](https://www.rectcircle.cn/series/software-project-management/rust-rfc/#%E6%9C%AA%E8%A7%A3%E5%86%B3%E7%9A%84%E9%97%AE%E9%A2%98)

- 在这个功能被合并之前，你期望通过RFC程序解决设计中的哪些部分？
- 在稳定化之前，你期望通过这个功能的实现来解决设计中的哪些部分？
- 您认为哪些相关问题不在本 RFC 的范围内，可以在未来独立于本 RFC 的解决方案解决？

### 未来的可能性 [](https://www.rectcircle.cn/series/software-project-management/rust-rfc/#%E6%9C%AA%E6%9D%A5%E7%9A%84%E5%8F%AF%E8%83%BD%E6%80%A7)

想一想你的提案的自然延伸和演变会是什么，以及它将如何整体地影响语言和项目。试着把这部分作为一个工具，在你的提案中更全面地考虑所有可能与项目和语言的互动。同时考虑这一切如何与项目和相关子团队的路线图相适应。

这也是一个 “倾倒想法 “的好地方，如果这些想法不在你正在编写的RFC范围内，但在其他方面是相关的。

如果你已经尝试过，但想不出任何未来的可能性，你可以简单地声明你想不出任何东西。

请注意，在未来可能性部分写下一些东西并不是接受当前或未来 RFC 的理由；这样的说明应该在本 RFC 或后续 RFC 的动机或理由部分。这一节只是提供补充信息。

## rfc 的例子

[1210-impl-specialization.md](https://github.com/rust-lang/rfcs/blob/master/text/1210-impl-specialization.md)
[0002-rfc-process.md](https://github.com/rust-lang/rfcs/blob/master/text/0002-rfc-process.md)
