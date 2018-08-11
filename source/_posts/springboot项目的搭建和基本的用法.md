---
title: springboot项目的搭建和基本的用法
comments: true
tags: [Springboot ,java]
categories: [微服务架构]
date: 2018-08-11 17:34:31
---
初涉springboot，学习小记，用于学习，简单介绍下如何去创建一个简单的的springboot工程。本人这里使用idea操作的。
**一.首先需要创建一个工程：new project,选择Spring Initalizr。**
![这里写图片描述](https://img-blog.csdn.net/20180509172029526?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
输入项目的目录结构和项目的名称，如图：
![这里写图片描述](https://img-blog.csdn.net/20180509172333830?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
然后选择你需要导入的jar包，如下：
![这里写图片描述](https://img-blog.csdn.net/20180509172423241?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
web是web项目的核心启动包，devtools是一个web项目的工具包。

**二.然后你的一个springboot项目便创建完成。**
如图我们可以创建这个项目的目录结构，但是需要注意一点的是，包的结构必须和TestApplication是在一级上。
 ![这里写图片描述](https://img-blog.csdn.net/20180509172740232?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在controller里面创建一个类，HelloController

```
@RestController
public class HelloController {

    @RequestMapping(value = "/hello")
    public  String sayHello(){
        return  "Hello , my First demo!";
    }

}

```
**然后我们启动这个工程，然后访问一下！**
![这里写图片描述](https://img-blog.csdn.net/20180509173207746?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ra19naXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
如上图所示我们的项目就已经构建完成了，然后具体的项目的热部署和项目的过滤器和测试等都在我下面的笔记中有，也可以看我的github，查看我的代码！

**笔记**：
> **1.springboot的创建：** 	idea的情况下，选择Spring Initializr 然后配置 web 和 devtools 一个是启动的web的核心依赖一个是开发工具包
> **2.创建热部署** 	idea下，pom中修改 devtools的optional为true 则为热部署开启 	plugin 下增加configuration --》fork 为true 	接下来在build
> 里面选择自动构建；ctrl+shift+a：输入register
> 窜则compile.automake.allow.when.app.running
> **3.单元测试** 	先创建mockmvc，静态的加载：ockMvcBuilders.standaloneSetup(new HelloWorldController()).build();
> 	mockmvc.perform()调用mockmvcRequestBuilders.post/get方法，.parm()加参数--》键值对类型,anddo（print()）可以打印
> **4json的支持** 	在 Spring Boot 体系中，天然对 Json 支持，@restController
> **5.请求传参** 	例如@RequestMapping(value="get/{name}", method=RequestMethod.GET) 	public User get(@PathVariable String name)
> { 		this.name = name; 	}
> **6.参数校验** 	实体类上可以加@NotEmpty，@Max，@Min，@Length来设置属性的校验 	Spring Boot 的参数校验其实是依赖于 hibernate-validator 来进行 	public void saveUser(@Valid User
> user,BindingResult result){ 		List<ObjectError> list =
> result.getAllErrors(); //能够得到所有的校验的结果 	}
> **7.自定义过滤** 	自定义的过滤必须实现Filter接口并实现方法，在doFilter里面写上自己的过滤处理 	@Configuration说明是个配置类，用来存放配置信息， 	@bean 说明该方法作为一个配置信息被spring监控， 
> 	public FilterRegistrationBean testFilterRegistration() {
> 	   FilterRegistrationBean registration = new FilterRegistrationBean();
> 	   registration.setFilter(new MyFilter());
> 	   registration.addUrlPatterns("/*");
> 	   registration.addInitParameter("paramName", "paramValue");
> 	   registration.setName("MyFilter");
> 	   registration.setOrder(1);
> 	   return registration;
>     }

[代码地址](https://github.com/zhaokuankuan/springboot-basic)
