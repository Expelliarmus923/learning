## 读《You Might Not Need jQuery》有感

清明出去玩了两天，今天刚回来刷了下github趋势榜单，看到一篇名为`You-Dont-Need-jQuery`的项目，一周内涨了2000+个star，联想到最近看到的一些文章也有提及，于是满怀好奇的点了进去。

看完全文，也做了一些思考。总得来说对文章的观点赞同和反对大致是7：3吧，结合当下我在公司所做的项目做了一些相应的总结：


1. 文章开头就提到由于React，Angular，Vue的流行，大家普遍形成了的思想已经确认直接操作DOM并不是一件好事。

    > 这点我双手赞成。联想到公司有些项目在需要更新数据的时候使用数据渲染模板后直接替换掉原有的一大片DOM节点的造成部分低端机页面出现闪屏的情况，大片操作DOM是我们需要避免的一点。
    
2. 文章提议使用`querySelector`与`querySelectorAll`代替$的选择器查找，并且在适时的时候考虑使用`getElementById`,`getElementsByClassName`,`getElementsByTagName`,来提高性能。

    > 这点让我想起了jQuery的移动端版本Zepto就做得无可挑剔,根据你浏览器的性能传入参数时进行一系列判断，来帮你选择最优的选择器，并返回继承了Zepto诸多方法的数组实例。

3. 涉及到节点之间的遍历的时候，文中都提出了一系列相应的解决方案。

    > 看了解决方案，突然觉得原生JS对于DOM树的遍历操作还是过于原始，虽然没有C语言操作树结构那么难于理解，但是使用起来依然存在诸多不便，不由感叹，我们真是被jQuery惯坏了

4. 关于CSS与Style，文中推荐使用classList这个API来操作class，推荐使用`getComputedStyle`,`setAttribute`来实现属性的读取。或者用他们的util。

    > classList方法是好，但是兼容性是个问题，我们依然要兼容4.0-4.4的安卓，和iOS 8.0以上的版本，所以暂时不能这么使用.这里感觉jQuery的兼容代码在不需要兼容IE的时候会显得冗余，但是用Zepto没什么不好的，在高版本上面做了很多的兼容处理，以及性能最优做法。
    
5. 关于width、height、position等位置属性的操作。文中每一个都给了很复杂的原生JS实现。

    > 如果你没有仔细研究过盒模型概念与DOM相关的位置获取API，你可能很难写出相关的计算代码。不过我们有一个接近万能的方法`getBoundingClientRect`能得到盒子的宽高和相对视口的坐标，所以根据这点来看，我们只要了解了一些计算规则是可以不依赖jQuery/Zepto的

6. 关于DOM变化，文章给出了实现`append`、`prepend`、`html`,`insertBefore`、`insertAfter`等常用方法。
    
    > 这里要自己实现不难，借助原生方法也能很好的实现，不过话说回来现在virtual-dom如此流行的趋势，其实借助数据来diff DOM节点的变化，再最小量的更改才是王道

7. 说到AJAX，文中直接建议用一个fetch库或者ajax库来实现。

    > 看过了zepto源码之后，你会觉得不是XMLHttpRequest不好，而是原生的功能要满足很多额外需求不得不要写更多的代码，而jQuery/Zepto开箱即用，更有很多库实现了promise规范，拦截器，等等诸多实用的功能

8. 关于事件处理。
    
    > 之前看过一个关于原生事件处理比jQuery/Zepto慢很多的帖子，确实在事件上面jQuery/Zepot做了很多的封装，无论是让不支持冒泡的某些事件，看起来像是支持冒泡；还是使用事件委托；还是回调添加boolean值功能取消默认事件和冒泡，框架的封装都帮我们做了很多的事情，我们没有必要抛弃这些

总结：看完该文章又结合下我现在正在维护的项目，我认为在不使用VUE的项目或者页面上面引入一个不足10k的zepto库能够帮我们节省很多麻烦的事情，所以是有必要的；而如果是使用了类似VUE，React等流行框架的项目时，使用jQuery/Zepto看起来确实是没有什么必要了，可以去除掉。

事实上，目前我们也正是这么做的，已经开始逐渐的去jQuery/Zepto化了，不得不说可能以后在页面控制台输入$就能使用jQuery的情况会逐渐的变少。

最后我也去看了该作者自己封装的一些处理库，都只有十几个star，所以我认为其实在现代浏览器的环境中Zepto还是帮我们做了很多的事情的，作为一个简化版本的jQuery，它应该还能存在很长一段时间。