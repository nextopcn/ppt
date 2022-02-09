# Firebase analytics

### 简介

Google Analytics（分析）是一款免费的应用分析解决方案，可提供关于应用使用情况和用户互动度的数据分析。
主要功能  

|分析报告|Analytics 可以针对最多 500 种不同类型的事件生成分析报告|
|----|----|
|受众群细分|可以根据设备数据、自定义事件或用户属性在 Firebase 控制台中自定义受众群体。定位新功能或通知消息时，这些受众群体信息可以与其他 Firebase 功能结合使用|

### 服务端设置



### 客户端设置

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

logEvent(analytics, 'crash_event', {version : '1.1.0', account_id : '178187273', user_agent : 'chrome', company : '1', route : 'express', env : 'test', });

```

### 通用事件

[sign_up](https://developers.google.com/gtagjs/reference/event#sign_up)
[login](https://developers.google.com/gtagjs/reference/event#login)
[exception](https://developers.google.com/gtagjs/reference/event#exception)
[page_view](https://developers.google.com/gtagjs/reference/event#page_view)
[screen_view](https://developers.google.com/gtagjs/reference/event#screen_view)
[search](https://developers.google.com/gtagjs/reference/event#search)

### 自定义事件

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

### 集成 BigQuery

```
select * from `web-test-9f920.analytics_300153741.events_20220206` where event_name = 'crash_event' and event_date = '20220206'
```

### 潜在风险

[will-google-analytics-banned-in-europe-in-2022](https://www.eseller365.com/will-google-analytics-banned-in-europe-in-2022/)