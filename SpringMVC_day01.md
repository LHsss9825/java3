# 学习目标

- SpringMVC概述
- SpringMVC入门
- RequestMapping注解
- controller方法返回值
- 参数接收
- 参数传递
- JSON数据处理

# 1.SpringMVC概述

## 1.1.MVC介绍

MVC是一种软件架构的思想，将软件按照模型、视图、控制器来划分

- M：Model，模型层，指工程中的JavaBean，作用是处理数据

     JavaBean分为两类：

  - 一类称为实体类Bean：专门存储业务数据的，如 Student、User 等
  - 一类称为业务处理 Bean：指 Service 或 Dao 对象，专门用于处理业务逻辑和数据访问。

- V：View，视图层，指工程中的html或jsp等页面，作用是与用户进行交互，展示数据


- C：Controller，控制层，指工程中的servlet，作用是接收请求和响应浏览器


MVC的工作流程：
用户通过视图层发送请求到服务器，在服务器中请求被Controller接收，Controller调用相应的Model层处理请求，处理完毕将结果返回到Controller，Controller再根据请求处理的结果找到相应的View视图，渲染数据后最终响应给浏览器

## 1.2.Spring MVC介绍

- Spring MVC 是Spring框架的一个模块，是一个基于 MVC 设计模式的轻量级 Web 开发框架，本质上相当于 Servlet。

- SpringMVC 是 Spring 为表示层开发提供的一整套完备的解决方案。在表述层框架历经 Strust、WebWork、Strust2 等诸多产品的历代更迭之后，目前业界普遍选择了 SpringMVC 作为 Java EE 项目表述层开发的**首选方案**。

  > 注：三层架构分为表示层、业务逻辑层、数据访问层，表述层表示前台页面和后台servlet



# 2.SpringMVC 的入门	

## 2.1.环境搭建

### 2.1.1.创建工程

<img src="assets/image-20211012185038920.png" alt="image-20211012185038920" style="zoom:67%;" />	

### 2.1.2.添加web支持

1. 右键项目选择`Add framework support...`

   <img src="assets/image-20211012185445977.png" alt="image-20211012185445977" style="zoom:67%;" />	

2.添加web支持

​		<img src="assets/image-20211012185550721.png" alt="image-20211012185550721" style="zoom:67%;" />	

3.效果

​		<img src="assets/image-20211012185626626.png" alt="image-20211012185626626" style="zoom:67%;" />	

- 注意：
  1. <font color=red>不要先添加打包方式</font>
  2. webapp目录要拖拽到main目录下

### 2.1.3.pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.qf</groupId>
    <artifactId>SpringMVC_day01</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--打包方式-->
    <packaging>war</packaging>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <!-- 配置Tomcat插件 -->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <!--端口号-->
                    <port>8080</port>
                    <!--项目名-->
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## 2.2.入门案例

### 2.2.1.index.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
  <a href="/hello">hello</a>
  </body>
</html>
```

### 2.2.2.controller

```java
@Controller
public class HelloController {

    @RequestMapping("/hello")
    public ModelAndView hello() {
        //ModelAndView对象封装了模型数据和视图名称
        ModelAndView mv = new ModelAndView();
        //添加数据，request.setAttribute(“hello”,”hello springmvc!!”)
        mv.addObject("hello", "欢迎你 springmvc");
        //设置逻辑视图路径
        mv.setViewName("success");
        //返回数据和视图
        return mv;
    }
}
```

### 2.2.3.springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
	<!-- 配置创建 spring 容器要扫描的包 -->
    <context:component-scan base-package="com.qf"></context:component-scan>

    <!-- 配置视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

    <!--开启springmvc注解支持：配置HandlerMapping和HandlerAdapter-->
    <mvc:annotation-driven></mvc:annotation-driven>
</beans>
```

### 2.2.4.success.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
	<h2>${msg}</h2>
