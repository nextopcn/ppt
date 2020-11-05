
## 前端 - PWA


### PWA

`Progressive` Web App

将 Web 和 App 各自的优势融合在一起,提升 Web App 的性能，改善 Web App 的用户体验

- **白屏, 离线, 动画, 渲染等问题**

- **唤起**

相关概念:

- AMP

- `PRPL` (push/preload, render, recache, lazy-load)




### 特点

  - 渐进式 - 适用于所有浏览器，因为它是以渐进式增强作为宗旨开发的
  - 连接无关性 - 能够借助 Service Worker 在离线或者网络较差的情况下正常访问
  - 类似应用 - 由于是在 App Shell 模型基础上开发，因为应具有 Native App 的交互和导航，给用户 Native App 的体验
  - 持续更新 - 始终是最新的，无版本和更新问题
  - 安全 - 通过 HTTPS 协议提供服务，防止窥探和确保内容不被篡改
  - 可索引 - 应用清单文件和 Service Worker 可以让搜索引擎索引到，从而将其识别为『应用』
  - 粘性 - 通过推送离线通知等，可以让用户回流
  - 可安装 - 用户可以添加常用的 webapp 到桌面，免去去应用商店下载的麻烦
  - 可链接 - 通过链接即可分享内容，无需下载安装

### 相关技术/API


#### Web App Manifest


```
<link rel="manifest" href="/manifest.json">
```

[Manifest doc](https://developer.mozilla.org/en-US/docs/Web/Manifest)

`入口`


#### Service Worker


- 改善应用对网络的依赖

  > Service Worker是PWA中必不可少的一个服务，主要提供资源缓存、离线访问等功能，并可动态进行数据更新.简单来说，Service Worker 就是一段运行在 Web 浏览器中，并为应用`管理缓存`的脚本，会拦截所有由应用发出的 HTTP 请求，并选择如何给出响应 (网络代理)


- 文档无关的生命周期

![](https://yqfile.alicdn.com/08f326c3cae549f0408f0bb25624c191668bd25a.png)

- 浏览器导航流程



#### SYNC & PUSH  NOTIFICATION

- SYNC: 后台处理请求
- `Web Push` 协议 (chrome:  Firebase FCM)
- 通知

支持情况

### Project Fugu (フグ)

![](http://yanxuan.nosdn.127.net/c0683dfda92bd1d9e6439f36d0e7e551.png)

更进一步大道 app 的能力


- 定义 api,补全功能
- w3c 标准化推进
-

##### APIS...

- badge

```
window.ExperimentalBadge.set(15);
```

- Native File System Api

- NFC Api

- Contract Picker Api

- SMS Receiver Api

- Wake Lock Api

- Shape Detection Api

```
const faces = await new FaceDetector().detect(img);
```

- [apis](https://docs.google.com/spreadsheets/d/1de0ZYDOcafNXXwMcg4EZhT0346QM-QFvZfoD8ZffHeA/edit#gid=557099940)



[miniapp 比较](https://w3c.github.io/miniapp/white-paper/comparison.html)


##### 安全

- https, iframe, 用户操作
- 授权管理



##### Origin Trial

相对比较成熟的Web Platform APIs的试验场，面向Web开发者有限 范围试用，收集反馈用来完善API标准以及实现。


```
<meta http-equiv="origin-trial" content="**your token**">
```


- [token](https://developers.chrome.com/origintrials/#/trials/active)
- [chrome://flag](chrome://flags)


### 其他 web 试验特性(补充)

- [跳转优化 Portals](https://storage.googleapis.com/web-dev-assets/hands-on-portals/portals_h264.mp4)

### refs

- [简单介绍一下Progressive Web App(PWA)](https://juejin.im/post/5a6c86e451882573505174e7)
- [【PWA学习与实践】(3) 让你的WebApp离线可用](https://github.com/alienzhou/blog/issues/4)
- [Web 推送技术](https://villainhr.com/page/2017/01/08/Web%20%E6%8E%A8%E9%80%81%E6%8A%80%E6%9C%AF)
- [我们真的需要网页版App吗？Google PWA的困局](https://www.leiphone.com/news/201606/UEiart497WUzS62u.html)
- [饿了么的 PWA 升级实践 - 黄玄的博客 | Hux Blog](https://huangxuan.me/2017/07/12/upgrading-eleme-to-pwa/)
- [lavas](https://lavas.baidu.com/)
- [workbox](https://codelabs.developers.google.com/codelabs/workbox-lab-cn/index.html)

