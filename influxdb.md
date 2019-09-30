# influxdb

## 1. 简介
InfluxDB是一个用于存储和分析时间序列数据的开源数据库。时间序列数据库主要用于指处理带时间标签（按照时间的顺序变化，即时间序列化）的数据，带时间标签的数据也称为时间序列数据。

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

1. measurement : system_jgroups
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
rp = autogen, measurement = system_jgroups, tag set = (app = web_admin_13, facade = bytes_sent,    module = web)  
rp = autogen, measurement = system_jgroups, tag set = (app = server_1,     facade = messages_sent, module = system)  
rp = autogen, measurement = system_jgroups, tag set = (app = web_admin_13, facade = messages_sent, module = web)  
rp = autogen, measurement = system_jgroups, tag set = (app = server_1,     facade = bytes_sent,    module = web)  
```

7. point :  
```java  
name: system_jgroups
-----------------
time                      value                 app                facade                module
2015-08-18T00:00:00Z      1200                web_admin_13       bytes_sent              web
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

## 5. influxQL

1. 查询语句
```java  

# 指定数据库
use thorin;

# 查询h2o_feet这个measurement所有tag和field
SELECT * FROM "h2o_feet"

# 查询h2o_feet这个measurement指定的tag和field
SELECT "level description","location","water_level" FROM "h2o_feet"

# 查询h2o_feet这个measurement所有的field
SELECT *::field FROM "h2o_feet"

# 查询时做基本的数学运算
SELECT ("water_level" * 2) + 4 from "h2o_feet"

# 指定全量的前缀， db名 + 保留策略 
SELECT * FROM "NOAA_water_database"."autogen"."h2o_feet"

# 如果不指定field, 那么不会返回任何数据，至少指定一个field 
SELECT "location" FROM "h2o_feet"

# 带有where条件的查询
SELECT * FROM "h2o_feet" WHERE "water_level" > 8

# where条件中进行数学运算
SELECT * FROM "h2o_feet" WHERE "water_level" + 2 > 11.9

# 多个条件
SELECT "water_level" FROM "h2o_feet" WHERE "location" <> 'santa_monica' AND (water_level < -0.59 OR water_level > 9.95)

# 指定时间段的where条件
SELECT * FROM "h2o_feet" WHERE time > now() - 7d

# group by 
SELECT MEAN("water_level") FROM "h2o_feet" GROUP BY "location"

# 多个条件group by 
SELECT MEAN("index") FROM "h2o_quality" GROUP BY location,randtag

# 以时间段group by
SELECT COUNT("water_level") FROM "h2o_feet" WHERE "location"='coyote_creek' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' GROUP BY time(12m)

# fill 操作， 在时间段内查不出数据的时候插入一条100
SELECT MAX("water_level") FROM "h2o_feet" WHERE "location"='coyote_creek' AND time >= '2015-09-18T16:00:00Z' AND time <= '2015-09-18T16:42:00Z' GROUP BY time(12m) fill(100)

# fill 操作， 在时间段内查不出数据的时候把之前一条数据插入进去
SELECT MAX("water_level") FROM "h2o_feet" WHERE location = 'coyote_creek' AND time >= '2015-09-18T16:24:00Z' AND time <= '2015-09-18T16:54:00Z' GROUP BY time(12m) fill(previous)

# 线性fill操作 在时间段内查不出数据的时候算出一个线性值
SELECT MEAN("tadpoles") FROM "pond" WHERE time > '2016-11-11T21:24:00Z' AND time <= '2016-11-11T22:06:00Z' GROUP BY time(12m) fill(linear)

# fill(null)
SELECT MEAN("tadpoles") FROM "pond" WHERE time > '2016-11-11T21:24:00Z' AND time <= '2016-11-11T22:06:00Z' GROUP BY time(12m) fill(null)

# order by 
SELECT "water_level" FROM "h2o_feet" WHERE "location" = 'santa_monica' ORDER BY time DESC

# limit
SELECT "water_level","location" FROM "h2o_feet" LIMIT 3

# slimit <N> 选取N个series的所有数据
SELECT "water_level" FROM "h2o_feet" GROUP BY * SLIMIT 1

# offset
SELECT "water_level","location" FROM "h2o_feet" LIMIT 3 OFFSET 3

# soffset <N> 从第N个series开始选取
SELECT "water_level" FROM "h2o_feet" GROUP BY * SLIMIT 1 SOFFSET 1
```

2. 数据管理
```java  

# 创建数据库
CREATE DATABASE "NOAA_water_database"

# 删除数据库
DROP DATABASE "NOAA_water_database"

# 删除series
DROP SERIES FROM "h2o_feet"
DROP SERIES FROM "h2o_feet" WHERE "location" = 'santa_monica'
DROP SERIES WHERE "location" = 'santa_monica'

# 与delete的区别， delete不删除tag的index， drop不支持时间段删除
DELETE FROM "h2o_feet"
DELETE FROM "h2o_quality" WHERE "randtag" = '3'
DELETE WHERE time < '2016-01-01'

# 删除MEASUREMENT
DROP MEASUREMENT "h2o_feet"

# 创建保留策略
CREATE RETENTION POLICY "one_day_only" ON "NOAA_water_database" DURATION 23h60m REPLICATION 1 DEFAULT

# 删除保留策略同时删除数据
DROP RETENTION POLICY "what_is_time" ON "NOAA_water_database"
```

3. 重要的函数
```java  

COUNT()
MEAN()
MEDIAN()
SUM()
FIRST()
LAST()
MAX()
MIN()
TOP(field_key,N)
ABS()
CEIL()
POW()
ROUND()
```
## 6. 备份与还原

```java  
influxd backup -portable <path-to-backup>
influxd restore -portable <path-to-backup>

influxd backup -portable -database mydatabase -host <remote-node-IP>:8088 <path-to-backup>
influxd restore -portable -db mydatabase <path-to-backup>

influxd backup  -portable -database mytsd -start 2017-04-28T06:49:00Z -end 2017-04-28T06:50:00Z <path-to-backup>
influxd restore -portable -db mytsd <path-to-backup>
```

## 7. 配置
```java  
# 配置文件位置
/etc/influxdb/influxdb.conf

# 关键配置项

# influxdb联网收集信息选项，默认开启，要设置为false
reporting-disabled = false

# 绑定的ip地址
bind-address = "127.0.0.1:8088"

# meta信息，如数据库，用户，保留策略等等
[meta]
dir = "/var/lib/influxdb/meta"

# 自动创建一个autogen的保留策略
retention-autocreate = true

# influxdb数据存储位置
[data]
dir = "/var/lib/influxdb/data"

# 预写日志位置
wal-dir = "/var/lib/influxdb/wal"

# fsync等待时间， 大于0可以执行fsync批量写入，hdd推荐设置100ms以内
wal-fsync-delay = "0s"

# 内存中创建index, 也可以创建disk index, 把值改为tsi1
index-version = "inmem"

# 缓存占用的内存大小
cache-max-memory-size = "1g"

# 限制数据库的级数，该值为0时取消限制，默认值：1000000
max-series-per-database = 1000000  

# 一个tag最大的value数，0取消限制，默认值：100000
max-values-per-tag = 100000  

# 小的值会引发经常性数据压缩， 调大可以减少数据压缩的次数，增加写入的吞吐量
max-index-log-file-size = "1m"

# series 缓存大小， 按每个数据库series数量调整
series-id-set-cache-size = 100

```