---
title: Node.js开发实践，前端工程师的MVP利器
date: 2021-07-04 11:53:36
tags:
---
什么是MVP，来自伟大的百科:

> **Minimum Viable Product –最简化可实行产品**。
>
> 是指**以最低成本尽可能展现核心概念的产品策略，即是指用最快、最简明的方式建立一个可用的产品原型**，这个原型要表达出你产品最终想要的效果，然后通过迭代来完善细节。该术语由弗兰克·罗宾逊和埃里克·里斯推广于Web应用程序 ，它也可能涉及到进行市场前手的分析。


### 前言：

Node是前端工程师的贵人，拓宽了前端工程师的能力边界，对比前几年用[Dreamweaver](https://baike.baidu.com/item/Adobe%20Dreamweaver/10015800?fromtitle=DREAMWEAVER&fromid=216984)写table页面的我来说，感受到的变化是日新月异；前端搞搞工程化和框架什么的也就算了，竟然连编辑器都自己搞🤔，**js你说你是不是有点过分了？**

当然，这个过分的js帮助了我很多，从被后端大佬揪着耳朵按到工位上温声细语的说：“**我套完页面样式乱了，帮我调下样式**”，演变成大佬气冲冲的跑到我工位慈眉善目的拍着桌子说：“**TM接口参数传错了**”。

感谢Node吧，至少我可以在自己的工位上改自己写的Bug了🤓。


言归正传，再这么贫真就写不下去代码了，随着Node能力的发展，我自己感觉出来自己有点飘了，因为有用Kindle看书的陋习，一直觉的市面上所有的kindle笔记软件都是垃圾🤥，于是自己写了一个满意的垃圾；这都不算啥，我居然因为要减肥，就写了个体重记录小程序，上线以后我冲着镜子里浑身赘肉的自己喊：“**以为自己就是Node吗？过分**”🤧。

体重记录小程序的故事并没有突兀的结束，有些用户反馈有bug，我借口taro更新太快项目跑不起来了，而且腾讯云函数我用的很不方便，于是很不负责的停更了；在年后疫情期间，因为实在太闲就打开了后台留言，看到有一个**莫名其妙的留言说寻求合作**🤷‍♂️。

我忐忑的拨通了电话，在说明了我是小程序的开发者以后，这个人上来就开始说瞎话：“你这个小程序太好了”🤦‍♂️，他阐述了一下自己的经历，是一位开了8年健身房的教练，后来混不下去把健身房关了，做在线减脂指导，竟然收入还不错，真是造化弄人🤪，**他咨询我可以一起做一个减脂管理系统吗？不要钱那种，我恬不知耻的说：“好呀”**。

不久我们见面了，约在北京东五环外的常营龙湖·长楹天街，他问我可以吃川菜吗？我说可以，于是我们找了一家老屋川菜馆，坐在我面前的这个人因为吃了一口自己点的辣子鸡，然后拿着纸巾一把鼻涕一把泪的忏悔自己是个假重庆人🤦‍♂️，我羞涩的吐掉了口中的辣椒，一起构想了我们小程序的未来，**现在回想起自己的高谈阔论，都有些不好意思**😌。


### 正文

上边的段子根据个人情况改编，纯属娱乐，如果没有Node的开发能力，别人可能会安慰我：**“你还是去写个页面把”**，😭 苦涩的泪水从眼眶涌出。

简单介绍了下最近折腾的3个项目的由来，从第一个体重记录小程序，到Kindle笔记工具，再到现在的一套小程序 + 后台，作为一个前端程序员独立作出一套可以跑起来的小系统还是比较有成就感的，虽然可能会被吐槽：**不就是增删改查吗？** 但是不用担心被吐槽：**又没写过增删改查懂个屁？😝**

下边内容介绍了3个项目的积累，重点贴一下第三个项目Node用到的代码。**共同交流，恳请斧正**。


### 21天体重记录小程序

累计7千用户和每天不超过20个活跃用户的数据，还有3篇实践笔记。
**小程序提供的Node云函数 + 数据库，可以不花一毛钱就能跑起来自己的小程序**，最早是原生写法，后来切换到`Taro React`语法，效率提高很多，对小程序登录流程、云开发有了一些经验积累，也意识到自己对表结构设计的欠缺。

奇怪的是竟然累计了7千用户，用户从哪来的呢，难道是因为名字起的好吗？🤔后续准备再更新探索下。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/699419ab5d774792932fff67a6c6b3b2~tplv-k3u1fbpfcp-watermark.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b977eb4d796a482b9034975fbdd03634~tplv-k3u1fbpfcp-watermark.image)

