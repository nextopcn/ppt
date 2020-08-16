# Java9 module简介

# module的三种形式
* Full qualified name module (带module-info.java文件)
* Automatic module (META-INF/MANIFEST.MF上有Automatic-Module-Name属性)
* Unnamed module (未升级到java9模块系统的遗留jar包)

## Full qualified name module

* 一个简单的例子

```java  
src/
├── main
│   └── java
│       ├── com
│       │   └── moilioncircle
│       │       └── redis
│       │           └── replicator
│       │               ├── AbstractReplicator.java
│       │               ├── AbstractReplicatorListener.java
│       │               ├── FileType.java
│       │               ├── rdb
│       │               │   ├── BaseRdbParser.java
│       │               │   ├── DefaultRdbValueVisitor.java
│       │               │   ├── DefaultRdbVisitor.java
│       │               └── util
│       │                   ├── Arrays.java
│       │                   ├── ByteArray.java
│       │                   ├── ByteArrayList.java
│       │                   ├── ByteArrayMap.java
│       └── module-info.java

```

* 导出上述package
  
```java  

module com.moilioncircle.redis.replicator {
    exports com.moilioncircle.redis.replicator;
    exports com.moilioncircle.redis.replicator.rdb;
    exports com.moilioncircle.redis.replicator.util;
    requires org.slf4j;
}
```

* module文件的语法

```java  
module name
open module name
exports package
exports package to package
opens package
opens package to package
require module
require transitive module
require static module 

```

## Automatic module

## Unnamed module

## 模块系统的一些工具链
* jdeps
* jmods
* jlinks
