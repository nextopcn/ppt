# influxdb

## 1. 简介
InfluxDB是一个用于存储和分析时间序列数据的开源数据库。　时间序列数据库主要用于指处理带时间标签（按照时间的顺序变化，即时间序列化）的数据，带时间标签的数据也称为时间序列数据。

## 2. 安装
1. 硬件配置
```java  

CPU : 8+ cores
RAM : 32+ GB
IOPS : 1000+  
推荐用SSD 而非HDD

```

2. 安装
```java  

sudo yum install influxdb
sudo systemctl start influxdb

# 默认的配置文件位置
/etc/influxdb/influxdb.conf

# 指定配置文件
influxd -config /etc/influxdb/influxdb.conf

```

## 3. 一些概念

```java  

name: system_jgroups
-————————————
time                      value                 app                facade                module
2015-08-18T00:00:00Z      1200                web_admin_13       bytes_sent              web
2015-08-18T00:00:00Z      10                  server_1           messages_sent           system
2015-08-18T00:06:00Z      11                  web_admin_13       messages_sent           web
2015-08-18T00:06:00Z      3                   server_1           bytes_sent              web

```

1. measurement : census
2. field key : value
3. tag key : app, facade, module
4. field set : value = 1200, value = 10, value = 11, value = 3
5. tag set :   
```java  
app = web_admin_13, facade = bytes_sent,    module = web  
app = server_1,     facade = messages_sent, module = system  
app = web_admin_13, facade = messages_sent, module = web  
app = server_1,     facade = bytes_sent,    module = web  
```

6. series :   
```java  
rp = autogen, measurement = census, tag set = (app = web_admin_13, facade = bytes_sent,    module = web)  
rp = autogen, measurement = census, tag set = (app = server_1,     facade = messages_sent, module = system)  
rp = autogen, measurement = census, tag set = (app = web_admin_13, facade = messages_sent, module = web)  
rp = autogen, measurement = census, tag set = (app = server_1,     facade = bytes_sent,    module = web)  
```

7. point :  
```java  
name: census
-----------------
time                    butterflies honeybees   location    scientist
2015-08-18T00:00:00Z    1           30          1           perpetua
```

### 层级结构
database -> retention policy -> measurement -> point

### 和传统数据库比较
1. InfluxDB measurement 比较像传统数据库的表
2. InfluxDB tags 比较像传统数据库中带索引的列
3. InfluxDB fields 比较像传统数据中不带索引的列
4. InfluxDB points 比较像传统数据库中一行数据

## 4. 客户端

```java  
            <dependency>
                <groupId>org.influxdb</groupId>
                <artifactId>influxdb-java</artifactId>
                <version>2.15</version>
            </dependency>
```

```java  

InfluxDB r = InfluxDBFactory.connect("http://172.30.1.136:8086", "nextop", "nextop");
r.setDatabase("thorin_dev").setRetentionPolicy("30days").enableBatch(BatchOptions.DEFAULTS).enableGzip();

Point p = Point.measurement("system_jgroups").addField("time", 1569299564814L).addField("value", 1024).addField("avg", 0.234d)
		.tag("app", "server_1").tag("module", "web").tag("type", "COUNTER")).tag("facade", "BYTES_SENT").build();
		
r.write(p);
```

### 4.1. Batch 插入

```java  
BatchOptions opt = DEFAULTS;
opt.consistency(ONE);     // 单机设置成ONE
opt.jitterDuration(1000); // 写入的时候增加一个随机的时间
opt.actions(256);         // 批处理大小
opt.bufferLimit(8192);    // 失败的写入会存到这个buffer里重试， lru替换最旧的数据 
opt.flushDuration(2000);  // 每隔2秒写入一次
opt.exceptionHandler(...);// 写入异常处理
opt.threadFactory(...);   // 设置线程池
```

### 4.2 行协议
weather,location=us-midwest,season=summer temperature=82 1465839830100400200\n  
weather,location=us-midwest,season=winter temperature=10 1465839830100400200\n  

### 4.3 Field 数据类型
1. 浮点数   82,  10
2. 整数     82i, 10i
3. 字符串   "82"
4. 布尔型   t,T,true,True,TRUE, f,F,false,False, FALSE

## 5. 备份与还原

```java  
influxd backup -portable <path-to-backup>
influxd restore -portable <path-to-backup>

influxd backup -portable -database mydatabase -host <remote-node-IP>:8088 <path-to-backup>
influxd restore -portable -db mydatabase <path-to-backup>

influxd backup  -portable -database mytsd -start 2017-04-28T06:49:00Z -end 2017-04-28T06:50:00Z <path-to-backup>
influxd restore -portable -db mytsd <path-to-backup>
```

## 6. influxql