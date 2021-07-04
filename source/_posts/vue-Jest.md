---
title: Vue业务系统落地单元测试
---
一直对单测很感兴趣，但对单测覆盖率、测试报告等关键词懵懵懂懂，最近几个月一直在摸索如何在Vue业务系统中落地单元测试，看到慢慢增长的覆盖率，慢慢清晰的模块，对单元测试的理解也比以前更加深入，也有一些心得和收获。

今天把自己的笔记分享出来，和大家一起交流我在2个较为复杂的Vue业务系统中落地单测的一些思路和方法，算是入门实践类的笔记，资深大佬还请跳过。


## 大纲
1. 定义
2. **安装与使用**
3. **常用API**
4. **落地单元测试**
5. **演进：构建可测试的单元模块**
6. **可维护的单元模块**
7. 回顾
8. 讨论 && Thank


## 1. 定义

**单元测试定义：**

单元测试是指对软件中的**最小可测试单元进行检查和验证**。单元在质量保证中是非常重要的环节，根据测试金字塔原理，越往上层的测试，所需的测试投入比例越大，效果也越差，而单元测试的成本要小的多，也更容易发现问题。


也有不同的测试分层策略（冰淇淋模型、冠军模型）。




**![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c916b65f711945a19d6f61ecdc386882~tplv-k3u1fbpfcp-zoom-1.image)**



## 2. 安装与使用
#### 1. vue项目添加 @vue/unit-jest [文档](https://vue-test-utils.vuejs.org/zh/guides/#%E5%AE%89%E8%A3%85)
```bash
$ vue add @vue/unit-jest
```
安装完成后，在package.json中会多出test:unit脚本选项，并生成jest.config.js文件。
```javascript
// package.json
{
  "name": "avatar",
  "scripts": {
    "test:unit": "vue-cli-service test:unit", // 新增的脚本
  },
  "dependencies": {
    ...
  },
  "devDependencies": {
    ...
  },
}
```


生成测试报告的脚本：增加--coverage自定义参数
```javascript
// package.json
{
  "name": "avatar",
  "scripts": {
    "test:unit": "vue-cli-service test:unit",
    "test:unitc": "vue-cli-service test:unit  --coverage", // 测试并生成测试报告
  },
  "dependencies": {
    ...
  },
  "devDependencies": {
    ...
  },
}
```
#### 2. VScode vscode-jest-runner 插件配置
作用：VS Code打开测试文件后，可直接运行用例。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4df9e177d49140d3833c73235b1ff00a~tplv-k3u1fbpfcp-zoom-1.image)
运行效果：
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2617ab4e4ba24c6c825ae557e0ef8284~tplv-k3u1fbpfcp-zoom-1.image)
不通过效果：
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7adf5ee537f342948bf8e1429b8ccf5e~tplv-k3u1fbpfcp-zoom-1.image)


安装插件：[https://marketplace.visualstudio.com/items?itemName=firsttris.vscode-jest-runner](https://marketplace.visualstudio.com/items?itemName=firsttris.vscode-jest-runner)
配置项：设置 => jest-Runner Config


- Code Lens Selector：匹配的文件，**/*.{test,spec}.{js,jsx,ts,tsx}
- Jest Command：定义Jest命令，默认为Jest 全局命令。



将Jest Command替换为 test:unit，使用vue脚手架提供的 test:unit 进行单元测试。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/326c34d8ec3a45e09ed532267c395892~tplv-k3u1fbpfcp-zoom-1.image)
#### 3. githook 配置
作用：在提交时执行所有测试用例，有测试用例不通过或覆盖率不达标时取消提交。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/571482fd44e6457694730be6fe93ccb6~tplv-k3u1fbpfcp-zoom-1.image)
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8063d92f2b9244cf9e57306460369803~tplv-k3u1fbpfcp-zoom-1.image)


安装：
```bash
$ npm install husky --save-dev
```
配置：
```javascript
// package.json
{
  "name": "avatar",
  "scripts": {
    "test:unit": "vue-cli-service test:unit",
    "test:unitc": "vue-cli-service test:unit  --coverage", // 测试并生成测试报告
  },
  "husky": {
    "hooks": {
      "pre-commit": "npm run test:unitc" // commit时执行参单元测试 并生成测试报告
    }
  },
}
```
设置牵引指标：jest.config.js，可全局设置、对文件夹设置、对单个文件设置。
```javascript
module.exports = {
  preset: '@vue/cli-plugin-unit-jest',
  timers: 'fake',
  coverageThreshold: {
   global: { // 全局
      branches: 10,
      functions: 10,
      lines: 10,
      statements: 10
    },
    './src/common/**/*.js': { // 文件夹
      branches: 0,
      statements: 0
    },
    './src/common/agoraClientUtils.js': { // 单个文件
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
}

