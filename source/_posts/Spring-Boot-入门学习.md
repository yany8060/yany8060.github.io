---
title: Spring-Boot 入门学习
date: 2017-01-25 14:03:40
tags: [spring-boot,spring]
categories: [java]
---
> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run". We take an opinionated view of the Spring platform and third-party libraries so you can get started with minimum fuss. Most Spring Boot applications need very little Spring configuration.
> 简言之：方便创建一个最小规模的Spring工程，且它只需要很少的Spring配置

### 特征
* Create stand-alone Spring applications
* Embed（内置、嵌入） Tomcat, Jetty or Undertow directly (no need to deploy WAR files)
* Provide opinionated 'starter' POMs to simplify your Maven configuration
* Automatically configure Spring whenever possible
* Provide production-ready features such as metrics, health checks and externalized configuration（提供产品化的功能）
* Absolutely no code generation and no requirement for XML configuration


### Spring-Boot工程简单搭建
> Spring-boot文档：http://docs.spring.io/spring-boot/docs/current/reference/html/
> 具体代码详见：https://github.com/yany8060/SpringDemo

#### pom.xml详情（本工程以maven方式搭建）：
官方建议引入父工程
```
<properties> 
	<java.version>1.7</java.version>
	<spring.boot.version>1.4.3.RELEASE</spring.boot.version>
</properties>


<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>${spring.boot.version}</version>
	<type>pom</type>
	<scope>import</scope>
</dependency>
```

在子工程中引入自己所需要的依赖（版本号有父工程管理）：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>

```
并添加maven编译插件：
```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-compiler-plugin</artifactId>
</plugin>
```
更多SpringBoot Maven插件详见：http://docs.spring.io/spring-boot/docs/1.4.3.RELEASE/maven-plugin/index.html

#### 创建一个启动入口（应用类）
我们创建一个Application类：
```java
@EnableAutoConfiguration
@ComponentScan(basePackages = "com.yany")
@Configuration
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }

}
```
`@EnableAutoConfiguration`
开启自动配置
这个注解告诉Spring Boot根据添加的jar依赖猜测你想如何配置Spring
`@ComponentScan`
Spring原注解，扫描包加载bean
`@Configuration`
标注一个类为配置类

我们再创建一个Controller层：
```java
@RestController
public class Example {

    @RequestMapping(value = "/hello", method = {RequestMethod.GET})
    public String test() {
        return "Hello world";
    }

}
```
`@RestController`
相当于同时添加@Controller和@ResponseBody注解。

这时一个基本的Spring-Boot的工程就创建完了，里面包含一个了Mapping映射，可通过访问：http://localhost:8080/SpringBoot/hello 来得到返回值

#### 启动
以main方法方式启动Application类，

在浏览器中输入：http://localhost:8080/SpringBoot/hello
得到返回：Hello world
![](/img/work/14853239880359.jpg)


具体输入日志如下：
```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.4.3.RELEASE)

2017-01-25 13:55:36.298  INFO 30221 --- [           main] com.yany.Application                     : Starting Application on yanyongdeMacBook-Pro.local with PID 30221 (/Users/yanyong/GitHub/YanY/SpringDemo/SpringBoot/target/classes started by yanyong in /Users/yanyong/GitHub/YanY/SpringDemo)
2017-01-25 13:55:36.301  INFO 30221 --- [           main] com.yany.Application                     : No active profile set, falling back to default profiles: default
2017-01-25 13:55:36.373  INFO 30221 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@2d8b20a0: startup date [Wed Jan 25 13:55:36 CST 2017]; root of context hierarchy
2017-01-25 13:55:37.264  WARN 30221 --- [           main] o.m.s.mapper.ClassPathMapperScanner      : No MyBatis mapper was found in '[com.yany]' package. Please check your configuration.
2017-01-25 13:55:37.570  INFO 30221 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration' of type [class org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration$$EnhancerBySpringCGLIB$$baf0e866] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2017-01-25 13:55:37.997  INFO 30221 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
2017-01-25 13:55:38.009  INFO 30221 --- [           main] o.apache.catalina.core.StandardService   : Starting service Tomcat
2017-01-25 13:55:38.010  INFO 30221 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.6
2017-01-25 13:55:38.136  INFO 30221 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2017-01-25 13:55:38.136  INFO 30221 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1767 ms
2017-01-25 13:55:38.298  INFO 30221 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Mapping servlet: 'dispatcherServlet' to [/]
2017-01-25 13:55:38.302  INFO 30221 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2017-01-25 13:55:38.302  INFO 30221 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2017-01-25 13:55:38.303  INFO 30221 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2017-01-25 13:55:38.303  INFO 30221 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2017-01-25 13:55:38.613  INFO 30221 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@2d8b20a0: startup date [Wed Jan 25 13:55:36 CST 2017]; root of context hierarchy
2017-01-25 13:55:38.674  INFO 30221 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/hello],methods=[GET]}" onto public java.lang.String com.yany.controller.Example.test()
2017-01-25 13:55:38.678  INFO 30221 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2017-01-25 13:55:38.678  INFO 30221 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2017-01-25 13:55:38.712  INFO 30221 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-01-25 13:55:38.712  INFO 30221 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-01-25 13:55:38.752  INFO 30221 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-01-25 13:55:39.333  INFO 30221 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2017-01-25 13:55:39.385  INFO 30221 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-01-25 13:55:39.388  INFO 30221 --- [           main] com.yany.Application                     : Started Application in 3.752 seconds (JVM running for 4.078)

```


