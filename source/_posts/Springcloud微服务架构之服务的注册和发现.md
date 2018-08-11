---
title: Springcloud微服务架构之服务的注册和发现
comments: true
tags: [SpringCloud ,java]
categories: [微服务架构]
date: 2018-08-11 17:34:31
---

**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;微服务可以在"自己的程序"中运行，并通过"轻量级设备与HTTP型API进行沟通"。关键在于该服务可以在自己的程序中运行。通过这一点我们就可以将服务公开与微服务架构(在现有系统中分布一个API)区分开来。在服务公开中，许多服务都可以被内部独立进程所限制。如果其中任何一个服务需要增加某种功能，那么就必须缩小进程范围。在微服务架构中，只需要在特定的某种服务中增加所需功能，而不影响整体进程。**
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上解释来自百度，综上可以看出对于微服务架构来说，服务的注册和服务的发现就很关键了，由于本人接触Dubbo和zookeeper较少因此谈谈Springcloud的注册和发现把。
**一.新建一个空的maven工程（任何东西都不选)如图**
![这里写图片描述](https://img-blog.csdn.net/20180507173152623?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
	Next： 然后填写 GroupId 和 ArtifactId 分别写入的是 包的层次结构基本上是公司的域名或者项目的域名。例 com.kk 或者  com.baidu等
**二.在新建好的该maven项目中新建两个module分别为server和service作为服务的注册中心和服务的提供方(创建方法类似)。**
![这里写图片描述](https://img-blog.csdn.net/2018050717322387?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180507173256574?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180507173310227?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180507173322191?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
注册中心选择server  , 服务提供者选择discovery
**三.Server的配置，创建一个application.yml文件**
然后配置yml

```
server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false #表示是否将自己注册到Eureka Server
    fetch-registry: false #表示是否从Eureka Server获取注册信息
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/   #注册中心的地址

```

然后启动server的这个项目，访问http://localhost:8761/   你就看到如下的界面
![这里写图片描述](https://img-blog.csdn.net/2018050717333577?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
说明注册中心已经配置完成，只是里面没有注册服务。
Ps：在这里说一下，因为我们在做分布式部署的时候为了保证项目的健壮性我们通常会采用集群部署，因为可以参考微笑的博客来设置注册中心的集群。
**四.接下来我们应该向注册中心注册服务了，首先创建application.yml文件**

```
在ServiceApplication类中开启服务发现的注解
@SpringBootApplication
@EnableEurekaClient
public class ServiceApplication {

   public static void main(String[] args) {
      SpringApplication.run(ServiceApplication.class, args);
   }
}

```
然后配置yml

```
`server:
  port: 8762 #设置服务的端口号
client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
spring:
  application:
    name: service-A
```

`
然后再次访问注册中心，你就发现我们的服务已经注册进去了。
![这里写图片描述](https://img-blog.csdn.net/20180507173351441?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
**到此，springcloud的服务的注册和发现已经完成。**
在此感谢，两位大佬的博客，我是根据以上大佬的博客学习的！
[方志鹏的springcloud微服务架构](https://blog.csdn.net/forezp/article/details/70148833)
[纯洁的微笑](http://www.ityouknow.com/spring-cloud.html)
