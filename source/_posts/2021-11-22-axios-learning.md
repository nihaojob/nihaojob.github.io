---
title: Axios源码学习
date: 2021-11-22 19:26:07
tags:
---

## 目录


1. 介绍
1. **初始化**：axios 、Axios、intance
1. **整体流程**：request => deispatchRequest => Adapter
1. 数据转换：despatchRequest
1. 适配器：Adapter
1. **拦截器**：request => deispatchRequest => response 
1. **取消请求**：cancelToken => adapter => cancel
1. 回顾



## 介绍


特点：

1. 支持浏览器与Node.js
1. API promise
1. 请求/响应拦截器
1. 请求/响应格式化方法
1. 取消请求



用法：
```javascript
// 直接发送请求
axios(config)
axios(url[, config])
axios.request(config)

// 别名方法
axios.get(url[, config])
axios.delete(url[, config])
axios.post(url[, data, config])
axios.put(url[, data, config])

axios.defaults.xxx: 请求的默认全局配置 
axios.interceptors.request.use(): 添加请求拦截器 
axios.interceptors.response.use(): 添加响应拦截器

axios.create([config]): 创建一个新的 axios

// 只有默认axios有的方法，新创建的没有
axios.Cancel(): 用于创建取消请求的错误对象 
axios.CancelToken(): 用于创建取消请求的 token 对象 
axios.isCancel(): 是否是一个取消请求的错误 
axios.all(promises): 用于批量执行多个异步请求 
axios.spread(): 
```
​

目录结构
```
├── adapters
│   ├── README.md
│   ├── http.js   // Node环境请求
│   └── xhr.js    // 浏览器环境请求
├── axios.js      // 入口文件 初始化
├── cancel        // 请求取消模块
│   ├── Cancel.js  
│   ├── CancelToken.js
│   └── isCancel.js
├── core          // 核心模块
│   ├── Axios.js  // 初始化
│   ├── InterceptorManager.js   // 拦截器管理
│   ├── buildFullPath.js
│   ├── createError.js
│   ├── dispatchRequest.js       // 请求调用
│   ├── enhanceError.js
│   ├── mergeConfig.js
│   ├── settle.js
│   └── transformData.js         // 数据转换
├── defaults.js                  // 默认配置
├── env
│   ├── README.md
│   └── data.js
├── helpers
│   ├── README.md
│   ├── bind.js                  // bind
│   ├── buildURL.js
│   ├── combineURLs.js
│   ├── cookies.js
│   ├── deprecatedMethod.js
│   ├── isAbsoluteURL.js
│   ├── isAxiosError.js
│   ├── isURLSameOrigin.js
│   ├── normalizeHeaderName.js
│   ├── parseHeaders.js
│   ├── spread.js
│   └── validator.js
└── utils.js          // 类型判断、对象方法（forEach、merge、extend）
```
​

## 初始化
**带着这个问题开始  axios与Axios的关系？**
入口文件lib/axios.js，通过 **工厂函数 createInstance 初始化**，得到axios，并给axios挂载Cancel、CancelToken等方法。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5581a72d43d7414e81f610a0d1d2eefd~tplv-k3u1fbpfcp-zoom-1.image)


​

初始化的流程开始从工厂函数 createInstance开始：

1. Axios构造函数生成对象带有defaults属性和拦截器属性的对象
1. **bind(request, content) 返回一个闭包的request方法。**
1. 拷贝Axios.prototype(get/post/request) 
1. 拷贝context上的 defaults 和 拦截器属性。



![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc7acb10099b4fb393e04b7029c44590~tplv-k3u1fbpfcp-zoom-1.image)
​

#### 逐行分析 + debugger：

1. Axios构造函数生成对象带有defaults属性和拦截器属性的对象
1. **bind(request, content) 返回一个闭包的request方法。**
```sql
var context = new Axios(defaultConfig);
var instance = bind(Axios.prototype.request, context);
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2495b480e814127a072d36cad79f7f9~tplv-k3u1fbpfcp-zoom-1.image)

3. 拷贝Axios.prototype(get/post/request) 
```sql
utils.extend(instance, Axios.prototype, context);
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68ff11904a3545f680057f002f236dbc~tplv-k3u1fbpfcp-zoom-1.image)
**​**


