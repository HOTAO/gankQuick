快应用发布快两周啦。这两天有空，就捣鼓了一个快应用。整体感觉来说，交互很流畅，基本功能和组件都有。上手也很快。希望官网推广能做好。好了，话不多说，先上 gitHub [(传送门)](https://github.com/HOTAO/gankQuick)

## gankQuick-快应用（妹子篇）

[快应用官方文档](https://doc.quickapp.cn/)

p 个 s: 环境搭建之类的 跟官网上面写的基本一样。照着做就行了。

* 注意：部分机子可能会出现手机上可以正常访问，浏览器上出现白屏，不要怀疑你的配置是不是有问题。有问题是那个机子可能  设配不了

### 现在我们来搞一个项目。

##### 一、建个项目，并且跑起来

到你喜欢的目录下执行

```
hap init <你的项目名称>
```

命令执行后会在当前目录下生成名为 你的项目名称 的文件夹，作为项目根目录，目录结构如下：

```
├── node_modules
├── sign                      rpk包签名模块
│   └── debug                 调试环境
│       ├── certificate.pem   证书文件
│       └── private.pem       私钥文件
├── src
│   ├── Common                公用的资源文件和组件文件
│   │   └── logo.png          manifest.json中配置的icon
│   ├── Demo                  页面目录
│   |   └── index.ux          页面文件，文件名不必与父文件夹相同
│   ├── app.ux                APP文件（用于包括公用资源）
│   └── manifest.json         项目配置文件（如：应用描述、接口申明、页面路由等）
└── package.json              定义项目需要的各种模块及配置信息，npm install根据这个配置文件，自动下载所需的运行和开发环境
```

在项目根目录下，运行如下命令安装模块到 node_modules 目录

```
npm install
```

执行命令（如果你按照官网，已经执行过这句了。那这里就跳过）

```
hap update --force
```

执行命令

```
npm run server
```

再开一个 DOS 窗口，执行命令

```
npm run watch
```

手机上打开，打开 <font color="blue">快应用调试器</font>,扫码安装。安装成功是这样的 ↓

![](https://doc.quickapp.cn/readme.png)

回到快应用调试器，点击 <font color="grey">开始调试</font> 按钮,手机和浏览器都会开始运行项目

##### 二、项目配置

官网写的贼详细，如果看不懂，可能要小心你的语文老师了，[项目配置传送门](https://doc.quickapp.cn/tutorial/getting-started/project-configuration.html)
这里贴出我的配置。可以跟着我的来写。

1、修改 manifest.json 的 配置（默认的不够用，另外，还有包名，应用名，版本号等等需要修改或添加）

```
{
  "package": "com.htSelf.gankQuick",
  "name": "gankQuick",
  "versionName": "1.0.0",
  "versionCode": "1",
  "minPlatformVersion": "101",
  "icon": "/Common/meizi.png",
  // features >>> 接口列表，这里是这个项目需要用到的快应用内置的接口。
  // 一定要先在features里声明，才可以使用
  "features": [
    { "name": "system.prompt" },
    { "name": "system.router" },
    { "name": "system.fetch" },
    { "name": "system.shortcut" },
    { "name": "system.prompt" },
    { "name": "system.webview" }
  ],
  "permissions": [{ "origin": "*" }],
  // config >>> 系统配置信息
  "config": {
    "logLevel": "debug"
  },
  // router >>> 路由信息
  "router": {
    "entry": "Home",
    "pages": {
      "Home": {
        "component": "index"
      }
    }
  },
  // display >>> UI显示相关配置
  "display": {
    "titleBarBackgroundColor": "#f2f2f2",
    "titleBarTextColor": "#333333",
    "menu": true,
    "pages": {
      "Home": {
        "titleBarText": "首页",
        "menu": false
      }
    }
  }
}
```

上面的配置中：配置了 router 路由。很简单，就只有一个路由 Home，后续会有更多路由。跟着来一步步走

2、整理项目代码

* 删除 Demo 文件夹
* 创建 Home 文件夹
* 在 Home 文件夹下，创建 index.ux 文件
* 开始编码

```
<template>
  <div id="Meizi">
      <list class="list" @scrollbottom="loadMore">
        <block for="{{meiziInfo.list}}">
          <list-item type="imgItem" class="img-item">
            <image @click="shouModel($item.url)" class="img" src="{{$item.url}}" />
          </list-item>
        </block>
        <list-item type="loadMore" class="load-more">
          <progress type="circular"></progress>
          <text>{{meiziInfo.hasMore?'加载更多':'已经没有更多妹子啦~'}}</text>
        </list-item>
      </list>
  </div>
</template>
<style lang="less" scoped>
#Meizi {
  font-size: 16px;
  padding: 20px 0;
  .list {
    columns: 2;
  }
  .img-item {
    margin: 0 10px 20px;
    height: 400px;
  }
  .img {
    width: 100%;
    height: 100%;
  }
  .load-more {
    display: flex;
    justify-content: center;
    padding-top: 20px;
  }
  .mask {
    position: fixed;
    height: 100%;
    align-items: center;
    background-color: rgba(0, 0, 0, 0.5);
    height: 100%;
    .big-img {
      width: 100%;
    }
  }
}
</style>
<script>
export default {
  data: {
    pageSize: 10,
    page: 1,
    // 妹子列表
    meiziInfo: {
      list: [],
      hasMore: true
    }
  },
  getMeiziList() {
    this.$app.$def.fetch.fetch({
      url: `http://gank.io/api/data/福利/${this.pageSize}/${this.page}`,
      success: data => {
        const results = JSON.parse(data.data).results
        if (results.length <= 0) {
          this.meiziInfo.hasMore = false
        } else {
          this.meiziInfo.list.push(...results)
        }
      },
      fail: (error, code) => {
        console.log('handling fail, error=' + error)
        console.log('handling fail, code=' + code)
      }
    })
  },
  loadMore() {
    this.page++
    this.getMeiziList()
  },
  onInit() {
    this.$page.setTitleBar({ text: '妹子妹子~~' })
    this.getMeiziList()
  },
}
</script>
```

效果图：![](https://user-gold-cdn.xitu.io/2018/3/29/1626fa9a61014d66?w=1080&h=2160&f=jpeg&s=192838)
别问为什么没有 安卓的那三个功能键，因为，我的隐藏了...

页面用了 list 、list-item 和 block 组件。做一个列表循环。并且获取数据，与展示。

<font color="red">这里非常感谢 gank.io 提供的 api</font>

在生命周期来到，onInit 的时候，做了两件事：

* 1、修改 title 为"妹子妹子~~"
* 2、获取妹子的列表数据

在 list 上监听 scrollbottom 事件，触发事件的时候，获取更多的妹子 😍，也就是 滚动加载更多数据

OK 就先写到这里吧。后续在接上吧

项目 github 地址[(完整代码在此)](https://github.com/HOTAO/gankQuick)

下期精彩预告：(其实代码写好了，github 上可以看到。只是还没写文章.😝)

* tabs
* webView
