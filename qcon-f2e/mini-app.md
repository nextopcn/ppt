

## 前端 - 小程序


Web技术 应用场景扩展: Chrome App / 轻应用 / 快应用 / Hybrid 应用 / Electron / PWA / 小程序

- app 强大原生能力 + web 灵活开发模式
- Hybrid 标准化

### 微信小程序

优势:

- 快速的加载 (缓存,分包)
- 更强大的能力(宿主提供)
- 原生的体验(渲染优化)
- 易用且安全的微信数据开放(云生态)
- 高效和简单的开发(web 技术栈)


#### 技术相关


- 渲染和逻辑分离: webview + jscore

    - 管控与安全
    - 异步
    - 数据驱动UI

- 统一应用生命周期

- wxml / wxss / javascript

    - wxml -> js
    - wxss -> css

- web 组件与原生组件

    - canvas/camera/map/input/textarea/video/..


### 苏宁小程序

#### 转换工具


<details>
<summary>微信小程序 --> Babel --> 目标小程序</summary>

![](https://user-gold-cdn.xitu.io/2017/2/10/262609f85a934feff1933a10190bfb1e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

</details>



- 转换  view 层 关系代码
- 复写 登录,支付 等业务逻辑
- 遵从书写规范

#### 苏宁小程序

- 允许在苏宁系 app 中
- api 与微信保持同步
- react-native

### 360 小程序平台

![](http://yanxuan.nosdn.127.net/5dd64379c27b633926ae7fa473ea46ec.png)

- pc桌面 (小程序,小游戏)
- 基于 vue 框架,部分兼容 vue 生态
- 与主流小程序 api , 生命周期 兼容
- 单引擎
- 增强安全性 (安全域/CSP/禁用危险api)

### 钉钉小程序选型建议

- Native：重度用户高频场景，视觉还原度要求高，优选Native；
- H5：小程序支持不了的Web技术标准，优选H5；
- 小程序(含Native渲染)：新业务，创新产品，优选小程序；



### 小程序组件化开发工具\框架

- [Taro](https://github.com/NervJS/taro)
- [WePy 小程序组件化开发框架](https://wepyjs.github.io/wepy-docs/)
- [uni-app](https://uniapp.dcloud.io/)
- [Tencent/omi](https://github.com/Tencent/omi)