4. 拷贝context上的 defaults 和 拦截器属性。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40450fe97003425d8ea621ef1b028f36~tplv-k3u1fbpfcp-zoom-1.image)


Axios构造函数：
delete、get等方法都是request的别名，只不过是合并了一下不同的配置config配置。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f3c1979227540d5a3eec61e5973a448~tplv-k3u1fbpfcp-zoom-1.image)


### axios 与 Axios 的关系

1. 语法上：axios 不是Axios的对象实例
1. 功能上：axios 具备 Axios实例上的所有功能（方法：Axios.porotype、属性：defaults/interceptors）
1. **axios 是 Axios.porotype.request 函数 bind返回的 warp 函数，可直接使用**。



### axios.created 创建的对象 与 axios 的区别
相同：直接发送请求、具有request、get/post方法，具有defaults/interceptors属性。
不同：**新的instance 不具有 Cancel、CancelToken方法**。




![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39b51e1e7168475489a9bb95a39704c2~tplv-k3u1fbpfcp-zoom-1.image)




15
## 整体流程
#### 责任链模式
> 行为模式负责对象间的高效沟通和职责委派。
> ​

> **责任链模式**是一种行为设计模式， 允许你将请求沿着处理者链进行发送。 收到请求后， 每个处理者均可对请求进行处理， 或将其传递给链上的下个处理者。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b8ee45588c6b491582ea5797ea6cbded~tplv-k3u1fbpfcp-zoom-1.image)


模块流程：
Axios.porotype.request(config) ==> despatchRequest(config) ==> adapter(config)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67e90f6988b0482aa0420da37c1dcbc7~tplv-k3u1fbpfcp-zoom-1.image)
​

重要模块与职责：

1. request： 请求拦截 => despatchRequest => 响应拦截，通过promise串联
1. despatchRequest： 请求数据转换 => adapter => 响应数据转换
1. adpter：根据环境确定请求函数 返回promise



整体流程：
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26fa4c002c574c2f9ac5c01a3f863fc9~tplv-k3u1fbpfcp-zoom-1.image)
​

​

## 转换数据 despatchRequest
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1dd24ddd71f457bba35d66eaf5fecc4~tplv-k3u1fbpfcp-zoom-1.image)
​

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72611ca0631f451793d33098428ecc5a~tplv-k3u1fbpfcp-zoom-1.image)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e27b0982b364db2a85eaeae357803df~tplv-k3u1fbpfcp-zoom-1.image)
​

默认转换
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b437dff8f77b46d389c86b269d1d8dc8~tplv-k3u1fbpfcp-zoom-1.image)
​

## 适配器
> 结构型模式介绍如何将对象和类组装成较大的结构， 并同时保持结构的灵活和高效。
**适配器模式**是一种结构型设计模式， 它能使接口不兼容的对象能够相互合作。

​

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c813ac1d1a7d4eb9b72878578e44d35c~tplv-k3u1fbpfcp-zoom-1.image)


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54c23a0e88354f75a9509076d4d76d65~tplv-k3u1fbpfcp-zoom-1.image)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4971605580de4afeb2b6d2b621f65d52~tplv-k3u1fbpfcp-zoom-1.image)


## 拦截器
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d9db3c6abbd4a5fb5fead75e827274b~tplv-k3u1fbpfcp-zoom-1.image)










### 拦截器管理


**取消拦截器？**
**拦截器执行顺序？**
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8640e451543f4c2bbb9996d5b7941337~tplv-k3u1fbpfcp-zoom-1.image)




lib/core/InterceptorManager.js

- user：添加
- eject：删除
- forEach：获取所有

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be60083f6201418a9bbab9799a10f2c6~tplv-k3u1fbpfcp-zoom-1.image)


添加/取消拦截器
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc8f2b5341fa4181bcf2fe4562b3a050~tplv-k3u1fbpfcp-zoom-1.image)
​