- [【小程序 + 云开发】体重记录小程序 上手笔记](https://juejin.cn/post/6844903894816915463)
- [【小程序 + 云开发 】 随机读取数据并生成分享图片 上手笔记](https://juejin.cn/post/6844903896943427597)
- [【小程序 + 云开发】体重排行榜 上手笔记](https://juejin.cn/post/6844903918896414727)


### kindle 笔记整理工具
最早是在本地开发，开发了用户注册、密码找回、书籍管理、笔记管理的功能，然后买服务器部署到线上。

前端使用的 `Ant Pro`，因为引入了`Echarts`，没有做按需引入，所以第一次进入比较慢，目前只有自己用，就没优化。

整套流程跑起来以后，用着自己做的小工具觉得还挺香，也有信心做一些更大的挑战了。

地址：[http://nihaojob.com/](http://nihaojob.com/)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f1e2d2d174a4ea4a855eb77275ac6f1~tplv-k3u1fbpfcp-watermark.image)
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f71ee9beadf40c1a1b988176fdfdb6a~tplv-k3u1fbpfcp-watermark.image)
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7da7d10e790f4e9783d35e145e1016e9~tplv-k3u1fbpfcp-watermark.image)



### 减脂管理系统开发

终于到今天的主题了，先说下应用场景，学员在报名减脂教练的课程后，教练需要先了解学员日常饮食、睡眠、运动等生活习惯，然后根据学员状况定制运动计划和饮食方案，以及日常的运营如对学员的饮食和运动的打卡审核、积分减重排行榜、知识库等。

**主要的6个功能：**

1. 教练账号管理
2. 问卷收集
3. 方案下发
4. 打卡审核
5. 知识管理
6. 积分、减重排行榜


**后台预览：**
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0262dd330c6640c485721bf0fcaf4c41~tplv-k3u1fbpfcp-watermark.image)
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7561f7a05aa34daf87e27cdb84a0488e~tplv-k3u1fbpfcp-watermark.image)
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7aec2460b51b48398186b66ca71178d0~tplv-k3u1fbpfcp-watermark.image)

小程序预览：
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e9148c286444be3a845334a05d352f5~tplv-k3u1fbpfcp-watermark.image)
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35e30326f1564f5393f98ff2065f5ed4~tplv-k3u1fbpfcp-watermark.image)



### 知识点



#### 服务器 域名 备案
我是从滴滴云上买的服务器，一年才几百块，域名是之前在腾讯买的`nihaojob.com`，
备案过程中滴滴说有政策调整，花了20多天的时间，备案建议提前做准备，备案期间可以把Nginx + Node + Mongodb环境搭建起来。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36897f6db6244568bd5eec75a1ea6ab2~tplv-k3u1fbpfcp-watermark.image)


#### HTTPS证书申请与Nginx配置

微信小程序的开发域名必须是`HTTPS`，滴滴云有免费的证书。
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/442186f0063f46aaa006b917567e45ac~tplv-k3u1fbpfcp-watermark.image)
证书申请后需要在域名解析汇总增加TXT记录，不懂就问滴滴云客服，服务很nice。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b72cbd8c9a97425a9ec9f28a272f1485~tplv-k3u1fbpfcp-watermark.image)
证书申请成功后，把证书上传到服务器，在Nginx的`/etc/nginx/conf.d`目录下，`https.conf`文件中`ssl_certificate`、`ssl_certificate_key`配置证书路径。

