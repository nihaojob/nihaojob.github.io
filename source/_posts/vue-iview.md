---
title: vue + iview 项目实践总结
date: 2021-07-04 12:13:27
tags:
---

> 一直想把一大篇的总结写完、写好，感觉自己拖延太严重还总想写完美，然后好多笔记都死在编辑器里了，以后还按照一个小节一个小节的更新吧，小步快跑😂，先发出来，以后再迭代吧。

最近我们参与开发了一个（年前了）BI项目，前端使用vue全家桶，项目功能基本开发完成，剩下的修修补补，开发过程还算顺畅，期间遇到好多问题，也记录了一下，发出来一起交流，主要是思路，怎么利用vue给的API实现功能，避免大家在同样的坑里待太长时间，如果有更好实现思路可以一起交流讨论😎🤗。

前后端分离形式开发，`vue+vueRouter+vueX+iviewUI+elementUI`，大部分功能我们都用的iviewUI，有部分组件我们用了elementUI，比如表格、日历插件，我们没接mock工具，接口用文档的形式交流，团队氛围比较和谐，三个PHP三个前端，效率还可以，两个前端伙伴比较厉害，第一次使用vue，就承担了90%的开发工作任务，我没到上线就跑回家休陪产假了，特别感谢同事们的支持，我才能回家看娃。

前端其实不太复杂，但是只要用vue开发基本上都会遇到的几个问题，比如菜单组件多级嵌套、刷新后选中当前项、

涉及几个点，表格表头表体合并、文件上传、富文本编辑器、权限树等等。
### 项目介绍

系统的主要功能就是面向各个部门查看报表数据，后端同学们很厉害，能汇总到一个集团的所有数据，各种炫酷大数据技术；

**菜单功能：**
- **数据看板：** 筛选、展示日期和表格分页
- **业务报表：** 报表类型，日期筛选、表格分页
- **数据检索：** 筛选项联动、表格分页
- **损耗地图：** 筛选项、关系图插件
- **展开分析：** 筛选项、分类、卡片、表格
- **系统信息：** 版本发布、步骤条、富文本编辑
- **数据源上传：** 手动上传、表格展示
- **权限管理：** 用户管理、角色管理（权限菜单配置）