​

lib/core/Axios.js  **Axios.prototype.request**
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5f23db8338b4b30b9f44bc277cb932a~tplv-k3u1fbpfcp-zoom-1.image)
### 异步执行
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a20a2a95ec7420594b541697fc221c8~tplv-k3u1fbpfcp-zoom-1.image)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0af9955c127419382709d939fbaa3a3~tplv-k3u1fbpfcp-zoom-1.image)


### 同步执行
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91170dc400e8457a92b59059e3ad427b~tplv-k3u1fbpfcp-zoom-1.image)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6fd3b0fbb12461c9cb795d5201da31e~tplv-k3u1fbpfcp-zoom-1.image)


### 执行顺序
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17cf0596de65468f92771dcf1e20f358~tplv-k3u1fbpfcp-zoom-1.image)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa67dec98fa4487c8dffbc41224947b6~tplv-k3u1fbpfcp-zoom-1.image)




写一个demo
```sql
const config = {a:'111'}
const promise = Promise.resolve(config);
const arr = [promise]
let resault = promise

arr.push((config) => {
    config['user1'] = 111
    console.log(config, '111')
    return config
})

arr.push((config) => {
    config['user2'] = 2222
    console.log(config, '222')
    return config
})

arr.push((config) => {
    config['user3'] = 3333
    console.log(config, '333')
    return config
})

while (arr.length) {
    resault = promise.then(arr.shift())
}
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b144847eb274400db84221d14eaf7b33~tplv-k3u1fbpfcp-zoom-1.image)
## 取消请求
取消的使用

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad71e5f80547421e903d51940cf40fca~tplv-k3u1fbpfcp-zoom-1.image)
​

lib/cancel/CancelToken.js  **主函数与subscribe**
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f25715bcb1164575968fbef8254cd823~tplv-k3u1fbpfcp-zoom-1.image)


订阅函数
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6488499b46a4d349e22bcc0f989c53e~tplv-k3u1fbpfcp-zoom-1.image)
​

第一步 生成
cancelToken = new axios.CancelToken(c => cancel = c)   
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef146c5b72494f74a480023072784b24~tplv-k3u1fbpfcp-zoom-1.image)
​

第二步
xhr内  cancelToken.subscribe 存储取消事件
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab8d364d94cc4892ada8e1f35834cd80~tplv-k3u1fbpfcp-zoom-1.image)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/819ff9b682e94878b9861fabbbd83f9e~tplv-k3u1fbpfcp-zoom-1.image)
​

第三步
执行通过subscribe存储的取消事件
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e005fcbb3f14898b9c90b49f57531ed~tplv-k3u1fbpfcp-zoom-1.image)


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66fd77ef63aa4f50bdb906b9f00dd57c~tplv-k3u1fbpfcp-zoom-1.image)
​

观察者模式
> **观察者模式**是一种行为设计模式， 允许你定义一种订阅机制， 可在对象事件发生时通知多个 “观察” 该对象的其他对象。



![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c79aa6c1f6c1447eaf1535b60ba197b9~tplv-k3u1fbpfcp-zoom-1.image)
​

​

**观察者模式与发布订阅的关系 讨论？**
观察者模式包含发布订阅。
发布订阅是观察者的升级变种。
观察者模式与发布订阅的区别是否有事件调度中心。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6cf25846770474aa0719cdc02078688~tplv-k3u1fbpfcp-zoom-1.image)


## 回顾


1. axios 、Axios 的区别、初始化流程
1. config责任链，关键模块 request、despatchRequest、adpter 的职责, 请求整体流程。
1. despatchRequest的default转换 自动转json
1. 适配器实现
1. request 拦截器实现，取消拦截器、执行顺序。
1. 取消请求的实现

​

感悟：

- 模块职责清晰
- 适配器易扩展
- 拦截器、格式转换灵活
- 设计模式的分类：创建型、结构型、行为型
- 技巧与方法 && 持续学习

[
](https://refactoringguru.cn/design-patterns/catalog)


