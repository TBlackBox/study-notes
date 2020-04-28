# JavaEE简介
JavaEE是java应用最普遍的一个领域，

# JavaEE,JavaWeb，JavaSE,JavaME的区别

## JavaEE
全称Java平台企业版（Java Platform Enterprise Edition），是Sun公司为企业级应用推出的标准平台。JavaEE是个大杂烩，包括Applet、EJB、JDBC、JNDI、Servlet、JSP等技术的标准，运行在一个完整的应用服务器上，用来开发大规模、分布式、健壮的网络应用。

## JavaWeb
主要指以Java语言为基础，利用JavaEE中的Servlet、JSP等技术开发动态页面，方便用户通过浏览器与服务器后台交互。Java Web应用程序可运行在一个轻量级的Web服务器中，比如Tomcat。

## JavaSE 
$java_home/jre/lib/rt.jar里的全部内容，比如java目录下的lang包、util包、io/nio包等14个包的内容，一般是一些基础类库，例如包装类、collection、concurrent并发包、函数式接口、反射、注解等，简单粗暴的讲，JavaSE就是jdk包含的内容；

## JavaME,Java Micro Edition
Java的微型版，用于android和一些嵌入式设备。

## 相互间的区别
### JavaSE与JavaEE
JavaSE包含annotation和自定义注解API（Pluggable Annotation Processing API）的内容，JavaEE则是提出JSR269规范，所有框架中定义的注解都必须符合该规范，典型的例如lombok中的@Getter/@Setter注解就是符合JSR269规范的，当然你也可以依据JSR269的要求，自己实现一个@Getter/@Setter注解；

### JavaEE与JavaWeb
可以粗略地认为JavaWeb就是JavaEE的一部分，里面用的是JavaEE规范的东西。


# Servlet
Servlet是JavaEE最根本的组件之一，Servlet3带来了异步提高了其处理请求 的性能。

