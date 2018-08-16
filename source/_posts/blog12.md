---
title: 基于SpringMVC的文件上传
comments: true
tags: [Spring,SpringMvc,java]
categories: [JavaWeb]
date: 2018-08-11 17:34:31
---
由于是一个新手菜鸟，所以说对很多东西都不是很了解，最近刚好在做一个项目需要做文件的上传和下载，以前直接是用写好的，这个自己动手写了一下用了半天时间跟大家分享一下。

一.环境是SSM+Maven

      首先，你需要搭建好springMC的环境，如果不会搭建的话请自己百度，他会告诉你的。

二.导入需要的Jar包

因为这个附件的上传和下载是基于SpringMVC做的，因此我们需要导入一下jar包。如下，我直接在pom中引入就可以了。





```
<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3</version>
		</dependency>
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.2</version>
		</dependency>
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3</version>
		</dependency>
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.2</version>
		</dependency>
```










三.接下来比较重要了，就是需要去配置下你的springMVC的配置文件，配置信息如下。





```
 <bean id="multipartResolver"
	        class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	        <!-- 上传文件大小上限，单位为字节（10MB） -->
	        <property name="maxUploadSize">
	            <value>10485760</value>
	        </property>
	        <property name="defaultEncoding">
	            <value>UTF-8</value>
	        </property>
	    </bean>
		<bean id="multipartResolver"
	        class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	        <!-- 上传文件大小上限，单位为字节（10MB） -->
	        <property name="maxUploadSize">
	            <value>10485760</value>
	        </property>
	        <property name="defaultEncoding">
	            <value>UTF-8</value>
	        </property>
	    </bean>






```


四。接下来就是写html界面(最重要的一点是一定要在form中加上 enctype="multipart/form-data" )

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h3>文件上传</h3>
	<form enctype="multipart/form-data"  action="http://localhost:8080/yangfan-server/upload" method="post" >
	    <input type="file" name="file" id="file"/>
	    参数inputStr:<input type="text" name="des">
	        <input type="submit" value="submit" />
	</form>
	<hr/>

	<h3>文件下载</h3>
		<a href="download?filename=50769870/Desert.jpg">
		  Penguins.jpg
		</a>
</body>
</html>


```








五.接下来就是controller的实现

这个是文件上传的，因为我自己写了一个方法来生成一个随机的路径，你们也可以自己定义自己的路径。



```
//上传文件会自动绑定到MultipartFile中
    @RequestMapping(value="/upload",method={RequestMethod.POST})
    public Result upload(HttpServletRequest request,
           @RequestParam("des") String des,
           @RequestParam("file") MultipartFile file) throws Exception {

    	Result result = new Result();
       System.out.println(des);
       //如果文件不为空，写入上传路径
       if(!file.isEmpty()) {
    	   long uuid = NumberUtil.createId();
    	   //上传文件路径
    	   String path = request.getServletContext().getRealPath("/images/") + File.separator + uuid;
           //上传文件名
           String filename = file.getOriginalFilename();
           File filepath = new File(path,filename);
           //判断路径是否存在，如果不存在就创建一个
           if (!filepath.getParentFile().exists()) {
               filepath.getParentFile().mkdirs();
           }
           //将上传文件保存到一个目标文件当中
           file.transferTo(new File(path + File.separator +filename));
           result.setSuccess(true);
           result.addDefaultModel("fileName", "fileName");
           result.addDefaultModel("fileDownLoad",uuid+"/"+filename);
           return result;
       } else {
           return result;
       }

    }

```








这个是下载的deomo


```

 @RequestMapping(value="/download",method={RequestMethod.POST,RequestMethod.GET})
    public ResponseEntity<byte[]> download(HttpServletRequest request,
            @RequestParam("filename") String filename,
            Model model)throws Exception {
       //下载文件路径
       String path = request.getServletContext().getRealPath("/images/");
       File file = new File(path + File.separator + filename);
       HttpHeaders headers = new HttpHeaders();
       //下载显示的文件名，解决中文名称乱码问题
       String downloadFielName = new String(filename.getBytes("UTF-8"),"iso-8859-1");
       String[] strArr = downloadFielName.split("/");
       //通知浏览器以attachment（下载方式）打开图片
       headers.setContentDispositionFormData("attachment", strArr[1]);
       //application/octet-stream ： 二进制流数据（最常见的文件下载）。
       headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
       return new ResponseEntity<byte[]>(FileUtils.readFileToByteArray(file),
               headers, HttpStatus.CREATED);
    }
```

以上就是上传和下载的代码了，不过现在只是上传单个文件的，后续会加上多个文件的批量操作的。



欢迎转载，请注明出处