``` NGINX
server {
  listen    443 ssl;
  server_name  api.nihaojob.com;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
  ssl_prefer_server_ciphers on;
  ssl_certificate /root/sslcert/cert.crt;
  ssl_certificate_key /root/sslcert/private.key;
  location / {
      alias /root/coach/coach-fe/dist/;
  }
  location /prod-api/ {
     proxy_pass http://127.0.0.1:3000/;
  }
 }
```



#### 跨域设置
这里设置了跨域请求头，因为`Origin`是根据入参来的，很容易造成`CROS`攻击，对安全系数有要求的系统还是用别的方案吧，也可以使用`express`推荐的`cors`中间件。

```js
app.all('*', function (req, res, next) {
  res.header("Access-Control-Allow-Origin", req.headers.origin);
  res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept, token');
  res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");
  res.header("Access-Control-Allow-Credentials", "true");
  res.header("Content-Type", "application/json;charset=utf-8");
  if (req.method == 'OPTIONS') {
    res.send(200); /*让options请求快速返回*/
  }
  else {
    next();
  }
});
```


#### express jwt使用

前端在登录时根据用户`id`生成一个`Token`发给前端，前端之后的所有请求都携带这个`Token`，后端根据`Token`解开后的用户id来进行数据操作。

利用`jsonwebtoken`生成`Token`，`express-jwt`进行校验和非必需登录接口检查。

个人认为开发同学都应该深挖一下无状态`Token`机制与有状态`session`机制的知识点。

```js
//引入
const jwt = require('jsonwebtoken');
const expressJwt = require('express-jwt');

//定义签名字符串
const secret = 'anyThingString';

//使用中间件验证token合法性
app.use(expressJwt({
  secret: secret,
  getToken: function fromHeaderOrQuerystring(req) {
    if (req.cookies && req.cookies.token || req.headers.token) { // 使用query.token
      return req.headers.token;
    }
    return null;
  }
}).unless({
  path: ['/login','/users/register', '/students/login', ]  //除了这些地址，其他的URL都需要验证
}));

//拦截器
app.use(function (err, req, res, next) {
  //当token验证失败时会抛出如下错误
  if (err.name === 'UnauthorizedError') {
    //这个需要根据自己的业务逻辑来处理（ 具体的err值 请看下面）
    res.status(401).send({ code: -1, msg: '未登录', status: 41002 });
  }
});

// 解析token中间件 后续所有接口可直接使用req.tokenDecode获取参数
app.use((req, res, next) => {
  // 获取token
  let token = req.cookies.token || req.headers.token;
  if (token) {
    let decoded = jwt.decode(token, secret);
    req.tokenDecode = decoded
  }
  next()
});

//返回token给客户端.
app.use('/login', async function (req, res) {
  const { mail, password }= req.body;
  // 格式校验
  if( mail === '' || password === '' ){
    return res.send({ code: -1, msg: '非法参数' });
  }
  const userInfo  = await userModel.findOne({ mail: mail, password: password  })
  if (!userInfo) {
    res.send({ code: -1, msg: '信息错误' });
  } else {
    const { _id, mail, mobile, type } = userInfo;
    //生成token
    const token = jwt.sign({ _id, mail, mobile, type, }, secret, {
      expiresIn: 3600 * 2 //秒到期时间
    });

    res.cookie('token', token, {
      maxAge: 1000 * 60 * 60 * 24 * 30,
      httpOnly: true
    });

    res.send({ code: 1, msg: '登录成功', status: 'ok', data:{ userInfo, token } });
  }
});

```

#### 环境变量
环境变量在`npm script`中设置，本地开发时用`nodemon yarn start`，部署线上环境时使用`pm2 start --name coEnd npm -- run startPro`。

需要根据环境变量走不同的数据库连接地址和图片前缀地址，如果公众号或者小程序有区分测试和正式环境，也可以在这里配置`APPID`和`SECRET`。

