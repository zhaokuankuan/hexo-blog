---
title: Springboot的用法之整合Mybatis
comments: true
tags: [MyBatis,Springboot]
categories: [javaWeb]
date: 2018-08-16 12:25:07
---

***Springboot的用法之整合Mybatis***


----------


因为Springboot在现在的JaveWeb开发中使用越来越多，今天就出一个Springboot整合Mybatis的文章，因为Springboot一直崇尚的就是“约定大于配置”，因为在本篇只会有很少的配置文件。


----------








一.  准备工作
	首先你需要先创建好一个Springboot的工程，具体的步骤可参考上一篇==》[springboot项目的搭建和基本的用法][2]
    本文将使用的是Springboot2.0版本 
    PS:开发的过程中最好启动热部署，那样的话会让工作量大大的减少。


----------


    
二 . 开始集成Mybatis
	

 1. 首先需引入Mybatis和Mysql的相关依赖包
   

``` 
		<!--mysql的核心包-->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.45</version>
			<scope>runtime</scope>
		</dependency>
		<!--mybatis工具类-->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.1</version>
		</dependency>
		<!--swagger依赖文件-->
```


 2 然后开始配置Springboot的配置文件，在application.yml中增加数据库的配置，如下：
 

``` 
#数据库的配置信息
spring:
  datasource:
    driverClassName: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/testuseUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root

```

 

```
  然后再开启sql的打印，增加如下配置：
  #配置sql的打印 //包路径为mapper文件包路径
  configuration:
      log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```


 - 至此，配置已基本完成，我的工程的目录结构如图示：
   ![这里写图片描述](/blog13/1.png)
   
 3 具体的代码实现：
 	
 - controller==>DemoController
 

``` stylus
package com.kk.Springbootmanger.controller;

import com.kk.Springbootmanger.dao.UserDao;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import javax.management.Query;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * @author :zhaokk
 * @date: 2018/8/13 - 16:27
 */
@RestController
public class DemoController {

    @Autowired
    private UserDao userDao;


    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    public String sayHello(String name){
        return name;
    }

    @RequestMapping(value = "/getAll",method = {RequestMethod.GET,RequestMethod.POST})
    public Object getAll(){
        return userDao.getAll();
    }
}

```


   
 - domain==>User.java
   

``` stylus
package com.kk.Springbootmanger.domain;

import com.kk.Springbootmanger.common.UserSexEnum;

/**
* @author :Mr.kk
* @date: 2018/8/14/9:41
*/
public class User {
 private Long id;

 private String userName;

 private String passWord;

 private UserSexEnum userEnum;

 private String nickName;

 public User() {
 }

 public User(Long id, String userName, String passWord, UserSexEnum userEnum, String nickName) {
     this.id = id;
     this.userName = userName;
     this.passWord = passWord;
     this.userEnum = userEnum;
     this.nickName = nickName;
 }

 public void setId(Long id) {
     this.id = id;
 }

 public void setUserName(String userName) {
     this.userName = userName;
 }

 public void setPassWord(String passWord) {
     this.passWord = passWord;
 }

 public void setUserEnum(UserSexEnum userEnum) {
     this.userEnum = userEnum;
 }

 public void setNickName(String nickName) {
     this.nickName = nickName;
 }

 public Long getId() {
     return id;
 }

 public String getUserName() {
     return userName;
 }

 public String getPassWord() {
     return passWord;
 }

 public UserSexEnum getUserEnum() {
     return userEnum;
 }

 public String getNickName() {
     return nickName;
 }
}

```


 - dao==>UserDao.java
   

``` stylus
package com.kk.Springbootmanger.dao;

import com.kk.Springbootmanger.domain.User;

import java.util.List;

/**
* @author :Mr.kk
* @date: 2018/8/14-9:44
*/
public interface UserDao {
  //全查
  List<User> getAll();
  //根据id进行查询
  User getOne(Long id);
}

```


 - mapper==>UserMapper.xml
   

``` stylus
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.kk.Springbootmanger.dao.UserDao" >

<resultMap id="BaseResultMap" type="com.kk.Springbootmanger.domain.User" >
   <id column="id" property="id" jdbcType="BIGINT" />
   <result column="userName" property="userName" jdbcType="VARCHAR" />
   <result column="passWord" property="passWord" jdbcType="VARCHAR" />
   <result column="user_sex" property="userEnum" javaType="com.kk.Springbootmanger.common.UserSexEnum"/>
   <result column="nick_name" property="nickName" jdbcType="VARCHAR" />
</resultMap>

<sql id="Base_Column_List" >
   id, userName, passWord, user_sex, nick_name
</sql>


<select id="getAll" resultMap="BaseResultMap"  >
   SELECT
   <include refid="Base_Column_List" />
   FROM users
</select>

<select id="getOne" parameterType="Long" resultMap="BaseResultMap" >
   SELECT
   <include refid="Base_Column_List" />
   FROM users
   WHERE id = #{id}
</select>



</mapper>
```


 5.至此，基本的配置文件和配置已经完成


----------


三.启动，验证
	启动该工程，然后访问 http://localhost:8081/manager/getAll ，如图所示即成功了，顺便看到控制台打印出来的sql,完整的代码已经上传至[Gihub][4]
   ![这里写图片描述](/blog13/2.png)


  [1]: http://www.struggling-bird.cn
  [2]: https://blog.csdn.net/zhaokk_git/article/details/79608197
  [4]: https://github.com/zhaokuankuan/springboot-mybatis
