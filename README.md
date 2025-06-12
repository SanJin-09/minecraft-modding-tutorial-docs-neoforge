# Minecraft模组开发文档指南 基于NeoForge API
_——一篇为使用NeoForge API的Minecraft模组开发者提供中文指导的文档指南_
## 一些 Q & A
1. **什么是NeoForge，为什么使用它开发Minecraft模组？**\
NeoForge 是 Minecraft Forge 的官方分支项目，由原 Forge 开发团队几乎全体成员于 2023 年 7 月创建146。它继承了 Forge 的模组开发功能，并进行了多项技术革新。\
总的来说，如果你想要进行高版本的Minecraft模组开发（1.20.4+），并且你期望实现的内容不是那么轻量（如果是，那么你可以考虑Fabric API），那么我更推荐你使用NeoForge API，它兼具了Forge的强大功能与多模组的加载性能优化，更适用于高版本的Minecraft游戏。~~拜托，Forge真的很老了~~
2. **这篇文档可以帮助我做些什么？**\
如果你是一名编程小白，但你又希望为你热爱的游戏——Minecraft，开发模组，那么这篇文档将带你从基础的自定义方块与物品开始，到自定义事件与战利品，自定义的生物与AI行为，再到网络通信与数据包基础，Mixin使用等等，构建属于你自己的模组，与你的Minecraft好友~~炫耀~~分享并将它发布到其他网站与平台。\
本文档同时也将涉及一些关于Java语言的使用规范，项目的结构构件，当程序运行报错或崩溃时你应该怎么做，以及其他的一些你将会用到的第三方工具（例如BlockBench）的使用教程。\
当然，如果你对Java或Kotlin语言有至少基础的掌握，你可以选择性的跳过这部分内容。
3. **为什么编写这篇文档？**\
目前，关于NeoForge API的使用指南并不多，国内资源平台上只有一些搬运内容，而即使是在外网平台（YouTube），基于NeoForge API的Minecraft模组编程完整指导频道也只有一个，其余的则是一些在Reddit或是类似的社区平台上的零散内容。\
NeoForge官方有自己的使用文档：**NeoForge Documentation**，但是它的内容对刚接触Minecraft模组开发的新手并不太友好，更重要的是，它是英文的，我相信你和我一样都不能忍受糟糕的浏览器翻译，这篇文档便因此诞生，**我们将用中文给出一份完整的，适于模组开发新手的，高版本Minecraft的模组开发指南**。\
本文档不会涉及太深奥的功能实现，Minecraft的结构并不复杂，写出优质的模组更多需要的是创想力与扎实的项目编写能力。
## 你可能会用到
- NeoForge官方API使用文档：https://docs.neoforged.net/
- 适用于Java与Kotlin专业开发的Intellij IDEA：https://www.jetbrains.com/zh-cn/idea/
- 用于创建物品纹理，方块与生物模型以及生物动画的BlockBench：https://www.blockbench.net/
