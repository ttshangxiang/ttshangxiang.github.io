# WebComponents了解一下

## 前言

WebComponents其实我很早之前就听过，但是仅止于概念。当时的想法是，东西是好可是太新了，浏览器支持遥遥无期，所以没太在意。  

前不久因为看游戏攻略视频上了Youtube，逛了一会儿，突然想了解一下，这个世界上最大的视频网站的前端在用什么技术，结果打开F12发现了一些奇怪的东西。  
* 奇怪一：html上到处都是自定义标签。  
在我知道的前端框架中，虽然很多都可以写自定义标签，但是实际上在浏览器运行的还是div，能直接用自定义标签很新奇。
* 奇怪二：css中有很多看不懂的属性。  
像“—app-drawer-width: 240px”，“background: var(--yt-main-app-background);”，看上去有点像less，但是不编译成css就能直接用真的好吗?

基于以上的疑问，我查到了polymer，webcomponents，webcomponentsjs等一些东西，正好最近公司公众号分享轮到我了，自己有兴趣还能完成任务，觉得很nice。

## 简介

以下部分摘抄自MDN，想看完整的请[点击跳转](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components)，下面还有例子，我就不重复摘抄了。
> Web Components是一套不同的技术，允许您创建可重用的定制元素（它们的功能封装在您的代码之外）并且在您的web应用中使用它们。  

> 它们可以一起使用来创建封装功能的定制元素，可以在你喜欢的任何地方重用，不必担心代码冲突  

理解：从这两句话来看，我之前应该有些理解错误，WebComponents似乎不是用来替代React、Vue等框架，而是适合作为一个小部件混入其中。像select，radio这种比较丑、表现不一的标签，通常需要重写，但如果你有jquery项目又有react项目，那就要写两份，因为不能兼容。react开发者通常也耻于引入jquery(笑)。其实这个用原生Javascript插件也可以解决，但是会有一些问题，比如css冲突、js冲突，这些问题WebComponents有考虑到。

### 它由四项主要技术组成
> Custom elements（自定义元素）：一组JavaScript API，允许您定义custom elements及其行为，然后可以在您的用户界面中按照需要使用它们。

<font color=#999 size=2>理解：就是自定义组件，可以拿video标签作为参考，原本只有官方可以定义标签，现在GM开放权限，你也可以自定义标签了呢。</font>

> Shadow DOM（影子DOM）：一组JavaScript API，用于将封装的“影子”DOM树附加到元素（与主文档DOM分开呈现）并控制其关联的功能。通过这种方式，您可以保持元素的功能私有，这样它们就可以被脚本化和样式化，而不用担心与文档的其他部分发生冲突。

<font color=#999 size=2>理解：可以理解成一个隔离的环境，css，js基本上不会互相影响。前面提到的css，js冲突问题，在这里就可以得到解决。写组件的人也不用因为害怕css冲突，写很多中短线下划线，用组件的人也不用担心组件css被自己覆盖。</font>

> HTML templates（HTML模板）：\<template\> 和 \<slot\> 元素使您可以编写不在呈现页面中显示的标记模板。然后它们可以作为自定义元素结构的基础被多次重用。

<font color=#999 size=2>理解：和目前热门的框架的template、slot功能差不太多，但是只实现了最基本的功能。</font>

> HTML Imports（HTML导入）：一旦定义了自定义组件，最简单的重用它的方法就是使其定义细节保存在一个单独的文件中，然后使用导入机制将其导入到想要实际使用它的页面中。HTML 导入就是这样一种机制。

<font color=#999 size=2>解读：就是这样 \<link rel="import" href="myfile.html"\></font>

## 兼容