```




#### 4. 测试报告
生成的测试报告在跟目录下的`coverage`文件夹下，主要是4个指标。

> - 语句覆盖率（statement coverage）每个语句是否都执行
> - 分支覆盖率（branch coverage）每个if代码块是否都执行
> - 函数覆盖率（function coverage）每个函数是否都调用
> - 行覆盖率（line coverage) 每一行是否都执行了


根目录截图
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad0e6c648ab14d048149a6eb149e606b~tplv-k3u1fbpfcp-zoom-1.image)
文件夹目录截图：三种颜色代表三种状态：红色、黄色、绿色。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d4d1d67eec04889820a6a0736debe3e~tplv-k3u1fbpfcp-zoom-1.image)
单个文件截图：红色行为未覆盖，绿色行为运行次数。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a4abfd2c214484e8ee7dcd24b64dcd5~tplv-k3u1fbpfcp-zoom-1.image)



## 3. 常用API

抛砖引玉，只展示简单的用法，具体可参见文档。

**Jest常用方法：**
[文档](https://jestjs.io/zh-Hans/)
```javascript
// 例子
describe('versionToNum 版本号转数字', () => {
  it('10.2.3 => 10.2', () => {
    expect(versionToNum('10.2.3')).toBe(10.2)
  })
  it('11.2.3 => 11.2', () => {
    expect(versionToNum('11.2.3')).toBe(11.2)
  })
})

/*------------------------------------------------*/

// 值对比
expect(2 + 2).toBe(4); 
expect(operationServe.operationPower).toBe(true)
// 对象对比
expect(data).toEqual({one: 1, two: 2}); 
// JSON 对比
expect(data).toStrictEqual(afterJson)


// 每次执行前
beforeEach(() => {
	// do  some thing....
  // DOM 设置
  document.body.innerHTML = `<div id="pc" class="live-umcamera-video" style="position: relative;">
        <div style="width:200px; height:300px; position:absolute; top:20px; left:500px;">
            <video style="width:300px; height:400px;"
                autoplay="" muted="" playsinline=""></video>
        </div>
    </div>`
})

// Mock
const getCondition =  jest.fn().mockImplementation(() => Promise.resolve({ ret: 0, content: [{ parameterName: 'hulala' }] }))

