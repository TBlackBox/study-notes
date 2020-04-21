# Servlet 创有三种方式

## 实现 Servlet 接口

因为是实现 Servlet 接口，所以我们需要实现接口里的方法。

下面我们也说明了 Servlet 的执行过程，也就是 Servlet 的生命周期。

```
//Servlet的生命周期:从Servlet被创建到Servlet被销毁的过程
//一次创建，到处服务
//一个Servlet只会有一个对象，服务所有的请求
/*
 * 1.实例化（使用构造方法创建对象）
 * 2.初始化  执行init方法
 * 3.服务     执行service方法
 * 4.销毁    执行destroy方法
 */
public class ServletDemo1 implements Servlet {

    //public ServletDemo1(){}

     //生命周期方法:当Servlet第一次被创建对象时执行该方法,该方法在整个生命周期中只执行一次
    public void init(ServletConfig arg0) throws ServletException {
                System.out.println("=======init=========");
        }

    //生命周期方法:对客户端响应的方法,该方法会被执行多次，每次请求该servlet都会执行该方法
    public void service(ServletRequest arg0, ServletResponse arg1)
            throws ServletException, IOException {
        System.out.println("hehe");

    }

    //生命周期方法:当Servlet被销毁时执行该方法
    public void destroy() {
        System.out.println("******destroy**********");
    }
//当停止tomcat时也就销毁的servlet。
    public ServletConfig getServletConfig() {

        return null;
    }

    public String getServletInfo() {

        return null;
    }
}
```

## 继承 GenericServlet 类

它实现了 Servlet 接口除了 service 的方法，不过这种方法我们极少用。

```
public class ServletDemo2 extends GenericServlet {

    @Override
    public void service(ServletRequest arg0, ServletResponse arg1)
            throws ServletException, IOException {
        System.out.println("heihei");

    }
}
```

## 继承 HttpServlet 方法

```
public class ServletDemo3 extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        System.out.println("haha");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        System.out.println("ee");
        doGet(req,resp);
    }

}
```

#  HttpServlet、GenericServlet 和 Servlet 的关系

对于一个 Servlet 类，我们日常最常用的方法是继承自 HttpServlet 类，提供了 Http 相关的方法，HttpServlet 扩展了 GenericServlet 类，而 GenericServlet 类又实现了 Servlet 类和 ServletConfig 类。

## Servlet

Servlet 类提供了五个方法，其中三个生命周期方法和两个普通方法，关于 Servlet 类的方法，不再赘述。

## GenericServlet

GenericServlet 是一个抽象类，实现了 Servlet 接口，并且对其中的 init() 和 destroy() 和 service() 提供了默认实现。在 GenericServlet 中，主要完成了以下任务：

-  将 init() 中的 ServletConfig 赋给一个类级变量，可以由 getServletConfig 获得；
-  为 Servlet 所有方法提供默认实现；
-  可以直接调用 ServletConfig 中的方法；

基本的结构如下：

```
abstract class GenericServlet implements Servlet,ServletConfig{
 
   //GenericServlet通过将ServletConfig赋给类级变量
   private trServletConfig servletConfig;
 
   public void init(ServletConfig servletConfig) throws ServletException {

      this.servletConfig=servletConfig;

      /*自定义init()的原因是：如果子类要初始化必须覆盖父类的init() 而使它无效 这样
       this.servletConfig=servletConfig不起作用 这样就会导致空指针异常 这样如果子类要初始化，
       可以直接覆盖不带参数的init()方法 */
      this.init();
   }
   
   //自定义的init()方法，可以由子类覆盖  
   //init()不是生命周期方法
   public void init(){
  
   }
 
   //实现service()空方法，并且声明为抽象方法，强制子类必须实现service()方法 
   public abstract void service(ServletRequest request,ServletResponse response) 
     throws ServletException,java.io.IOException{
   }
 
   //实现空的destroy方法
   public void destroy(){ }
}
```

以上就是 GenericServlet 的大致实现思想，可以看到如果继承这个类的话，我们必须重写 service() 方法来对处理请求。

## HttpServlet

HttpServlet 也是一个抽象类，它进一步继承并封装了 GenericServlet，使得使用更加简单方便，由于是扩展了 Http 的内容，所以还需要使用 HttpServletRequest 和 HttpServletResponse，这两个类分别是 ServletRequest 和 ServletResponse 的子类。代码如下：

