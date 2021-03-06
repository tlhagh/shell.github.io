---
layout: post
title:  "[译] CSS编写的禅道"
date:   2017-09-13
categories: CSS
author: 张翔
location: ShangHai, China
cover: "https://meetsup.oss-cn-hangzhou.aliyuncs.com/blog-images/article12/programming-1832991_640.png"
description: 现在很多人不喜欢写css，这是有很多原因导致的，但是归根结底：css是不可预测的。如果你以前从未有过调整过样式规则而意外的打破了完全无关的布局，那么你可能是个新手或者是一个比我们更好的程序员。
---
---
现在很多人不喜欢写css，这是有很多原因导致的，但是归根结底：css是不可预测的。如果你以前从未有过调整过样式规则而意外的打破了完全无关的布局，那么你可能是个新手或者是一个比我们更好的程序员。

所以JavaScript社区开始解决这方面的问题，在过去的几年中，出现了使得css规范化的寒武纪大爆炸时期，统称为CSS-in-JS

你可能没有意识到的是，**CSS最大的问题可以不用CSS-in-JS就能解决**。没有这些问题，编写CSS不再难以忍受，相反会变得很愉快。你不必因为CSS-in-JS引入的其他问题的寻找解决方案。

这篇文章不是批评CSS-in-JS社区的辛苦工作。它是JS生态系统最活跃的角落之一，每周都会有新的想法。相反，我的目的是介绍一种令人愉快地替代方法-基于单文件组件的真正的CSS。

## CSS最大的问题
CSS中样式都是全局的。因此，修改样式代码可能会影响另外的样式。开发人员经常采用不同的命名空间约定（不是规则，因为很难执行），但是这样只会增加RSI的风险。

尤其是当你在一个团队中工作时，这样的做法会变得更加糟糕。没有人敢去修改别人写的样式代码，因为我们不清楚它们是做什么的，以及它们应用在哪个标签上，删除他们项目会不会崩溃。

所有这些的是因为是**仅追加样式表（append-only stylesheet）**。没有办法知道哪些代码可以被安全的删除，所以即使在较小的项目上通常会使用另一个更具体样式代码来撤销已存在的样式。

## 单文件组件

SFC系统背后的想法很简单： 将组件写入HTML文件（可选地），html文件包含描述组件样式和行为的`<style>`和`<script>`属性。`Svelte`，`Ractive`，`Vue`和`Polymer`都遵循这一基本模式。

（本文中的其余部分，我们将会使用Svelte，我们不应该害怕使用模板语言，这个我们以后再谈，现在你的SFCs中使用能让你使用JSX的Vue框架）

结果发生了一些奇妙的事情：


* 你的样式文件作用于组件范围，没有相互干扰，没有不可预测的级联。也没有更多令人厌烦的类名设计来防止冲突。
* 你不需要通过你的文件夹结构去寻找破坏你的东西的规则。
* 编译器（在Svelte的情况下）可以**识别和删除未使用的样式**。没有更多的附加样式表！

让我们在实践中看看：

![Image1.png](https://meetsup.oss-cn-hangzhou.aliyuncs.com/blog-images/article12/Image%20%282%29.png)

![Image2.png](
https://meetsup.oss-cn-hangzhou.aliyuncs.com/blog-images/article12/Image2.png)

![Image3.png](https://meetsup.oss-cn-hangzhou.aliyuncs.com/blog-images/article12/Image3.png)

![Image4.png](https://meetsup.oss-cn-hangzhou.aliyuncs.com/blog-images/article12/Image4.png)

运行之后查看网页localhost:5000

![Image5.png](https://meetsup.oss-cn-hangzhou.aliyuncs.com/blog-images/article12/Image5.png)

修改App.html的style里面的样式文件如下：

```css
h1 {
    color: purple;
    background-color: #1A83EB;
    text-transform: uppercase;
}
.unused{
    color: #000;
}
```

可以在浏览器看到css的h1后面加了一个[svelte-1424936598]，框架默认防止css冲突，编写的unused类没有被使用，也不会被编译进来。

![Image6.png](https://meetsup.oss-cn-hangzhou.aliyuncs.com/blog-images/article12/Image6.png)

编辑器一般都能检测CSS，所以代码能自动补全、linting(css代码检查以及语法高亮显示等等，也不需要安装其他检测工具。

而且这是真正的CSS，而不是驼峰命名引用，我们可以利用“打开devtools，粘贴我的 源代码”工作流模式，我写代码不能没有这个工作流。注意到CSS源代码开箱即用，我们能立即找出有问题的行。很难向你说明这一点的重要性：当你使用所见即所得的模式，你没有考虑你的组件树。所以有一个完整可行的方案来确定错误位置至关重要。如果别人以及写了这个组件，那么就会事半功倍。（我保证，这会在你CSS工作流中最大的提升你的效率，如果你在没有源代码的情况下编写样式，那么肯定会浪费大量时间，至少对我来说是这样的。）

Svelte会转换css选择器（使用影响元素的属性，尽管确切的机制不重要，并可能会更改）以实现范围界定。它会预警并删除任何未使用的规则样式，然后将结果缩小，并将其写入.css文件。还有一个实验性的新选项来编译Web组件，使用shadow DOM封装样式，如果这是你对这方面感兴趣可以去了解。

这些都是有可能的，因为CSS被解析（使用css树），并在标记的上下文中进行静态分析。静态分析打开了未来各种令人兴奋的可能性的大门 - 更智能的优化，a11y代码检测，如果你的样式在运行时动态计算，这将更加困难。我们这仅仅是个开始。

## 我们可以使用工具来做[x]!
如果你觉得上面的视频还不错，但是如果我们使用TypeScript并为每个编辑器写入插件，那么我们可以获得自动完成和语法高亮 - 换句话说，如果您认为追求CSS的平等，而且建立、记录、推广和维护一个配套项目队伍是有意义的，那么，你和我可能永远不能达成一致！

## 目前我们还不能解答所有的问题
话虽如此，CSS-in-JS给出了有些纠结问题的答案：

* 我们如何从npm安装样式？
* 我们如何在一个地方重用定义的常量？
* 我们如何编写声明？

就我个人而言，我没有发现引入的问题超过上述方法带来的好处。你可能有一套不同的优先处理事项，并且可能有充分的理由放弃CSS。
但是在一切结束之后，你也必须了解CSS。无论是爱它或是恨它，你都必须学习它。作为网络管理员，我们有一个选择：创建抽象，使Web开发人员学习曲线更加深入，或者一起来修复CSS的不好的部分。我知道我选择了什么。