</body>
</html>
```

### 2.2.5.web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 配置初始化参数，用于读取 SpringMVC 的配置文件 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!-- 表示容器在启动时立即创建servlet对象 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

### 2.2.6.测试

访问：http://localhost:8080/hello

<img src="assets/image-20211012194536051.png" alt="image-20211012194536051" style="zoom:67%;" />	

## 2.3.springmvc组件

### 2.3.1.DispatcherServlet前端控制器

用户请求到达前端控制器，它就相当于mvc模式中的c，<font color=red>dispatcherServlet 是整个流程控制的中心，由它调用其它组件处理用户的请求</font>，dispatcherServlet 的存在降低了组件之间的耦合性。

### 2.3.2.HandlerMapping处理器映射器

<font color=red>HandlerMapping负责根据用户请求找到 Handler 即处理器</font>，SpringMVC 提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

### 2.3.3.Handler处理器

它就是我们开发中要编写的具体业务控制器。由DispatcherServlet 把用户请求转发到 Handler。由Handler对具体的用户请求进行处理。	

### 2.3.4.HandlAdapter处理器适配器

<font color=red>通过 HandlerAdapter 对处理器进行执行</font>，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

<img src="assets/image-20211012194905002.png" alt="image-20211012194905002" style="zoom:67%;" />	

适配器对应的处理器以及这些处理器的作用：

1. **AnnotationMethodHandlerAdapter** 主要是适配注解类处理器，注解类处理器就是我们经常使用的 **@Controller 的这类处理器**
2. **HttpRequestHandlerAdapter** 主要是适配静态资源处理器，静态资源处理器就是**实现了 HttpRequestHandler 接口的处理器**，这类处理器的作用是处理通过 SpringMVC 来访问的静态资源的请求
3. **SimpleControllerHandlerAdapter** 是 Controller 处理适配器，适配**实现了 Controller 接口或 Controller 接口子类的处理器**，比如我们经常自己写的 Controller 来继承 MultiActionController.
4. **SimpleServletHandlerAdapter** 是 Servlet 处理适配器, 适配**实现了 Servlet 接口**或 **Servlet 的子类的处理器**，我们不仅可以在 web.xml 里面配置 Servlet，其实也可以用 SpringMVC 来配置 Servlet，不过这个适配器很少用到，而且 SpringMVC 默认的适配器没有他，默认的是前面的三种。

### 2.3.5.View Resolver视图解析器

<font color=red>ViewResolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象返回给DispatcherServlet </font>

### 2.3.6.View视图渲染器

<font color=red>view对象</font>会调用render<font color=red>将model中的数据全部存放到request中</font>完成了请求的处理，源码如下：

```java
public interface View {
    String RESPONSE_STATUS_ATTRIBUTE = View.class.getName() + ".responseStatus";
    String PATH_VARIABLES = View.class.getName() + ".pathVariables";
    String SELECTED_CONTENT_TYPE = View.class.getName() + ".selectedContentType";