// Promise 方法
it('获取预置埋点 - pages', () => {
  return getCondition('hz', 'pages').then(() => {
    // logType不包含presetEvent、不等于 pages，获取预置埋点
    expect($api.analysis.findPresetList).toBeCalled()
})

// 定时器方法  
it('定时器 新建 执行', () => {
  const timer = new IntervalStore()
  const callback = jest.fn()
  timer.start('oneset', callback, 2000)
  expect(callback).not.toBeCalled()
  jest.runTimersToTime(2000) // 等待2秒
  expect(callback).toBeCalled()
})
 
```


**@vue/test-utils常用方法：**
[文档](https://vue-test-utils.vuejs.org/zh/)
```javascript
// 例子
import { mount } from '@vue/test-utils'
import Counter from './counter'

describe('Counter', () => {
  // 现在挂载组件，你便得到了这个包裹器
  const wrapper = mount(Counter)

  it('renders the correct markup', () => {
    expect(wrapper.html()).toContain('<span class="count">0</span>')
  })

  // 也便于检查已存在的元素
  it('has a button', () => {
    expect(wrapper.contains('button')).toBe(true)
  })
})

/*------------------------------------------------*/


import { shallowMount, mount, render, renderToString, createLocalVue } from '@vue/test-utils'
import Component from '../HelloWorld.vue'

// router模拟
import VueRouter from 'vue-router'
const localVue = createLocalVue()
localVue.use(VueRouter)
shallowMount(Component, { localVue })

// 伪造
const $route = {
  path: '/some/path'
}
const wrapper = shallowMount(Component, {
  mocks: {
    $route
  }
})


// store 模拟
const store = new Vuex.Store({
      state: {},
      actions
 })
shallowMount(Component, { localVue, store })


it('错误信息展示', async () => {
    // shallowMount  入参模拟
    const wrapper = shallowMount(cloudPhone, {
      propsData: {
        mosaicStatus: false,
        customerOnLine: true,
        cloudPhoneState: false,
        cloudPhoneError: true,
        cloudPhoneTip: '发生错误',
        delay: ''
      }
    })
    
    
    // 子组件是否展示
    expect(wrapper.getComponent(Tip).exists()).toBe(true)
    // html判断
    expect(wrapper.html().includes('发生错误')).toBe(true)
    // DOM 元素判断
    expect(wrapper.get('.mosaicStatus').isVisible()).toBe(true)
   	// 执行点击事件
    await wrapper.find('button').trigger('click')
  	// class
    expect(wrapper.classes()).toContain('bar')
    expect(wrapper.classes('bar')).toBe(true)
		// 子组件查找  	
  	wrapper.findComponent(Bar)
    // 销毁
   	wrapper.destroy()
    // 
   	wrapper.setData({ foo: 'bar' })
  	
  	// axios模拟
   	jest.mock('axios', () => ({
      get: Promise.resolve('value')
    }))
  
  })
```



## 4. 落地单元测试



❌ 直接对一个较大的业务组件添加单元测试，需要模拟一系列的全局函数，无法直接运行。

**问题：**
1. 逻辑多：业务逻辑不清楚，1000+ 行
2. 依赖多：`$dayjs、$api、$validate、$route、$echarts、mixins、$store`...
3. 路径不一致：有`@`、`./`、`../`


> 单元测试是用来对一个模块、一个函数或者一个类来进行正确性检验的测试工作。
> -- 廖雪峰的官方网站


**落地：**

✅ 对业务逻辑关键点，**抽出纯函数、类方法、组件，并单独增加测试代码**。


**例子：**
获取分组参数，由7个接口聚合。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59831dcee1794629955d3d5a0ad2618f~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b06a2326738f40d1ab942eb6540e84d4~tplv-k3u1fbpfcp-watermark.image)


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b45f82ac9d542788aa3b2126d1252d8~tplv-k3u1fbpfcp-watermark.image)

原有逻辑：
系统参数存全局变量，自定义参数存全局变量

- 无法看出多少种类型与接口数量
- 无法在多个位置直接复用
```javascript
getCondition (fIndex, oneFunnel) { // 添加限制条件，如果该事件没有先拉取
      const {biz, logType, event, feCreateType} = oneFunnel
      return new Promise((resolve, reject) => {
        // 私有限制条件为空，且不是预置事件 或 页面组，就拉取私有限制条件
        try {
          this.$set(this.extraParamsList.parameterList, fIndex, {})
          if (logType !== 'pages' && logType.indexOf('presetEvent') === -1) {
            this.$api.analysis[`${logType}ParameterList`]({
              biz: logType === 'server' && feCreateType === 0 ? '' : biz,
              event: event,
              terminal: this.customType[logType],
              platform: logType === 'server' && feCreateType === 0 ? 'common' : '',
              pageNum: -1
            }).then(res => {
              if (res.ret === 0) {
                res.content.forEach(element => {
                  this.$set(this.extraParamsList.parameterList[fIndex], element.parameterName || element.parameter_name, element)
                })
                resolve()
              } else {
                reject('获取事件属性失败，请联系后台管理员')
              }
            })
          } else if ((logType === 'presetEvents' ||  logType === 'presetEventsApp')) {
            this.$api.analysis.findPresetList({
              biz,
              appTerminal: logType,
              operation: event
            }).then(res => {
              if (res.code === 0) {
                res.data.forEach(item => {
                  item.description = item.name
                  this.$set(this.extraParamsList.parameterList[fIndex], item.name, item)
                })
                resolve()
              }
            })
          } else {
            resolve('无需拉取')
          }
        } catch (e) {
          reject(e)
        }
      })
    },
      
     getGlobalCondition (funnelId) { // 获取 全局 基础选项
      return new Promise((resolve, reject) => {
        this.$api.analysis.getGlobalCondition({
          funnelId: funnelId,
          type: this.conditionMode
        }).then(res => {
          if (res.code === 0) {
            const {bizList, expressions, expressionsNumber, comBizList} = res.data
            this.bizList = Object.assign(...bizList)
            this.comBizList = Object.assign(...comBizList)
            this.comBizKeyList = Object.keys(this.comBizList)
            this.operatorList = expressions
            this.numberOperatorList = expressionsNumber
            this.comBizKey = Object.keys(this.comBizList)
            this.getComBizEvent()
            resolve(res)
          } else {
            this.$message.error('获取基础选项失败，请联系后台管理员')
            reject('获取基础选项失败，请联系后台管理员')
          }
        })
      })
    },  
      
   setCommonPropertiesList (data) { // 初始化 公共限制条件列表 commonPropertiesList
      const commonPropertiesList = {
        auto: data.h5AutoCommonProperties,
        pages: data.h5PagesCommonProperties,
        presetEvents: data.h5PresetCommonProperties, // h5 预置事件 公共属性
        customH5: data.h5CustomCommonProperties,
        customApp: data.appCustomCommonProperties,
        presetEventsApp: data.appPresetCommonProperties, // App 预置事件 公共属性
        server: data.serverCommonProperties,
        customWeapp: data.weappCustomCommonProperties,
        presetEventsWeapp: data.weappPresetCommonProperties, // Weapp 预置事件 公共属性
        presetEventsServer: data.serverPresetCommonProperties || [], // Server 预置事件 公共属性
        presetEventsAd: data.adPresetCommonProperties
      }
      for (let type in commonPropertiesList) { // 将parameter_name的值作为key，item作为value，组合为k-v形式
        let properties = {}
        if (!commonPropertiesList[type]) continue
        commonPropertiesList[type].forEach(item => {
          properties[item.parameter_name] = item
        })
        commonPropertiesList[type] = properties
      }
      this.commonPropertiesList = commonPropertiesList
    },
      
```
拆分模块后：
建立GetParamsServer主类，该类由2个子类构成，并聚合子类接口。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/754d1c384cf841c69b5a558c87341f6a~tplv-k3u1fbpfcp-zoom-1.image)




这是其中一个子类，获取私有参数的单元测试：
```javascript
import GetParamsServer, { GetPrivateParamsServer } from '@/views/analysis/components/getParamsServer.js'

describe('GetPrivateParamsServer 私有参数获取', () => {
  let $api
    beforeEach(() => {
      $api = {
        analysis: {
          findPresetList: jest.fn().mockImplementation(() => Promise.resolve({
            code: 0, data: [{ name: 'hulala', description: '234234', data_type: 'event' }]
          })), // 预置埋点
          serverParameterList: jest.fn().mockImplementation(() => Promise.resolve({
            ret: 0, content: [{ parameterName: 'hulala' }]
          })), // 服务端埋点
          autoParameterList: jest.fn().mockImplementation(() => Promise.resolve({
            ret: 0, content: [{ parameter_name: 'hulala' }]
          })), // H5全埋点
          customH5ParameterList: jest.fn().mockImplementation(() => Promise.resolve({
            ret: 0, content: [{ parameterName: 'hulala' }]
          })), // H5自定义
          customWeappParameterList: jest.fn().mockImplementation(() => Promise.resolve({
            ret: 0, content: [{ parameter_name: 'hulala', description: '234234', data_type: 'event' }]
          })), // Weapp自定义
          customAppParameterList: jest.fn().mockImplementation(() => Promise.resolve({
            ret: 0, content: [{ parameterName: 'hulala', description: 'asdfafd', data_type: 'event' }]
          })) // App自定义
        }
      }
    })
  describe('GetPrivateParamsServer 不同类型获取', () => {
    it('获取预置埋点 - pages', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getCondition('hz', 'pages').then(() => {
        // logType不包含presetEvent、不等于 pages，获取预置埋点
        expect($api.analysis.findPresetList).toBeCalled()
      })
    })

    it('获取预置埋点 - presetEvent ', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getCondition('hz', 'presetEvent').then(() => {
        // logType不包含presetEvent、不等于 pages，获取预置埋点
        expect($api.analysis.findPresetList).toBeCalled()
      })
    })

    it('获取非预置埋点 - 其他', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getCondition('hz', '12312').then(() => {
        expect($api.analysis.findPresetList).not.toBeCalled()
      })
    })
    

    it('获取非预置埋点 - server', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getCondition('hz', 'server').then(() => {
        expect($api.analysis.serverParameterList).toBeCalled()
      })
    })

    it('获取非预置埋点 - auto', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getCondition('hz', 'auto').then(() => {
        expect($api.analysis.autoParameterList).toBeCalled()
      })
    })

    it('获取非预置埋点 - customH5', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getCondition('hz', 'customH5').then(() => {
        expect($api.analysis.customH5ParameterList).toBeCalled()
      })
    })

    it('获取非预置埋点 - customWeapp', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getCondition('hz', 'customWeapp').then(() => {
        expect($api.analysis.customWeappParameterList).toBeCalled()
      })
    })

    it('获取非预置埋点 - customApp', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getCondition('hz', 'customApp').then(() => {
        expect($api.analysis.customAppParameterList).toBeCalled()
      })
    })

    it('获取非预置埋点 - 不存在类型', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getCondition('hz', '哈哈哈哈').then(res => {
        expect(res.length).toBe(0)
      })
    })
  })


  describe('GetPrivateParamsServer 结果转换为label', () => {
    it('获取预置埋点 - pages', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getConditionLabel('hz', 'pages').then((res) => {
        expect(res.length).toBe(1)
        expect(!!res[0].value).toBeTruthy()
        expect(!!res[0].label).toBeTruthy()
        expect(res[0].types).toBe('custom')
        expect(res[0].dataType).toBe('event')
      })
    })

    it('获取非预置埋点 - customWeapp', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getConditionLabel('hz', 'customWeapp').then((res) => {
        expect(res.length).toBe(1)
        expect(!!res[0].value).toBeTruthy()
        expect(!!res[0].label).toBeTruthy()
        expect(res[0].types).toBe('custom')
        expect(res[0].dataType).toBe('event')
      })
    })

    it('获取非预置埋点 - customApp', () => {
      const paramsServer = new GetPrivateParamsServer()
      paramsServer.initApi($api)
      return paramsServer.getConditionLabel('hz', 'customApp').then((res) => {
        expect(res.length).toBe(1)
        expect(!!res[0].value).toBeTruthy()
        expect(!!res[0].label).toBeTruthy()
        expect(res[0].types).toBe('custom')
        expect(res[0].dataType).toBe('event')
      })
    })
  })
})
```


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/415670ec89c54c75b61c2f1b1200ef84~tplv-k3u1fbpfcp-zoom-1.image)

**从测试用例看到的代码逻辑：**

- 6个接口
- 6种事件类型
- 类型与接口的对应关系
- 接口格式有三种



**作用：**

1. 复用：将复杂的业务逻辑封闭在黑盒里，更方便复用。
1. 质量：模块的功能通过测试用例得到保障。
1. 维护：测试即文档，方便了解业务逻辑。


实践：在添加单测的过程中，抽象模块，重构部分功能，并对单一职责的模块增加单测。




## 5. 演进：构建可测试单元模块


将业务代码代码演变为可测试代码，重点在：


1. 设计：将业务逻辑**拆分为单元模块**（UI组件、功能模块）。
1. 时间：可行的重构目标与重构方法，要有**长期重构**的心理预期。


**为单一职责的模块设计测试用例，才会对功能覆盖的更全面，所以设计这一步尤为重要。**


> 如果挽救一个系统的办法是重新设计一个新的系统，那么，**我们有什么理由认为从头开始，结果会更好呢**? 
> --《架构整洁之道》


原来模块也是有设计，我们如何保证重构后真的比之前更好吗？还是要根据设计原则客观的来判断。



**设计原则 SOLID：**

- SRP-单一职责
- OCP-开闭：易与扩展，抗拒修改。
- LSP-里氏替换：子类接口统一，可相互替换。
- ISP-接口隔离：不依赖不需要的东西。
- DIP-依赖反转：构建稳定的抽象层，单向依赖（例：A => B => C， 反例：A  => B => C => A）。


在应接不暇的需求面前，还要拆模块、重构、加单测，无疑是增加工作量，显得不切实际，《重构》这本书给了我很多指导。

**重构方法：**

- 预备性重构
- 帮助理解的重构
- 捡垃圾式重构（营地法则：遇到一个重构一个，像见垃圾一样，让你离开时的代码比来时更干净、健康）
- 有计划的重构与见机行事的重构
- 长期重构



**业务系统1的模块与UI梳理：**

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d341a479c8349418e1be072284936e2~tplv-k3u1fbpfcp-watermark.image)


**业务系统2的模块与UI梳理：**

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af1688492c8449aabf15f1797b0569f6~tplv-k3u1fbpfcp-watermark.image)


## 6. 可维护的单元模块


> 避免重构后再次写出坏味道的代码，提取执行成本更低的规范。



**代码坏味道：**

- **神秘命名-无法取出好名字，背后可能潜藏着更深的设计问题。**
- 重复代码
- **过长函数-小函数、纯函数**。
- 过长参数
- **全局数据-数量越多处理难度会指数上升。**
- 可变数据-不知道在哪个节点修改了数据。
- 发散式变化-只关注当前修改，不用关注其他关联。
- **霰弹式修改-修改代码散布四处**。
- 依恋情结-与外部模块交流数据胜过内部数据。
- **数据泥团-相同的参数在多个函数间传递。**
- 基本类型偏执
- 重复的switch
- 循环语句
- 冗赘的元素
- 夸夸其谈通用性
- 临时字段
- 过长的消息链
- 中间人
- 内幕交易
- **过大的类**
- 异曲同工的类
- 纯数据类
- 被拒绝的遗赠-继承父类无用的属性或方法
- **注释-当你感觉需要撰写注释时，请先尝试重构，试着让所有注释都变得多余。**



**规范：**

- 全局变量数量：20 ±
- 方法方法行数：15 ±
- 代码行数：300-500
- 内部方法、内联方法：下划线开头



**技巧：**

- 使用class语法：将紧密关联的方法和变量封装在一起。
- 使用Eventemitter 工具库：实现简单发布订阅。
- 使用vue  provide语法：传递实例。
- 使用koroFileHeader插件： 统一注释规范。
- 使用Git-commit-plugin插件： 统一commit规范。
- 使用eslint + stylelint（未使用变量、误改变量名、debugger，自动优化的css）。

示例代码：
```javascript
/*
 * @name: 轻量级message提示插件
 * @Description: 模仿iview的$message方法，api与样式保持一致。
 */

