# spring boot 简介
SpringBoot是由Pivotal团队在2013年开始研发, 2014年4月发布第一个版本的全新开源的轻量级框架。它基于Spring4.0设计，不仅继承了Spring框架原有的优秀特性，而且还通过简化配置来进一步简化了Spring应用的整个搭建和开发过程。另外SpringBoot通过集成大量的框架使得依赖包的版本冲突，以及引用的不稳定性等问题得到了很好的解决。

# 依赖

```
Java 8+
Spring Framework 5.2.10.RELEASE+
Maven 3.3+
```

* Servlet 容器

| 名称 | Servlet版本 |
| ---- | ---- |
| Tomcat 9.0 | 4.0 |
| Jetty 9.4 | 3.1 |
| Undertow 2.0 | 4.0 |

# 简单使用

1. 创建maven工程

```
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.3.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
            <version>2.3.5.RELEASE</version>
        </dependency>
    </dependencies>
```

2. 创建`Main.java`, `TestController.java`以及`application.properties`

```
├───src
│   ├───main
│   │   ├───java
│   │   │   └───com
│   │   │       └───demo
│   │   │           │───rest
│   │   │           │   └───TestController.java
│   │   │           └───Main.java
│   │   └───resources
│           └───application.properties
```

3. Main.java

```java  
@SpringBootApplication
public class Main {
	public static void main(String[] args) {
		SpringApplication.run(Main.class, args);
	}
}

```

4. TestController.java

```java  
@RestController
public class TestController {
	
	@RequestMapping(value = "/test", method = GET, produces = APPLICATION_JSON_VALUE)
	public Map<String, String> test() {
		Map<String, String> map = new HashMap<>();
		map.put("response", "hello world");
		return map;
	}
}
```

5. 运行Main.java

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.4.RELEASE)

2020-11-05 16:10:15.349  INFO 17492 --- [           main] com.demo.Main                            : Starting Main on MI04-chenby with PID 17492 (C:\workspace\spring-boot-demo\target\classes started by chenby in C:\workspace\spring-boot-demo)
2020-11-05 16:10:15.351  INFO 17492 --- [           main] com.demo.Main                            : No active profile set, falling back to default profiles: default
2020-11-05 16:10:16.100  INFO 17492 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-11-05 16:10:16.107  INFO 17492 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-11-05 16:10:16.107  INFO 17492 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.38]
2020-11-05 16:10:16.166  INFO 17492 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-11-05 16:10:16.167  INFO 17492 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 760 ms
2020-11-05 16:10:16.289  INFO 17492 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-11-05 16:10:16.413  INFO 17492 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-11-05 16:10:16.421  INFO 17492 --- [           main] com.demo.Main                            : Started Main in 1.352 seconds (JVM running for 2.098)
2020-11-05 16:10:21.791  INFO 17492 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2020-11-05 16:10:21.792  INFO 17492 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2020-11-05 16:10:21.795  INFO 17492 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 3 ms
``` 

6. 浏览器打开`http://localhost:8080/test`

```
{"response":"hello world"}
```

# 自动配置

1. @SpringBootApplication注解

```
1. @SpringBootConfiguration
2. @EnableAutoConfiguration
3. @ComponentScan
```

2. @EnableAutoConfiguration

```
├───src
│   ├───main
│   │   ├───java
│   │   └───resources
│           └───META-INF
│               └───spring.factories

org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.package.MyAutoConfiguration,\
                                                               ....,\
```

3. MyAutoConfiguration 和 MyBean
```
@Configuration(proxyBeanMethods = false)
public class MyAutoConfiguration {
	@Bean
	public MyBean myBean() {
		return new MyBean();
	}
}

public class MyBean {
	public String hello() {
		return "hello world";
	}
}
```

# 优雅关闭

```
public class Main {
	private static final Logger LOGGER = LoggerFactory.getLogger(Main.class);
	
	@PreDestroy
	public void shutdown() {
		LOGGER.info("shutdown");
	}
	
	public static void main(String[] args) {
		SpringApplicationBuilder builder = new SpringApplicationBuilder();
		builder.sources(Main.class).registerShutdownHook(true).run(args);
	}
}
```

# application.properties

```

```

# 非web的spring项目改造成spring-boot

```
@SpringBootConfiguration
@ImportResource({"classpath:your-spring-file.xml"})
public class Main {
	public static void main(String[] args) {
		SpringApplicationBuilder builder = new SpringApplicationBuilder();
		builder.sources(Main.class).registerShutdownHook(true).web(WebApplicationType.NONE).run(args);
	}
}
```

# 既有的spring mvc项目升级到spring-boot

```
@SpringBootConfiguration
@ImportResource({"classpath:your-spring-webmvc-file.xml"})
public class Main {
	public static void main(String[] args) {
		SpringApplicationBuilder builder = new SpringApplicationBuilder();
		builder.sources(Main.class).registerShutdownHook(true).web(WebApplicationType.SERVLET).run(args);
	}
}
```

# spring boot 条件注解

# spring boot 使用原则

# References

* [spring boot reference](https://docs.spring.io/spring-boot/docs/current/reference/html/)
* [configure jetty server](https://howtodoinjava.com/spring-boot/configure-jetty-server/)
* [spring app migration from xml to java based config](https://www.robinhowlett.com/blog/2013/02/13/spring-app-migration-from-xml-to-java-based-config/)
