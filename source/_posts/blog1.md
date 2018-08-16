---
title: Vue+Webpack+node构建web和App项目
comments: true
tags: [Vue,Node,Webpack]
categories: Web前端
date: 2018-08-11 18:16:02
---
       先说一下，本猿自今年接触到vue之后，感到vue全家桶之大，不能穷也，虽然这个框架的越来越成熟，越来越多的对应的组件框架伴随而出，例如，web端的elementUI,移动端的mintUi等。本猿在学习和使用了半年之久，然后粗略的总结下搭建和使用的过程，方便自己以后继续深入学习和帮助一下刚入门的新猿们。
 	 这次主要是以APP端的mintUI为例子讲解下，其实web端的elementUI和这个基本使用的方式一样。

 一.环境的准备
 	node ， vue ，mintUI
 	首先下载node。
 	js.Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。 Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。 Node.js 的包管理器 npm，是全球最大的开源库生态系统。(介绍来自百度)
 	下载完成之后可以在dos中输入： node -v   和   npm -v查看你所安装的node的版本，一般情况下你在安装好node后，自动就给你安装好了npm。
 	接下来你需要安装淘宝镜像，具体的原因你可以百度，这里给小白说下，你不安装也是可以的，默认是npm是国外的，cnpm是国内的(淘宝)。npm install -g cnpm -registry=https://registry.npm.taobao.org
    安装完淘宝的镜像之后就可以安装Webpack了，我这里用cnpm装	cnpm install webpack -g 。(介绍来自百度) webpack是近期最火的一款模块加载器兼打包工具，它能把各种资源，例如JS（含JSX）、coffee、样式（含less/sass）、图片等都作为模块来使用和处理。
    安装完Webpack之后，现在就是安装vue的脚手架了，cnpm install vue-cli -g（给你们个建议，基本上我是在我需要创建项目的目录下安装这个脚手架的）。
    接下来就是见证奇迹的时刻了，就是创建我们的项目了，这个你想存放在哪个目录，你就cd到哪个目录之下执行， vue init webpack  项目名称。 然后你会进入一个引导的目录，根据目录填写你的项目名称和是否使用路由等。
    ![这里写图片描述](/blog1/20171220162547881-1.png)
 	上面这个图来自网络，可以按照这个配置。到此所有的创建工作就全部完成了，然后 cd 到该项目底下   npm  run  dev ，在浏览器中localhost：8080 就可以看见你新创建的项目。
 	二.下面上一下这个项目的目录结构


 ```
 ├── build --------------------------------- webpack相关配置文件
 │   ├── build.js --------------------------webpack打包配置文件
 │   ├── check-versions.js ------------------------------ 检查npm,nodejs版本
 │   ├── dev-client.js ---------------------------------- 设置环境
 │   ├── dev-server.js ---------------------------------- 创建express服务器，配置中间件，启动可热重载的服务器，用于开发项目
 │   ├── utils.js --------------------------------------- 配置资源路径，配置css加载器
 │   ├── vue-loader.conf.js ----------------------------- 配置css加载器等
 │   ├── webpack.base.conf.js --------------------------- webpack基本配置
 │   ├── webpack.dev.conf.js ---------------------------- 用于开发的webpack设置
 │   ├── webpack.prod.conf.js --------------------------- 用于打包的webpack设置
 ├── config ---------------------------------- 配置文件
 ├── node_modules ---------------------------- 存放依赖的目录
 ├── src ------------------------------------- 源码
 │   ├── assets ------------------------------ 静态文件
 │   ├── components -------------------------- 组件
 │   ├── main.js ----------------------------- 主js
 │   ├── App.vue ----------------------------- 项目入口组件
 │   ├── router ------------------------------ 路由
 ├── package.json ---------------------------- node配置文件
 ├── .babelrc--------------------------------- babel配置文件
 ├── .editorconfig---------------------------- 编辑器配置
 ├── .gitignore------------------------------- 配置git可忽略的文件
 ```

 到现在这个项目只是搭建好了一个Vue的架子，我们需要把我们需要的插件引入，可以利用 cnpm install  插件名称  导入插件，我这里导入一下mintUI的插件 cnpm install --save mint-ui   然后需要在main.js中引入刚才导入的mintui和css。[这个是mintUI的官网](http://mint-ui.github.io/#!/zh-cn)
 	import MintUI from 'mint-ui'
 	import 'mint-ui/lib/style.css'
 	Vue.use(MintUI)
 三.做好了准备工作，接下来就是我们的第一个项目了
 		直接上几张效果图。
 		![这里写图片描述](/blog1/20171220163848571-2.png)

   这个有一个校验当没有账号和密码的时候，会提示让你填写，有的话就会跳转到这
   ![这里写图片描述](/blog1/20171220164015050-3.png)
   然后我把我的代码上传到github上，你们可以down。https://github.com/zhaokuankuan/app.git
    提示一下，你们在安装的时候需要安装好node 然后 cd到down下来的目录下，cnpm install  就安装完成好所有的jar包，然后npm  run   dev就可以了。