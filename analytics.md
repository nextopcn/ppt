# Firebase analytics

### 简介

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

### 自定义事件

### 最佳实践

### 集成 BigQuery

```
select * from `web-test-9f920.analytics_300153741.events_20220206` where event_name = 'crash_event' and event_date = '20220206'
```