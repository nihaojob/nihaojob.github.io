---
title: 使用fabric.js 快速开发一个图片编辑器
date: 2022-10-17 11:06:51
tags:
---


最近自己开发了一个图片编辑器，把源码也放在了GitHub上，顺便也总结下使用fabric.js开发一个编辑器需要用到哪些知识点。

- 预览地址：https://nihaojob.github.io/vue-fabric-editor/
- GitHub地址：https://github.com/nihaojob/vue-fabric-editor

![Foxmail20220904011310.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88ca4329b4c24dcea91e624ccadf4127~tplv-k3u1fbpfcp-watermark.image?)


### 架构设计
**选型：** fabric.js 和 konva.js都是强大的canvas库，功能上类似，konva.js比较新中文文档也多一些，因为比较熟悉fabric就没有采用konva。

**要点：** 因为框架用的vue，主要解决如何把fabric的实例对象共享给各个功能组件，区分出是未选中、单选、多选状态，然后将选中、取消选中事件暴露给各个功能组件，子组件根据状态进行独立的功能开发。

我的方法是在入口文件中初始化实例，然后与mixins结合，在mixins中定义了选择类型（多选、单选、未选中）、选中元素类型、选中id等属性，以及选中、取消选中的事件，子组件通过引入mixins来开发对应功能；如子组件需要对fabric对象进行操作，则可以通过inject获得原始对象。


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97d79b361a2549eea30d199e59b36344~tplv-k3u1fbpfcp-watermark.image?)

入口文件：
https://github.com/nihaojob/vue-fabric-editor/blob/161faae7b0adcf6f55d9efaa38cb6d7c6d2c01b5/src/views/HomeView.vue#L113

mixins文件：
https://github.com/nihaojob/vue-fabric-editor/blob/main/src/mixins/select.js




### 初始化
初始化比较简单，fabric.js创建对象，用EventEmitter创建事件发射器，可订阅单选、多选、取消选择事件。
通过vue的provide语法把fabric对象、EventEmitter对象向下传递，在mixins中保存选中的元素和选中状态。

**初始化：**
https://github.com/nihaojob/vue-fabric-editor/blob/161faae7b0adcf6f55d9efaa38cb6d7c6d2c01b5/src/views/HomeView.vue#L111


**事件发射器：**

```
import EventEmitter from 'events'

class EventHandle extends EventEmitter {

    init(handler){
        this.handler = handler
        this.handler.on("selection:created", (e) => this._selected(e));
        this.handler.on("selection:updated",  (e) => this._selected(e));
        this.handler.on("selection:cleared", (e) => this._selected(e));
    }

    // 暴露单选多选事件
    _selected(e) {
        const actives = this.handler.getActiveObjects()
        if(actives && actives.length === 1) {
            this.emit('selectOne', actives)
        }else if(actives && actives.length > 1){
            this.mSelectMode = 'multiple'
            this.emit('selectMultiple', actives)
        }else{
            this.emit('selectCancel')
        }
    }
}

export default EventHandle
```
**mixins:**
```
export default {
  inject: ['canvas', 'fabric', 'event'],
  data() {
    return {
      mSelectMode: '', // one | multiple
      mSelectOneType: '', // i-text | group ...
      mSelectId: '', // 选择id
      mSelectIds: [], // 选择id
    }
  },
  created(){
    this.event.on('selectOne', (e) => {
      this.mSelectMode = 'one'
      this.mSelectId = e[0].id
      this.mSelectOneType = e[0].type
      this.mSelectIds = e.map(item => item.id)
    })

    this.event.on('selectMultiple', (e) => {
      this.mSelectMode = 'multiple'
      this.mSelectId = ''
      this.mSelectIds = e.map(item => item.id)
    })

    this.event.on('selectCancel', () => {
      this.mSelectId = ''
      this.mSelectIds = []
      this.mSelectMode = ''
      this.mSelectOneType = ''
    })
  },
  methods: {
    /**
     * @description: 保存data数据
     * @param {Object} data 房间详情数据
     */
    _mixinSelected({ event, selected }) {
      if(selected.length === 1) {
        const selectItem = selected[0]
        this.mSelectMode = 'one'
        this.mSelectOneType = selectItem.type
        this.mSelectId = [selectItem.id]
        this.mSelectActive = [selectItem]
      }else if(selected.length > 1){
        this.mSelectMode = 'multiple'
        this.mSelectActive = selected
        this.mSelectId = selected.map(item => item.id)
      }else{
        this._mixinCancel()
      }
    },
    /**
     * @description: 保存data数据
     * @param {Object} data 房间详情数据
     */
     _mixinCancel(data) {
      this.mSelectMode =''
      this.mSelectId= []
      this.mSelectActive =[]
      this.mSelectOneType = ''
    },
  }
}

```


