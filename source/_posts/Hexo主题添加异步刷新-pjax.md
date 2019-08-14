---
title: Hexo主题 icarus 添加异步刷新-pjax
date: 2019-08-11 14:02:24
tags: Hexo 
thumbnail: /gallery/3232.jpg
categories: Coder
---

这两天更新Blog的时候,找到一个很好用的主题, 也就是现在这款:

>  [A simple, delicate, and modern theme for the static site generator Hexo. ](https://github.com/ppoffice/hexo-theme-icarus)

也可通过右下角footer中的github图标进入.

但是原版的刷新模式是基于同步刷新, 无论点什么链接都会整屏刷新会导致浏览器整个页面闪一下, 感受很不好, 遂寻找异步刷新的解决方案.

<!-- more -->
经过一番搜索后, pjax进入了眼帘

>   [GitHub: pushState + ajax = pjax](https://github.com/defunkt/jquery-pjax)

将引入代码添加到项目中正确到位置. 如果和我一样对前端比较生疏的人记得找准位置, 加载位置要在jquery 后面

```html
<script src="//cdn.bootcss.com/jquery.pjax/2.0.1/jquery.pjax.min.js"></script>
```
在 icarus 中, 我是将其放在了 `scrips.ejs` 中, 也就是刚好在 jquery 引入后

当然也可以通过npm

> $ npm install jquery-pjax

等其它方式引入



![scripts.ejs](1.png)

引入之后, 在找到对应的容器div, 即主体内容的加载容器对其进行全局唯一ID的定义
在 icarus 中, 是 `layout.ejs`

![layout.ejs](2.png)

将在包裹着 `<- body ->` 的div  声明一个Id

最后, 在 `main.js` 中初始化 pjax, 具体在哪声明, 根据不同主题的主函数定义位置

```javascript
$(document).pjax('a', '#pjax-container', {
    fragment: '#pjax-container',
    timeout: 5000,
    cache: false
});
```

![main.js](3.png)

至此, 异步刷新界面达成