WebComponents目前的兼容性还很差，其实并不推荐用在生产中。但是也不用太担心啊，毕竟也有大厂在用，可以用他们的兼容方案 - webcomponentsjs。  
webcomponentsjs是一个兼容库，这个库提供了Custom Elements、Shadow DOM、CSS custom properties等特性的兼容。  
在百度搜webcomponentsjs的第一个网页是知乎的，打开后发现大牛们都在说他的不好，甚至有人说“Webcomponent名存实亡”，估计很多人是给这吓退的。  
但是看npm的周下载量也有26000+，而且还是在上升的趋势，说明也不是不能用的，我觉得对兼容性没那么严格的项目来说可以尝试。谷歌的polymer也在使用webcomponentsjs，实际上这就是polymer那一群人做出来的，看github上的贡献者就会发现。既然Youtube在用，那也不用那么怕，我这样安慰道。

在v2中用法就是引入webcomponents-bundle.js，其他高级用法可参照polymer的设置[点击跳转](https://polymer-zh.cn/3.0/docs/polyfills)

## 扩展
webcomponents是提供了template，但是用过其他主流框架的人会发现，这个template根本不够用。既没有数据绑定，也没用事件绑定，条件、循环这些通通都没有。这个时候呢[webcomponents.org](https://www.webcomponents.org/introduction)网站介绍了一些库：  
* [Bosonic](https://bosonic.github.io/)
* [Polymer](https://www.polymer-project.org/)
* [SkateJS](https://github.com/skatejs/skatejs)
* [Slim.js](http://slimjs.com/)
* [Stencil](https://stenciljs.com/)
* [X-Tag](https://x-tag.github.io/)

X-Tag是比较老库，新版的github无人问津，我看他的介绍发现写法和官方WebComponents的写法差别很大，所以我不太喜欢。  
Slim.js是比较能接受的了，绑定和条件循环都有，文件也很小，看名字也知道定位是精致小巧。  
Polymer则是一整套，看到很多和angular和vue功能类似的东西，数据绑定、数据监听、计算属性这些，甚至还有命令行构建工具，可以说是很完整了。  
其他的没有仔细看。   

## 看polymer文档时的其他收获
这些不属于webcomponents，可能是很久以前的概念了，但是对于我来说还很新鲜。
### 自定义css属性
应该很容易看懂，特别用过sass/less的玩家。
```
// 定义
element {
  --custom-color: blue;
}
// 使用
element {
  color: var(--custom-color, #fff);
}
```
这个@apply现在需要polyfill，引入@webcomponents/shadycss/entrypoints/apply-shim.js。
```
// 定义
selector {
  --mixin-name: {
    /* rules */
  };
}
// 使用
selector {
  @apply --mixin-name;
}
```
查看更多请[点击跳转](https://polymer-zh.cn/3.0/docs/devguide/custom-css-properties)

### 自定义事件 - CustomEvent
polymer中有提到这个，感觉可以用来作为父子组件传值，vue就是这么做的。vue还推荐在简单的项目里面不用vuex管理状态，而用一个new Vue()来注册、触发自定义事件，使用这个CustomEvent也可以模仿。
```
// 添加一个适当的事件监听器
obj.addEventListener("cat", function(e) { process(e.detail) })

// 创建并分发事件
var event = new CustomEvent("cat", {"detail":{"hazcheeseburger":true}})
obj.dispatchEvent(event)
```

### 监听DOM - MutationObserver
用这个实现双向绑定的dom到数据的环节。搜索时还搜到有人在推荐用这个实现水印不可删除，就是监听后删掉就给它补上，对方一定很烦。
当然写个循环监听也可以做到了，但是性能和反应速度就不用说了。  
它还有个兄弟ResizeObserver，如果需要监听元素尺寸变化就得用这个，不过这个的兼容性还很差，需要的话可以[polyfill](https://link.zhihu.com/?target=https%3A//github.com/WICG/ResizeObserver)。

## 结语
新特性虽然兼容性差，但是你看HTML5、ES6这些不都是慢慢的变得普遍，不支持新特性的老旧浏览器也在慢慢的被抛弃，说不定过几年webcomponents变成了热门呢？了解一下总是不坏的。
