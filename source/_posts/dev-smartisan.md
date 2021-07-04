---
title:  快速开发一个锤子便签
date: 2021-07-04 12:08:05
tags:
---

> 借助Vue、Marked、dom-to-image，40行代码快速开发一个锤子便签。

- 项目地址：https://github.com/nihaojob/markdown-css-smartisan
- 使用编辑器：[地址](https://nihaojob.github.io/markdown-css-smartisan/examples/index.html)
- 预览主题：[地址](https://nihaojob.github.io/markdown-css-smartisan/)
- 欢迎点start😍，鼓励我一下。

![2021-03-28 11.45.58.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/deac83d6405e49ce9e8c80f8c4c40b44~tplv-k3u1fbpfcp-watermark.image)


![image (3).png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a8f5f4c50374363bdd27ee68b9bfbd2~tplv-k3u1fbpfcp-watermark.image)


## 前言
我是锤子便签的深度用户，除了写日记、备忘、文章外，偶尔也用来写技术笔记，有一段短时间把设计模式和《重构》的笔记用便签 + 代码的形式发布到掘金沸点记录下来。

但是锤子便签不能把主题迁移到别处使用，想复制一份复用，就在夜深人静时Fork了`Markdown css`，把心结了了。

![Foxmail20210328114339.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/414060b30509436188e74257971485ac~tplv-k3u1fbpfcp-watermark.image)

样式复制完成后，仅仅有一个个`CSS`和`HTML`，无法体验开发成果😌，又接接入了`Marked`，**一共40行逻辑代码**，满足了自己的产品瘾。


![Foxmail20210328115457.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fef93cf03d94b9087c55d24a18ea55f~tplv-k3u1fbpfcp-watermark.image)



## 实现

使用工具vue、marked、dom-to-img、splitpanes、lodash。

1. 框架：Vue
2. 解析markdown字符串：marked
3. 转图片：dom-to-img
4. 节流：lodash
5. 左右拖拽：splitpanes
6. css主题：markdown-css-smartisan

```js
 new Vue({
    el: "#editor",
    data: {
        downIng: true,
        input: ''},
    computed: {
        compiledMarkdown: function() {
        return marked(this.input, { sanitize: true });
        }
    },
    components: { Splitpanes: splitpanes.Splitpanes, Pane:splitpanes.Pane },
    created:function(){
        if(localStorage.getItem('smartisan')){
        this.input = localStorage.getItem('smartisan')
            if(this.input === '') this.input = md
        } else {
            this.input = md
        }
    },
    methods: {
        update: _.debounce(function(e) {
            this.input = e.target.value;
        }, 300),
        save:function(){
            localStorage.setItem('smartisan', this.input)
            alert('缓存成功')
        },
        down:function(){
            domtoimage.toJpeg(
                document.getElementById('markdown-down-id'), 
                { quality: 0.95 }).then(function (dataUrl) {
                    var link = document.createElement('a');
                    link.download = 'Markdown-Smartisan.jpeg';
                    link.href = dataUrl;
                    link.click();
            });
        },
        clear: function(){
        this.input = ''
        }
    }
});
```


## 总结
JavaScript生态繁荣，开发一个功能只需要引入几个工具包就轻松完成，用着自己开发的便签小工具还是挺有成就感的😄。

> 每一个生命来到世间，都注定改变世界，这是你的宿命，你别无选择。你要么把世界变得好一点，要么把世界变得坏一点。
> 
> 摘自《生命不息，折腾不止》----罗永浩

