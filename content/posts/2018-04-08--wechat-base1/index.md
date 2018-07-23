---
title: 微信小程序基础
subTitle: 关于小程序开发
cover: folder.png
category: WeChat
---

## 开发过程中的发现

- wx:for 可以变量对象，wx:for-index 为对象的 key，wx:for-item 为对象的值
- wx.getBackgroundAudioManager() 获取到的值，是可能随时发生变化的，如下情况的代码，在两次输出时可能会返回不同的值
```
const backgroundAudioManager = wx.getBackgroundAudioManager();
console.log(backgroundAudioManager.src);
...
console.log(backgroundAudioManager.src);
```
- WXML 中参数命名为 data-live-id，在 js 中调用的事件处理函数中通过参数 e.target.dataset.liveId 获取；命名为 data-liveId，通过参数 e.target.dataset.liveid 获取
- scroll-view 设置 scroll-into-view 有时候没有效果，目前已知在先设置了 scroll-into-view 值，但是页面由于数据的处理还没有完成进行的是初始空列表的渲染，此时无法滚动到指定位置
- 在点击页面返回按钮时如果想指定返回层级数，可以在 onUnload() 生命周期方法中，调用 getCurrentPages() 获取整个路由栈中的所有信息，来决定返回返回几级页面 `wx.navigateBack({ delta: 1 });`，但是可能存在中间页面显示一下就消失的效果
- 图片的名称不要用汉字，在真机上（华为荣耀8）可能找不到资源
- 对于图片高度未知的情况，可能存在图片被压缩了然后又正常显示的情况，可以设置图片的 `height: auto;` 或者添加 image 组件的 bindload 回调，控制图片在加载完成后再显示出来
- 如果有富文本的内容需要后端返回，因为 web-view 有一些限制，为了降低开发成本可以使用长图的形式
- onUnload 是页面元素已经卸载后的回调，目前小程序没有提供页面元素卸载前的回调，如果在 onUnload 中进行获取页面元素的操作可能会报错
- 官方文档中描述，wx:if 有更高的切换消耗而 hidden 有更高的初始渲染消耗，如果需要频繁切换的情景下，用 hidden 更好，如果在运行时条件不大可能改变则 wx:if 较好
- 确定了在 android 手机（华为荣耀8），临时文件和保存文件的存储路径，在文件系统是可以看到的，路径如下，其中 `wx2a4a59eb34efaf3b` 为 appid，`tmp_` 表示临时文件，`store_` 表示保存的文件，也就说明如果使用了文件的存储，需要处理文件被用户手动删除的处理
    `/Tecent/MicroMsg/wxafiles/wx2a4a59eb34efaf3b/tmp_df3c24e50376e0354dc22798e6dccfda9e88f20b6d7ed0e0.jpg`
    `/Tecent/MicroMsg/wxafiles/wx2a4a59eb34efaf3b/store_df3c24e50376e0354dc22798e6dccfda9e88f20b6d7ed0e0.jpg`
- 100vh 指的是去掉顶部 Header 和底部 TabBar (如果存在) 后的高度，wx.getSystemInfo() 返回的理论上也是去掉顶部 Header 和底部 TabBar (如果存在) 后的高度，但存在适配问题，在 android 机上不同页面调用可能返回不同的值

## 一个值得思考的 bug

问题描述：A 页面 -> B 页面 -> C 页面，在 C 页面中，进行背景音频播放，并添加了背景音频播放的监听，onPlay、onPause、onTimeUpdate、onEnded、onStop，这些监听中主要是更新当前页面的 data 值，来控制页面中的显示，如显示播放/暂停按钮，显示当前播放进度等，**并更新全局的当前播放音频的信息**，在从 C 页面返回到 B 页面及 A 页面时，会显示一个悬浮播放的控件，显示正在播放的背景音频的信息及播放进度。问题是在从 C 页面返回后，如果是从背景音频点击的暂停，悬浮播放控件中显示的当前播放进度会显示从 C 页面返回时的进度，而不是当前播放进度。

分析原因：经过大概1小时的 bug 追踪，发现问题的原因是背景音频播放的监听是在 C 页面内的，在 onPause 监听中，更新全局播放音频信息的代码是如下代码中的第一段，但从 C 页面返回后，C 页面已经销毁了，这时再获取 this.data.tipRecord 是销毁页面前的数据（刚开始这么写的时候我以为会报错），也就是从 C 返回时的数据。将代码修改为第二段，主要是更新全局提示记录的处理是基于全局 store 中的数据，而不是当前页面的 data，该问题就解决了。

