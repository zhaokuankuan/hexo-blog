---
title: Springcloud微服务架构之路由网关(zuul)
comments: true
tags: [SpringCloud ,java]
categories: [微服务架构]
date: 2018-08-11 17:34:31
---
**Springcloud微服务架构之路由网关(zuul)**

**Zuul的主要功能是路由转发和过滤器:**
    **1.路由功能是微服务的一部分**，比如将 API-A转发到service-hi服务,zuul默认和Ribbon结合        实现了负载均衡的功能。
    **2.zuul不仅只是路由，并且还能过滤，做一些安全验证**。
    


----------


**一．	接下来我们分别看一下，Zuul的路由转发和过滤器**
		首先还是在上一节的[服务的注册和发现](https://blog.csdn.net/zhaokk_git/article/details/80228880)的基础上进行的，我们需要先启动Server(服务的注册中心):8761，然后启动我们的一个服务Service(需要注册的服务):8762。
**二．	路由转发：路由功能是微服务的一部分，比如将API-A转发到service-hi**
**a)	创建一个module项目rest-client**
 ![这里写图片描述](https://img-blog.csdn.net/2018052415363735?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
**b)	然后新建一个appliacation.yml,**配置内容如下，我们将这个服务注册进eureka注册中心中，然后设置zuul的路由，当访问API-A 的时候我们将请求的理由到service-HI服务中去。

```
server:
  port: 8088
eureka:
  client:
    service-url:
     defaultZone: http://localhost:8761/eureka
spring:
  application:
    name: service-zuul
zuul:
  routes:
    API-A:
      path: /API-A/**
      serviceId: SERVICE-HI
```

**c)   然后我们修改RestClientApplication 这个类，增加zuul开启路由的注解**

```
@SpringBootApplication
@EnableZuulProxy
@EnableEurekaClient
public class RestClientApplication {
   public static void main(String[] args) {
      SpringApplication.run(RestClientApplication.class, args);
   }
}
```
	
**d)	分别访问http://localhost:8761/你会看到如下的图示**，有我们注入的zuul和需要路由的serice-hi服务。
 ![这里写图片描述](https://img-blog.csdn.net/2018052415365712?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在访问**http://localhost:8088/API-A/hello?name=kk**，你会看到，我们的如图所示，即我们的路由已经成功的将api-a转发到了serive-hi的服务，并且输出了service-hi的端口，具体的代码可以看**文章的结尾**。
 
![这里写图片描述](https://img-blog.csdn.net/20180524153705952?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
**三   过滤功能：zuul不仅可以做为路由使用，还可以用作过滤来做一些逻辑操作**
**a)	在上面的基础上进行改造，创建一个类MyFilter具体的说明已经注释出来了。**

```
/**
 * @author :zhaokk
 * @date: 2018/5/11 - 14:58
 */
@Component
public class MyFilter extends ZuulFilter {

    private static Logger logger = LoggerFactory.getLogger(ZuulFilter.class);

    @Override
    public String filterType() {
        return "pre";
        //pre：路由之前
        //routing：路由之时
        //post： 路由之后
        //error：发送错误调用
    }

    @Override
    public int filterOrder() {
        return 0;   //优先级  0 最高
    }

    @Override
    public boolean shouldFilter() {
        return true; //true为永远过滤
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest httpServletRequest = requestContext.getRequest();
        Object o = httpServletRequest.getParameter("token");
        if(o == null){
            logger.warn("token is empty");
            requestContext.setSendZuulResponse(false);
            requestContext.setResponseStatusCode(401);
            try {
                requestContext.getResponse().getWriter().write("token is empty!");
            }catch (Throwable throwable){
                return  null;
            }
        }
        logger.info("ok");
        return null;
    }
}
```

**b)	然后我们继续访问http://localhost:8088/API-A/hello?name=kk**，这时你会看到，界面如图，因为我们的代码中获取不到token，所以被拦截了，
![这里写图片描述](https://img-blog.csdn.net/20180524153718401?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 **c） 当我们输入http://localhost:8088/API-A/hello?name=kk&token=22**，会显示 如下的界面，即我们的过滤功能有效了！
![这里写图片描述](https://img-blog.csdn.net/20180524153724592?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 
附上我的完整的博文： 
**[从零开始学习SpringCloud](https://blog.csdn.net/zhaokk_git/article/details/80228420)** 
**[代码地址](https://github.com/zhaokuankuan/springcloud/tree/master/SpringCloudZuul)** 
**在此感谢，两位大佬的博客，我是根据以上大佬的博客学习的！**
 [方志鹏的springcloud微服务架构](https://blog.csdn.net/forezp/article/details/70148833) 
 [纯洁的微笑](http://www.ityouknow.com/spring-cloud.html)