class Message {
    constructor() {
        this._prefixCls = 'i-message-';
        this._default = {
            top: 16,
            duration: 2
        }
    }
    info(options) {
        return this._message('info', options);
    }
    success(options) {
        return this._message('success', options);
    }
    warning(options) {
        return this._message('warning', options);
    }
    error(options) {
        return this._message('error', options);
    }
    loading(options) {
        return this._message('loading', options);
    }
    config({ top = this._default.top, duration = this._default.duration }) {
        this._default = {
            top,
            duration
        }
        this._setContentBoxTop()
    }
    destroy() {
        const boxId = 'messageBox'
        const contentBox = document.querySelector('#' + boxId)
        if (contentBox) {
            document.body.removeChild(contentBox)
        }
        this._resetDefault()
    }

    /**
     * @description: 渲染消息
     * @param {String} type 类型
     * @param {Object | String} options 详细格式
     */
    _message(type, options) {
        if (typeof options === 'string') {
            options = {
                content: options
            };
        }
        return this._render(options.content, options.duration, type, options.onClose, options.closable);
    }

    /**
     * @description: 渲染消息
     * @param {String} content 消息内容
     * @param {Number} duration 持续时间
     * @param {String} type 消息类型
     */
    _render(content = '', duration = this._default.duration, type = 'info',
        onClose = () => { }, closable = false
    ) {
        // 获取节点信息
        const messageDOM = this._getMsgHtml(type, content, closable)
        // 插入父容器
        const contentBox = this._getContentBox()
        contentBox.appendChild(messageDOM);
        // 删除方法
        const remove = () => this._removeMsg(contentBox, messageDOM, onClose)
        let removeTimer
        if(duration !== 0){
            removeTimer = setTimeout(remove, duration * 1000);
        }
        // 关闭按钮
        closable && this._addClosBtn(messageDOM, remove, removeTimer)
    }

