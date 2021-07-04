---
title: 使用TypeScript + React发布组件到Npm
date: 2021-07-04 12:09:20
tags:
---
最近封装了项目中使用的React地图组件，摸爬滚打发布到npm上；学到的知识点也比较散，如TypeScript、Commit规范/版本语义化、React组件测试、Npm发布更新、Readme模板、组件文档搭建等，有的知识点也是浅尝辄止（一知半解😱），先记录下来，后期有时间深挖。

- ✨ GitHub主页：[https://github.com/nihaojob/mapLine](https://github.com/nihaojob/mapLine)
- 文档地址：[http://nihaojob.gitee.io/carui/#/carui/maps](http://nihaojob.gitee.io/carui/#/carui/maps)
- Npm主页：[https://www.npmjs.com/package/maplib2](https://www.npmjs.com/package/maplib2)

### 目录

1. 脚手架选择
2. tsdx使用与配置
3. TypeScript基础
4. Commit规范
5. Npm发布/配置/更新
6. Readme模板
7. React测试
8. 组件文档搭建

### 1. 脚手架选择

发布前看了很多教程，都是自己配置Webpack、Babel、Rollup，辅助的还有ESLint、Jest，想想都头大，社区有很多**开箱即用的零配置脚手架**，我用的`tsdx`，自行参考吧。

- [create-react-library](https://github.com/transitive-bullshit/create-react-library#readme) 有中文文档
- [create-react-hook](https://github.com/Hermanya/create-react-hook#create-react-hook) Hook + TypeScript
- [nwb](https://github.com/insin/nwb)
- [tsdx](https://github.com/formik/tsdx)

有推荐或更好用的脚手架还请告知😘，留着下次用。

### 2. tsdx使用与配置

按照文档生成项目即可，需自己手动配置Less、ESLint，如下：

#### Less配置

安装依赖插件

```bash
$ yarn add rollup-plugin-postcss autoprefixer cssnano less --dev
```

tsdx.config.js配置文件修改

```javascript
const postcss = require('rollup-plugin-postcss');
const autoprefixer = require('autoprefixer');
const cssnano = require('cssnano');

module.exports = {
  rollup(config, options) {
    config.plugins.push(
      postcss({
        plugins: [
          autoprefixer(),
          cssnano({
            preset: 'default',
          }),
        ],
        inject: false,
        // only write out CSS for the first bundle (avoids pointless extra files):
        // extract: !!options.writeMeta,
        extract: 'mapLine.min.css',
      })
    );
    return config;
  },
};
```

#### ESLint配置

脚手架创建的项目没有`.eslintrc.js`和`.eslintignore`文件，需手动添加。

也可以用`yarn lint --write-file`生成，文档有说明。

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/602ed25a76874764a4ea67887a7a09a8~tplv-k3u1fbpfcp-zoom-1.image)

[`.eslintrc.js`](https://github.com/nihaojob/mapLine/blob/master/.eslintrc.js)文件：

```js
module.exports = {
  "extends": [
    "react-app",
    "prettier/@typescript-eslint",
    "plugin:prettier/recommended" // 重点
  ],
  "settings": {
    "react": {
      "version": "detect"
    }
  }
}
```

#### Start 预览

源码在`src`目录下，`example`目录可预览，需要分别在根目录和`example`下安装依赖和`Start`，先在根目录`Start`再`example Start`。

[参考教程](https://juejin.im/post/6844904162497757192)

### 3. TypeScript基础

刚开始使用`TypeScript`很可能会写出`AnyScript`风格的代码，画风如下🙋（我写的）。

阮一峰推荐教程：[TypeScript 入门教程](https://ts.xcatliu.com/)很赞 👍。

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a7f4dfe2dcc4e2bb8e995159c74dabc~tplv-k3u1fbpfcp-zoom-1.image)

我是先用jsx写出功能，又迁移到`TypeScript`，记录几个常用的`Interface`定义语法。

#### 非必选

在类型定义中，有些属性为非必须参数，使用`?`标识。


```ts
interface anime {
  show?: boolean;
  icon?: string;
  pathColor?: string;
  type?: string;
}
```

#### 数组类型

在数组中放置对象，可使用`Array<InterfaceName>`或`InterfaceName[]`定义。

```ts
interface pathItem {
  iconText: string;
  title: string;
  theme?: number;
}

interface defaultOptions {
  path: Array<pathItem>;
  pathColor?: string;
  donePath?: pathItem[];
}
```

#### 未知属性、继承

已知属性名但不知道类型，可以用`any`定义，未知属性名已知类型可使用`[propName: string]: string;`定义；为了不重复定义，使用`extends`关键词继承其他`interface`。

代码：

```ts
interface pathItem extends donePathItem {
  iconText: string;
  title: string;
  [propName: string]: any;
}

interface donePathItem {
  LT: number[];
}
```

### 4. Commit规范

之前对Git规范不是很深入，只有使用 `git rebase`合并一些无用`commit`信息，推荐下开源的规范文档。

- [阮一峰：Commit message 和 Change log 编写指南](https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
- [约定式提交](https://www.conventionalcommits.org/zh-hans/v1.0.0-beta.4/)
- [Angular 提交准则 英文](https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#commit)

#### 7个类别

翻看了几个开源项目的`commit`和规范说明，主要是标明`commit`的7个类别。

```
- feat：新功能（feature）
- fix：修补bug
- docs：文档（documentation）
- style： 格式（不影响代码运行的变动）
- refactor：重构（即不是新增功能，也不是修改bug的代码变动）
- test：增加测试
- chore：构建过程或辅助工具的变动
```

#### VsCode插件使用

我个人主要是在`VsCode`中提交`Commit`，推荐2个插件。

快捷生成message：[git-commit-plugin](https://marketplace.visualstudio.com/items?itemName=redjue.git-commit-plugin)

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0cc28373ef8c4dfb8af54ebb65c1bf82~tplv-k3u1fbpfcp-zoom-1.image)

Commit记录展示：[Git History](https://marketplace.visualstudio.com/items?itemName=donjayamanne.githistory)

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c91a18711a664c538a94f702c199b5d1~tplv-k3u1fbpfcp-zoom-1.image)

#### 命令行配置
在命令行中使用`Git`可以使用`Commitizen`生成`Commit message`模板，并用`Commitlint`检查；参见[教程](https://zhuanlan.zhihu.com/p/34223150)。

### 5. Npm发布/配置/更新

注册不再赘述，我这么笨的人都可以搞定，聪明的你一定没问题👩‍🎓‍，以下是细节。

#### 打包路径

使用`yarn build`打包完以后，文件生成到`dist`目录，检查`package.json`中的`main`与打包路径一致即可。

#### 主页与仓库地址

正确配置`homepage`和`repository`才能在`Npm`介绍中展示出来。

```json
{
  "homepage": "http://nihaojob.gitee.io/carui/#/carui/maps",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/nihaojob/mapLine.git"
  },
}
```

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b30d109f86f42e69dcb4ad60375df80~tplv-k3u1fbpfcp-zoom-1.image)

#### 发布

国内很多小伙伴设置过npm镜像源，记得还原为官方，然后按照提示输入信息就成功了，简单吧💁 ？


```bash
$ npm config set registry https://registry.npmjs.org/
$ npm publish
```

#### 更新

修改Readem文件后，使用以下命令更新。

官方文档：[https://docs.npmjs.com/about-package-readme-files](https://docs.npmjs.com/about-package-readme-files)

```bash
$ npm version patch  // 1.0.1 表示小的bug修复
$ npm version minor // 1.1.0 表示新增一些小功能
$ npm version mmajor // 2.0.0 表示大的版本或大升级
$ npm version preminor // 1.1.0-0 后面多了个0，表示预发布
```

```bash
$ npm version patch
$ npm publish
```

版本也要遵循规范，没有来得及深挖，先留坑位🏌。

- 语义化版本 2.0.0：[https://semver.org/lang/zh-CN/](https://semver.org/lang/zh-CN/)

### 6. Readme模板

`Readme`应该告诉人们为什么应该使用以及如何安装、使用。留坑优化🏌。

#### 模板

推荐一份高Start的模板：[standard-readme](https://github.com/RichardLitt/standard-readme)。

也可以使用[readme-md-generator](https://github.com/kefranabg/readme-md-generator)交互式的生成`Readme`文件。

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b9fece8099f42429f4111729ddbd56a~tplv-k3u1fbpfcp-zoom-1.image)

#### 徽标生成

[https://shields.io](https://shields.io/)可根据GitHub仓库自动生成徽标。

**CI徽标**

GitHub配置的`Actions`成功执行，就可以从CI页面拷贝徽标。

[阮一峰：GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de6e915ee5f145aaac5762d5ec5fb7a2~tplv-k3u1fbpfcp-zoom-1.image)

**测试覆盖率徽标**

多种多样的的工具和方法，我还没学会，等我更新🏌。

### 7. React测试

tsdx并没有自动生成`jest.config.js`文件，如需定义可以手动增加，列下遇见的问题。

#### 组件中引入less报错

安装依赖

```bash
$ yarn add --dev identity-obj-proxy
```

`package.json`配置moduleNameMapper

```json
{
 "jest": {
    "moduleNameMapper": {
      "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/__mocks__/fileMock.js",
      "\\.(css|less)$": "identity-obj-proxy"
    }
  },
}
```

#### document is not defined

在测试文件头部增加环境说明，[例子](https://github.com/nihaojob/mapLine/blob/master/test/blah.test.tsx)。

```
/**
 * @jest-environment jsdom
 */
```

#### 引入第三方sdk

我在组件内有使用高德的全局变量，需要高德的SDK文件加载完毕后再执行测试，参考了[react-amap](https://github.com/ElemeFE/react-amap)的实现。

```js
// PromiseScript加载
function getAmapuiPromise() {
  const script = buildScriptTag(
    'https://webapi.amap.com/maps?v=1.4.15&key=你的key&plugin=AMap.Driving'
  );
  const p = new Promise(resolve => {
    script.onload = () => {
      resolve();
    };
  });
  document.body.appendChild(script);
  return p;
}
// 创建script标签
function buildScriptTag(src: string) {
  const script = document.createElement('script');
  script.type = 'text/javascript';
  script.async = true;
  script.defer = true;
  script.src = src;
  return script;
}

describe('it', () => {
  it('正常渲染', () => {
    // 加载高德后执行
    getAmapuiPromise().then(() => {
      initDemo();
    });
    function initDemo() {
      const div = document.createElement('div');
      ReactDOM.render(<Maps {...options} />, div);
      ReactDOM.unmountComponentAtNode(div);
    }
  });

});
```

### 8. 组件文档搭建

有很多类似的工具，主要是为了解决组件文档的问题。

- [bisheng](https://github.com/benjycui/bisheng) Ant 系列在用
- [dumi](https://d.umijs.org/) umi系列
- [VuePress](https://vuepress.vuejs.org/zh/) Vue
- [redemo](https://github.com/imweb/redemo) 腾讯出品 单个组件
- [Docz](https://www.docz.site/)
- [Gatsby.js](https://www.gatsbyjs.org/) React官网在用

我选了dumi，原因是文档好看💁，dumi刚发布不久，但使用体验真的不错。

官方介绍为：是一款基于 Umi 打造、为组件开发场景而生的文档工具。

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/604333ff894b470d88f8d50f3028ae36~tplv-k3u1fbpfcp-zoom-1.image)

文档源码：[https://gitee.com/nihaojob/CarUI](https://gitee.com/nihaojob/CarUI)

### 留坑与总结

还有很多没有搞定的事情，需要学的也越来越多，为了**质量和落地**，先列个ToDoList。

- [ ] VsCode Jest debug配置
- [ ] React Jest 补充测试用例
- [ ] 测试覆盖率徽标
- [ ] Readme按照模板修改
- [ ] 版本语义化落地
- [ ] np工具熟悉
- [ ] TypeScript深入学习

**才疏学浅，恳请斧正**，还请Stars、点赞鼓励💁。

GitHub项目：[https://github.com/nihaojob/mapLine](https://github.com/nihaojob/mapLine)