---
title: Springboot的用法之整合Swagger
comments: true
tags: [Springboot,Swagger]
categories: JavaWeb
date: 2018-08-16 12:31:59
---




百度百科：Swagger的目标是为REST API 定义一个标准的，与语言无关的接口，使人和计算机在看不到源码或者看不到文档或者不能通过网络流量检测的情况下能发现和理解各种服务的功能。当服务通过Swagger定义，消费者就能与远程的服务互动通过少量的实现逻辑。类似于低级编程接口，Swagger去掉了调用服务时的很多猜测。

这个是百度百科的介绍，我自己的理解其实Swagger就是一个Restful接口的在线生成工具，可以动态的根据注解新增和修改展示的内容，解放了人力去维护和编写接口文档的工作量。解决了在接口修改的时候需要重新修改接口文档的问题。


----------


一.在[上一节](https://blog.csdn.net/zhaokk_git/article/details/79608197)的基础上引入Swagger的Jar包，以下是我的POM配置文件
	

``` stylus
<!--swagger依赖文件-->
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger2</artifactId>
			<version>2.6.1</version>
		</dependency>
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger-ui</artifactId>
			<version>2.6.1</version>
		</dependency>
```


----------


二.配置Swagger的配置文件 ==>Swagger.java

``` stylus
package com.kk.Springbootmanger.configure;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * @author :Mr.kk
 * @date: 2018/8/14-17:07
 * springboot整合Swagger的入口
 */
@Configuration
@EnableSwagger2
public class Swagger {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.kk.Springbootmanger.controller"))
                .paths(PathSelectors.any())
                .build();
    }
    //配置在线文档的基本信息
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("springboot利用swagger构建api文档")
                .description("简单优雅的restfun风格，http://blog.csdn.net/zhaokk_git")
                .termsOfServiceUrl("http://blog.csdn.net/zhaokk_git")
                .version("1.0")
                .build();
    }
}

```
		到此，Springboot整合Swagger的配置就已经全部完成，惊喜不惊喜，意外不意外!


----------


三.接下来就是我们的使用了，分为两部分来介绍

1. 代码中需要的配置

 我抛一下我的代码，一起来看一下常用的一些东西==》DemoController.java

``` stylus
package com.kk.Springbootmanger.controller;

import com.kk.Springbootmanger.dao.UserDao;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
/**
 * @author :zhaokk
 * @date: 2018/8/13 - 16:27
 */
@RestController
@Api(value = "测试的demo",tags = {"搭建环境的测试代码"})
public class DemoController {

    @Autowired
    private UserDao userDao;

    @ApiOperation(value = "测试restful",notes = "测试工程")
    @ApiImplicitParam(value = "姓名",name="name")
    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    public String sayHello(String name){
        return name;
    }

    @ApiOperation(value = "测试数据库连通性",notes = "测试数据库连通性")
    @RequestMapping(value = "/getAll",method = {RequestMethod.GET,RequestMethod.POST})
    public Object getAll(){
        return userDao.getAll();
    }
}

```

   这个是官方的Wiki上的文档
      	![这里写图片描述](/blog14/s1.png)
       这个中文的翻译
 

``` undefined
常用注解： 
- @Api()用于类； 
表示标识这个类是swagger的资源 ，即类的注释
- @ApiOperation()用于方法； 
表示一个http请求的操作 ，即方法的注释
- @ApiParam()用于方法，参数，字段说明； 
表示对参数的添加元数据（说明或是否必填等） ，即参数的注释
- @ApiModel()用于类 
表示对类进行说明，用于参数用实体类接收 ，即返回实体的注释
- @ApiModelProperty()用于方法，字段 
表示对model属性的说明或者数据操作更改 ，即对返回的实体的属性的注释
- @ApiIgnore()用于类，方法，方法参数 
表示这个方法或者类被忽略 ，即忽略此方法不现实
- @ApiImplicitParam() 用于方法 
表示单独的请求参数 ，即单个参数的注释
- @ApiImplicitParams() 用于方法，包含多个 @ApiImplicitParam 
```


 2 界面的使用
   启动工程，然后访问http://localhost:8081/manager/swagger-ui.html则出现如图所示，即

![这里写图片描述](/blog14/s2.png)
	
    具体看下以下几个常用的注解：
    	

 - @Api(value = "测试的demo",tags = {"搭建环境的测试代码"})
 ![这里写图片描述](/blog14/s3.png)
 ----------
 - @ApiOperation(value = "测试数据库连通性",notes = "测试数据库连通性")
 ![这里写图片描述](/blog14/s4.png)
 ----------
 - @ApiImplicitParam(value = "姓名",name="name")
 ![这里写图片描述](/blog14/s5.png)
----------
    同时，你还可以在上图所示的位置输入参数，
    点击Try it out ! 进行接口的测试


----------


  到此，Springboot整合Swagger就已经全部完成了！代码已经上传到[Github][7]上。


  [1]: http://www.struggling-bird.cn
  [7]: https://github.com/zhaokuankuan/Springboot-manger