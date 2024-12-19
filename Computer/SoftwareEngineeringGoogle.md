# Software Engineering at Google

> Software Engineering at Google 笔记。

## 软件工程是什么？

> "软件工程是随着时间推移的编程。"编程当然是软件工程的一个重要部分：毕竟，编程首先是生成新软件的方式。如果你接受这一区别，那么很明显，我们可能需要在编程任务（开发）和软件工程任务（开发、修改、维护）之间进行划分。时间的增加为编程增加了一个重要的新维度。这是一个立方体三维模型不是正方形的二维模型，距离不是速度。软件工程不是编程。

软件工程可以定义为多人协作开发多版本程序。**人是软件工程的核心**，代码是重要的产出。

[文章地址](https://qiangmzsx.github.io/Software-Engineering-at-Google/#/zh-cn/Chapter-1_What_Is_Software_Engineering/Chapter-1_What_Is_Software_Engineering)


**编程和软件**之间有三个关键的区别

- 时间
- 规模
- 权衡取舍

在一个**软件工程项目**中，**工程师需要更多关注成本和需求变化**。 在软件工程中我们需要更加的关注的是**规模和效率**，无论是对于我们的生产的软件，还是对生产软件的组织。

如果你正在维护一个由其他工程师使用的项目，那么关于“有效”与“可维护”最重要的一课就是我们所说的**海勒姆定律**：

> 当一个API有足够多的用户时，在约定中你承诺的什么都无所谓，所有在你系统里面被观察到的行为都会被一些用户直接依赖。

当我们在讨论软件工程的时候，这本书重点是在讨论下面的两件事情

- 一个组织和一个程序员的策略，如何评估和改进你的最佳实践，以及用于可维护软件的工具和技术。
- 你如何维护你的代码，让它正常运行。

##  新成员如何融入团队？

> 软件开发是团队的努力。要在工程团队或任何其他创造性合作中取得成功，你需要围绕谦逊、尊重和信任的核心原则重新定义你的行为。

[文章地址](https://qiangmzsx.github.io/Software-Engineering-at-Google/#/zh-cn/Chapter-2_How_to_Work_Well_on_Teams/Chapter-2_How_to_Work_Well_on_Teams?id=%e7%ac%ac%e4%ba%8c%e7%ab%a0-%e5%a6%82%e4%bd%95%e8%9e%8d%e5%85%a5%e5%9b%a2%e9%98%9f)

- **早期分享**：如果你对世界隐藏着牛逼的想法，并在未完美之前拒绝向任何人进行展示，那么就是在进行一场巨大的赌博。因为在早期非常容易犯基本的设计错误。早起的失误的可能性非常高，你越早的征求反馈，这种风险就越低。**早失败，快失败，经常失败**是一句经得起考验的至理名言。

- **巴士因子**：团队里因巴士撞倒的多少人，会导致项目失败。简答的来说，你需要进行团队工作而不是独自工作，人们很容易忘记，独自工作是一项艰苦的工作，比人们自认为的慢的多。编程很难，软件工程很难，你需要另外一双眼睛。

总结：几乎任何规模的软件工作的基础都是一个运作良好的团队。尽管软件开发者单打独斗的 "天才神话 "仍然存在，但事实是，没有人能够真正地单干。一个软件组织要想经受住时间的考验，就必须有一种健康的文化，植根于谦逊、信任和尊重，围绕着团队而不是个人。此外，软件开发的创造性要求人们承担风险并偶尔失败；为了让人们接受这种失败，必须有一个健康的团队环境。


## 知识共享

> 你的组织需要一种学习文化，这需要创造一种心理上的安全感，允许人们承认自己缺乏知识。

在某些方面，知识是软件工程组织字重要的无形资产，而知识的共享对于使组织在面对变化时具有弹性和冗余至关重要。一种促进开放和诚实的知识共享的文化可以在整个组织内有效的分配这些知识，并使该组织能够随着时间的推移而扩散。在大多数情况下，对更容易的知识共享的投入会在一个公司的生命周期中获取很多倍的回报。


## 测量工程效率

> 如果你有一个蓬勃发展的业务想要扩增。据推测，为了增加你的业务范围，你也需要增加你的工程组织的规模。然而，随着组织规模的线性增长，沟通成本也呈二次曲线增长。增加人员对扩大业务范围是必要的，但沟通成本不会随着你增加人员而线性扩展。因此，你将无法根据你的工程组织的规模线性地扩大你的业务范围。



# 总结