    /**
     * @description: 删除消息
     * @param {Element} contentBox 父节点
     * @param {Element} messageDOM 消息节点
     * @param {Number} duration 持续时间
     */
    _removeMsg(contentBox, messageDOM, onClose) {
        messageDOM.className = `${this._prefixCls}box animate__animated animate__fadeOutUp`
        messageDOM.style.height = 0
        setTimeout(() => {
            contentBox.removeChild(messageDOM)
            onClose()
        }, 400);
    }

    /**
     * @description: 获取图标
     * @param {String} type
     * @return {String} DOM HTML 字符串
     */
    _getIcon(type = 'info') {
        const map = {
            info: `<svg style="color:#2db7f5" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
           <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
         </svg>`,
            success: `<svg style="color:#19be6b"  xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
           <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd" />
         </svg>`,
            warning: `<svg style="color:#ff9900" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
           <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
         </svg>`,
            error: `<svg style="color:#ed4014" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
           <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
         </svg>`,
            loading: `<svg style="color:#2db7f5" xmlns="http://www.w3.org/2000/svg" class="loading" viewBox="0 0 20 20" fill="currentColor">
           <path fill-rule="evenodd" d="M9.504 1.132a1 1 0 01.992 0l1.75 1a1 1 0 11-.992 1.736L10 3.152l-1.254.716a1 1 0 11-.992-1.736l1.75-1zM5.618 4.504a1 1 0 01-.372 1.364L5.016 6l.23.132a1 1 0 11-.992 1.736L4 7.723V8a1 1 0 01-2 0V6a.996.996 0 01.52-.878l1.734-.99a1 1 0 011.364.372zm8.764 0a1 1 0 011.364-.372l1.733.99A1.002 1.002 0 0118 6v2a1 1 0 11-2 0v-.277l-.254.145a1 1 0 11-.992-1.736l.23-.132-.23-.132a1 1 0 01-.372-1.364zm-7 4a1 1 0 011.364-.372L10 8.848l1.254-.716a1 1 0 11.992 1.736L11 10.58V12a1 1 0 11-2 0v-1.42l-1.246-.712a1 1 0 01-.372-1.364zM3 11a1 1 0 011 1v1.42l1.246.712a1 1 0 11-.992 1.736l-1.75-1A1 1 0 012 14v-2a1 1 0 011-1zm14 0a1 1 0 011 1v2a1 1 0 01-.504.868l-1.75 1a1 1 0 11-.992-1.736L16 13.42V12a1 1 0 011-1zm-9.618 5.504a1 1 0 011.364-.372l.254.145V16a1 1 0 112 0v.277l.254-.145a1 1 0 11.992 1.736l-1.735.992a.995.995 0 01-1.022 0l-1.735-.992a1 1 0 01-.372-1.364z" clip-rule="evenodd" />
         </svg>`
        }
        return map[type]
    }

