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
// 定义一个module
module name

// 定义一个module, 并允许其他module 引入这个module时可以反射调用私有方法
open module name

// 导出一个package, 其他module引入这个module可以调用其开放的package里的类以及方法
exports package

// 仅让some-module访问其导出的pakcage
exports package to some-module

// 开放一个package, 其他module引入这个module可以反射调用其私有方法
opens package

// 仅对some-module开放其package
opens package to some-module

// 引入一个module
require module

// 引入一个module, 并把依赖传递给上层
require transitive module

// 引入一个module, 当这个module没被运行时使用时，可以不加如到运行时环境类似maven scope optional
require static module 

// java SPI, 对interface提供某个实现
provides interface with implementment

// 使用ServiceLoader读取SPI时，此module需要添加uses interface
uses interface;
```

* 看[module-demo](https://github.com/leonchen83/module-demo)的实际的例子

## Automatic module

* MANIFEST.MF文件内容
```
Manifest-Version: 1.0
Created-By: Maven Jar Plugin 3.2.0
Build-Jdk-Spec: 11
Automatic-Module-Name: module1
```

* 利用maven jar plugin打包

```
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <archive>
                        <manifestEntries>
                            <Automatic-Module-Name>module1</Automatic-Module-Name>
                        </manifestEntries>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

## Automatic module的性质

1. exports 所有package
2. requires 所有在module path上的模块(包括Unnamed module)
3. 在META-INF/services里的SPI默认会导出(类似自动加上provides interface with implementment)

## Unnamed module的性质




## 模块系统的一些工具链
* jdeps
* jmods
* jlinks
