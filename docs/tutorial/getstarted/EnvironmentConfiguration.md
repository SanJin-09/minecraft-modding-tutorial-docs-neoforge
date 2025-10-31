# 环境与工具配置
> 从这里开始我们的旅程  

_在这一章节，你将学习如何获取并安置好所有你需要的工具_

## Java环境配置
Java语言是我们开发Minecraft mod的主流语言，为了能让Java程序在你的PC上正常运行，我们需要下载以下配置内容：
- [Java 21 Development Kit (JDK)](https://www.oracle.com/java/technologies/downloads/)
- [64-bit Java Virtual Machine (JVM)](https://www.java.com/en/download/manual.jsp)

## 安装你的编译器
下载并按照指示成功安装好以上环境配置内容之后，我们还需要一个编译器用于编写并运行Java文件。如果你是编程新手，我会推荐你使用 [IntelliJ IDEA](https://www.jetbrains.com/idea/)。

IntelliJ IDEA由JetBrains公司开发，社区版对个人免费，如果你是一名大学生且拥有.edu邮箱，你还可以通过教育认证免费使用专业版。当然，社区版用于开发Minecraft mod已经完全够用。

如果你拥有Java编程基础，或者其他编程语言基础，使用你最熟悉的现代编译器即可，只要它们能编译运行Java程序。

## 下载模版模组
NeoForge官方提供了一个Mod Generator工具（[NeoForge Mod Generator](https://neoforged.net/mod-generator/)），包含了mod的基础结构和gradle配置文件。

以我的教学示范模组Tutorial为例：输入你的mod的正式名：Tutorial，包名com.snajin.tutorial，选择你的mod版本，最后的Gradle Plugin一般选择默认值。点击网页下方的DOWNLOAD MOD PROJECT即可下载模版模组的压缩包。

解压你所下载的压缩包，打开我们的编译器（IDE），选择打开本地项目，选择解压后的文件夹，打开。

第一次打开项目文件时，IDE会开始自动下载所需的Gradle依赖，你要做的只是等待下载完成即可。

如果你出现了下载失败（控制台，也叫终端中出现了Build Fail字样），或是下载进度长时间未变动等情况，请打开你的vpn，选择一个稳定的节点，然后在控制台中输入：`gradlew build`指令重新构建

等待所有依赖下载完毕后，在IDE中选择Client并run运行（一般在右上角），如果你使用的是VS code一类的编译器，你可以在控制台中直接输入`./gradlew runClient`来调试运行。你可以看到Minecraft游戏窗口正常打开并运行。

## 其他工具
Minecraft mod 开发不止需要使用Java进行编程，我们还需要其他一些工具用于制作物品，方块和生物的贴图；特殊方块，生物的模型，生物的行为动画等等资源内容。

Minecraft中，所有的资源贴图都应该是.png格式，所以任何能够绘制并导出.png格式的绘图工具都可以作为你的贴图编辑器

我个人使用最多并且也很推荐的软件是[Blockbench](https://www.blockbench.net/)。Blockbench是一款免费开源的低多边形3D建模软件,专门用于Minecraft模型制作。它拥有简洁直观的界面,操作简单易上手,内置动画编辑器,支持骨骼绑定和关键帧动画。软件支持像素艺术纹理绘制,可直接在3D空间中绘画或使用2D纹理编辑器。还拥有丰富的插件系统扩展功能,支持多种导出格式用于游戏开发、3D打印等用途,是Minecraft模组和资源包制作的行业标准工具

简单说：2D贴图绘制，3D模型建造，动画编辑，Blockbench都能做到，完全满足一般的Minecraft mod开发需求。
