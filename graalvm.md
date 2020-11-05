# GraalVM

## 1. 安装

```java  

$ wget https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-20.2.0/graalvm-ce-java11-linux-amd64-20.2.0.tar.gz
$ tar xz graalvm-ce-java11-linux-amd64-20.2.0.tar.gz
export JAVA_PATH=/path/to/graalvm-ce-java11-20.2.0
export PATH=$JAVA_PATH/bin:$PATH

$ java -version
openjdk version "11.0.8" 2020-07-14
OpenJDK Runtime Environment GraalVM CE 20.2.0 (build 11.0.8+10-jvmci-20.2-b03)
OpenJDK 64-Bit Server VM GraalVM CE 20.2.0 (build 11.0.8+10-jvmci-20.2-b03, mixed mode, sharing)
```

## 2. 简单使用

```java  

# 运行Java

public class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello, World!");
  }
}

$ javac HelloWorld.java
$ java HelloWorld
Hello World!

# 运行JS
$ js
> JSON.stringify({ x: 5, y: 6 })
{"x":5,"y":6}
> 

# 运行NodeJS
$ npm install colors ansispan

# 创建 app.js
const http = require("http");
const span = require("ansispan");
require("colors");

http.createServer(function (request, response) {
    response.writeHead(200, {"Content-Type": "text/html"});
    response.end(span("Hello Graal.js!".green));
}).listen(8000, function() { console.log("Graal.js server running at http://127.0.0.1:8000/".red); });

$ node app.js
Graal.js server running at http://127.0.0.1:8000/

# 浏览器打开 http://192.168.1.201:8000/

```

## 3. Graal安装器

```
gu install ruby
gu install python
gu install native-image
```

## 4. 多语言

```
# js 调用 R
# Create a file polyglot.js:
var array = Polyglot.eval("R", "c(1,2,42,4)")
console.log(array[2]);

$ js --polyglot --jvm polyglot.js
42

# R 调用 js
array <- eval.polyglot("js", "[1,2,42,4]")
print(array[3L])

$ Rscript --polyglot --jvm polyglot.R
[1] 42

# python 调用 js

import polyglot
array = polyglot.eval(language="js", string="[1,2,42,4]")
print(array[2])

$ graalpython --polyglot --jvm polyglot.py
42

# 多语言shell
polyglot --jvm --shell

```

## 5. 编译成native image

### 前提条件

```
# 安装开发者套件
$ sudo yum -y groupinstall "Development tools"
$ sudo yum -y install zlib-devel

# 安装graal native image插件
$ gu install native-image
```

### 示例

```
$ native-image HelloWorld
$ ./helloworld
Hello, World!

# 多语言编译成native image示例

import java.io.*;
import java.util.stream.*;
import org.graalvm.polyglot.*;

public class PrettyPrintJSON {
  public static void main(String[] args) throws java.io.IOException {
    String json = "{\"GraalVM\":{\"description\":\"Language Abstraction Platform\",\"supports\":[\"combining languages\",\"embedding languages\",\"creating native images\"],\"languages\": [\"Java\",\"JavaScript\",\"Node.js\", \"Python\", \"Ruby\",\"R\",\"LLVM\"]}}";
    BufferedReader reader = new BufferedReader(new StringReader(json));
    String input = reader.lines().collect(Collectors.joining(System.lineSeparator()));
    try (Context context = Context.create("js")) {
      Value parse = context.eval("js", "JSON.parse");
      Value stringify = context.eval("js", "JSON.stringify");
      Value result = stringify.execute(parse.execute(input), null, 2);
      System.out.println(result.asString());
    }
  }
}

$ javac PrettyPrintJSON.java
$ native-image --language:js --initialize-at-build-time PrettyPrintJSON
[13:56:53 chenby@thorin-dev test]$ native-image --language:js --initialize-at-build-time PrettyPrintJSON
[prettyprintjson:11109]    classlist:  15,950.80 ms,  0.96 GB
[prettyprintjson:11109]        (cap):   5,227.13 ms,  0.96 GB
[prettyprintjson:11109]        setup:  19,694.98 ms,  0.96 GB
[prettyprintjson:11109]     (clinit):   5,545.74 ms,  4.57 GB
[prettyprintjson:11109]   (typeflow): 194,238.24 ms,  4.57 GB
[prettyprintjson:11109]    (objects): 154,005.80 ms,  4.57 GB
[prettyprintjson:11109]   (features):  71,607.54 ms,  4.57 GB
[prettyprintjson:11109]     analysis: 442,223.36 ms,  4.57 GB
[prettyprintjson:11109]     universe:  16,337.57 ms,  4.60 GB
9713 method(s) included for runtime compilation
[prettyprintjson:11109]      (parse):  61,944.98 ms,  4.60 GB
[prettyprintjson:11109]     (inline):  70,668.88 ms,  4.99 GB
[prettyprintjson:11109]    (compile): 258,455.04 ms,  5.77 GB
[prettyprintjson:11109]      compile: 419,052.84 ms,  5.77 GB
[prettyprintjson:11109]        image:  64,615.34 ms,  5.71 GB
[prettyprintjson:11109]        write:  25,121.60 ms,  5.71 GB
[prettyprintjson:11109]      [total]: 1,022,098.93 ms,  5.71 GB
[14:14:04 chenby@thorin-dev test]$ 

$ ./prettyprintjson

```