```
abstract class HttpServlet extends GenericServlet{
 
   //HttpServlet中的service()
   protected void service(HttpServletRequest httpServletRequest,
                       HttpServletResponse httpServletResponse){
        //该方法通过httpServletRequest.getMethod()判断请求类型调用doGet() doPost()
   }
 
   //必须实现父类的service()方法
   public void service(ServletRequest servletRequest,ServletResponse servletResponse){
      HttpServletRequest request;
      HttpServletResponse response;
      try{
         request=(HttpServletRequest)servletRequest;
         response=(HttpServletResponse)servletResponse;
      }catch(ClassCastException){
         throw new ServletException("non-http request or response");
      }
      //调用service()方法
      this.service(request,response);
   }
}
```

我们可以看到，HttpServlet 中对原始的 Servlet 中的方法都进行了默认的操作，不需要显式的销毁初始化以及 service()，在 HttpServlet 中，自定义了一个新的 service() 方法，其中通过 getMethod() 方法判断请求的类型，从而调用 doGet() 或者 doPost() 处理 get,post 请求，使用者只需要继承 HttpServlet，然后重写 doPost() 或者 doGet() 方法处理请求即可。

我们一般都使用继承 HttpServlet 的方式来定义一个 servlet。





# Servlet 实例

Servlet 是服务 HTTP 请求并实现 **javax.servlet.Servlet** 接口的 Java 类。Web 应用程序开发人员通常编写 Servlet 来扩展 javax.servlet.http.HttpServlet，并实现 Servlet 接口的抽象类专门用来处理 HTTP 请求。

## Hello World 示例代码

下面是 Servlet 输出 Hello World 的示例源代码：

```
// 导入必需的 java 库
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;

// 扩展 HttpServlet 类
public class HelloWorld extends HttpServlet {
 
  private String message;

  public void init() throws ServletException
  {
      // 执行必需的初始化
      message = "Hello World";
  }

  public void doGet(HttpServletRequest request,
                    HttpServletResponse response)
            throws ServletException, IOException
  {
      // 设置响应内容类型
      response.setContentType("text/html");

      // 实际的逻辑是在这里
      PrintWriter out = response.getWriter();
      out.println("<h1>" + message + "</h1>");
  }
  
  public void destroy()
  {
      // 什么也不做
  }
}
```

## 编译 Servlet

让我们把上面的代码写在 HelloWorld.java 文件中，把这个文件放在 C:\ServletDevel（在 Windows 上）或 /usr/ServletDevel（在 UNIX 上）中，您还需要把这些目录添加到 CLASSPATH 中。

假设您的环境已经正确地设置，进入 **ServletDevel** 目录，并编译 HelloWorld.java，如下所示：

```
$ javac HelloWorld.java
```

如果 Servlet 依赖于任何其他库，您必须在 CLASSPATH 中包含那些 JAR 文件。在这里，我只包含了 servlet-api.jar JAR 文件，因为我没有在 Hello World 程序中使用任何其他库。

该命令行使用 Sun Microsystems Java 软件开发工具包（JDK）内置的 javac 编译器。为使该命令正常工作， PATH 环境变量需要设置 Java SDK 的路径。

如果一切顺利，上面编译会在同一目录下生成 HelloWorld.class 文件。下一节将讲解已编译的 Servlet 如何部署在生产中。

## Servlet 部署

默认情况下，Servlet 应用程序位于路径 <Tomcat-installation-directory>/webapps/ROOT 下，且类文件放在 <Tomcat-installation-directory>/webapps/ROOT/WEB-INF/classes 中。

如果您有一个完全合格的类名称 **com.myorg.MyServlet**，那么这个 Servlet 类必须位于 WEB-INF/classes/com/myorg/MyServlet.class 中。

现在，让我们把 HelloWorld.class 复制到 <Tomcat-installation-directory>/webapps/ROOT/WEB-INF/classes 中，并在位于 <Tomcat-installation-directory>/webapps/ROOT/WEB-INF/ 的 **web.xml** 文件中创建以下条目：

```
<web-app>      
    <servlet>
        <servlet-name>HelloWorld</servlet-name>
        <servlet-class>HelloWorld</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>HelloWorld</servlet-name>
        <url-pattern>/HelloWorld</url-pattern>
    </servlet-mapping>
</web-app>  
```

上面的条目要被创建在 web.xml 文件中的 <web-app>...</web-app> 标签内。在该文件中可能已经有各种可用的条目，但不要在意。