    /**
     * @description: 获取消息节点
     * @param {String} type 类型
     * @param {String} content 消息内容
     * @return {Element} 节点DOM对象
     */
    _getMsgHtml(type, content) {
        const messageDOM = document.createElement("div")
        messageDOM.className = `${this._prefixCls}box animate__animated animate__fadeInDown`
        messageDOM.style.height = 36 + 'px'
        messageDOM.innerHTML = `
                <div class="${this._prefixCls}message" >
                    ${this._getIcon(type)}
                    <div class="${this._prefixCls}content-text">${content}</div>
                </div>
        `
        return messageDOM
    }

    /**
     * @description: 添加关闭按钮
     * @param {Element} messageDOM 消息节点DOM
     */
    _addClosBtn(messageDOM, remove, removeTimer) {
        const svgStr = `<svg class="${this._prefixCls}btn" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
        </svg>`
        const closBtn = new DOMParser().parseFromString(svgStr, 'text/html').body.childNodes[0];
        closBtn.onclick = () => {
            removeTimer && clearTimeout(removeTimer)
            remove()
        }
        messageDOM.querySelector(`.${this._prefixCls}message`).appendChild(closBtn)
    }

    /**
     * @description: 获取父节点容器
     * @return {Element} 节点DOM对象
     */
    _getContentBox() {
        const boxId = 'messageBox'
        if (document.querySelector('#' + boxId)) {
            return document.querySelector('#' + boxId)
        } else {
            const contentBox = document.createElement("div")
            contentBox.id = boxId
            contentBox.style.top = this._default.top + 'px'
            document.body.appendChild(contentBox)
            return contentBox
        }
    }

