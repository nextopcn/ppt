# Firebase analytics

### 简介

Google Analytics（分析）是一款免费的应用分析解决方案，可提供关于应用使用情况和用户互动度的数据分析。
主要功能  

|分析报告|Analytics 可以针对最多 500 种不同类型的事件生成分析报告|
|----|----|
|受众群细分|可以根据设备数据、自定义事件或用户属性在 Firebase 控制台中自定义受众群体。定位新功能或通知消息时，这些受众群体信息可以与其他 Firebase 功能结合使用|

### 服务端设置

[firebase console](https://console.firebase.google.com/)

### 客户端设置

```js  
npm install firebase
```

```js

import { initializeApp } from "firebase/app";
import { getAnalytics, logEvent } from "firebase/analytics";

const firebaseConfig = {
    apiKey: "********************************",
    authDomain: "************************",
    projectId: "****************",
    storageBucket: "************************",
    messagingSenderId: "************",
    appId: "********************************",
    measurementId: "***********"
};

const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);

logEvent(analytics, 'crash', {version : '1.1.1', account_id : '178187233', user_agent : 'firefox', company : '2', route : 'express', env : 'test', });

```

### 通用事件

1. [sign_up](https://developers.google.com/gtagjs/reference/event#sign_up)
2. [login](https://developers.google.com/gtagjs/reference/event#login)
3. [exception](https://developers.google.com/gtagjs/reference/event#exception)
4. [page_view](https://developers.google.com/gtagjs/reference/event#page_view)
5. [screen_view](https://developers.google.com/gtagjs/reference/event#screen_view)
6. [search](https://developers.google.com/gtagjs/reference/event#search)

```js

logEvent(analytics, 'sign_up', {   method: 'facebook' });
logEvent(analytics, 'login', {   method: 'google' });
logEvent(analytics, 'page_view', { page_location: 'https://example.com/about', page_path: '/about', page_title: 'About' });
logEvent(analytics, 'screen_view', { screen_name: 'About' });
logEvent(analytics, 'exception', {   description: 'Missing required field.', fatal: false });
logEvent(analytics, 'search', { search_term: 'computer'});

```

### 设置维度

[Custom Definitions](https://console.firebase.google.com/project/web-test-9f920/analytics/app/web:NTAwNzk3OTQtMzkwOC00YThhLWJiMjMtMTM4NDcyOGFkNDU0/userproperty/~2F%3Ft%3D1644483590861&fpn%3D659923416702&swu%3D1&sgu%3D1&sus%3Dupgraded&params%3D_u..pageSize%253D25&cs%3Dapp.m.userproperties.overview&g%3D1)

### 限制

|内容|限制|
|----|----|
|事件数量|500 个事件|
|事件名称的长度|40 个字符|
|事件参数|25 个参数|
|事件参数名称长度|40 个字符|
|事件参数值长度|100 个字符|
|用户属性|25 个属性|
|用户属性名称长度|24 个字符|
|用户属性值长度|36 个字符|
|User-Id值长度|256 个字符|

### 最佳实践

1. 做好模块封装，随时切到换其他数据分析工具。
2. 定义crash分析事件的时候，不能把所有堆栈都放到事件里，挑取重要信息。
3. 注意不要收集用户的隐私，包括搜索关键字，下单价格等。

### 集成 BigQuery

[BigQuery](https://console.cloud.google.com/bigquery?project=web-test-9f920)

```
select * from `web-test-9f920.analytics_300153741.events_20220209` where event_name = 'crash_event'
```

[BigQuery 实战宝典](https://support.google.com/analytics/answer/4419694#zippy=%2C%E6%9C%AC%E6%96%87%E5%8C%85%E5%90%AB%E7%9A%84%E4%B8%BB%E9%A2%98)

### 潜在风险

[will-google-analytics-banned-in-europe-in-2022](https://www.eseller365.com/will-google-analytics-banned-in-europe-in-2022/)

### 相似工具

* [datadog](https://www.datadoghq.com/)
* [appdynamics](https://www.appdynamics.com/)