### 背景设置
![背景.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2286f31ad81545579f8a0a4caf060d60~tplv-k3u1fbpfcp-watermark.image?)
主要包括设置画布大小、设置背景颜色、设置背景图片，也可以设置背景重复方向。
[代码：](https://github.com/nihaojob/vue-fabric-editor/blob/main/src/components/bgBar.vue)
``` 
// 设置大小
setSize() {
      this.canvas.c.setWidth(this.width);
      this.canvas.c.setHeight(this.height);
      this.canvas.c.renderAll()
},
// 设置背景图片
setBgImg(target) {
      const imgEl = target.cloneNode(true);
      imgEl.onload = () => {
        // 可跨域设置
        const imgInstance = new this.fabric.Image(imgEl, { crossOrigin: 'anonymous' });
        // 渲染背景
        this.canvas.c.setBackgroundImage(imgInstance, this.canvas.c.renderAll.bind(this.canvas.c), {
          scaleX: this.canvas.c.width / imgInstance.width,
          scaleY: this.canvas.c.width / imgInstance.width,
        });
        this.canvas.c.renderAll()
        this.canvas.c.requestRenderAll();
      }
},
// 背景颜色设置
setColor(color) {
      this.canvas.c.setBackgroundColor(color, this.canvas.c.renderAll.bind(this.canvas.c))
      this.canvas.c.backgroundImage = ''
      this.canvas.c.renderAll()
}
```

### 插入元素
![插入元素.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43d110915d044f91818a238df07bf767~tplv-k3u1fbpfcp-watermark.image?)
主要包括插入基础元素文字、正方形、圆形、三角形、SVG元素，详见[代码](https://github.com/nihaojob/vue-fabric-editor/blob/main/src/components/tools.vue#L122)：
```
addText() {
      const text = new this.fabric.IText('万事大吉', {
        ...defaultPosition,
        fontSize: 40, id: uuid(),
      });
      this.canvas.c.add(text)
      this.canvas.c.setActiveObject(text);
},
addTriangle() {
      const triangle = new this.fabric.Triangle({
        top: 100,
        left: 100,
        width: 100,
        height: 100,
        fill: '#92706B'
      })
      this.canvas.c.add(triangle)
      this.canvas.c.setActiveObject(triangle);
},
```
导入SVG元素时，可以导入SVG文件或者字符串进行导入，调用fabric的loadSVGFromURL、loadSVGFromString方法进行导入，[详见代码](https://github.com/nihaojob/vue-fabric-editor/blob/main/src/components/importSvg.vue)。


### 属性调整
![属性调整.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e780a1c7c0414f0896bf80ca69b4a35c~tplv-k3u1fbpfcp-watermark.image?)
不同元素的属性会有差异，但通用属性是一致的，如填充颜色、坐标、旋转角度、透明度等，也有很多特定元素的特定属性，如文字的字体属性、图片的滤镜属性等，[详见代码](https://github.com/nihaojob/vue-fabric-editor/blob/main/src/components/attribute.vue)。
字体属性可以自定义字体，需要先下载字体后再进行设置，可以通过fontfaceobserver工具库下载指定字体，成功后在设置字体名称。
```
// 字体加载
var font = new FontFaceObserver(fontName);
font.load(null, 150000).then(() => {
    const activeObject = this.canvas.c.getActiveObjects()[0]
    activeObject && activeObject.set('fontFamily', fontName);
    this.canvas.c.renderAll()
    this.$Spin.hide();
}).catch((err) => {
    this.$Spin.hide();
})
```



### 元素对齐
![对齐.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9dd4ffa48d104810b895c82c705701d2~tplv-k3u1fbpfcp-watermark.image?)
元素对齐区分单选元素与多选元素，单选元素时只支持相对于画布水平、垂直、水平垂直对齐。
```
// name为 centerH | centerV | center
position(name){
  const activeObject = this.canvas.c.getActiveObject()
  if(activeObject){
    activeObject[name]()
    this.canvas.c.renderAll()
  }
}
```
多元素对齐有上下左右对齐、水平、垂直对齐，主要是通过获得最边缘元素的坐标，然后进行计算排序，如顶部对齐代码：
```
const activeObject = this.canvas.c.getActiveObject();
  if (activeObject && activeObject.type === 'activeSelection') {
        const activeSelection = activeObject;
        console.log(activeSelection)
        const activeObjectTop = -(activeObject.height / 2);
        activeSelection.forEachObject(item => {
          item.set({
                top: activeObjectTop,
            });
            item.setCoords();
            this.canvas.c.renderAll();
        });
    }
}
```
平均分配会复杂一些，需要计算出边缘与元素间距，再进行设置，[详见代码](https://github.com/nihaojob/vue-fabric-editor/blob/main/src/components/align.vue)。

### 其他用法
编辑器经常需要给元素进行分组/拆分组合、调整层级、回退、快捷键、画布放大/缩小、导入/导出文件等功能，不再一一罗列，这个小编辑器都已经支持，大家感兴趣的可以看源码。

- [组合](https://github.com/nihaojob/vue-fabric-editor/blob/main/src/components/group.vue)
- [层级调整](https://github.com/nihaojob/vue-fabric-editor/blob/main/src/components/layer.vue)
- [快捷键实现](https://github.com/nihaojob/vue-fabric-editor/blob/main/src/plugin/hotkeys.js)
- [画布放大缩小](https://github.com/nihaojob/vue-fabric-editor/blob/main/src/components/zoom.vue)
- [导入/导出](https://github.com/nihaojob/vue-fabric-editor/blob/main/src/components/save.vue)


### 总结

fabric.js的功能很强大，可以很轻松的开发出一个简版的图片编辑器，自定义素材、模板、字体文件；还可以结合数据接口拼接模板生成图片，很轻松的实现定制模板 + 生成图片的功能，比如我的朋友借助我的功能 + 成语接口生成成语图片，在小红书上斩获了八千多的粉丝。

最后希望大家能够通过这个项目学习到fabric.js的基础用法，感兴趣的话可以一起维护这款小编辑器，欢迎star。

https://github.com/nihaojob/vue-fabric-editor