    /**
     * @description: 重新设置父节点高度
     */
    _setContentBoxTop() {
        const boxId = 'messageBox'
        const contentBox = document.querySelector('#' + boxId)
        if (contentBox) {
            contentBox.style.top = this._default.top + 'px'
        }
    }
    /**
     * @description: 恢复默认值
     */
    _resetDefault() {
        this._default = {
            top: 16,
            duration: 2
        }
    }
}

if (typeof module !== 'undefined' && typeof module.exports !== 'undefined') {
    module.exports = new Message();
} else {
    window.$message = new Message();
}
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f9c76b639ef46999ed7ac6553208dff~tplv-k3u1fbpfcp-zoom-1.image)



## 6. 回顾

- 定义
- 安装与使用（安装、调试、git拦截、测试报告）
- 常用API（jest、vue组件）
- 落地单元测试（拆分关键模块加单测）
- 演进：构建可测试单元模块（设计原则、重构）
- 可维护的单元模块（代码规范）



**落地线路：**

① 安装使用 => ② API学习 => ③ 落地：拆分关键模块加单测 =>  ④ 演进：架构设计与重构 =>  ⑤ 代码规范 


**未来：**

⑥ 文档先行(待探索) 

在较为复杂的业务系统开发过程中，从第一版代码到逐步划分模块、增加单测，还是走了一段弯路。
如果能够养成文档先行的习惯，先设计模块、测试用例，再编写代码，会更高效。


**理解：**

- 单元测试有长期价值，也有执行成本。
- 好的架构设计是单测的土壤，**为单一职责的模块设计单测、增加单元测试更加顺畅**。
- 每个项目的业务形态与阶段不一样，不一定都适合，**找到适合项目的平衡点**。




## 7. 讨论 && Thank


感谢各位能够看到最后，前半部偏干，后半部分偏水，为内部分享笔记，部分代码和图片经过处理，重在分享和大家一起交流，恳请斧正，有收获还请点赞收藏。


