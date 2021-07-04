---
title: 离职后才搞懂vue项目开发流程中的疑惑点
date: 2021-07-04 12:14:49
tags:
---
在离职的最后一个月，帮两位同事申请加薪，确切的说，申请加薪是导火索，我被扣上了哄抬同事工资以提高自己工资的帽子，在推动前后端分离工作中处处碰壁，点燃了压抑许久的离职冲动，领导培养自己四五年，不让声张，答应悄悄离开。

离开时原来公司项目里剩下很多问题没有解决，现在自己还在做vue和动画的项目，现在或许之前的问题已经解决了，但我还是把思路写下来，也算对的起自己悄悄离职的事情了，看到你们获得优秀团队奖的照片了，很棒，祝福你们👍👏👏👏。


### 自动部署
这边用的是gitLab做git服务器，可以配置commit的钩子函数，实现自动部署和线上发布，就相当服务器监听你的提交，在你commit之后，服务器自动执行了一下`npm run build`，放到对应的测试服务器目录，配置文件在根目录下有`.gitlab-ci.yml`文件，起作用的是下边一段代码，`script`是linux脚本，拷贝文件到指定的静态资源CDN目录和web服务器目录，这块知识点是`gitlab-ci 持续集成`，可以关注一下，svn应该也有类似的配置，让运维帮忙给配一下吧。

```
npm-build-test:
  image: cdn路径
  stage: build
  cache:
    untracked: true
    paths:
      - node_modules/
  before_script:
    - export BI_ENV="test"
  script:
    - "npm install --registry=http://代理地址 --sass_binary_site=https://npm.taobao.org/mirrors/node-sass/" 
    - "npm run build"
    - "rsync -auvz dist/index.html  ip::服务器开发分支目录/trunk/resources/views/index/"
    - "rsync -auvz dist/* 静态资源cdn目录/trunk/bi/"
  only:
    - master  分支名称

```

### 版本管理
以前咱们经常出现这种情况， v0.1的需求已经上线，v0.2的需求已经提测了，v0.3的需求在开发中，现在要修复一下v1.0的线上bug，用svn的话可能就是把修复后的文件更新到三个分支上继续开发，更新来更新去就`lock`了。

如果用git做版本管理，就方便很多，按照分支规范，常用4个分支，
`Develop`持续开发分支，`Release`版本分支，
`Hotfix`紧紧热修复分支，`Master`上线版本主分支，
可以看看`GitFlow工作流`方面的资料，真的比svn的分支好用太多了。

![](https://user-gold-cdn.xitu.io/2019/1/24/1687b99acf98287f?w=614&h=380&f=png&s=35398)

### 脚手架升级与优化
我们目前在用webpack 4.0 和 vue-cli3.0，编译很快，建议升级，记得之前项目用vue-cli2.0编译和打包时间很长。


### Ajax全局设置

原来项目里用的是`jQuery.ajax`方法，其实也可以全局设置拦截，我们用的是`axios`，在`main.js`中引用，设置**根路径、状态码、token、超时时间**等全局设置，代码如下:

```
// 引入axios
import axios from 'axios'
// axios配置
Vue.prototype.$http = axios

// 配置默认axios参数
axios.defaults.baseURL = '/'
axios.defaults.timeout = 1000000
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=UTF-8'

// 添加请求拦截器
axios.interceptors.request.use(function (config) {
  let token = localStorage.getItem('token')
  if(token== null && router.currentRoute.path == '/login'){// 本地无token,不为登录 跳转至登录页面
    router.push('/login')
  }else{
    if(config.data==undefined){
      config.data = {
        "token":token
      }
    }else{
      Object.assign(config.data,{"token":token})
    }

  }
  return config
}, function (error) {
  iView.Message.error('请求失败')
  return Promise.reject(error)
})

// 返回结果拦截
axios.interceptors.response.use(function (response) {
  if(response.hasOwnProperty("data") && typeof response.data == "object"){
      if(response.data.code === 998){// 登录超时 跳转至登录页面
          iView.Message.error(response.data.msg)
          router.push('/login')
          return Promise.reject(response)
      }else if (response.data.code === 1000) {// 成功
        return Promise.resolve(response)
      }
  } else {
    return Promise.resolve(response)
  }

}, function (error) {
  // 请求取消 不弹出
  if(error.message != '0000'){
    iView.Message.error('请求失败')
  }

  // 请求错误时做些事
  return Promise.reject(error)
})

```
### 异步操作与数据遍历
原来的项目是保险项目，大家没有前后端分离的业务系统经验，都是**最基础的接口**，一个一个接口都是从数据字典中取出，业务逻辑往前端移，**导致前端有很多的串行、并行的异步操作**，请求各种接口，然后遍历合并各种数据，串行并行我们用`promise`写，异步操作的问题就解决了，数据的操作我们用`lodash.js`，效率比手写快，这两块可以加深一下。
```
initializationTab(obj){
    let This = this
    return new Promise((resolve, reject) => {
        This.$http
            .post('/api/show/ograde-header',obj)
            .then(function(response) {
              return resolve(response.data.datas)
            })
            .catch(function(error) {

                //数据丢失的状态
                This.isContent=false   //是否展示加载后内容
                This.isLoading=false   //是否展示loading
                This.isContentError=true 
                This.isReady = false //是否展示数据准备中状态
                reject(error)
            });
    })
}
```


### 登录
原来项目登录是跳转到jsp登录页面，然后再跳回静态页面，`sessionID`就存到`cookie`里了，建立会话，**也可以`Form`提交做登录，** 正常走接口，也会在`cookie`里存上`sessionID`建立会话。

当然前后端分离最好用`JWT`方案，把生成的`token`放在`redis`里，建立`事务`,`token`过期后自动删除，**如有提前退出，则给改`token`打上标识，不通过该`token`通过即可**，**续签也好解决**，在如果`token`在到期5分钟前访问，则生成新`token`返回给前端，给即将过期的`token`打上标识，到期后自动删除。


### H5动画
我们H5动画做了很多尝试，民生银行的蓝精灵主题、租房分期、招聘3D、消消乐等，在适配、时间轴、精灵图、动画性能等方面有了一定积累，笔记夭折在我的MWeb编辑器里，如果后边有时间再更新出来吧。

之前蓝精灵动画需求用的`TweenMax.js`和`gka`生成的css帧动画做了那么复杂的一个效果，性能不是特别好，毕竟操作的是DOM，如果动画需求还多，就多熟悉熟悉`PIXI.js+TweenMax.js`两个工具吧，我刚用它们做了一个需求，`PIXI.js`有加载器、精灵图、滤镜、物理引擎、音视频等好多辅助工具，基本可以实现大部分我们在朋友圈看到的H5效果，`PIXI.js`支持`canvas`和`webGL`两种渲染。

![](https://user-gold-cdn.xitu.io/2019/1/24/1687bd5f866be363?w=483&h=269&f=gif&s=2014168)

![](https://user-gold-cdn.xitu.io/2019/1/24/1687be36bb53bac3?w=493&h=263&f=gif&s=4394991)


### 通读API
最新的项目是自己搭建的vue架子，功能和场景也慢慢复杂起来，还是要多看api和文档，多了解原理，才能游刃有余的使用这些工具，高效的完成开发任务，比如vue的组件递归、缓存、强制刷新、混入、过滤器，Axios的默认配置、CancelToken等等，最近的项目笔记总结还没有写完，先把目录贴出来，期望进步吧。

![](https://user-gold-cdn.xitu.io/2019/1/24/1687bf0c6a18e16a?w=992&h=1024&f=png&s=225533)

如果你们还在从事前端，相忘于江湖吧🤣😂。