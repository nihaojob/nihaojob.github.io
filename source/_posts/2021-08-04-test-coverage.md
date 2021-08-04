---
title: 生成Github JS 仓库的测试覆盖率徽标
date: 2021-08-04 16:56:07
tags:
---


今天给我的开源项目[popular-message](https://github.com/nihaojob/popular-message)增加了一下测试覆盖率的图标，覆盖率提高到了88%，不过这个覆盖率的图标还真不是直接放个图片链接这么简单。

不过也难不到哪里去，除了jest单元测试框架的语法，主要是借助[travis-ci](https://travis-ci.com/)、[coveralls.io](https://coveralls.io/)这2工具实现测试报告自动上报。


快速的写下笔记备忘，如果你在搞单元测试，恰巧也要增加测试覆盖率图表，希望能够帮到你，大神跳过。


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c04644be6c534feca262245878df6dd3~tplv-k3u1fbpfcp-zoom-1.image)


涉及工具：

1. Jest：Js测试框架
2. Travis-CI：CI 持续集成服务平台
3. Coveralls.io：测试报告展示



### 流程
首先选择一个单元测试框架，我用的Jest，编写完单元测试代码以后，我们要确保[travis-ci](https://travis-ci.com/)、[coveralls.io](https://coveralls.io/)这2个网站**使用GitHub账号授权登录，并有响应的读写权限**，然后再按照流程配置就轻车熟路了。


1. GitHub账号授权登录[travis-ci](https://travis-ci.com/)、[coveralls.io](https://coveralls.io/)
1. 安装Jest，编写单元测试代码
1. 安装 Coveralls， 增加测试报告上报脚本
1. 配置Travis 文件，提交代码后自动执行上报
1. 提交代码触发CI，查看覆盖率



### 1. GitHub账号授权登录[travis-ci](https://travis-ci.com/)、[coveralls.io](https://coveralls.io/)
这一步很简单，只需要授权登录就好，但是必须的，否则不能根据仓库自动执行。


授权后会有项目列表：
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5cb12f51fd3140d7b78934b12296206b~tplv-k3u1fbpfcp-zoom-1.image)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9e642816b9d49a2be91296ac15703af~tplv-k3u1fbpfcp-zoom-1.image)


### 2. 安装Jest，编写单元测试代码
安装依赖，编写单测代码，增加Script选项，然后直接运行即可。

```bash
$ yarn add jest -D
```


[测试代码](https://github.com/nihaojob/popular-message/blob/main/test/index.test.js)不再贴进来，可在Github查看。


package.json增加测试脚本
```json

{
  "scripts": {
    "test": "jest"
  }
}
```


执行结果：
![2021-08-04 19.02.48.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d80f723a707e4fd5860a734d643879ab~tplv-k3u1fbpfcp-zoom-1.image)
### 3. 安装 Coveralls， 增加测试报告上报脚本
本地执行 jest --coverage 时会生成测试报告HTML文件， Coveralls工具会把测试报告上传到[coveralls.io](https://coveralls.io/)网站，可以展示测试报告并生成徽章。




安装coveralls：
```bash
$ yarn add coveralls -D
```


package.json增加上报脚本：
```json
{
  "name": "popular-message",
  "scripts": {
    "test": "jest",
    "coveralls": "jest --coverage --coverageReporters=text-lcov | coveralls", // 上报脚本
  }
}
```


本地执行生成覆盖率的效果，这一步仅演示覆盖率生成，与上报无关。
![1.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75404c00583141efb62b50b2e792812a~tplv-k3u1fbpfcp-zoom-1.image)


### 4. 配置Travis 文件，提交代码后自动执行上报
授权Github账号授权Travis后，在每次提交会按照项目中的`.travis.yml`配置文件自动执行脚本，只配置自动上报测试报告脚本。


```javascript
language: node_js
node_js:
  - 14 # use nodejs v10 LTS
cache: npm
script:
  - yarn coveralls # generate static files
```




### 5. 提交代码触发CI，查看覆盖率


提交代码后，就可以在Travis-CI后台看到执行过程了，执行成功后等几分钟去[coveralls.io](https://coveralls.io/)查看报告，这是我项目的[测试报告](https://coveralls.io/github/nihaojob/popular-message)。


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a5e2dd5d3b7493c9d6a23d5a7e6ea8c~tplv-k3u1fbpfcp-zoom-1.image)


### ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b50e6c759484b6295df557e4d720bd3~tplv-k3u1fbpfcp-zoom-1.image)
点击EMBED按钮获得带分辨率的徽章，拷贝到自己的项目ReadMe文件里就可以了。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92185563e95844558bb817d248497a25~tplv-k3u1fbpfcp-zoom-1.image)
## 结尾


自己最近的一篇笔记是[《Vue业务系统落地单元测试》](https://juejin.cn/post/6978831511164289055)，对单元测试的空白算是补上了一点，趁着热乎劲把自己的小插件也加了一下单元测试，如果你也在学习单元测试，大家一起Star、相互鼓励学习吧。

看到自己的开源小插件[popular-message](https://github.com/nihaojob/popular-message)从0到200多Star，真的是满心欢喜，感谢阮一峰老师博客的介绍，感谢公众号的推送，感谢素未谋面的朋友提来PR和Issue。


