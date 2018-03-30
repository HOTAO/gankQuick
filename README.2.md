OK，上一篇文章我们做了个妹子的页面[(传送门)](https://juejin.im/post/5abb6732f265da23a2292a96)

## gankQuick-快应用（妹子篇）

[快应用官方文档](https://doc.quickapp.cn/)

感谢 gank.io 提供的 api

今天接着上一次的文章，继续往下做：

#### 1、先来做个 tabs 吧

* 1、修改 Home 文件夹为 MeiZi
* 2、新建 Home 文件夹，里面新建一个 index.ux 文件

* 3、开始编码

Home/index.ux

```
<import name="meizi" src="../Meizi/index"></import>

<template>
  <div id="home">
    <div class="flexible-tabs">
      <tabs onchange="changeTabactive" index="{{currentIndex}}">
        <tab-content class="flexible-tab-content">
          <div class="tab-content-section">
            <meizi if="currentIndex===0"></meizi>
          </div>
          <div class="tab-content-section">
            <text if="currentIndex===1">其他<text>
          </div>
          <div class="tab-content-section">
            <text if="currentIndex===2">about me<text>
          </div>
        </tab-content>
      </tabs>
      <div class="flexible-tabbar">
        <div class="tab-item" for="{{tabList}}" @click="clickTabBar($idx)">
          <text class="{{currentIndex === $idx ? 'active' : 'tab-text'}}">{{$item.name}}</text>
        </div>
      </div>
    </div>
  </div>
</template>
<style lang="less" scoped>
#home {
  font-size: 16px;
  .flexible-tabs {
    display: flex;
    flex-direction: column;
  }
  .flexible-tabbar {
    display: flex;
    border-top-width: 1px;
    border-top-color: #eeeeee;
    .tab-item {
      display: flex;
      padding: 20px;
      font-size: 12px;
      justify-content: center;
      flex: 1;
    }
    .tab-text {
      color: #aaaaaa;
      font-size: 24px;
    }
    .active {
      font-size: 24px;
      color: #24b9ff;
    }
  }
}
</style>
<script>
export default {
  data: {
    currentIndex: 0,
    tabList: [
      {
        name: '妹子',
        icon: ''
      },
      {
        name: '其他',
        icon: ''
      },
      {
        name: 'Me',
        icon: ''
      }
    ]
  },
  // 监听change事件，触发时动态修改tabs的index属性
  // 左右滑动屏幕的时候就会需要用到这里
  changeTabactive(evt) {
    this.currentIndex = evt.index
  },
  //点击tab的时候，记录当前选中tab的下标
  clickTabBar(index) {
    this.currentIndex = index
  }
}
</script>
```

tabs 搞定, 这里有 changeTabactive 和 clickTabBar 两个函数，不只是为记录下标，好做样式和页面切换。

还有一个作用是下面这段代码中，在 if 里用作判断。 这样也是一种懒加载的方式

```
<div class="tab-content-section">
  <meizi if="currentIndex===0"></meizi>
</div>
<div class="tab-content-section">
  <text if="currentIndex===1">其他<text>
</div>
<div class="tab-content-section">
  <text if="currentIndex===2">about me<text>
</div>
```

这里是 flex 布局。说实话，有点不太适应。

但是，用好了，真的很爽。

###### 这里解释一下布局吧。会的大佬可以跳过...

因为之前也很少有道 flex 做整体的布局，所以今天我也记录一下。

对 flex 不懂的，看[阮一峰大佬的这篇教程，真的是包教包会](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

```
.flexible-tabs {
  display: flex;
  flex-direction: column;
}
```

这里的意思是：纵向从上往下排列.

![](https://user-gold-cdn.xitu.io/2018/3/30/16274e37d00beb61?w=956&h=724&f=jpeg&s=86527)

我们的 DOM 结构是这样的：

```
<div class="flexible-tabs">
  <tabs></tabs>
  <div></div>
</div>
```

这句样式 `flex-direction: column` 会让里面的 `tabs` 和 `div` 垂直排列

其实，快应用的 dispaly 默认值就是 flex，所以，其实可以这么写

```
.flexible-tabs {
  flex-direction: column;
}
```

效果是一样的，我写上只是为了方便阅读，你们可以不写的。

上面写完，效果是这样的 ↓

![](https://user-gold-cdn.xitu.io/2018/3/30/16275800013cfd24?w=320&h=565&f=gif&s=2456865)

现在我们去完善一下会 Meizi 页面。

Meizi/index.ux

```
<template>
  <div id="Meizi">
    <refresh @refresh="refreshList" refreshing="{{isRefreshing}}">
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
    </refresh>
    <stack show="{{model.show}}" class="mask" @click="closeModel">
      <image class="big-img" src="{{model.url}}" />
    </stack>
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
    isRefreshing: false,
    // 妹子列表
    meiziInfo: {
      list: [],
      hasMore: true
    },
    // 显示model
    model: {
      show: false,
      url: ''
    }
  },
  refreshList(evt) {
    this.isRefreshing = evt.refreshing
    this.getMeiziList()
  },
  getMeiziList() {
    this.$app.$def.fetch.fetch({
      url: `http://gank.io/api/data/福利/${this.pageSize}/${this.page}`,
      success: data => {
        if (this.isRefreshing) {
          this.$app.$def.prompt.showToast({
            message: '刷新成功'
          })
          this.isRefreshing = false
        }
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
  shouModel(url) {
    this.model = {
      show: true,
      url
    }
  },
  closeModel() {
    this.model = {
      show: false,
      url: ''
    }
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

增加了 refresh 组件，做下拉刷新操作。增加了一个 stack 组件，做查看大图操作。

效果图：

![](https://user-gold-cdn.xitu.io/2018/3/30/1627596c4330ab71)

好今天先到这.完整项目地址在 [这里](https://github.com/HOTAO/gankQuick)

可以点一下 start，感恩的 ❤️

下期精彩预告：

* list
* web