    @Nullable
    default String getContentType() {
        return null;
    }
   //把model里的数据存放到request，request和response负载跳转	
   void render(Map<String, ?> model, HttpServletRequest request, 
               			HttpServletResponse response) throws Exception;
}
```



## 2.4.SpringMVC执行流程

![img](assets/image-20211012195252387.png)	

- 具体步骤

  Ø 第一步：发起请求到前端控制器(DispatcherServlet)

  Ø 第二步：前端控制器请求HandlerMapping查找 Handler 

  Ø 第三步：处理器映射器HandlerMapping向前端控制器返回Handler，HandlerMapping会把请求映射为HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象，多个HandlerInterceptor拦截器对象），通过这种策略模式，很容易添加新的映射策略

  Ø 第四步：前端控制器调用处理器适配器去执行Handler

  Ø 第五步：处理器适配器HandlerAdapter将会根据适配的结果去执行Handler

  Ø 第六步：Handler执行完成给适配器返回ModelAndView

  Ø 第七步：处理器适配器向前端控制器返回ModelAndView （ModelAndView是springmvc框架的一个底层对象，包括 Model和view）

  Ø 第八步：前端控制器请求视图解析器去进行视图解析 （根据逻辑视图名解析成真正的视图），通过这种策略很容易更换其他视图技术，只需要更改视图解析器即可

  Ø 第九步：视图解析器向前端控制器返回View

  Ø 第十步：前端控制器进行视图渲染 （将数据(在ModelAndView对象中)填充到request域）

  Ø 第十一步：前端控制器向用户响应结果

# 2.RequestMapping注解

## 2.1.使用说明

- 作用：用于建立请求URL和处理请求方法之间的对应关系。

- 出现位置：
  - 类上：

    请求 URL的第一级访问目录。此处不写的话，就相当于应用的根目录。写的话需要以/开头。它出现的目的是为了使我们的 URL 可以按照模块化管理，例如： 

    账户模块： 

    ​		<font color=red>/account</font>/add 

    ​		<font color=red>/account</font>/update 

    ​		<font color=red>/account</font>/delete ... 

    订单模块： 

    ​		<font color=red>/order</font>/add 

    ​		<font color=red>/order</font>/update 

    ​		<font color=red>/order</font>/delete 

    红色的部分就是把RequsetMappding写在类上，使我们的URL更加精细。

  - 方法上：

    请求URL的第二级访问目录，可以窄化请求路径

- 属性：

  value：用于指定请求的URL。它和path属性的作用是一样的。 

  method：用于指定请求的方式。 

  params：用于指定限制请求参数的条件。它支持简单的表达式。要求请求参数的key和value必须和配置的一模一样。

  例如： params = {"accountName"}，表示请求参数必须有accountName 

  注意：以上四个属性只要出现2个或以上时，他们的关系是与的关系。

## 2.2.窄化路径示例

- 使用二级目录访问

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
      @RequestMapping("/findAccount")
      public ModelAndView findAccount() {
          ModelAndView mv = new ModelAndView();
          mv.addObject("msg", "欢迎你 springmvc");
          mv.setViewName("success");
          return mv;
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount">窄化路径</a>
  ```

## 2.3.method属性示例

- 描述需要使用指定的请求方式来请求该方法

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  	//指定的请求方式
      @RequestMapping(value = "/findAccount1", method = RequestMethod.POST)
      public ModelAndView findAccount1() {
          ModelAndView mv = new ModelAndView();
          mv.addObject("msg", "欢迎你 springmvc");
          mv.setViewName("success");
          return mv;
      }
  }
  ```

- 测试：在index.jsp里使用get方式请求

  ```html
  <a href="/account/findAccount1">请求方式</a>
  ```

  结果：

  <img src="assets/image-20211012201250207.png" alt="image-20211012201250207" style="zoom:67%;" />	

- 我们再换一种请求方式

  ```html
    <form action="account/findAccount1" method="post">
      <input type="submit" value="保存账户，post 请求">
    </form>
  ```

  结果：

  <img src="assets/image-20211012194536051.png" alt="image-20211014162605063" style="zoom:67%;" />		

# 3.controller方法返回值

## 3.1.返回ModelAndView

- 说明：controller方法中定义ModelAndView对象并返回，对象中可添加model数据、指定view

## 3.2.返回字符串

### 3.2.1.逻辑视图名

- 说明：controller方法返回字符串可以指定逻辑视图名，通过视图解析器解析为物理视图地址。

- 返回字符串

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
      @RequestMapping(value = "/findAccount2")
      public String findAccount2(Model model) {
          //添加数据
          model.addAttribute("msg", "欢迎你 springmvc");
          return "success";
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount2">返回字符串</a>
  ```

### 3.2.2.Redirect重定向

