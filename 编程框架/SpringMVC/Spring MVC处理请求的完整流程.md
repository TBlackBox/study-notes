# 简介

Spring MVC框架是高度可配置的，包含多种视图技术，例如  技术、Velocity、Tiles、iText 和 POI。
MVC 框架并不关心使用的视图技术，也不会强迫开发者只使用 JSP 技术，但教程中使用的视图是 JSP，本节主要介绍 Spring MVC 框架处理用户请求的完整流程和处理中包含的 4 个接口。Spring MVC 框架主要由 Dispatcher、处理器映射、控制器、视图解析器、视图组成。


 # 工作原理

![](../../sources/img/5-1ZG2095404c8.png)

1. 客户端请求提交到 DispatcherServlet（在web.xml中配置，是SpringMVC的前置控制器）。
2. 从HandlerMapping中匹配此次请求信息的Handler,匹配条件：请求路径，请求方法，header信息等。有下面几个常用的几个Handler Mapping。
- SimpleUrlHandlerMapping:简介映射一个URL到一个Handler上。
- RequestMappingHandlerMapping:扫描RequestMapping注解，根据相关配置，绑定到URL到一个Handler。
3. 获取对应的Handler(Controller,控制器)，这里有一个关键组件HandlerAdapter，是Controller的适配器，SpringMVC最终通过HandlerAdapter来调用实际的Controller方法，下面是两个常用的适配器：
- SimpleControllerHandlerAdapter:处理实现了Controller接口的Controller。
- RequestMappingHandlerAdapter:处理类型为HandlerMethod的Handler,这里使用RequestMapping注解的Controller的方法就是一种HandlerMethod。
4. Handler 调用业务逻辑处理后返回 ModelAndView。
5. DispatcherServlet 寻找一个或多个 ViewResolver 视图解析器(这个需要配置)，找到 ModelAndView 指定的视图。ViewResolver常用的几个视图解析器。
- UrlBasedViewResolver:通过配置文件，根据URL把一个视图名交给一个View来处理。
- InternalResourceViewResolver:根据配置好的资源路径，解析jsp视图，支持jstl.
- FreeMakerViewResolver:freemarker的视图解析器。
6. 生成视图返回给用户，有下面几个常用的视图类：
- MappingJackson2View：使用mappingJackson输出JSON数据的视图，数据来源于ModelMap。
- FreeMarkerView:使用FreeMarker模板引擎的视图。
- JSTLView:输出jsp页面，可以使用jsp标签库。





# HandlerIntercepter组件
HandlerInterceptor是请求路径上的拦截器，需要自己实现这个接口以拦截请求，做一些对Handler的前置或后置处理工作。几个方法的描述：

- preHandle(..)：在执行实际的处理程序之前

- postHandle(..)：执行处理程序后

- afterCompletion(..)：完成完整的请求后

该preHandle(..)方法返回一个布尔值。您可以使用此方法来中断或继续执行链的处理。当此方法返回时true，处理程序执行链继续。当它返回false时，DispatcherServlet 假定拦截器本身已经处理了请求（例如，渲染了适当的视图），并且不会继续执行执行链中的其他拦截器和实际处理程序。

# HandlerExceptionResolver
这个是异常处理器，其可以实现直接的处理器在全局层面的拦截Handler抛出的Exeception,在做进一步的处理。SpringMVC自带了以下几个处理器。

## SimpleMappingExceptionResolver

异常类名称和错误视图名称之间的映射。对于在浏览器应用程序中呈现错误页面很有用。

## DefaultHandlerExceptionResolver

解决Spring MVC引发的异常，并将其映射到HTTP状态代码。另请参见替代ResponseEntityExceptionHandler和REST API异常。

## ResponseStatusExceptionResolver

使用@ResponseStatus注释解决异常，并根据注释中的值将其映射到HTTP状态代码。

## ExceptionHandlerExceptionResolver

通过调用@ExceptionHandler @Controller或 @ControllerAdvice中的方法来解决异常。