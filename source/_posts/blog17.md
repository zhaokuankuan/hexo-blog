---
title: maven多模块应用的搭建和dubbo的整合
comments: true
tags: [Springboot,Dubbo]
categories: 微服务架构
date: 2019-03-03 14:31:00
---



**准备环境**：
idea
jdk1.8
dubbo
zookeeper
springboot2.0

**一.先新建一个空的父工程 dubbo-application**
(1).创建一个空的Maven工程作为父Maven工程
(2).删除掉父工程里面的src文件夹 只保留pom,如图:
![在这里插入图片描述](/blog17/1.png)

**二.依此创建对应的maven的子模块，如图所示**
(1).选中父工程，右键选择new一个Module
![在这里插入图片描述](/blog17/2.png)


(2).直接点击next默认创建出空的Maven子模块
![在这里插入图片描述](/blog17/3.png)

(3).然后填写子模块的信息,如图:
![在这里插入图片描述](/blog17/4.png)
然后点击next，完善Module的信息，然后点击Finish。
![在这里插入图片描述](/blog17/5.png)
(4).如图便创建好了controller子模块，接下来我们一次创建别的子模块和子模块下的包结构：

```stylus
controller    工程的接口层                  com.kk.controller        
service       工程的服务层                  com.kk.service
api           外部暴露的接口                com.kk.api
rpc           注册到zk中提供服务的工程       com.kk.rpc
dao           dao层，即数据库的连接层        com.kk.dao
common        公共的组建和文件配置           com.kk.common
domain        实体类所在的层                com.kk.domain
```


创建完成之后如图所示：
![在这里插入图片描述](/blog17/6.png)
到此，项目的目录的创建就完成了。
**(三).接下来进行工程的配置**
因为是springboot整合dubbo进行分布式调用，这里就需要将rpc和controller分别作为服务的提供者和服务的调用者发布出去。
(1).首先,在父工程的pom包中引入项目中需要用到的依赖包，代码如下：

```stylus
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging>
    <groupId>com.kk</groupId>
    <artifactId>Dubbo-Application</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>dubbo-application-controller</module>
        <module>dubbo-application-service</module>
        <module>dubbo-application-api</module>
        <module>dubbo-application-common</module>
        <module>dubbo-application-domain</module>
        <module>dubbo-application-dao</module>
        <module>dubbo-application-rpc</module>
    </modules>
    <properties>
        <springBoot.version>2.0.4.RELEASE</springBoot.version>
        <lombok.version>1.16.22</lombok.version>
        <spring.mybatis>1.3.1</spring.mybatis>
        <mysql>5.1.25</mysql>
        <mysql.druid>0.2.23</mysql.druid>
        <swagger.version>2.6.1</swagger.version>
        <fastjson.version>1.2.46</fastjson.version>
        <common.lange.version>3.8.1</common.lange.version>
    </properties>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
    </parent>


    <dependencies>
        <!--spring-boot-starter-dubbo-->
        <dependency>
            <groupId>com.gitee.reger</groupId>
            <artifactId>spring-boot-starter-dubbo</artifactId>
            <version>1.1.2</version>
        </dependency>
        <!--lombok的版本号管理-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
        </dependency>
        <!--spring的核心包-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${springBoot.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <version>${springBoot.version}</version>
            <optional>true</optional>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>${springBoot.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
            <version>${springBoot.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${spring.mybatis}</version>
        </dependency>
        <!--数据库和链接池-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql}</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>${mysql.druid}</version>
        </dependency>
        <!--swagger依赖文件-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>${swagger.version}</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>${swagger.version}</version>
        </dependency>
        <!--redis的依赖包-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--工具包-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>${common.lange.version}</version>
        </dependency>
    </dependencies>

</project>
```


(2).因为刚开始创建子模块的时候都是创建的空的子Maven模块，因为我们应该对controller和rpc进行改造为Springboot工程。
分别创建ControllerApplication和RpcApplication两个启动类,和application.yml文件配置如下：
ControllerApplication.java

```stylus
/**
 * 启动类
 * @author :Mr.kk
 * @date: 2019/2/14 17:43
 */
@SpringBootApplication
@ComponentScan(basePackages = {"com.kk.*"})
public class ControllerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ControllerApplication.class, args);
    }
}
```


application.yml

```stylus
server:
  port: 8761
  servlet:
    context-path: /dubbo-controller
```



在controller包中，编写一个测试用例TestController.java，并且启动工程运行，再浏览器输入

```stylus
package com.kk.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 测试的类
 * @author :Mr.kk
 * @date: 2019/2/14 18:07
 */
@RestController
public class TestController {

    @GetMapping("/test")
    public String test(){
        return "test is ok";
    }

}

```