``` JSON
"scripts": {
    "start": "NODE_ENV=development node ./bin/www",
    "startPro": "NODE_ENV=production node ./bin/www"
  },
```


```JS
// 全局配置文件 /utils/config.js
const configBase = {
	mongoDb: { // 数据库
        development:'mongodb://127.0.0.1:27017/coach',
        production:'mongodb://127.0.0.1:27017/coach',
    },
    fileUrl:{ // 图片地址
        development:'http://127.0.0.1:3000/',
        production:'https://api.nihaojob.com/prod-api/',
    }
}
// 引用的配置对象 在各分模块中调用
let infoConfig = {}
let envKey = process.env.NODE_ENV
// 预知环境变量
let keys = ['development','production']
// 拼接引用的配置对象
Object.keys(configBase).forEach(item => {
	// 预知环境变量
	if(keys.includes(envKey)){
		infoConfig[item] = configBase[item][envKey]
	}else{// 本地
		infoConfig[item] = configBase[item].host
	}
})
module.exports = infoConfig;

// 使用
const { fileUrl } = require("../utils/config");
```

#### Mongooes 连接

在`app.js`中执行 `require('./utils/dbs')()`，并且把DB实例挂到`global`上。

```JS
// 配置文件
var baseConfig = require('./config.js');
const dbs = async function (env) {
    const mongoose = require('mongoose');
    mongoose.connect(baseConfig.mongoDb, { useNewUrlParser: true, auto_reconnect: true, poolSize: 10 });
    const db = mongoose.connection;
    db.on('error', console.error.bind(console, '数据库链接失败'));
    db.once('open', callback => {
        console.log(`数据库链接成功，地址：${baseConfig.mongoDb}`)
    });
    global.db = db;
}
module.exports = dbs;
```
#### Mongooes 增删改

这部分不多说了，利ORM框架`Mongooes`增删改特别简单，先创建模型再根据模型操作。

``` JS
const mongoose = require('mongoose');
const { db } = global;
// 创建Model
const model = new mongoose.Schema({
    coachId:String, // 教练ID
}, {
    timestamps: true, // 自动增加创建、更新时间
});

const dbManage = db.model('tag', model);
// 创建
dbManage.create({ name:'' })
// 更新 前边为查询条件 后边为更新内容
dbManage.updateOne({_id:'id'},{ name:'' })
// 删除
dbManage.remove({_id : 'id'})
// 查找
dbManage.find({_id : 'id' })
dbManage.findOne({_id : 'id' })
```

#### Mongooes 查询
查询的功能比较多了，比如字符串模糊查询，常见的分页、排序，时间范围搜索等。
直接贴一个模板吧，copy直接用版，哈哈。

``` js
router.get('/getList', async function(req, res, next) {
    const { nickName = '', name = '', tel = '', newStatus = '', start = '1900-01-01', end = '2222-01-01'  } = req.query
    const { current = 1, pageSize = 20  } = req.query
    const startTime = moment(start).startOf('day')
    const endTime = moment(end).startOf('day')
    const queryParams = {
        $and: [
            { nickName: { $regex: new RegExp(nickName, 'i') } }, // 昵称模糊查询
            { name: { $regex: new RegExp(name, 'i') } }, // 姓名模糊查询
            { tel: { $regex: new RegExp(tel, 'i') } },   //手机号模糊查询
        ],
        coachId: req.tokenDecode._id,
        createdAt: {
            $gte: startTime.toDate(),
            $lte: moment(endTime).endOf('day').toDate()
            // $lte: moment(today).endOf('day').toDate() // 查询当天
        }
    }
    if (newStatus !== ''){
        // 数字状态模糊查询
        queryParams.$and.push({ newStatus: newStatus },)
    }
    // 分页 skip跳过数 limit每页数 sort排序方式
    const resault = await studentsModel.find(queryParams).skip(Number(pageSize) * Number(current-1)).limit(Number(pageSize)).sort({ createdAt: 'desc' });
    // 总数
    const total = await studentsModel.find(queryParams).count()
    res.send({
        code:1,
        data:{
            list: resault,
            page: {
                total,
                current: Number(current), // 当前页
                pageSize: Number(pageSize)
            }
        },
    });
});
```

