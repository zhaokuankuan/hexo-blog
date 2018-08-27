---
title: Java代码生成平台(Springboot+Mybatis+Swagger)
comments: true
tags: [Springboot,Swagger]
categories: JavaWeb
date: 2018-08-23 16:45:16
---

**Java代码生成平台(Springboot+Mybatis+Swagger)**

[个人博客传送门](http://www.struggling-bird.cn/)


----------


由于最近本猿想做一个开源的项目，为了避免开发冗余代码的弊端，因此开始研究代码生成的工具，在看了**[xxl](https://github.com/xuxueli/xxl-code-generator)**的开源项目时眼前一亮，开始深入理解，但是后来随着深入的加深，发现由于技术的更新和开发的风格大不同，因此在该项目的基础上定制和升级，现将部署的项目分享给大家，可以自己定制化自己的工具，也可以直接用我的平台直接生成。


----------


**在这里我只讲解如何去使用本猿搭建的环境**：

一.准备环境
  Springboot(Spring和SpingMVC),Mybatis,Swagger
  搭建好上面的环境之后便可以开始生成代码，如果不会搭建Springboot和Mybatis的环境，本猿已经默认集成了Swagger了。
  [搭建教程传送门](https://blog.csdn.net/zhaokk_git/article/details/81738366)
  


----------


 二.生成代码
 1. 打开生成平台
 
  [代码生成平台传送门]( http://coder.struggling-bird.cn:8080/create_code/#)
  
 出现如图所示的界面，便开始生成代码了！ 
 
   ![这里写图片描述](/blog15/1.png)
    

2.已经给出了一个测试用的sql了，你可以直接点击按钮生成按钮，注意在生成的时候sql的格式需要如上图的测试sql所示。

3.生成的代码如图

![这里写图片描述](/blog15/2.png)

如上所示，所有的代码已经生成完毕了，接下来开始将生成的代码集成到我们的工程中去。


----------


三.将生成的代码集成到我们的工程中去
  1.先向大家展示一下我的项目的结构，对应的如下目录
  
   controller    service==》impl   dao   domain  mapper 
   
  ![这里写图片描述](/blog15/3.png)

  
  2.需要修改的地方如下： 我们需要修改dao和domain的路径
  
  ![这里写图片描述](/blog15/4.png)
  
  
 3.接下来我们便可以运行项目，调用对应的接口进行调试了。
 
 


----------


 以上，便是生成代码的平台的具体的使用方法，大家可以根据自己的风格和需求定制化自己的生成工具！
 [代码地址](https://github.com/zhaokuankuan/xxl-code-generator)