### 运行时间比较
```java  

time java PrettyPrintJSON > /dev/null

real    0m8.329s
user    0m4.627s
sys     0m1.086s

time ./prettyprintjson > /dev/null

real    0m0.058s
user    0m0.009s
sys     0m0.019s

```

### 处理反射, 动态代理, native方法调用以及资源加载

* 如果代码包含反射, 动态代理, native方法调用以及资源加载. 那么需要配置如下配置文件

```java  


META-INF/
└── native-image
    └── groupID
        └── artifactID
            └── native-image.properties
            └── jni-config.json
            └── proxy-config.json
            └── reflect-config.json
            └── resource-config.json

一个比较简单的方法是通过java agent自动拦截动态方法调用，自动生成配置文件
$mkdir -p META-INF/native-image
java -agentlib:native-image-agent=config-output-dir=META-INF/native-image class...
```

* 示例程序

```java  

import java.io.*;
import java.util.*;
import java.lang.reflect.*;

class StringReverser {
    static String reverse(String input) {
        return new StringBuilder(input).reverse().toString();
    }
}

class StringCapitalizer {
    static String capitalize(String input) {
        return input.toUpperCase();
    }
}

interface XProxy { String invoke(); }

public class Example {
    public static void main(String[] args) throws Exception {
        // reflection example
        String className = args[0];
        String methodName = args[1];
        String input = args[2];
        
        Class<?> clazz = Class.forName(className);
        Method method = clazz.getDeclaredMethod(methodName, String.class);
        Object result = method.invoke(null, input);
        System.out.println(result);
        
        // proxy example
        XProxy x = (XProxy) Proxy.newProxyInstance(Example.class.getClassLoader(), new Class[]{XProxy.class}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                return "proxy";
            }
        });
        x.invoke();
        
        // resource example
        Properties properties = new Properties();
        try(InputStream in = Example.class.getClassLoader().getResourceAsStream("test.properties")) {
            properties.load(in);
        } catch (IOException e) {}
    }
}

执行如下命令
$ java -agentlib:native-image-agent=config-output-dir=META-INF/native-image Example StringReverser reverse "hello"
$ java -agentlib:native-image-agent=config-merge-dir=META-INF/native-image Example StringCapitalizer capitalize "hello"

```

* 得到如下配置文件

```
$ cat jni-config.json 
[
{
  "name":"java.lang.ClassLoader",
  "methods":[
    {"name":"getPlatformClassLoader","parameterTypes":[] }, 
    {"name":"loadClass","parameterTypes":["java.lang.String"] }
  ]
},
{
  "name":"java.lang.ClassNotFoundException"
},
{
  "name":"java.lang.NoSuchMethodError"
},
{
  "name":"java.lang.String"
}
]

$ cat proxy-config.json 
[
  ["XProxy"]
]

$ cat reflect-config.json 
[
{
  "name":"StringCapitalizer",
  "methods":[{"name":"capitalize","parameterTypes":["java.lang.String"] }]
},
{
  "name":"StringReverser",
  "methods":[{"name":"reverse","parameterTypes":["java.lang.String"] }]
}
]

$ cat resource-config.json 
{
  "resources":[{"pattern":"\\Qtest.properties\\E"}],
  "bundles":[]
}
```

* 执行 native-image Example 生成example可执行文件

```java  

./example StringReverser reverse "hello"
./example StringCapitalizer capitalize "hello"
```

### 系统属性

```
public class App {
    public static void main(String[] args) {
        System.getProperties().list(System.out);
    }
}
# 编译时可用
native-image -Dfoo=bar App

# 运行时可用
app -Dfoo=bar
```

### 构建无任何依赖的static native image

[StaticImages](https://www.graalvm.org/reference-manual/native-image/StaticImages/)
```
step1: build musl-1.2.0
step2: build zlib-1.2.11
step3: cp /usr/lib/gcc/x86_64-redhat-linux/4.4.7/libstdc++.a /path/to/musl/lib
step4: native-image --static --libc=musl ...
```

### native-image的限制

1. Unsafe使用
```
Unsafe.objectFieldOffset()
需要在reflect-config.json配置
"fields" : [ { "name" : "hash", "allowUnsafeAccess" : true }, ... ]
```
2. JCA (Java Cryptography Architecture) 
```
开启JCA需要加如如下option --enable-all-security-services
```
3. class初始化可以添加参数`--initialize-at-build-time`在编译时初始化
4. `invokedynamic`以及`cglib`等字节码工具不支持，`finalize`方法以及`Thread.stop`等方法不支持

# 6. References

* [native-image](https://www.graalvm.org/reference-manual/native-image/)
* [使用 GraalVM 开发多语言应用](https://developer.ibm.com/zh/technologies/java/articles/j-use-graalvm-to-run-polyglot-apps/)
* [Cannot build native image for trivial app with SLF4J and Log4J2](https://github.com/oracle/graal/issues/2008)

