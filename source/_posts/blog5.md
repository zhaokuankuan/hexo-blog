---
title: Springcloud微服务架构之断路器(Hystrix)
comments: true
tags: [SpringCloud ,java]
categories: [微服务架构]
date: 2018-08-11 17:34:31
---
**在微服务架构中，根据业务来拆分成一个个的服务，服务与服务之间可以相互调用（RPC），在Spring Cloud可以用RestTemplate+Ribbon和Feign来调用，详细使用见[上一章](https://blog.csdn.net/zhaokk_git/article/details/80256356)。为了保证其高可用，单个服务通常会集群部署。由于网络原因或者自身的原因，服务并不能保证100%可用，如果单个服务出现问题，调用这个服务就会出现线程阻塞，此时若有大量的请求涌入，Servlet容器的线程资源会被消耗完毕，导致服务瘫痪。服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。为了将这种影响降到最低，提出了断路器的概念**
**一．	断路器介绍**
Netflix开源了Hystrix组件，实现了断路器模式，SpringCloud对这一组件进行了整合。
当较底层的服务出现故障时，会导致连锁故障。当一个服务的不可用达到一定的阈值断路器将会被打开。
![这里写图片描述](/blog5/20180510151201743-1.png)
断路打开后，可用避免连锁故障，fallback方法返回一个出现故障时的处理方法
**二．	接下来便分别对Ribbon+resttemplate和feign两种消费服务的方式分别使用断路器进行说明。**
在此应该首先启动上一节中的服务注册中心和需要注册的服务server和service。
Server的端口号是8761，server1和server2的端口号分别是8762和8763。
**三．	在Ribbon+resttemplate中使用断路器**
a)	需要先引入断路器的依赖包，如下

```
<!--断路器的依赖包-->
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-hystrix</artifactId>
   <version>1.3.1.RELEASE</version>
</dependency>
```

b)	在程序的启动类中开启断路器的注解


```
@SpringBootApplication
@EnableHystrix
public class RibbonApplication {

   public static void main(String[] args) {
      SpringApplication.run(RibbonApplication.class, args);
   }

   @Bean
   @LoadBalanced
   public RestTemplate restTemplate(){
      return  new RestTemplate();
   }
}
```

c)	修改调用服务的方法Service，如下：

```
@Service
public class service {

    @Autowired
    private RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod = "hasError" )
    public  String  sayHello(String name){
        return  this.restTemplate.getForObject("http://service-hi/hi?name=" + name,String.class);
    }

    public String hasError(String name){
        return  "I have a error, so You must restart me!" + name;
    }
}
```

d)	然后停掉第一步启动的两个service1和service2,再次访问**http://localhost:8080/hello?name=kk**，会出现如下图的界面
 ![这里写图片描述](/blog5/20180510145048845-2.png)
即，断路器增加成功！当你再次启动一个服务，如server1时，访问上述地址，会得到如下的图示：


**四．	在feign中使用断路器**：
Feign是自带断路器的，在D版本的Spring Cloud中，它没有默认打开。需要在配置文件中配置打开它，在配置文件加以下代码：
feign.hystrix.enabled=true
a)	这个只需要修改接口即可，在上节的基础上我们修改Iserver，首先需要创建一个类实现Iservice接口的方法，如下：

```
public class IserviceImpl implements Iservice {
    @Override
    public String SayHello(String name) {
        return  "I have a error, so You must restart me!" + name;
    }
}
```

b)	然后在修改Iservice中修该fallback指向刚才的实现类，当出现断路时会调用实现类中的方法。

```
@FeignClient(value = "service-hi",fallback = IserviceImpl.class)
@Service
public interface Iservice {

    @RequestMapping("/hi")
    String SayHello(@RequestParam(value = "name") String name);
}
```

c)	然后访问，出现我们负载均衡的调用service1和service2，如下图：
![这里写图片描述](/blog5/20180510150843425-3.png)

d)	然后我们停掉service2和service2，再次访问上述的路径，会出现如下图的显示，即说明断路器发挥了作用。
![这里写图片描述](/blog5/20180510150923308-4.png)

**五.  断路器仪表盘**
	这里ribbon和feign两中方式都可以进行，我这里以feign为例，ribbon的和这个一样。
a)	首先我们需要在pom中引入，该仪表盘的依赖，如下：
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
<version>1.4.1.RELEASE</version>
</dependency>
```

b)	然后在Application上加上注释，开启注解

```
@SpringBootApplication
@EnableFeignClients
@EnableHystrixDashboard
public class FeignApplication {

   public static void main(String[] args) {
      SpringApplication.run(FeignApplication.class, args);
   }
}
```

然后启动feign,访问该网址**http://localhost:8080/hystrix**，出现图示的界面，
 ![这里写图片描述](/blog5/20180510150940272-5.png)

附上我的完整的博文：
**[从零开始学习SpringCloud](https://blog.csdn.net/zhaokk_git/article/details/80228420)**
**[代码地址](https://github.com/zhaokuankuan/springcloud/tree/master/springcloud)**
**在此感谢，两位大佬的博客，我是根据以上大佬的博客学习的！**
[方志鹏的springcloud微服务架构](https://blog.csdn.net/forezp/article/details/70148833)
[纯洁的微笑](http://www.ityouknow.com/spring-cloud.html)








	