出现如下的结果，证明controller和rpc两个子模块已经完成。
![在这里插入图片描述](/blog17/7.png)

**四.接下来进行dubbo的整合,在这之前你需要先搭建好zookeeper，后续会出一篇关于zookeeper的搭建。**
(1).dubbo的包我在pom中已经引入了，支持最新的boot2.0
先配置controller的application.yml

```stylus
server:
   port: 8761
   servlet:
     context-path: /dubbo-controller

spring:
  dubbo:
    scan:
      base-packages: com.kk.controller
    application:
      id: dubbo-controller
      name: dubbo-controller
    protocol:
      id: dubbo
      name: dubbo
    registry:
      address: zookeeper://zkserver:2181
    consumer:
      timeout: 5000
      check: true         # 服务启动时检查被调用服务是否可用
      retries: 1
```


(2). 在配置rpc的application.yml

```stylus
server:
  port: 8762
  servlet:
    context-path: /dubbo-rpc

spring:
  dubbo:
    scan:
      base-packages: com.kk.rpc
    application:
      id: dubbo-rpc
      name: dubbo-rpc
    protocol:
      id: dubbo
      name: dubbo
    registry:
      address: zookeeper://zkserver:2181
    consumer:
      timeout: 5000
      check: true      # 服务启动时检查被调用服务是否可用
```


**五.基本上的配置就已经完成了，接下来测试一下！**
(1).先编写一个api的接口，代码如下：


```stylus
package com.kk.api;
/**
 * @author :Mr.kk
 * @date: 2019/2/15 16:13
 */
public interface TestService {

    /** 调用本地测试接口
     * @param str
     * @return
     */
    public String TestLocalServer(String str);

    /** 调用rpc测试接口
     * @param str
     * @return
     */
    public String TestRpcServer(String str);

}

```


(2).编写Rpc服务和本地自身调用的Service的实现类：
 本地：

```stylus
package com.kk.service;

import com.kk.api.TestService;
import org.springframework.stereotype.Service;

/**
 * @author :Mr.kk
 * @date: 2019/2/15 16:35
 */
@Service
public class TestServiceImpl implements TestService{


    /**
     * 调用本地测试接口
     *
     * @param str
     * @return
     */
    @Override
    public String TestLocalServer(String str) {
        return "local server" + str;
    }

    /**
     * 调用rpc测试接口
     *
     * @param str
     * @return
     */
    @Override
    public String TestRpcServer(String str) {
        return null;
    }
}

```


rpc：这里需要注意下，因为框架对dubbo的二次封装这边不能使用Dubbo的service发布服务，改用@Export
package com.kk.rpc;

```stylus
import com.kk.api.TestService;
import com.reger.dubbo.annotation.Export;

/**
 * @author :Mr.kk
 * @date: 2019/2/15 16:35
 */
@Export
public class TestServiceImpl implements TestService{


    /**
     * 调用本地测试接口
     *
     * @param str
     * @return
     */
    @Override
    public String TestLocalServer(String str) {
        return null;
    }

    /**
     * 调用rpc测试接口
     *
     * @param str
     * @return
     */
    @Override
    public String TestRpcServer(String str) {
        return "rpc server" + str;
    }
}

```


(3).编写Controller的调用，这里可以用@Reference,也可以用@Inject，代码如下：

```stylus
package com.kk.controller;

import com.alibaba.dubbo.config.annotation.Reference;
import com.kk.api.TestService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;

/**
 * 测试的类
 * @author :Mr.kk
 * @date: 2019/2/14 18:07
 */
@RestController
public class TestController {

    @Reference
    private TestService testService;

    @GetMapping("/test")
    public String test(){
        return "test is ok";
    }

    @GetMapping("/testRpc")
    public String testRpc(){
        return testService.TestRpcServer(new Date().toString());
    }


}

```


(4).然后分别启动Rpc和Controller的启动类，可以在Dubbo的监控台看到，即表示成功，f发布到zookeeper成功
![在这里插入图片描述](/blog17/9.png)



在浏览器输入，显示：
![在这里插入图片描述](/blog17/10.png)

**到此整合完成,附上代码地址  [github][10]**


  [1]: ./images/1.png "1.png"
  [2]: ./images/2.png "2.png"
  [3]: ./images/3.png "3.png"
  [4]: ./images/4.png "4.png"
  [5]: ./images/5.png "5.png"
  [6]: ./images/6.png "6.png"
  [7]: ./images/8.png "8.png"
  [8]: ./images/9.png "9.png"
  [9]: ./images/10.png "10.png"
  [10]: https://github.com/zhaokuankuan/Dubbo-Application
	