# Java9 module简介

JDK9引入了一个新的特性叫做JPMS（Java Platform Module System），也可以叫做Project Jigsaw。模块化的本质就是将一个大型的项目拆分成为一个一个的模块，
在module中会有元数据来描述该模块的信息和该模块与其他模块之间的关系。这些模块组合起来，构成了最后的运行程序。

# module的三种形式

* Explicit module (带module-info.java文件)
* Automatic module (META-INF/MANIFEST.MF上有Automatic-Module-Name属性或者自动根据jar包生成一个module name)
* Unnamed module (未升级到java9模块系统的遗留jar包)

## Explicit module

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

// 引入一个module, 当这个module没被运行时使用时，可以不加入到运行时环境类似maven scope optional
require static module 

// java SPI, 对interface提供某个实现
provides interface with implementment

// 使用ServiceLoader读取SPI时，此module需要添加uses interface
uses interface;
```

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

* 利用jar包自动生成名字

```
sub-module1-1.0-SNAPSHOT.jar 则会自动生成一个module sub-module1@1.0-SNAPSHOT
```

## Automatic module的性质

1. exports 所有 package
2. requires 所有在 module path上的模块(包括Unnamed module)
3. 在--module-path上的jar包会自动成为Automatic module

```
如果自动模块依赖Explicit module,那么需要在运行时添加--add-modules ${explicit-module})
如果自动模块依赖Automatic module,那么不需要在运行时添加--add-modules
```

## Unnamed module的性质

1. exports 所有 package
2. requires 所有在 module path上的模块包括其他Unnamed module
3. 在--class-path参数上的jar包会自动成为Unnamed module

```
如果未命名模块依赖Explicit module,那么需要在运行时添加--add-modules ${explicit-module})
如果未命名模块依赖Automatic module,那么不需要在运行时添加--add-modules
```

## 实际例子

* [module-demo](https://github.com/leonchen83/module-demo)

## 应用jlink进行JRE裁剪

1. 创建jlink的maven module并依赖所有自己写的java模块
```
<dependencies>
    <dependency>
        <groupId>cn.nextop</groupId>
        <artifactId>main-demo</artifactId>
        <version>1.0-SNAPSHOT</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>cn.nextop</groupId>
        <artifactId>sub-module1</artifactId>
        <version>1.0-SNAPSHOT</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>cn.nextop</groupId>
        <artifactId>sub-module2</artifactId>
        <version>1.0-SNAPSHOT</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```
2. 配置maven jlink plugin插件
```
<pluginRepositories>
    <pluginRepository>
        <id>apache.snapshots</id>
        <url>http://repository.apache.org/snapshots/</url>
        <releases>
            <enabled>false</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>

<build>
    <plugins>
        <plugin>
            <artifactId>maven-jlink-plugin</artifactId>
            <version>3.0.0-alpha-2-SNAPSHOT</version>
            <extensions>true</extensions>
        </plugin>
    </plugins>
</build>
```
3. 在maven jlink module 下执行`mvn clean jlink:jlink`并解压`jlink-1.0-SNAPSHOT.zip`
4. bin目录下执行`java --module main/cn.nextop.main.demo.Main`

## Maven multiRelease与toolchain

#### Maven multiRelease

* [JEP238](http://openjdk.java.net/jeps/238)

```java  
jar结构

A.class
B.class
C.class
D.class
META-INF/MANIFEST.MF { Multi-Release: true }
         versions/9 /A.class
                     B.class
                 /10/A.class
                     C.class
```

```java  

	├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── baeldung
│   │   │           └── multireleaseapp
│   │   │               ├── DefaultVersion.java
│   │   │               └── App.java
│   │   └── java9
│   │       └── com
│   │           └── baeldung
│   │               └── multireleaseapp
│   │                   └── DefaultVersion.java

```

```java  
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-toolchains-plugin</artifactId>
    <version>3.0.0</version>
    <executions>
        <execution>
            <id>toolchain</id>
            <goals>
                <goal>toolchain</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <toolchains>
            <jdk>
                <version>9</version>
            </jdk>
            <jdk>
                <version>1.8</version>
            </jdk>
        </toolchains>
    </configuration>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.0</version>
    <executions>
        <execution>
            <id>java-8</id>
            <goals>
                <goal>compile</goal>
            </goals>
            <configuration>
                <jdkToolchain>
                    <version>1.8</version>
                </jdkToolchain>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </execution>
        <execution>
            <id>java-9</id>
            <phase>compile</phase>
            <goals>
                <goal>compile</goal>
            </goals>
            <configuration>
                <jdkToolchain>
                    <version>9</version>
                </jdkToolchain>
                <release>9</release>
                <compileSourceRoots>
                    <compileSourceRoot>${project.basedir}/src/main/java9</compileSourceRoot>
                </compileSourceRoots>
                <multiReleaseOutput>true</multiReleaseOutput>
            </configuration>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <archive>
            <manifestEntries>
               <Multi-Release>true</Multi-Release>
            </manifestEntries>
        </archive>
    </configuration>
</plugin>
```

#### Toolchain

```java  
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-toolchains-plugin</artifactId>
    <version>3.0.0</version>
    <executions>
        <execution>
            <id>toolchain</id>
            <goals>
                <goal>toolchain</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <toolchains>
            <jdk>
                <version>9</version>
            </jdk>
            <jdk>
                <version>1.8</version>
            </jdk>
        </toolchains>
    </configuration>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <executions>
        <execution>
            <id>base-compile</id>
            <goals>
                <goal>compile</goal>
            </goals>
            <configuration>
                <jdkToolchain>
                    <version>1.8</version>
                </jdkToolchain>
                <source>1.8</source>
                <target>1.8</target>
                <excludes>
                    <exclude>module-info.java</exclude>
                </excludes>
            </configuration>
        </execution>
        <execution>
            <id>default-compile</id>
            <configuration>
                <jdkToolchain>
                    <version>9</version>
                </jdkToolchain>
                <release>9</release>
            </configuration>
        </execution>
    </executions>
    <configuration>
        <encoding>UTF-8</encoding>
    </configuration>
</plugin>
```

## 参考文档
* [migrate-to-java11](https://github.com/leonchen83/share/blob/master/java/migrate-to-java11.md)
* [modules](https://www.logicbig.com/tutorials/core-java-tutorial/modules.html)
* [understanding-java-9-modules](https://www.oracle.com/corporate/features/understanding-java-9-modules.html)
* [java-9-modules-cheat-sheet](https://www.jrebel.com/blog/java-9-modules-cheat-sheet)
* [maven-multi-release-jars](https://www.baeldung.com/maven-multi-release-jars)
* [maven multi-release](https://maven.apache.org/plugins/maven-compiler-plugin/multirelease.html)