- 说明：
  - Contrller方法返回结果重定向到一个url地址，如下商品修改提交后重定向到商品查询方法，参数无法带到商品查询方法中。
  - redirect方式相当于“response.sendRedirect()”，转发后浏览器的地址栏变为转发后的地址，因为转发即执行了一个新的request和response。
  - 由于新发起一个request原来的参数在转发时就不能传递到下一个url，如果要传参数可以/item/queryItem后边加参数，如下：`/item/queryItem?...&…..`

- 重定向

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
      
      @RequestMapping(value = "/findAccount3")
      public String findAccount3() {
          return "redirect:/account/findAccount4";
      }
  
      @RequestMapping(value = "/findAccount4")
      public String findAccount4(Model model) {
          //添加数据
          model.addAttribute("msg", "这是springmvc的重定向");
          return "success";
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount3">重定向</a>
  ```

### 3.2.3.forward转发

- 说明：
  - controller方法执行后继续执行另一个controller方法，如下商品修改提交后转向到商品查询页面，修改商品的id参数可以带到商品查询方法中。
  - forward方式相当于`request.getRequestDispatcher().forward(request,response)`，转发后浏览器地址栏还是原来的地址。转发并没有执行新的request和response，而是和转发前的请求共用一个request和response。所以转发前请求的参数在转发后仍然可以读取到。

- 重定向

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
      
      @RequestMapping(value = "/findAccount3")
      public String findAccount3() {
          return "forward:/account/findAccount4";
      }
  
      @RequestMapping(value = "/findAccount4")
      public String findAccount4(Model model) {
          //添加数据
          model.addAttribute("msg", "这是springmvc的重定向");
          return "success";
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount3">重定向和请求转发</a>
  ```

# 4.参数接收

Springmvc中，接收页面提交的数据是通过方法形参来接收：

- 处理器适配器调用springmvc使用反射将前端提交的参数传递给controller方法的形参

- springmvc接收的参数都是String类型，所以spirngmvc提供了很多converter（转换器）在特殊情况下需要自定义converter，如对日期数据

## 4.1.基本数据类型

- 编写controller

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
      @RequestMapping("/findAccount5")
      public String findAccount5(String username,Model model){
          model.addAttribute("msg", username);
          return "success";
      }
      
      @RequestMapping("/findAccount6")
      public String findAccount6(String username,Integer age,Model model){
          model.addAttribute("msg", username+" "+age);
          return "success";
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount5?username=eric">参数接收-基本数据类型</a>
  <a href="/account/findAccount6?username=eric&age=22">参数接收-多个基本数据类型</a>
  ```

  


## 4.2.POJO类型参数绑定

- 编写pojo

  ```java
  public class Account implements Serializable {
      private Integer id;
      private String name;
      private Float money;
      private Address address;
     //省略get set toString方法
   }
  ```

  ```java
  public class Address implements Serializable {
      private String provinceName;
      private String cityName;
       //省略get set toString方法
   }
  ```

- 编写controller

  ```java
  package com.qf.controller;
  
  import com.qf.pojo.Account;
  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RequestMethod;
  import org.springframework.web.servlet.ModelAndView;
  
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
      @RequestMapping("/saveAccount")
      public String saveAccount(Account account, Model model){
          model.addAttribute("msg", account);
          return "success";
      }
  }
  ```
  
  
  
- 在index.jsp里面添加表单

  ```html
  <form action="account/saveAccount" method="post">
      账户名称：<input type="text" name="name"><br/>
      账户金额：<input type="text" name="money"><br/>
      账户省份：<input type="text" name="address.provinceName"><br/>
      账户城市：<input type="text" name="address.cityName"><br/>
      <input type="submit" value="保存">
  </form>
  ```

## 4.3.restful风格

- restful概述：

(Representational State Transfer，表现层状态转移)：URL定位资源时，用HTTP动词（GET，POST，DELETE，PUT）描述操作

### 4.3.1.restful风格URL

- 在Restful之前的操作：
   http://127.0.0.1/user/query?id=1 根据用户id查询用户数据
   http://127.0.0.1/user/save 新增用户
   http://127.0.0.1/user/update?id=1 修改用户信息
   http://127.0.0.1/user/delete?id=1  删除用户信息
- RESTful用法：
   http://127.0.0.1/user/1 GET  根据用户id查询用户数据
   http://127.0.0.1/user  POST 新增用户
   http://127.0.0.1/user  PUT 修改用户信息
   http://127.0.0.1/user/1  DELETE 删除用户信息

- RESTful总结：

  Restful风格就是请求url统一，根据不同的请求方式，请求不同的后台方法。如果需要携带参数，在url上使用/{}占位符。

### 4.3.2.@PathVaribale

- 作用

  用于绑定url中的占位符。例如：/account/{id}，这个{id}就是url占位符

  url支持占位符是spring3.0之后加入的，是springmvc支持rest风格url的重要标志。

- 描述需要使用指定的请求方式来请求该方法

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
      
  	@RequestMapping(value="/findAccount7/{id}")
      public String findAccount11(@PathVariable Integer id, Model model){
          model.addAttribute("msg", id);
          return "success";
      }
  }
  ```

- 测试：在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount7/123">restful传参</a><br>
  ```




## 4.3.请求参数乱码问题

### 4.3.1.POST请求方式解决乱码问题

- 在web.xml里面设置编码过滤器

  ```xml
  <filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>
      org.springframework.web.filter.CharacterEncodingFilter
    </filter-class>
    <!-- 设置过滤器中的属性值 -->
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
    <!-- 启动过滤器 -->
    <init-param>
      <param-name>forceEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <!-- 过滤所有请求 -->
  <filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

- 测试

  ![image-20211012204316863](assets/image-20211012204316863.png)	

### 4.3.2.GET请求方式解决乱码问题

- tomcat对GET和POST请求处理方式是不同的，GET请求的编码问题，要改tomcat的 配置信息，如下：

  ```xml
  <plugin>
      <groupId>org.apache.tomcat.maven</groupId>
      <artifactId>tomcat7-maven-plugin</artifactId>
      <version>2.2</version>
      <configuration>
          <port>8080</port>
          <path>/</path>
          <!--按UTF-8进行编码-->
          <uriEncoding>UTF-8</uriEncoding>
      </configuration>
  </plugin>
  ```

## 4.4.自定义类型转换器

### 4.4.1.使用场景

- 在index.jsp里面添加日期类型

  ```html
      <form action="account/saveAccount" method="post">
        账户名称：<input type="text" name="name"><br/>
        账户金额：<input type="text" name="money"><br/>
        账户省份：<input type="text" name="address.provinceName"><br/>
        账户城市：<input type="text" name="address.cityName"><br/>
        开户日期：<input type="text" name="date"><br/>
        <input type="submit" value="保存">
      </form>
  ```

- 在pojo里面添加日期类型

  ```java
  public class Account implements Serializable {
      private Integer id;
      private String name;
      private Float money;
      private Address address;
      //添加日期类型
      private Date date;
      //省略get set toString方法
  }    
  ```
  
- 测试

  <img src="assets/image-20211012203811849.png" alt="image-20211013214746211" style="zoom:67%;" />		

  <img src="assets/image-20211012205919985.png" alt="image-20211012205919985" style="zoom:80%;" />	

- 原因

  我们前台传递的是字符串类型的参数，但是后台使用的是Date类型接收的。我们期望springmvc可以帮我们做数据类型的自动转换，显然没有做，所以我们需要自己自定义类型转换器。

### 4.4.2.自定义类型转换器

1. Converter接口说明：

   <img src="assets/image-20211012210055752.png" alt="image-20211012210055752" style="zoom:67%;" />	
   
2. 定义一个类，实现Converter接口

   ```java
   public class DateConverter implements Converter<String, Date> {
       @Override
       public Date convert(String source) {
           try {
               DateFormat format = new SimpleDateFormat("yyyy-MM-dd");
               return format.parse(source);
           } catch (Exception e) {
               e.printStackTrace();
           }
           return null;
       }
   }
   ```

3. 在 springmvc.xml配置文件中配置类型转换器

   ```xml
   	<!--开启springmvc注解支持-->
       <mvc:annotation-driven conversion-service="cs"></mvc:annotation-driven>
       <!-- 配置类型转换器工厂 -->
       <bean id="cs"
             class="org.springframework.context.support.ConversionServiceFactoryBean">
           <!-- 给工厂注入一个新的类型转换器 -->
           <property name="converters">
               <set>
                   <!-- 配置自定义类型转换器 -->
                   <bean class="com.qf.converter.DateConverter"></bean>
               </set>
           </property>
       </bean>
   ```

## 4.5.使用ServletAPI接收参数

- 编写controller

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
      @RequestMapping("/findAccount8")
      public String findAccount8(HttpServletRequest request, 
                                 HttpServletResponse response){
          String username = request.getParameter("name");
          String age = request.getParameter("age");
          request.setAttribute("msg",username+" "+age);
          return "success";
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount8?username=eric&age=19">Servlet接收参数</a>
  ```

# 5.参数传递

## 5.1.ModelAndView传递

- 编写controller

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
      //也可以不创建ModelAndView，直接在参数中指定
      @RequestMapping(value = "/findAccount9")
      public ModelAndView findAccount9(ModelAndView mv) {
          mv.addObject("msg", "欢迎你 springmvc");
          mv.setViewName("success");
          return mv;
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount9">ModelAndView参数传递</a>
  ```

## 5.2.Model传递

- 编写controller

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
      @RequestMapping(value = "/findAccount10")
      public String findAccount10(Model model) {
          model.addAttribute("msg", "欢迎你 springmvc");
          return "success";
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount10">Model参数传递</a>
  ```

## 5.3.ServletAPI传递

- 编写controller

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
      @RequestMapping("/findAccount11")
      public String findAccount11(HttpServletRequest request, 
                                  HttpServletResponse response){
          request.setAttribute("msg","欢迎你 springmvc");
          return "success";
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount11">ServletAPI传递</a>
  ```

# 6.JSON数据处理

## 6.1.添加json依赖

springmvc 默认使用jackson作为json类库,不需要修改applicationContext-servlet.xml任何配置，只需引入以下类库springmvc就可以处理json数据：

```xml
<!--spring-json依赖-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.9.0</version>
</dependency>
```

## 6.2.注解

- @RequestBody：作用是接收前端ajax传递给后端的json字符串，并将json格式的数据转为java对象
- @ResponseBody：作用是将java对象转为json格式的数据传递给前台ajax

## 6.3.案例

- 编写controller

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
      @RequestMapping("/saveAccount2")
      @ResponseBody
      public Map saveAccount2(@RequestBody Account account){
          Map<String, Object> map = new HashMap<String, Object>();
          map.put("status",200);
          map.put("msg",account);
          return map;
      }
  }
  ```

- 在index.jsp里面定义ajax请求

  - 添加按钮

    ```html
    <input type="button" value="测试ajax请求json和响应json" id="testJson"/>
    ```

  - 引入js库文件

    ```html
    <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js"></script>
    ```

  - 编写ajax代码

    ```html
      <script type="text/javascript">
        $(function(){
          $("#testJson").click(function(){
            $.ajax({
              type:"post",
              url:"/account/saveAccount2",
              contentType:"application/json;charset=UTF-8",
              data:'{"id":1,"name":"张二狗","money":999.0}',
              success:function(data){
                if(data.status == 200){
                  alert(data.msg.name);
                  alert(data.msg.money);
                }
              }
            })
          });
        })
      </script>
    ```

- 测试

  ![image-20211013231840953](assets/image-20211013231840953.png)