在 C 页面中的音频播放监听方法：
```
// 第一段，有问题的
this.backgroundAudioManager.onPause(() => {
    // 全局提示记录处理，更新 store 中的全局信息
    this.setTipRecord({
        ...this.data.tipRecord,
        show: true,
        paused: true,
    });
});

// 第二段，修正后的
this.backgroundAudioManager.onPause(() => {
    // 获取全局 store 中的音频播放信息
    const storeData = this.store.getState();
    // 全局提示记录处理，更新 store 中的全局信息
    this.setTipRecord({
        ...storeData.tipRecord,
        show: true,
        paused: true,
    });
});
```

## 参考组件

[弹幕 doomm](https://gitee.com/SolitaryAngel/doomm)
[滑动删除 wepy-swipe-delete（考虑封装）](https://github.com/GeoffZhu/wepy-swipe-delete)
[小程序官方示例](https://github.com/jingle1267/demo)
[小程序 Redux 绑定库](https://github.com/charleyw/wechat-weapp-redux)
[富文本解析 wxParse（考虑改为 ts）](https://github.com/icindy/wxParse)
[可滑动 tabs（考虑封装）](https://www.jianshu.com/p/c681007a6287)
[赞组件库 zanui-weapp](https://github.com/youzan/zanui-weapp)
[MinUI 组件库](https://github.com/meili/minui)
[wepy基础 UI 组件](https://github.com/wenshan/iwebchat)

## 声明周期方法

// 初始化，显示 Page1 页面
App === onLaunch
App === onShow
Page1 === onLoad
Page1 === onShow
Page1 === onReady

// 跳转到 Page2 页面
Page1 === onHide
Page2 === onLoad
Page2 === onShow
Page2 === onReady

// 从 Page2 返回 Page1 页面
Page2 === onUnload
Page1 === onShow

// 切后台
Page1 === onHide
App === onHide

## 自定义 Component

自定义 Component 需要注意的主要是 Component 中调用调用处 Page 的方法的方式，以及自定义子组件的传入方式。

#### Page 的代码

page.json
```
{
  "usingComponents": {
    "voice": "/components/voice/voice",
  }
}
```

page.js
```
Page({
    ...
    handleVoiceSeek(e) {
        // e.detail 内是 Component 传入的参数，还可通过 e.target.dataset.src 获取调用组件时传入的 data-src 参数
        this.backgroundAudioManager.seek(e.detail.value);
    },
});
```

page.wxml
```
<view>
    <voice
        data-src="{{particle.audio.uri}}"
        bind:_handleVoiceSeek="handleVoiceSeek"
        src="{{particle.audio.uri}}"
        playing="{{true}}"
        currentTime="{{particle.audio.currentTime}}"
        duration="{{particle.audio.duration}}"
        formatedCurrentTime="{{particle.audio.formatedCurrentTime}}"
        formatedEndedTime="{{particle.audio.formatedEndedTime}}"
    >
        <view slot="content">这里是插入到组件slot中的内容</view>
    </voice>
</view>
```

#### Component 的代码

component.json
```
{
  "component": true
}
```
component.ts
```
Component({
  options: {
    multipleSlots: true, // 在组件定义时的选项中启用多slot支持
  },
  methods: {
    _handleSeek(e) {
        // 此处的参数 e 是 slider 传入的，触发调用处的 bind:_handleVoiceSeek 对应的 js 中定义的方法并传入参数，也就是调用 handleVoiceSeek 方法
        this.triggerEvent('_handleVoiceSeek', { value: e.detail.value });
    },
  },
  properties: {
    src: {
      type: String,
    },
    playing: {
      type: Boolean,
    },
    currentTime: {
      type: Number,
    },
    duration: {
      type: Number,
    },
    formatedCurrentTime: {
      type: String,
    },
    formatedEndedTime: {
      type: String,
    },
  },
});
```
component.wxml
```
<view class="container">
    <image wx:if="{{playing}}" class="icon" src="/images/pause.png" />
    <image wx:else class="icon" src="/images/play.png" />
    <view class="sliderContainer">
        <slider min="0" max="{{duration}}" value="{{currentTime}}" bindchange="_handleSeek" />
        <view class="time">
            <text>{{formatedCurrentTime}}</text>
            <text>{{formatedEndedTime}}</text>
        </view>
    </view>
    <slot name="content" />
</view>
```

## 探索页面卸载前的监听、页面返回监听、退出小程序监听

目前小程序未提供这些功能！

## 探索是否自定义 Header 和 TabBar

在有些场景使用官方提供的 Header 和 TabBar 可能无法满足需求，官方会说可以使用自定的形式，但我查了下社区自定义可能存在不好处理的兼容性问题、android 原生事件无法监听的问题。如果没有真的特别需要、必须自定义，暂时还是不要自定义了。

## 探索小程序运行原理

微信小程序运行在三端：iOS、Android 和 开发者工具。在 iOS 上，小程序的 javascript 代码是运行在 JavaScriptCore 中（苹果开源的浏览器内核）；在 Android 上，小程序的 javascript 代码是通过 X5 内核来解析（QQ浏览器内核），X5 对 ES6 的支持不好，要兼容的话，可以使用 ES5 或者引入 babel-polyfill 兼容库；在开发工具上，小程序的 javascript 代码是运行在 nwjs（chrome内核）。

微信开发者工具编辑器的实现原理和方式：它本身也是基于WEB技术体系实现的，nwjs + react，nwjs 简单说就是node + webkit，node提供给本地api能力，webkit提供web能力，两者结合就能使用JS+HTML实现本地应用程序，wxml 转化为 html用的是 reactjs，包括里面整套的逻辑都是建构在 reactjs 之上的。既然有nodejs，打包时ES6转ES5可引入babel-core的node包，CSS补全可引入postcss和autoprefixer的node包（postcss和autoprefixer的原理看这里），代码压缩可引入uglifyjs的node包。

微信小程序的 JavaScript 运行环境即不是 Browser 也不是 Node.js。它运行在微信 App 的上下文中，不能操作 Browser context 下的 DOM，也不能通过 Node.js 相关接口访问操作系统 API。所以，严格意义来讲，微信小程序并不是 Html5，尽管开发过程和用到的技术栈和 Html5 是相通的。开发者写的所有代码最终将会打包成一份 JavaScript，并在小程序启动的时候运行，直到小程序销毁。

小程序的视图层目前使用 WebView 作为渲染载体，而逻辑层是由独立的 JavascriptCore 作为运行环境。在架构上，WebView 和 JavascriptCore 都是独立的模块，并不具备数据直接共享的通道。当前，视图层和逻辑层的数据传输，实际上通过两边提供的 evaluateJavascript 所实现。即用户传输的数据，需要将其转换为字符串形式传递，同时把转换后的数据内容拼接成一份 JS 脚本，再通过执行 JS 脚本的形式传递到两边独立环境。但是 evaluateJavascript 的执行会受很多方面的影响，数据到达视图层并不是实时的。

小程序代码包经过编译后，会放在微信的 CDN 上供用户下载，CDN 开启了 GZIP 压缩，所以用户下载的是压缩后的 GZIP 包，其大小比代码包原体积会更小。目前小程序打包是会将工程下所有文件都打入代码包内，也就是说，这些没有被实际使用到的库文件和资源也会被打入到代码包里，从而影响到整体代码包的大小。小程序启动时会从CDN下载小程序的完整包，一般是数字命名的，如：_-2082693788_4.wxapkg。

小程序正式部署使用 webpack 打的包，而在打包的过程中，把以下变量给屏蔽了：`window,document,frames,self,location,navigator,localStorage,history,Caches,screen,alert,confirm,prompt,XMLHttpRequest,WebSocket`。主要是为了管理和监控，如果这些对象你能访问，就可以像操作通常的网页一样操作小程序，这是绝对不被允许的。

{% img https://zhulichao.github.io/2018/04/08/wechat-base2/folder.png 小程序打包后的结构 %}

所有的小程序最后基本都被打成上面的结构，其中：
- WAService.js 框架JS库，提供逻辑层基础的API能力
- WAWebview.js 框架JS库，提供视图层基础的API能力
- WAConsole.js 框架JS库，控制台
- app-config.js 小程序完整的配置，包含我们通过app.json里的所有配置，综合了默认配置型
- app-service.js 我们自己的JS代码，全部打包到这个文件
- page-frame.html 小程序视图的模板文件，所有的页面都使用此加载渲染，且所有的WXML都拆解为JS实现打包到这里
- pages 所有的页面，这个不是我们之前的wxml文件了，主要是处理WXSS转换，使用js插入到header区域

微信小程序的框架包含两部分：AppView视图层，AppService逻辑层。AppView层用来渲染页面结构，所有的视图（wxml和wxss）都是单独的 WebView来承载。AppService层用来逻辑处理、数据请求、接口调用，整个小程序只有一个，并且整个生命周期常驻内存。它们在两个进程（两个 WebView）里运行，所以小程序打开至少就会有2个 WebView进程，由于每个视图都是一个独立的 WebView进程，考虑到性能消耗，小程序不允许打开超过5个层级的页面，当然同是也是为了体验更好。使用消息 publish 和 subscribe 机制实现两个 WebView之间的通信，实现方式就是统一封装一个 WeixinJSBridge 对象，而不同的环境封装的接口不一样。

对逻辑和UI进行了完全隔离，这个跟当前流行的react，agular，vue有本质的区别，小程序逻辑和UI完全运行在2个独立的WebView里面，而后面这几个框架还是运行在一个 WebView 里面的，如果你想还是可以直接操作 dom 对象进行 ui 渲染的。

微信自己写了2个工具：wcc 把 WXML 转换为 VirtualDOM；wcsc 把 WXSS 转换为一个 JS 字符串的形式通过 style 标签append 到 header 里。

## 探索图片适配问题

方式一：
```
@media  (-webkit-min-device-pixel-ratio: 2),(min-device-pixel-ratio: 2){
  .imgTest{
    background: url(https://images/2x.png) no-repeat;
  }
}
@media  (-webkit-min-device-pixel-ratio: 3),(min-device-pixel-ratio: 3){
  .imgTest{
    background: url(https://images/3x.png) no-repeat;
  }
}
```

方式二：
```
// 获取设备像素比，在 wxml 中根据不同像素比显示不同图片
const getPixelRatio = () => {
  let pixelRatio = 0
  wx.getSystemInfo({
    success: function (res) {
      pixelRatio = res.pixelRatio
    },
    fail: function () {
      pixelRatio = 0
    }
  })
  return pixelRatio
}
```

方式三：
只使用三倍图，或者说最大尺寸的图片，在样式中指定宽高。我个人和团队的 UI 设计师都倾向于使用这种方式，因为小程序限制上传代码的大小为 2M，尽量不使用多套图片，使用 UI 切好的大尺寸的图片在小尺寸手机上效果也不会有太大影响。

## 不同类型图片的适用场景

GIF（图形交换格式）：是位图的一种，但与JPEG或PNG不同，GIF文件最多使用来自256种颜色的色板中的颜色，虽然256种可能听起来像很多蜡笔，但复杂的照片通常有数千种色调。此颜色范围在GIF转换过程中丢失，而这也是彩色照片不使用GIF格式的关键原因。但是256色的限制可以帮助保持较小的文件大小，这对于即使是最慢的互联网速度也是理想的。无损，提供了网络透明度选项，适用于简单的动画、小图标、低像素间变化的图形（即大量的平面颜色，如标志和旗帜）。

JPEG（联合图像专家组）：可以将此格式称为“JPEG”或“JPG”，是一种16位格式，可以通过混合红色、蓝色和绿色光来显示数百万的颜色。这使得JPG非常“相片友好”。这就是为什么当涉及到市场上大多数的数码相机时，它会是一个标准格式的部分原因。JPEG格式还允许您灵活选择压缩图像的程度——从0％（重压缩）到100％（无压缩）。60％-75％的压缩程度会大幅缩减文件大小，同时能使您的图像在大多数屏幕看起来无差别。有损，不能保留透明度，适用于静止图像、摄影、图像具有复杂的颜色和动态。

PNG（便携式网络图形）：就像GIF和JPEG格式之间的结合，PNG-8格式与GIF相似，使用256色板，PNG-24与JPEG相似，允许用数百万种颜色来渲染图像。无损，提供了保留透明度选项，适用于需要透明度的Web图形、颜色重和复杂的照片和图形、需要重新编辑和重新导出的图片。

SVG（可缩放向量图形）：不是纯位图格式，是一个矢量格式。基于XML的标记，可以在任何文本编辑器中编辑，并通过JavaScript或CSS修改。因为矢量可以缩放到任何大小，同时保持清晰的图像质量，它们是响应设计的理想选择。SVG文件中嵌入位图图形是有可能的，正如在HTML中嵌入JPEG图片，这给了SVG无可挑剔的灵活性和力量。矢量/无损，适合显示在矢量图形应用程序（如Illustrator，Sketch和Inkscape）中制作的徽标，图标，地图，标记，图表和其他图形，视网膜屏幕显示。

## 探索样式中不同单位的区别 px、rpx、rem、pt 等（待完成）

在显示屏上，每一个画面都是由无数的点阵形成的，这个点阵中，每一个点叫做像素，就是 pixel（缩写为 px），1px 所能代表的尺寸并非是一成不变的，同样 px 尺寸的元素，在高分屏上显示会明显要比在低分屏显得更小。css 中的 px 单位，是多个设备像素，1个css像素所对应的物理像素个数是不一致的，每英寸的像素的数量保持在96左右，因此设置为12px的元素，无论使用什么样的设备查看网页，字体的大小始终为1/8英寸。

px 像素，是一个点，点的大小是会变的，也称“相对长度”
pt 全称 point，1/72英寸，用于印刷业，也称“绝对长度”
dp = px * (目标设备 dpi 分辨率 / 160)
rpx = px * (目标设备宽 px 值 / 750)
rem: 规定屏幕宽度为20rem；1rem = (750/20)rpx

dp 是以屏幕分辨率为基准的动态单位，而 rpx 是以长度为基准的动态单位，两者是不能直接进行互换的。

如果将微信小程序放到平板电脑上运行，屏幕的宽度 px 值有可能会变化（横竖屏、分屏模式等等），这时候，再以宽度为基准，就会出现元素显示不正确的问题，从这一点可以看出，微信团队目前并不希望将小程序扩展到手机以外的设备中。

## 多屏适配
