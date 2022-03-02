# 学习目标

- 拦截器
- 文件上传
- 异常处理器
- ssm整合

# 1.拦截器

## 1.1.什么是拦截器？

​		Spring MVC中的拦截器（Interceptor）类似于Servlet中的过滤器（Filter），它主要用于拦截用户请求并作相应的处理。例如通过拦截器可以进行权限验证、判断用户是否登录等。

​		拦截器依赖于web框架，在实现上基于Java的反射机制，属于面向切面编程（AOP）的一种运用。

## 1.2.自定义拦截器

### 1.2.1.创建拦截器

```java
package com.qf.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class CustomHandlerInterceptor implements HandlerInterceptor {
    /**
     * 在控制器方法调用前执行
     * 返回值为是否中断
     *      true：表示继续执行（下一个拦截器或处理器）
     *      false：则会中断后续的所有操作，所以我们需要使用response来继续响应后续请求
     */
    @Override
    public boolean preHandle(HttpServletRequest request,
                      HttpServletResponse response, Object object) throws Exception {
        System.out.println("HandlerInterceptor preHandle ....");
        return true;
    }

    /**
     * 在控制器方法调用后，解析视图前调用，我们可以对视图和模型做进一步渲染或修改
     * 可在modelAndView中加入数据，比如当前时间
     */
    @Override
    public void postHandle(HttpServletRequest request,HttpServletResponse response, 
                         Object object, ModelAndView modelAndView) throws Exception {
        System.out.println("HandlerInterceptor postHandle ....");
    }

    /**
     * 整个请求完成，即视图渲染结束后调用，这个时候可以做些资源清理工作，或日志记录
     */
    @Override
    public void afterCompletion(HttpServletRequest request,
        HttpServletResponse response,Object object, Exception e) throws Exception {
        System.out.println("HandlerInterceptor afterCompletion ....");
    }
}
```

### 1.2.2.配置拦截器

```xml
    <!--配置拦截器 -->
    <mvc:interceptors>
        <mvc:interceptor>
            <!-- 匹配的是url路径 -->
            <mvc:mapping path="/**"></mvc:mapping>
            <bean class="com.qf.interceptor.CustomHandlerInterceptor"></bean>
        </mvc:interceptor>
    </mvc:interceptors>
```

### 1.2.3.测试

- 编写controller

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
      @RequestMapping("/findAccount12")
      public String findAccount12(Model model) {
          model.addAttribute("msg", "欢迎你 springmvc");
          System.out.println("controller的方法执行了......");
          return "success";
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount12">拦截器</a>
  ```

## 1.3.登录拦截器

### 1.3.1.创建拦截器

```java
package com.qf.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request,
           HttpServletResponse response, Object object) throws Exception {
        String user = (String) request.getSession().getAttribute("user_session");
        if (user != null) {//已经登录，继续执行
            System.out.println("获得用户信息："+user);
            return true;
        } else {//未登录，跳转到登录页面
            response.sendRedirect(request.getContextPath() +"/login.jsp");
            return false;
        }
    }
}
```

### 1.3.2.配置拦截器

```xml
	<mvc:interceptors>
        <mvc:interceptor>
            <!-- 匹配的是url路径 -->
            <mvc:mapping path="/**"></mvc:mapping>
            <bean class="com.qf.interceptor.LoginInterceptor"></bean>
        </mvc:interceptor>
	</mvc:interceptors>
```

### 1.3.3.测试

- 编写controller

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
      @RequestMapping("/findAccount13")
      public String findAccount13(Model model) {
          model.addAttribute("msg", "欢迎你 springmvc");
          System.out.println("controller的方法执行了......");
          return "success";
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <%
     session.setAttribute("user_session","User{name:张二狗,age:18}");
  %>
  <a href="/account/findAccount13">登录拦截器</a>
  ```

# 2.文件上传

## 2.1.添加依赖

```xml
<!--文件上传-->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
```

## 2.2.配置文件上传解析器

```xml
<!--配置文件上传解析器-->
<bean id="multipartResolver" 
      class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="5242880" />
    <property name="defaultEncoding" value="UTF-8" />
</bean>
```

## 2.3.测试

