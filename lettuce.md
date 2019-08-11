# Lettuce

Lettuce是基于Netty，并且线程安全的Redis客户端，支持同步异步响应式API，并且支持Redis的单机，主从，哨兵，集群模式。

1. 简单使用
2. Redis URI
3. 命令相关
4. HA
5. 连接池
6. 配置

## 1. 简单使用

```xml  

<dependency>
    <groupId>io.lettuce</groupId>
    <artifactId>lettuce-core</artifactId>
    <version>5.1.3.RELEASE</version>
</dependency>

```

```java  

RedisClient client = RedisClient.create("redis://password@localhost:6379/0");
StatefulRedisConnection<String, String> connection = client.connect();
RedisCommands<String, String> sync = connection.sync();                       // ➀
// RedisAsyncCommands<String, String> sync = connection.async();              
// RedisReactiveCommands<String, String> reactive = connection.reactive();    


String value = sync.get("key");                                               // ➁
// RedisFuture<String> value = async.get("key");                
// Mono<String> value = reactive.get("key");

connection.close();                                                           // ➂
client.shutdown();                                                            // ➃ 
```

## 2. Redis URI

```java  

RedisURI standalone = RedisURI.create("redis://:password@host:port/db?timeout=6s");

RedisURI sentinel = RedisURI.create("redis-sentinel://:password@host1:port1,host2:port2,host3:port3/db?timeout=6s&sentinelMasterId=mymaster");

// ssl
RedisURI ssl = RedisURI.create("rediss://:password@host:port/db?timeout=6s");
```

## 3. 命令相关

### 3.1. Pub sub

```java  

StatefulRedisPubSubConnection<String, String> connection = client.connectPubSub()
connection.addListener(new RedisPubSubListener<String, String>() { ... })

RedisPubSubCommands<String, String> sync = connection.sync();
sync.subscribe("+switch-master");

```

### 3.2. Pipeline

```java  

StatefulRedisConnection<String, String> connection = client.connect();
RedisAsyncCommands<String, String> commands = connection.async();

// disable auto-flushing
commands.setAutoFlushCommands(false);

// perform a series of independent calls
List<RedisFuture<?>> futures = Lists.newArrayList();
for (int i = 0; i < iterations; i++) {
    futures.add(commands.set("key-" + i, "value-" + i));
    futures.add(commands.expire("key-" + i, 3600));
}

// write all commands to the transport layer
commands.flushCommands();
commands.setAutoFlushCommands(true);

// synchronization example: Wait until all futures complete
boolean result = LettuceFutures.awaitAll(5, TimeUnit.SECONDS, futures.toArray(new RedisFuture[futures.size()]));

```

### 3.3. Transaction

```java  

// 同步API
StatefulRedisConnection<String, String> connection = client.connect();
RedisCommands<String, String> sync = connection.sync();

sync.multi()             // return "OK"
sync.set(key1, value);   // return null
sync.set(key2, value);   // return null;
sync.exec();             // return list("OK", "OK")

// 异步API
RedisAsyncCommands<String, String> async = connection.async();

async.multi();
async.set(key1, value);
async.set(key2, value);
RedisFuture<List<Object>> exec = async.exec();

List<Object> list = exec.get();
list.get(0); // return OK;
list.get(1); // return OK;
```

## 4. HA

### 4.1. 哨兵
```java  

RedisClient client = RedisClient.create("redis-sentinel://localhost:26379,localhost:26380/0#mymaster");
StatefulRedisConnection<String, String> connection = client.connect();

RedisCommands<String, String> sync = connection.sync();
// RedisAsyncCommands<String, String> sync = connection.async();
// RedisReactiveCommands<String, String> reactive = connection.reactive();
...

connection.close();
client.shutdown();

```

### 4.2. 集群

```java  

RedisURI seed1 = RedisURI.create("node1", 6379);
RedisURI seed2 = RedisURI.create("node2", 6379);

RedisClusterClient client = RedisClusterClient.create(Arrays.asList(seed1, seed2));
StatefulRedisClusterConnection<String, String> connection = client.connect();
RedisAdvancedClusterCommands<String, String> syncCommands = connection.sync();

// RedisClusterCommands<String, String> node1 = connection.getConnection("host", 7379).sync();

...

connection.close();
client.shutdown();

```

## 5. 连接池

### 5.1. 同步连接池 
```java  

GenericObjectPool<StatefulRedisConnection<String, String>> pool = ConnectionPoolSupport
               .createGenericObjectPool(() -> client.connect(), new GenericObjectPoolConfig());

try (StatefulRedisConnection<String, String> connection = pool.borrowObject()) {

    RedisCommands<String, String> commands = connection.sync();
    commands.multi();
    commands.set("key", "value");
    commands.set("key2", "value2");
    commands.exec();
}

pool.close();
client.shutdown();

```

### 5.2. 异步连接池
```java  

RedisClient client = RedisClient.create();

AsyncPool<StatefulRedisConnection<String, String>> pool = AsyncConnectionPoolSupport.createBoundedObjectPool(
        () -> client.connectAsync(StringCodec.UTF8, RedisURI.create(host, port)), BoundedPoolConfig.create());

CompletableFuture<TransactionResult> r = pool.acquire().thenCompose(connection -> {

    RedisAsyncCommands<String, String> async = connection.async();

    async.multi();
    async.set("key", "value");
    async.set("key2", "value2");
    return async.exec().whenComplete((s, throwable) -> pool.release(c));
});

pool.closeAsync();
client.shutdownAsync();
```

## 6. 配置