#### 图片上传
很多地方都要用到图片上传，使用`formidable`插件，设置上传路径为`public`，根据环境变量 + 文件名拼接图片地址，单独把图片地址存到一张表中，方便其他地方复用。

```js
var express = require('express');
var router = express.Router();
const path = require('path');
const formidable = require('formidable');
var inFileModel = require("../model/inFile");
// 环境变量配置
const { fileUrl } = require("../utils/config");
// 上传文件
router.post('/', async function (req, res, next) {
    var imgPath = path.dirname(__dirname) + '/public';
    var form = new formidable.IncomingForm();
    form.encoding = 'utf-8'; //设置编辑
    form.uploadDir = imgPath; //设置上传目录
    form.keepExtensions = true; //保留后缀
    form.maxFieldsSize = 2 * 1024 * 1024; //文件大小
    form.type = true;
    const awaitFn = () => {
        return new Promise((resolve, reject) => {
            form.parse(req, (err, fields, files) => {
                let src = files.file.path.split('/');
                let urlString = src[src.length - 1]
                if (err) { reject(err); }
                resolve(urlString);
            });
        });
    };
    // 文件名
    const filePath = await awaitFn();
    // 储存上传记录
    const fileInfo = await inFileModel.create({ fileName:fileUrl + filePath, })
    res.send({ code:1, msg:'上传成功', data:fileInfo })
});
module.exports = router;
```

#### 定时获取accesstoken
有过微信开发经验的同学都知道，调用微信服务端api需要`accesstoken`，时效2小时，利用`CronJob`定时获取`accesstoken`并保存成文件，获取失败时利用`nodemailer`发送报警邮件。

在部署时单独跑一个PM2进程，`pm2 start cronTask.js`。

```js
// cronTask.js
var CronJob = require('cron').CronJob;
const moment = require('moment');
const { writeFile } = require('fs').promises
const axios = require('axios')
var weConfig = require('./utils/weConfig');
var nodemailer = require('nodemailer');

// 获取微信token
var getWeToken = new CronJob('0 0 0/1 * * *', async function () {
  console.log('获取token任务执行：', moment().format('YYYY-MM-DD HH:mm:ss'));
  const { APPID, SECRET } = weConfig
  const url = `https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=${APPID}&secret=${SECRET}`
  axios.get(url).then(res => {
    if (res && res.status === 200 && res.data) {
      // 保存accesstoken文件
      writeFile('./accesstoken.txt', res.data.access_token, 'utf8')
    } else {
      // 发送报警内容
      sendMail('nihaojob@163.com', '保存错误：' + JSON.stringify(res))
    }
  })
}, null, true, 'America/Los_Angeles');

function sendMail(mail, text) {
  // 发送邮箱配置
  var mailTransport = nodemailer.createTransport({
    host: 'smtp.163.com',
    port: 465,
    secureConnection: true, // 使用SSL方式（安全方式，防止被窃取信息）
    auth: {
      user: '邮箱',
      pass: '密码'
    },
  });
  const options = Object.assign(mailTransport, { from: 'nihaojob@163.com', to: mail }, {
    subject: 'token获取失败报警',
    text: text,
  })
  mailTransport.sendMail(options, (err, msg) => {
    if (err) {
      res.send({ code: -1, msg: err });
      return
    }
    mailTransport.close();
  });
}

getWeToken.start();
```

#### 生成小程序参数二维码
读取`accesstoken.txt`获取`token`，利用`axios`发送给微信服务器获取图片，这块有个点需要注意，请求会直接返回图片，需设置`responseType: 'arraybuffer'`直接把`buffer`数据保存为图片。