- 编写controller

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
      @RequestMapping(path="/upload")
      public String upload(HttpServletRequest request, 
                               MultipartFile upload,Model model) throws IOException {
          System.out.println("springmvc方式的文件上传");
          //获取要上传的文件目录
          String path = 
              request.getSession().getServletContext().getRealPath("/uploads");
          System.out.println("path:"+path);
          //根据文件上传的目录创建File对象，如果不存在则创建1个File对象
          File file = new File(path);
          if(!file.exists()){
              //创建一个file对象
              file.mkdirs();
          }
          //获取文件上传名称
          String filename = upload.getOriginalFilename();
          //完成文件上传
          upload.transferTo(new File(path,filename));
  
          model.addAttribute("msg", "欢迎你 springmvc");
          return "success";
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
      <form action="/account/upload" method="post" enctype="multipart/form-data">
          文件: <input type="file" name="upload"></input>
          <input type="submit" value="提交">
      </form>
  ```

# 3.异常处理器

如果不加以异常处理，错误信息肯定会抛在浏览器页面上，这样很不友好，所以必须进行异常处理。

## 3.1.异常处理思路

系统的dao、service、controller出现都通过throws Exception向上抛出，最后由springmvc前端控制器交由异常处理器进行异常处理，如下图：

<img src="assets/1587697450668.png" alt="1587697450668" style="zoom: 80%;" />	

## 3.2.创建异常处理器

```java
@Component
public class CustomExceptionResolver implements HandlerExceptionResolver {

	@Override
	public ModelAndView resolveException(HttpServletRequest request,
					HttpServletResponse response, Object handler, Exception ex) {
		ModelAndView modelAndView = new ModelAndView();
		modelAndView.addObject("message", ex.getMessage());
		modelAndView.setViewName("error");
		return modelAndView;
	}

}
```

## 3.3.测试

- 编写controller

  ```java
  @Controller
  @RequestMapping("/account")
  public class AccountController {
  
     @RequestMapping("/findAccount14")
      public String findAccount14(Model model) {
          model.addAttribute("msg", "欢迎你 springmvc");
          //模拟异常信息
          int i = 10/0;
          return "success";
      }
  }
  ```

- 在index.jsp里面定义超链接

  ```html
  <a href="/account/findAccount14">异常处理器</a>
  ```

# 4.ssm整合

## 4.1.创建表

```sql
CREATE TABLE `account` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  `money` double DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 4.2.创建工程

<img src="assets/image-20211014224425949.png" alt="image-20211014224425949" style="zoom:67%;" />	

## 4.3.pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.qf</groupId>
    <artifactId>ssm</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <spring.version>5.1.8.RELEASE</spring.version>
        <slf4j.version>1.6.6</slf4j.version>
        <log4j.version>1.2.12</log4j.version>
        <mysql.version>5.1.47</mysql.version>
        <mybatis.version>3.4.5</mybatis.version>
        <druid.version>1.1.0</druid.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
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

        <dependency>
            <groupId>jstl</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <!-- log start -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>        
        <!-- log end -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.0</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.0</version>
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
                    <!--按UTF-8进行编码-->
                    <uriEncoding>UTF-8</uriEncoding>
                </configuration>
            </plugin>
            <!-- java编译插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
        </resources>
    </build>
</project>
```

## 4.4.log4j.properties

```properties
log4j.rootLogger=DEBUG,A1

log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} - %c%l%m%n
```



## 4.5.db.properties

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ssm?characterEncoding=utf-8
jdbc.username=root
jdbc.password=1111
```

## 4.6.applicationContext-dao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- 加载配置文件 -->
    <context:property-placeholder location="classpath:db.properties" />
    <!-- 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
        <property name="driverClassName" value="${jdbc.driver}" />
        <property name="maxActive" value="10" />
        <property name="minIdle" value="5" />
    </bean>
    <!-- 让spring管理sqlsessionfactory 使用mybatis和spring整合包中的 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 数据库连接池 -->
        <property name="dataSource" ref="dataSource" />
        <!-- 别名-->
        <property name="typeAliasesPackage" value="com.qf.pojo"></property>
    </bean>
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.qf.mapper"></property>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
    </bean>
</beans>
```

## 4.7.applicationContext-tx.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- 事务管理器 -->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 数据源 -->
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- 通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!-- 传播行为 -->
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>
    <!-- 切面 -->
    <aop:config>
        <aop:advisor advice-ref="txAdvice"
                     pointcut="execution(* com.qf.service.*.*(..))" />
    </aop:config>
</beans>
```

## 4.8.applicationContext-service.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- 扫描service的包 -->
    <context:component-scan base-package="com.qf.service"></context:component-scan>
</beans>
```

## 4.9.springmvc.xml

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
    <context:component-scan base-package="com.qf.controller"></context:component-scan>

    <!-- 配置视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```

## 4.10.web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--配置Spring的监听器 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext-*.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- 解决post乱码 -->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>


    <!-- springmvc的前端控制器 -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!-- 拦截/，例如：/user/add  -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

## 4.11.pojo

```java
public class Account {
    private Integer id;
    private String name;
    private Double money;
    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Double getMoney() {
        return money;
    }

    public void setMoney(Double money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", money=" + money +
                '}';
    }
}
```

## 4.12.mapper

```java
public interface AccountMapper {
    List<Account> selectAccount();
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.qf.mapper.AccountMapper">
    <select id="selectAccount" resultType="Account">
		select * from account
	</select>
</mapper>
```

## 4.13.service

```java
@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountMapper accountMapper;

    @Override
    public List<Account> selectAccount() {
        return accountMapper.selectAccount();
    }
}
```

## 4.14.controller

```java
@Controller
@RequestMapping("/account")
public class AccountController {

    @Autowired
    private AccountService accountService;

    @RequestMapping("/selectAccount")
    public String selectAccount(Model model){
        List<Account> list = accountService.selectAccount();
        model.addAttribute("list",list);
        return "select_account";
    }
}
```

## 4.15.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h2>查询所有账户</h2>
<table width="30%" border="1" cellspacing="0" cellpadding="0">
    <tr>
        <th>id</th>
        <th>name</th>
        <th>money</th>
    </tr>
    <c:forEach var="list" items="${list}">
        <tr>
            <td>${list.id}</td>
            <td>${list.name}</td>
            <td>${list.money}</td>
        </tr>
    </c:forEach>
</table>
</body>
</html>
```

## 4.16.测试

<img src="assets/image-20211014230119111.png" alt="image-20211014230119111" style="zoom:67%;" />	