**项目预览图：**
    
    
![](https://user-gold-cdn.xitu.io/2019/3/8/1695b9e86c0f2431?w=1008&h=503&f=gif&s=271766)

![](https://user-gold-cdn.xitu.io/2019/3/8/1695ba7011ff8f6a?w=951&h=445&f=gif&s=243118)


![](https://user-gold-cdn.xitu.io/2019/3/8/1695ba693b32253a?w=880&h=425&f=gif&s=61438)




对勾为已更新。

- [x] 1. 使用v-if解决异步传参
- [x] 2. 使用$refs调用子组件方法
- [x] 3. 组件递归实现多级菜单
- [x] 4. 使用watch监听路由参数重新获取数据
- [x] 5. 页面刷新后Menu根据地址选中当前菜单项
- [x] 6. 使用Axios统一状态码判断、统一增加token字段
- [x] 7. 点击左侧菜单选中项点击刷新页面
- [x] 8. 使用Axios.CancelToken切换路由取消请求
- [x] 9. 使用element的table组件实现 表头表体合并
- [x] 10. iview的Menu组件+vuex实现面包屑导航
- [x] 11. iview上传组件手动上传与富文本编辑器接入
- [x] 12. 使用cheerio获取表格数据
- [x] 13. keep-live组件缓存
- [x] 14. 让数据保持单向流动（不要在子组件中操作父组件的数据）

### 1. 使用v-if解决异步传参组件重绘
大部分的交互的流程都是 “ajax请求数据=>传入组件渲染”，很多属性需要异步传入子组件然后进行相关的计算，如果绑定很多computed或者watch，性能开销会很大，而且有些场景并不需要使用computed和watch，我们只需要在最初创建的时候获取一次就够了。

如下gif例子,点击上方TAB后重新刷新折线组件：

![](https://user-gold-cdn.xitu.io/2019/3/8/1695b5069d34e6ff?w=1086&h=549&f=gif&s=74828)

``` HTML
<!--模板-->
<mapBox v-if="mapData" :data="mapData"></mapBox>
```

```JS
<!--点击搜索后执行-->
let This = this
// setp1 重点
this.mapData = false

this.$http
.post('/api/show/mapcondition',{key:key,type:type})
.then(function(response){
// setp2 重点
    this.mapData = response.data
})
```

有时候会出现DOM元素与数据不同步，可以使用使用其他方式让DOM强刷
```
- setTimeou
- $forceUpdate()
- $nextTick()
- $set()
```

### 2. 使用$refs调用子组件方法
有时候会涉及到父组件调用子组件方法的情况，例如，iview的Tree组件暴露出来的`getCheckedAndIndeterminateNodes`方法，详见官网文档[link](https://cn.vuejs.org/v2/guide/components-edge-cases.html#%E8%AE%BF%E9%97%AE%E5%AD%90%E7%BB%84%E4%BB%B6%E5%AE%9E%E4%BE%8B%E6%88%96%E5%AD%90%E5%85%83%E7%B4%A0)。

``` HTML
<!--模板-->
<Tree v-if="menu" :data="menu" show-checkbox multiple ref="Tree"></Tree>
```

```JS
let rules = this.$refs.Tree.getCheckedAndIndeterminateNodes();
```

### 3. 组件递归实现多级菜单
递归组件用的很多，我们的左侧菜单还有无限拆分的表格合并，都用到了递归组件，详见官网链接[link](https://cn.vuejs.org/v2/guide/components-edge-cases.html#%E9%80%92%E5%BD%92%E7%BB%84%E4%BB%B6)。

效果图：

![](https://user-gold-cdn.xitu.io/2019/3/8/1695b53d364ac870?w=207&h=534&f=gif&s=142519)


大致思路就是先创建一个子组件，然后再创建一个父组件，循环引用，拿左侧菜单说明，代码如下，数据结构也在父组件中。

```html
<!--index.vue  父组件 数据接口在default中-->
<template>
    <Menu width="auto"
        theme="dark"
        :active-name="activeName"
        :open-names="openNames"
        @on-select="handleSelect"
        :accordion="true"
    >

      <template v-for="(item,index) in items">
        <side-menu-item
            v-if="item.children&&item.children.length!==0"
            :parent-item="item"
            :name="index+''"
            :index="index"
        >
        </side-menu-item>
        <menu-item v-else
            :name="index+''"
            :to="item.path"
        >
            <Icon :type="item.icon" :size="15"/>
            <span>{{ item.title }}</span>
        </menu-item>
      </template>
    </Menu>
</template>

<script>
import sideMenuItem from '@/components/Menu/side-menu-item.vue'
export default {
    name: 'sideMenu',
    props: {
        activeName: {
            type: String,
            default: 'auth'
        },
        openNames: {
            type: Array,
            default: () => [
                'other',
                'role',
                'auth'
            ]
        },
        items: {
            type: Array,
            default: () => [
                {
                    name : 'system',
                    title : '数据看板',
                    icon : 'ios-analytics',
                    children: [
                        { name : 'user', title : '用户管理', icon : 'outlet',
                          children : [
                                { name : 'auth', title : '权限管理1', icon : 'outlet' },
                                { name : 'auth', title : '权限管理', icon : 'outlet',
                                  children:[
                                    { name : '334', title : '子菜单', icon : 'outlet' },
                                    { name : '453', title : '子菜单', icon : 'outlet' }
                                  ]
                                }
                            ]
                         }
                    ]
                },
                {
                    name : 'other',
                    title: '其他管理',
                    icon : 'outlet',
                }
            ]
        }
    },
    components: {
        sideMenuItem
    },
    methods: {
        handleSelect(name) {
           this.$emit('on-select', name)
        }
    }
}
</script>
```

```html
<!--side-menu-item.vue  子组件-->
<template>
    <Submenu :name="index+''">
        <template slot="title" >
            <Icon :type="parentItem.icon" :size="10"/>
            <span>{{ parentItem.title }}</span>
        </template>
        <template v-for="(item,i) in parentItem.children">
            <side-menu-item
                v-if="item.children&&item.children.length!==0"
                :parent-item="item"
                :to="item.path"
                :name="index+'-'+i"
                :index="index+'-'+i"
            >
            </side-menu-item>
            <menu-item v-else
                :name="index+'-'+i"  :to="item.path">
                <Icon :type="item.icon" :size="15" />
                <span>{{ item.title }}</span>
            </menu-item>
        </template>
    </Submenu>
</template>

<script>
export default {
    name: 'sideMenuItem',
    props: {
        parentItem: {
            type: Object,
              default: () => {}
        },
        index:{}
    },
    created:function(){
    }
}
</script>
```

### 4. 使用watch监听路由参数重新获取数据
**很多菜单项都只是入参不一样**，是不会重新走业务逻辑的，我们就用watch监听$router，如果改变就重新请求新的数据。
``` js
export default {
    watch: {
    '$route':'isChange'
    },
    methods:{
        getData(){
            // Do something
        },
        isChange(){
            this.getData()
        },
    }
}
```

### 5. 刷新：根据地址选中当前菜单项
页面刷新后左侧菜单的默认选中项就和页面对应不上了，我们用$router的beforeEnter方法做判断，根据地址获得路由的key（每一个路由都有一个key的参数），储存到localStorage中，然后菜单组件再从localStorage中取出key，再遍历匹配到当前选项目，比较冗余的是我们要在beforeEnter中获取一遍菜单数据，然后到菜单组件又获取一次数据，请求两次接口。

![](https://user-gold-cdn.xitu.io/2019/3/8/1695b57c9899d00e?w=782&h=802&f=png&s=513174)

```
step1 router.js中设置beforeEnter方法，获得地址栏中的key 存储到localStorage

step2 菜单组件取出localStorage中key，递归匹配
```
### 6. Axios统一状态码判断、统一增加token字段
Axios的interceptors方法有request和response两个方法对请求的入参和返回结果做统一的处理。
``` js
<!--request 除登录请求外，其他均增加token字段 -->
axios.interceptors.request.use(function (config) {
  let token = localStorage.getItem('token')
  if(token== null && router.currentRoute.path == '/login'){// 本地无token,未登录 跳转至登录页面
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

<!--response 返回状态统一处理 -->
axios.interceptors.response.use(function (response) {
  if(response.hasOwnProperty("data") && typeof response.data == "object"){
      if(response.data.code === 998){// 登录超时 跳转至登录页面
          iView.Message.error(response.data.msg)
          router.push('/login')
          return Promise.reject(response)
      }else if (response.data.code === 1000) {// 成功
        return Promise.resolve(response)
      } else if (response.data.code === 1060){ //数据定制中
         return Promise.resolve(response)
      }else {// 失败
        iView.Message.error(response.data.msg)
        return Promise.reject(response)
      }
  } else {
    return Promise.resolve(response)
  }

}, function (error) {
  iView.Message.error('请求失败')
  // 请求错误时做些事
  return Promise.reject(error)
})
```

### 7. 点击左侧菜单选中项点击刷新页面
测试同学提出bug，左侧菜单选中后，再次点击选中项没有刷新，用户体验不好，产品同学一致通过，我们就用野路子来解决了。
给菜单组件设置on-select事件，点击后存储当前选中项的path，每次执行当前点击的path和存储的path做对比，如果一致，跳转到空白页，空白页再返回到当前页，实现假刷新，注：不知道是router.push有节流控制还是怎么回事，不加setTimeout不管用。

``` js
<!--菜单的handleSelect事件-->
handleSelect(name) {
    let This = this
    if((this.selectIndex == 'reset') || (name == this.selectIndex)){
        // 点击再次刷新
        setTimeout(function function_name(argument) {
          This.$router.push({
              path: '/Main/about',
              query: {
                t: Date.now()
              }
            })
        },1)
    }
    this.selectIndex = name
    this.$emit('on-select', name)
},
```

``` js
<!--空白页-->
created(){
    let This = this
    setTimeout(function function_name(argument) {
      This.$router.go(-1);
    },1)
}
```

### 8. 使用Axios.CancelToken切换路由取消请求
有一部分情况是切换路由时，只改变参数，在“4. 使用watch监听路由参数重新获取数据”中提到过，还有一部分功能的接口数据返回的特别慢，会出现切换菜单后，数据才加载出来，需要增加切换菜单后取消原来的请求，代码注释中 setp1、2、3为顺序

``` js
export default {
  data(){
    return {
      // setp1 创建data公共的source变量 
      source:''                
    }
  },
  created:function(){
    // 获取搜索数据
    this.getData()
  },
  watch:{
    '$route':'watchGetSearchData',
  },
  methods:{
    getData(){
      // setp2 请求时创建source实例 
      let CancelToken = this.$http.CancelToken
      this.source = CancelToken.source();
    },
    watchGetSearchData(){
      // setp3 切换路由时取消source实例 
      this.source.cancel('0000')
      this.getData()
      this.$http
        .post('/api/show/map',data,{cancelToken:this.source.token})
        .then(function(response){
            
        })
    }
  }
}
```

### 9. element的table组件实现 表头表体合并
我们项目用到的的组件表格有两种，一种用iview的table，带操作按钮的表格，支持表头跨行跨列，另一种element的table组件，纯数据展示，支持表头和标题的跨行跨列。

![](https://user-gold-cdn.xitu.io/2019/3/8/1695b61f1c01782a?w=992&h=346&f=png&s=72840)


![](https://user-gold-cdn.xitu.io/2019/3/8/1695b6330fb064f0?w=913&h=484&f=gif&s=49381)

![](https://user-gold-cdn.xitu.io/2019/3/8/1695b63510cefa99?w=913&h=484&f=gif&s=260205)

element的table组件支持表头标题合并，我们定义数据结构包含三部分，表头、表体、表体合并项。
表头直接使用递归组件嵌套就可以了，表体数据直接扔给table组件，合并通过cellMerge方法遍历合并项数据遍历合并，代码如下。

**数据结构**
``` js
data:{
    historyColumns:[  // 表头数据
        {
            "title": " ",
            "key": "column"
        },
        {
            "title": "指标",
            "key": "target"
        },
        {
            "title": "11/22",
            "key": "11/22"
        },
        {
            "title": "日环比",
            "key": "日环比"
        },
        {
            "title": "当周值",
            "key": "当周值"
        },
        {
            "title": "上周同期",
            "key": "上周同期"
        },
        {
            "title": "周环比",
            "key": "周环比"
        },
        {
            "title": "近7日累计",
            "key": "近7日累计"
        },
        {
            "title": "当月累计",
            "key": "当月累计"
        }
    ],
    histories:[  // 表体数据
        {
            "target": "在售量",
            "11/22": 912,
            "日环比": "-",
            "当周值": 912,
            "上周同期": 0,
            "周环比": "100%",
            "近7日累计": 912,
            "当月累计": 912,
            "column": "基础指标"
        },
        {
            "target": "-在售外库车量",
            "11/22": 29,
            "日环比": "-",
            "当周值": 29,
            "上周同期": 0,
            "周环比": "100%",
            "近7日累计": 29,
            "当月累计": 29,
            "column": "基础指标"
        }
    ],
    merge:[  // 表体合并项
        {
            "rowNum": 0,
            "colNum": 0,
            "ropSpan": 1,
            "copSpan": 4
        },
        {
            "rowNum": 4,
            "colNum": 0,
            "ropSpan": 1,
            "copSpan": 27
        }
    ]
}
```

**表体合并说明：** 表格有cellMerge方法，每一td在渲染时都会执行这个方法，在cellMerge里遍历merge数据，根据cellMerge的入参行、列定位到td，如果是要合并的表格，则return出要合并的行数和列数，如果在合并的范围内，则要return [0,0]，隐藏当前td。

比如要把A、B、C、D，merge的数据rowNum为A的行、colNum为A的列、ropSpan为2、copSpan为2，在cellMerge方法中，如果坐标为A的单元格，return ropSpan和copSpan，**如果坐标为B、C、D则要return [0,0]隐藏，否则会出现表格错乱**。

![](https://user-gold-cdn.xitu.io/2019/3/8/1695b6429ddbdb1b?w=1176&h=746&f=png&s=91959)
**merge方法代码：**
``` js
// 表格合并主方法  row:行数组  column:列数据  rowIndex、columnIndex行列索引
cellMerge({ row, column, rowIndex, columnIndex }) {

  let This = this;
  if(This.configJson){
      for(let i = 0; i < This.configJson.length; i++){

      let rowNum = This.configJson[i].rowNum   // 行
      let colNum = This.configJson[i].colNum   // 列

      let ropSpan = This.configJson[i].ropSpan // 跨列数
      let copSpan = This.configJson[i].copSpan // 跨行数

      if(rowIndex == rowNum && columnIndex == colNum ){// 当前表格index 合并项
        return [copSpan,ropSpan]
      // 隐藏范围内容的单元格
      // 行范围 rowNum <= rowIndex && rowIndex < (rowNum+copSpan)
      // 列范围 colNum <= columnIndex && columnIndex < (colNum+ropSpan)
      }else if( rowNum <= rowIndex && rowIndex < (rowNum+copSpan) && colNum <= columnIndex && columnIndex < (colNum+ropSpan) ){

        return [0,0]
      }

    }
  }

}

```

**表头合并说明：**element和iview的表头合并数据格式可以一样，都是递归形式，区别是iview的table组件直接把数据扔给组件就可以了，而element需要自己封装一下表头。

``` JS
// 子组件
<template>
  <el-table-column :prop="thList.key" :label="thList.title" align="center">

    <template v-for="(item,i) in thList.children" >

        <tableItem  v-if="item.children&&item.children.length!==0"
        :thList="item" /></tableItem>

        <el-table-column align="center" v-else
              :prop="item.key"
              :label="item.title"
              :formatter="toThousands"
            >
            
        </el-table-column>
      </template>
  </el-table-column>
</template>
<script>
export default {
    name: 'tableItem',
    props: {
        thList: {
            type: Object,
              default: () => {}
        },
    },
}
</script>
```

**封装后的table组件：**
``` JS
<template>
  <div>
    <el-table :data="Tbody" :stripe="stripe" :border="true" :span-method="cellMerge" align="center" :header-cell-style="tableHeaderColor"  height="600" >
      <template v-for="(item,i) in Thead">
          <template v-if="item.children&&item.children.length!==0" >

            <tableItem :thList="item" />

          </template>

          <template v-else >

            <el-table-column align="center"
              :prop="item.key"
              :label="item.title"
              :formatter="toThousands"
            >
            </el-table-column>

          </template>
        </template>
      </el-table>
  </div>
</template>
<script>
import tableItem from '@/components/table/tableHeader/table-Item.vue'
export default {
    name: 'table-header',
    props: {
        Thead: {
            type: Array,
            default: () => {}
        },
        Tbody:{
          type: Array,
          default: () => {}
        },
        stripe:{
          type:Boolean,
          default:false
        },
        cellMerge:Function,
          default:()=>{}
    },
    created:function(){
    },
    components:{
      tableItem
    },
    methods:{
      tableHeaderColor({ row, column, rowIndex, columnIndex }) {
        if (rowIndex === 0) {
          return 'background-color: #f8f8f9;'
        }
      }
    }
}
</script>
```
**其他页面复用table**
``` html
<!--引入-->
import TableList from '@/components/table/tableHeader/index.vue'
<!--调用-->
<TableList :Thead="historyColumns" :Tbody="historyData" :cellMerge="cellMerge" />
```

### 10. iview的Menu组件+vuex实现面包屑导航
iview的Menu组件有on-select方法，可以获得当选选中项的name，我们的name按照数据索引来遍历的，比如三级菜单，选中后会返回`2-0-1`这样的字符串，表示树菜单第3个菜单下的第1个子菜单下的第2个菜单项，通过这个字符串再筛选出数组`['业务报表','B2C报表','成交明细']`对应菜单的title，然后发给vuex的Store.state，然后面包屑组件通过计算数据属性监听Store.state拿属性展示就可以了。
``` js
<!-- 根据字符串筛出title数组 发给$store -->
toBreadcrumb(arrIndex){

      let This = this;
      let mapIndex = arrIndex.split('-');
      // 获取对应name
      let box={};

      let mapText = mapIndex.map(function(item,index){
          if(index == 0){
            box = This.MenuData[eval(item)];
          }else{
            box = box.children[eval(item)];
          }

          return box.title;
      });

      this.$store.commit('toBreadcrumb',mapText)
    }
```

vueX代码
``` JS
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
export default new Vuex.Store({
  state: {
    Breadcrumb:[],  // 面包屑导航
    userName: '',
    readyData:""
  },
  mutations: {
    toBreadcrumb(state,arr){
      state.Breadcrumb = arr;
    }
  },
  getters: {
    getBreadcrumb: state => {
      return state.Breadcrumb
    }
  }
})
```
面包屑组件
``` html
<template>
  <Header style="background: #fff;">
  	<Row>
        <Col span="12">
          <!-- {{doneTodosCount}} -->
        	<Breadcrumb>
		        <BreadcrumbItem v-for="item in doneTodosCount">{{item}}</BreadcrumbItem>
		    </Breadcrumb>
        </Col>
        <Col span="12">
        	<Login />
        </Col>
    </Row>
  </Header>
</template>
<script>
import Login from '@/components/Login'
export default {
  data(){
    return {
    }
  },
  created:function(){
    this.$store.commit('toBreadcrumb',['首页'])
  },
  computed: {
    doneTodosCount () {
      return this.$store.state.Breadcrumb
    }
  },
  components:{
    Login
  }
}
</script>
```
###  11. iview上传组件手动上传,接入富文本编辑器
iview提供的组件特别丰富，我们在做图片上传的时候，需要手动上传，需要调用子组件的file对象通过自己的post方法提交到服务端，actionDate为文件数据，然后再通过on-success回调反馈上传成功或失败。
**手动上传:**
``` html
<Upload
     ref="upload"
    :data= "actionDate"
    :on-success="handleSuccess"
    :format="['png','jpg']"
    action="/api/upload/ccupload">
    <Button icon="ios-cloud-upload-outline">点击上传文件</Button>
</Upload>
 <div v-if="file !== null">
    上传文件: {{ file.name }} 
    <Button type="text" @click="upload" :loading="loadingStatus">{{ loadingStatus ? 'Uploading' : '上传' }}</Button>
</div>
```

``` js
// upload 方法
let uploadFile = this.$refs.upload.file
this.$refs.upload.data = this.actionDate;
this.$refs.upload.post(uploadFile);
this.loadingStatus = true;

// handleSuccess 方法
this.loadingStatus = false;
if(res.code == 1000){
    this.$Message.success('上传成功')
}else{
    this.$Message.error('上传失败')
}
```
我们在联调的过程中后端说接收不到文件，我们只能用node来验证一下是不是组件有问题，于是用express写了一下文件上传。
``` JS
var express = require('express');
var router = express.Router();
let fs = require('fs')
var formidable = require('formidable');//表单控件
var path = require('path');
var app = express();
app.use(express.static('/public/'));

router.post('/test',(req,res)=>{
	var imgPath = path.dirname(__dirname) + '/public';
	var form = new formidable.IncomingForm();
	form.encoding = 'utf-8'; //设置编辑 
	form.uploadDir = imgPath; //设置上传目录
	form.keepExtensions = true; //保留后缀
	form.maxFieldsSize = 2 * 1024 * 1024; //文件大小
	form.type = true;

	form.parse(req, function(err, fields, files){
		let src = files.img.path.split('/');
		let urlString = src[src.length-1]
	  	if (err) {
	      console.log(err);
	      req.flash('error','图片上传失败');
	      return;
	  }
	  res.json({
	      code: '200',
	      type:'single',
	      url:'http://10.70.74.167:3000/'+urlString
	  })
	});
});

module.exports = router;
```

我们在测试的时候增加了一个图片test的转发配置，然后把组件的action地址替换一下为`/test/`就可以了，亲测无问题[阴险脸]。
**vue.config.js**
``` JS
module.exports = {
  baseUrl: baseUrl,
  devServer: {
    proxy: {
        '/api': { // 开发服务器
            target: ' http://*******',
            changeOrigin: true,
        },
        '/test': { // 图片上传测试
            target: ' http://10.70.74.167:3000',
            changeOrigin: true,
        }
    }
  },
  productionSourceMap: false,
}
```


富文本编辑器的图片上传有两种模式，一种是把图片转成base64，通过一个接口把html内容提交给服务端，另一种模式是两个接口，分别把图片上传到服务器，然后返回url字符串到编辑器中，再把编辑器中的html保存到服务器上，我们用的编辑器是`vue-quill-editor`，使用第二种模式，借助element的el-upload组件自动上传图片，然后返回地址插入到编辑器。

``` JS
import {format} from '@/lib/js/utils.js'
	import {quillEditor} from 'vue-quill-editor'
	const toolbarOptions = [
	    ['bold', 'italic', 'underline', 'strike'],        // toggled buttons
	    [{'header': 1}, {'header': 2}],               // custom button values
	    [{'list': 'ordered'}, {'list': 'bullet'}],
	    [{'indent': '-1'}, {'indent': '+1'}],          // outdent/indent
	    [{'direction': 'rtl'}],                         // text direction
	    [{'size': ['small', false, 'large', 'huge']}],  // custom dropdown
	    [{'header': [1, 2, 3, 4, 5, 6, false]}],
	    [{'color': []}, {'background': []}],          // dropdown with defaults from theme
	    [{'font': []}],
	    [{'align': []}],
	    ['link', 'image'],
	    ['clean']
	  ]
    export default {
        data () {
            return {
            	options2:{},
            	quillUpdateImg: false, // 根据图片上传状态来确定是否显示loading动画，刚开始是false,不显示
            	content:'', // 富文本内容
            	title:'新建',
                editorOption:{
	            	placeholder: '',
	      			theme: 'snow',  // or 'bubble'
	      			modules:{
			            toolbar: {
			              container: toolbarOptions,
			              handlers: {
			                'image': function (value) {
			                  if (value) {
			                    // 触发input框选择图片文件
			                    document.querySelector('.avatar-uploader input').click()
			                  } else {
			                    this.quill.format('image', false);
			                  }
			                }
			              }
			            }
			        }
                },
		        serverUrl: '/api/add/upload?key='+this.$route.params.key,  // 这里写你要上传的图片服务器地址
		        header: {
		          // token: sessionStorage.token
		        },
                current: 0,
                formValidate: {
                    device_name: '集团BI',
                    versions: '',
                    publish_time: '',
                    desc: '',
                },
                ruleValidate: {
                    device_name: [
                        { required: true, message: '请选择系统名称', trigger: 'change' }
                    ],
                    versions: [
                        { required: true, message: '请输入版本信息', trigger: 'blur' }
                    ],
                    publish_time: [
                        { required: true, type: 'date', message: '请选择发版时间', trigger: 'change' }
                    ],
                    desc: [
                        { required: true, message: '请输入对于该版本的总体描述', trigger: 'blur' },
                        { type: 'string', min: 20, message: '版本的总体描述不少于20个字', trigger: 'blur' }
                    ]
                },
                isFirst: true,
                isSecond: false,
                isThird: false,
                versionid:''
            }
        },
        created:function(){
        	this.limit();
        	this.initialization();
       	},
        methods: {
        	limit(){
                this.options2 =  {
                  disabledDate (date) {
                    return (date && date.valueOf() > new Date().getTime()) || (date && date.valueOf() < new Date("2017-12-31"))
                  }
                }
            },
        	//初始判定是新增/修改
        	initialization(){
        		let id = this.$route.params.id;
        		if(id !=0){
        			this.title = "编辑";
        			let obj = {};
        			obj.version_id = this.$route.params.id;
        			obj.key = this.$route.params.key;
        			this.$http
			            .post('/api/show/version',obj).then(response => (
			               this.formValidate.device_name = response.data.data.device_name,
			               this.formValidate.versions = response.data.data.versions,
			               this.formValidate.publish_time = response.data.data.publish_time,
			               this.formValidate.desc = response.data.data.desc,
			               this.content = response.data.data.pc_html
			        ))

        		}else{
        			this.title = "新建";
        		}
        	},
        	//第一步基本信息（发布）
        	firstSubmit(name){
        		this.$refs[name].validate((valid) => {
                    if (valid) {
                        this.$Message.success('信息添加成功');
                        this.current += 1;
                        this.isFirst = !this.isFirst;
                        this.isSecond = !this.isSecond;
                    }else{
                        this.$Message.error('请完善必填信息');
                    }
                })
        	},
        	//第二步的表单数据提交（发布）
        	save(){
        		let id = this.$route.params.id;
        		let addObj = this.formValidate;
        		addObj.publish_time = format(this.formValidate.publish_time);
        		addObj.pc_html = this.content;
        		addObj.key = this.$route.params.key;
        		if(this.$route.params.id != 0){
        			addObj.version_id = id;
        		}
        		this.$http
		            .post('/api/add/version',addObj).then(response => (
		               this.secondSubmit(response.data.version_id)
		        ))
        	},
        	//第二步提交成功后转至第三步（发布）
        	secondSubmit(id){
        		this.current += 1;
                this.isSecond = false;
                this.isThird = !this.isThird;
                this.versionid = id;
        	},
        	//第三步跳转至[预览]
        	preview(){
        		this.$router.push({ path:"/Main/VersionManagementInfo/system_versions/"+this.versionid});
        	},
        	//第三步发布
        	release(){
        		let status = this.$route.params.status;
        		if(status != 2){
        			let obj = {};
        			if(this.$route.params.id == 0){
        				obj.version_id = this.versionid;
        			}else{
        				obj.version_id = this.$route.params.id;
        			}
	        		obj.key = this.$route.params.key;
	        		this.$http
			            .post('/api/edit/publish/version',obj).then(response => (
			               this.releaseLink()
			        ))
        		}else{
        			this.releaseLink()
        		}

        	},
        	//第三步发布跳转
        	releaseLink(){
        		this.$router.push({ path:"/Main/VersionManagement/system_versions"});
        	},
        	//上一步操作
            returns () {
                if (this.current != 0) {
                    this.current -= 1;
                    this.isFirst = true;
                    this.isSecond = false;
                }
            },
            //富文本内容改变事件
            onEditorChange({editor, html, text}) {
		        this.content = html
		     },
		    //富文本图片上传前
		    beforeUpload() {
		        // 显示loading动画
		        this.quillUpdateImg = true
		    },
		    //富文本图片上传成功
		    uploadSuccess(res, file) {
		        // res为图片服务器返回的数据
		        // 获取富文本组件实例
		        console.log(res,file);
		        let quill = this.$refs.myQuillEditor.quill
		        // 如果上传成功
		        if (res.code == 1000 ) {
		          // 获取光标所在位置
		          let length = quill.getSelection().index;
		          // 插入图片  res.url为服务器返回的图片地址
		          quill.insertEmbed(length, 'image', res.data)
		          // 调整光标到最后
		          quill.setSelection(length + 1)
		        } else {
		          this.$message.error('图片插入失败')
		        }
		        // loading动画消失
		        this.quillUpdateImg = false
	      	},
		    // 富文本图片上传失败
		    uploadError() {
		        // loading动画消失
		        this.quillUpdateImg = false
		        this.$message.error('图片插入失败')
		    },
		 
        }
    }

```




### 12. 使用cheerio展示字符串表格
有一部分表格数据比较难处理，是后端直接把xlsx文件转成字符串发给前端，cheerio可以把字符串转为类似jquery对象的虚拟DOM，然后用jquery的api操作这个虚拟DOM。

```
import cheerio from "cheerio"
this.$http.post('/api/list/statement-table',p).then(function(response){
               if(response.data==""){
                  This.isShow=false;This.content=true;This.title=false//无数据时数据加载中和标题数据的盒子隐藏
                  This.message="<div style='text-align:center'>暂无数据</div>"
               }else{
                    //console.log(response)
                    This.isShow=false;
                    This.content=true;//有数据时 数据加载中隐藏 标题和表体显示
                    let $ = cheerio.load(response.data);
                    //删除自带的行内样式
                    $("body style").remove();
                    $("body table").css({"border":"1px solid #e8eaec" });
                    $("body table td").css({"border":"1px solid #e8eaec","padding":"10px","color":"#515a6e"});
                    //全文匹配 剔除&amp;quot;
                    This.message = $("body").html().replace(/&amp;quot;/g,"");
               }
           })


```


### 13. keep-live组件缓存

产品的需求是从列表页面点击查看按钮进入详情页面，详情页面再点击返回，列表页面要不能刷新，就需要把组件缓存起来。

![](https://user-gold-cdn.xitu.io/2019/3/11/1696bbd3e9ddfa00?w=1006&h=503&f=gif&s=256501)
组件缓存直接加`keep-live`就可以了，比较麻烦的是我们在这个组件里判断三种情况，1.第一次进入 2.从其他栏目进入 3.从详情页进入，如果从为1、2这两种情况，我们需要刷新页面，如果是3，则不刷新。

思路是：
`created`钩子中着增加`isFirstEnter`标识，`beforeRouteEnter`钩子中判断是否为详情页面返回，如果是则加上`meta.isBack`的标识，在`activated`钩子里判断是第几种情况，如果为1或2，则重新请求列表页数据，如果是3就不用动管了。

`router.js`增加标识meta的`keepAlive`和`isBack`
``` js
/******** 业务报表 Start ********/
{
    path: '/Main/BusinessReport/:key', // 业务报表-列表
    name: 'BusinessReport',
    meta: { keepAlive: true,isBack:false},
    component: () => import('./pages/BusinessReport/index.vue'),
},
{
    path: '/Main/BusinessReportInfo/:sn/:is_check/:key/:type/:cmd5/:time/:is_down', // 业务报表-详情
    name: 'BusinessReportInfo',
    component: () => import('./pages/BusinessReportInfo/index.vue'),
},
/******** 业务报表 End ********/
```

根据`mate.keepAlive`渲染不同的`router-view`（忘记为什么是这么写的了，感觉很low）。

```
<keep-alive>
    <router-view v-if="$route.meta.keepAlive" ></router-view>
</keep-alive>
<router-view v-if="!$route.meta.keepAlive" >
    <!-- 这里是不被缓存的视图组件，比如 Edit！ -->
</router-view>
```

组件代码钩子事件`created`、`beforeRouteEnter`、`activated`方法

``` js
data(){
    return{
        isFirstEnter:false
    }
},
created:function(){
    this.isFirstEnter = true;
},
beforeRouteEnter(to, from, next) {
    if(from.name === 'BusinessReportInfo') { //判断是从哪个路由过来的，若是BusinessReportInfo页面不需要刷新获取新数据，直接用之前缓存的数据即可
      to.meta.isBack = true
    }
    next();
},
activated() {
    if(!this.$route.meta.isBack || this.isFirstEnter) {
        this.data=""
        //如果isBack是false，表明需要获取新数据，否则就不再请求，直接使用缓存的数据
        this.getPath(); // ajax获取数据方法
    }
    this.$route.meta.isBack = false;
    this.isFirstEnter=false;
    //恢复成默认的false，避免isBack一直是true，导致下次无法获取数据
}

```

### 14. 不要在子组件中操作父组件的数据
确实可以在子组件中修改父组件的数据，但强烈建议不要在子组件中操作父组件数据，期间我接手过一个功能，梳理了半天逻辑，没找到触发点在哪里，原来是在子组件中操作了父组件的数据，不利于维护，我自己起了个名字，让数据保持单向流动，不知道是不是可以定义为单项数据了原则😂。

在开发的过程中我们发现，每个人写的业务组件代码风格都不一致，怎样是一致，关于业务组件，有没有好的规范或者原则呢？还希望大家给点资料和建议非常感谢。