```JS
var express = require('express');
var router = express.Router();
const { fileUrl } = require("../utils/config");
const { writeFile, readFile } = require('fs').promises

const axios = require('axios')
router.post('/getQrcode', async function (req, res, next) {
  const { path = '', scene = '' } = req.body;
  // 格式校验
  if (path === ''){
    res.send({ code: -1, msg: '非法参数' });
    return
  }

  const accessToken = await readFile('./accesstoken.txt')
  const url = `https://api.weixin.qq.com/wxa/getwxacodeunlimit?access_token=${accessToken.toString()}`
  const data = await axios.post(url,{
    scene: scene,
    page: path,
    width:430,
  },{
    headers:{
      "Content-Type": "application/json"
    },
    responseType: 'arraybuffer',
  })
  if (data.data){
    const fileName =  'qrCode_' + new Date().getTime() + '.jpg'
    await writeFile('./public/' + fileName, data.data)
    res.send({ code: 1, data: fileUrl + fileName });
  }else{
    res.send({ code:-1, msg:'请求错误' })
  }
});
module.exports = router;
```
#### for await使用
我们有一个用户列表，需要根据用户列表里的用户id查询另外一张列表里的用户详情，将他们拼接成一个新的列表返回给前端，我不太会用关联查询，探索出一个比较笨的方法，用`for await`这种方法实现的。

```JS
router.get('/getTopList', async function (req, res, next) {
    const { coachId } = req.tokenDecode;
    let data = await topModel.find({ coachId }).sort({ integral: 'desc' });
    if (data && data.length !== 0) {
        data = JSON.parse(JSON.stringify(data))
        for (const item of data) {
            const info = await studentsModel.findOne({ _id: item.studentsId})
            item.info = info
        }
    }
    res.send({code: 1, data })
});
```


#### taro小程序
这篇笔记的重点在Node上，就不展开聊了，简单写下登录、request封装、环境变量。

**登录**

登录的流程是，用户点击`openType`为`getUserInfo`的按钮发起授权，授权成功后调用`Taro.login`获取`code`，再把`code`发给后端，后端通过`code`、`APPID`、`SECRET`获取`openid`，剩下的就是用`openid`来绑定用户关系了。

> getUserInfo按钮 => 授权 => getCode => 获取openid

**taro-request**

taro官网上有一个`taro-request`的封装，蛮好用的[地址](https://github.com/TigerHee/taro-request)。


**环境变量**

Taro的环境变量从`process.env.NODE_ENV`中读取，内置环境变量为`development`、`production`，前端需要根据环境变量走不同的环境。

``` js
const getBaseUrl = (url) => {
  let BASE_URL = '';
  if (process.env.NODE_ENV === 'development') {
    //开发环境
    BASE_URL = 'http://localhost:3000'
  } else {
    // 生产环境
    BASE_URL = 'https://api.nihaojob.com/prod-api'
  }
  return BASE_URL
}
export default getBaseUrl;
```


#### 后台部分
后台使用`vue-element-admin`模板，几乎没有复杂的内容，接入了图表、富文本、图片上传，就不展开了，后续会开发发菜单、权限管理，有可能使用`node-casbin`或`acl`实现。


#### 部署
前端静态文件直接使用`Nginx`指定静态目录，后端接口通过PM2启动服务，并用`Nginx`的`proxy_pass`转到后端服务端口上。**HTTPS证书申请与Nginx配置**小节中有贴出，列一下自己最常用的几个PM2命令。

``` BASH
// npm script 启动应用
$ pm2 start --name 应用名称 npm -- run 'npm sctipt名称'
// 应用列表
$ pm2 list
// 查看日志
$ pm2 log
// 重启应用
$ pm2 restart '应用id'
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aed14f790ef64f66b907639f2e7acb1f~tplv-k3u1fbpfcp-watermark.image)

### 总结

开头段子中有调侃后端大佬，纯粹只是玩笑；互联网技术日新月异，大家都在齐头并进，前端的内容都学不完，又怎敢对不懂的行业指手画脚。希望自己**拥抱变化，保持敬畏**。

听说每个程序员都有一个创业梦，前端工程师真的可以借助Node跑起来自己的第一个MVP。
