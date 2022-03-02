# 学习目标

- Spring概述
- Spring IOC[重点]
- 基于注解的IOC配置
- Spring AOP[重点]
- 基于注解的AOP配置
- Spring整合MyBatis
- Spring事务控制[重点]
- 基于注解的事务控制

# 1.Spring概述

## 1.1.Spring介绍

​		Spring是轻量级Java EE应用开源框架（官网： http://spring.io/ ），它由Rod Johnson创为了解决企业应用开发的复杂性而创建

## 1.2.简化应用开发体现在哪些方面？

1. IOC

   解决传统Web开发中硬编码所造成的程序耦合

2. AOP

   实现在运行期间不修改源代码对程序进行增强

3. 声明式事务

   声明式方式灵活地进行事务的管理

4. 粘合剂

   Spring是一个超级粘合平台，除了自己提供功能外，还提供粘合技术和框架的能力

## 1.3.Spring的体系结构

Spring 框架根据功能的不同大体可分为 `Data Access/Integration（数据访问与集成）`、`Web`、`AOP、Aspects、Instrumentation（检测）、Messaging（消息处理）`、`Core Container（核心容器）`和 `Test`。

![](assets/1031053202-0.png)	

- Core Container： 框架的最基础部分，提供控制反转和依赖注入特性
- AOP ：提供了面向切面的编程的实现
- Data Access/Integration：简化了持久层的操作 
- Web：提供了Spring MVC Web 框架实现以及与Servlet、WebSocket的集成
- Test：方便程序的测试

## 1.4.Spring的发展历程

- 1997年IBM提出了EJB的思想

- 1998年，SUN制定开发标准规范EJB1.0

- 1999年，EJB1.1发布

- 2001年，EJB2.0发布

- 2003年，EJB2.1发布

  **Rod Johnson**（spring之父）

  ​		`Expert One-to-One J2EE Design and Development(2002)`

  ​		阐述了J2EE使用EJB开发设计的优点及解决方案

  ​		`Expert One-to-One J2EE Development without EJB(2004)`

  ​		阐述了J2EE开发不使用EJB的解决方式（Spring雏形）

- 2006年，EJB3.0发布

- 2017年9月发布了Spring的最新版本Spring5.0通用版x

# 2.Spring IOC	

## 2.1.程序的耦合

- 耦合：耦合指的就是对象之间的依赖性。对象之间的耦合越高，维护成本越高。

- 案例：没有引入IOC容器时系统的Web层、业务层、持久层存在耦合

  ```java
  /**
   * 持久层实现类
   */
  public class UserDaoImpl implements UserDao {
  
      @Override
      public void addUser(){
          System.out.println("insert into tb_user......");
      }
  }
  ```

  ```java
  /**
   * 业务层实现类
   */
  public class UserServiceImpl implements UserService {
      //硬编码：此处有依赖关系
      private UserDao userDao = new UserDaoImpl();
  
      public void addUser(){
          userDao.addUser();
      }
  }
  ```

  ```java
  /**
   * 模拟表现层
   */
  public class Client {
      public static void main(String[] args) {
          //硬编码：此处有依赖关系
          UserService userService = new UserServiceImpl();
          userService.addUser();
      }
  }
  ```

- 问题分析：

  上边的代码service层在依赖dao层的实现类，此时如果更改dao了层的实现类或此时没有dao层实现类，编译将不能通过。

- IOC(工厂模式)解耦：
  
  1. 把所有的dao和service对象使用配置文件配置起来
  2. 当服务器启动时读取配置文件
  3. 把这些对象**通过反射创建**出来并**保存在容器中**
  4. 在使用的时候，直接从工厂拿

## 2.2.IOC解决程序耦合

### 2.2.1.创建工程

<img src="assets/Spring.md" alt="image-20220103231029388" style="zoom: 80%;" />	

### 2.2.2.什么是IOC

- IOC (Inverse of Control)即控制反转：是指将原来程序中自己创建实现类对象的控制权反转到IOC容器中，程序只需要从IOC容器获取创建好的对象。

- 原来：

  ​		  我们在获取对象时，都是采用new的方式。是主动的。

  ​		 <img src="assets/wpsE503.tmp.jpg" alt="img" style="zoom:67%;" />

- 现在：

​		  	我们获取对象时，同时跟工厂要，有工厂为我们查找或者创建对象。是被动的。

​				<img src="assets/wps6DCC.tmp.jpg" alt="img" style="zoom:67%;" />

​			这种被动接收的方式获取对象的思想就是控制反转，它是spring框架的核心之一。

### 2.2.3.IOC(工厂模式)解耦

- 案例一

  ```java
  /**
   * bean工厂
   */
  public class BeanFactory {
  
      /**
       * 获得UserServiceImpl对象
       * @return
       */
      public static UserService getUserService(){
          return new UserServiceImpl();
      }
  
      /**
       * 获得UserDaoImpl对象
       * @return
       */
      public static UserDao getUserDao(){
          return new UserDaoImpl();
      }
  }
  ```

  问题：我们在开发中会有很多个service和dao，此时工厂类就要添加无数个方法。

- 案例二

  ```properties
  #1、配置要使用的dao和service
  UserDao=com.qf.dao.UserDaoImpl
  UserService=com.qf.service.UserServiceImpl
  ```

  ```java
  
  /**
   * bean工厂
   */
  public class BeanFactory {
  
      private static Properties prop = new Properties();
  
      /**
       * 根据全类名获取bean对象
       * @param beanName
       * @return
       * @throws ClassNotFoundException
       */
      public static Object getBean(String beanName) {
          try {
              //不能使用：web工程发布后没有src目录
              //InputStream is = new FileInputStream("src/bean.properties");
              InputStream is = 
              BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
              prop.load(is);
              return Class.forName(prop.getProperty(beanName)).newInstance();
          } catch (Exception e) {
              e.printStackTrace();
          }
          return null;
      }
  
      public static void main(String[] args) {
          System.out.println(prop.get("UserService"));
          System.out.println(getBean("UserService"));
      }
  }
  ```
  
  ```java
  /**
   * 业务层实现类
   */
  public class UserServiceImpl implements UserService {
      
      private UserDao userDao = (UserDao) BeanFactory.getBean("UserDao");
  
      public void addUser(){
        userDao.addUser();
      }
  }
  ```
  
  测试：
  
  ```java
  /**
   * 模拟表现层
   */
  public class Client {
      public static void main(String[] args) {
          //直接引用接口实现类
        for (int i = 0; i < 5; i++) {
              UserService userService = 
                (UserService)BeanFactory.getBean("UserService");
              System.out.println(userService);
          }
      }
  }
  ```
  
  <img src="assets/image-20210930122936696.png" alt="image-20210930122936696" style="zoom:67%;" />
  问题：
  
  1. 每次都会创建新的对象
  2. 程序运行时才创建对象(读取配置文件)
  
- 案例三

  ```java
  package com.qf.factory;
  
  import java.io.InputStream;
  import java.util.HashMap;
  import java.util.Map;
  import java.util.Properties;
  import java.util.Set;
  
  /**
   * bean工厂
   */
  public class BeanFactory {
  
      //定义一个容器，用于存放对象
      private static Map<String, Object> beans = new HashMap<>();
  
      /**
       * 加载配置文件
       */
      static {
          try {
              //2、读取配置文件
              //不能使用：web工程发布后没有src目录
              //InputStream is = new FileInputStream("src/bean.properties");
              InputStream is = 
              BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
  
              //3、通过反射创建对象,把对象存到容器中
              Properties prop = new Properties();
              prop.load(is);
              Set<Map.Entry<Object, Object>> entrySet = prop.entrySet();
              for (Map.Entry<Object, Object> entry : entrySet) {
                  String key = entry.getKey().toString();
                  String beanName = entry.getValue().toString();
                  Object value = Class.forName(beanName).newInstance();
                  beans.put(key, value);
              }
  
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  
      /**
       * 4、在使用的时候，直接从工厂拿
       * @param beanName
       * @return
       */
      public static Object getBean(String beanName) {
          try {
              return beans.get(beanName);
          } catch (Exception e) {
              e.printStackTrace();
          }
          return null;
      }
  
      public static void main(String[] args) {
          System.out.println(getBean("UserService"));
      }
  }
  ```

## 2.3.Spring的IOC解决程序耦合

### 2.3.1.创建工程

<img src="assets/image-20210930133749223.png" alt="image-20210930194741140" style="zoom:80%;" />	

#### 2.3.1.1.pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.qf</groupId>
    <artifactId>Spring_IOC_Xml</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <properties>
        <!-- 项目源码及编译输出的编码 -->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <!-- 项目编译JDK版本 -->
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    
    <dependencies>
        <!-- Spring常用依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

注意：Jar包彼此存在依赖，只需引入最外层Jar即可由Maven自动将相关依赖Jar引入到项目中。

|                Spring常用功能的Jar包依赖关系                 |
| :----------------------------------------------------------: |
| <img src="assets/image-20210930132833937.png" alt="image-20210930133222584" style="zoom:67%;" /> |

​	核心容器由 beans、core、context 和 expression（Spring Expression Language，SpEL）4个模块组成。

* spring-beans和spring-core模块是Spring框架的核心模块，包含了控制反转（Inversion of Control，IOC）和依赖注入（Dependency Injection，DI）。BeanFactory使用控制反转对应用程序的配置和依赖性规范与实际的应用程序代码进行了分离。BeanFactory属于延时加载，也就是说在实例化容器对象后并不会自动实例化Bean，只有当Bean被使用时，BeanFactory才会对该 Bean 进行实例化与依赖关系的装配。
* spring-context模块构架于核心模块之上，扩展了BeanFactory，为它添加了Bean生命周期控制、框架事件体系及资源加载透明化等功能。此外，该模块还提供了许多企业级支持，如邮件访问、远程访问、任务调度等，ApplicationContext 是该模块的核心接口，它的超类是 BeanFactory。与BeanFactory不同，ApplicationContext实例化后会自动对所有的单实例Bean进行实例化与依赖关系的装配，使之处于待用状态。
* spring-expression 模块是统一表达式语言（EL）的扩展模块，可以查询、管理运行中的对象，同时也可以方便地调用对象方法，以及操作数组、集合等。它的语法类似于传统EL，但提供了额外的功能，最出色的要数函数调用和简单字符串的模板函数。EL的特性是基于Spring产品的需求而设计的，可以非常方便地同Spring IoC进行交互。

#### 2.3.1.2.dao

```java
/**
 * 持久层实现类
 */
public class UserDaoImpl implements UserDao {

    @Override
    public void addUser(){
        System.out.println("insert into tb_user......");
    }
}
```

#### 2.3.1.3.service

```java
/**
 * 业务层实现类
 */
public class UserServiceImpl implements UserService {
    //此处有依赖关系
    private UserDao userDao = new UserDaoImpl();

    public void addUser(){
        userDao.addUser();
    }
}
```

### 2.3.2.IOC

#### 2.3.2.1.applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--1、注意：要导入schema约束-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <!--
		2、把对象交给spring来创建
       		id：给对象在容器中提供一个唯一标识。用于获取对象	
		   	class：指定类的全限定类名。用于反射创建对象。默认情况下调用无参构造函数
	-->
    <bean id="userDao" class="com.qf.dao.UserDaoImpl"></bean>
    <bean id="userService" class="com.qf.service.UserServiceImpl"></bean>
</beans>
```

注意：命名无限制，约定俗成命名有：spring-context.xml、applicationContext.xml、beans.xml

#### 2.3.2.2.测试

```java
/**
 * 模拟表现层
 */
public class Client {
    public static void main(String[] args) {
        //1.使用ApplicationContext接口，就是在获取spring容器
        ApplicationContext ac = new 
            ClassPathXmlApplicationContext("applicationContext.xml");
        //2.根据bean的id获取对象
        UserDao userDao = (UserDao) ac.getBean("userDao");
        System.out.println(userDao);

        UserService userService = (UserService) ac.getBean("userService");
        System.out.println(userService);
        userService.addUser();
    }
}
```

- 问题：service层仍然耦合

### 2.3.3.DI

概述：DI（Dependency Injection）依赖注入，在Spring创建对象的同时，为其属性赋值，称之为依赖注入。

#### 2.3.3.1.构造函数注入

顾名思义，就是使用类中的构造函数，给成员变量赋值。注意，赋值的操作不是我们自己做的，而是通过配置的方式，让spring框架来为我们注入。具体代码如下：

```java
/**
 * 业务层实现类
 */
public class UserServiceImpl implements UserService {

    private UserDao userDao;
    private String name;
    private Integer age;

    public UserServiceImpl(UserDao userDao, String name, Integer age) {
        this.userDao = userDao;
        this.name = name;
        this.age = age;
    }

    public void addUser(){
        System.out.println(name+","+age);
        userDao.addUser();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--1、注意：要导入schema约束-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--2、把对象交给spring来创建-->
    <bean id="userDao" class="com.qf.dao.UserDaoImpl"></bean>
    <bean id="userService" class="com.qf.service.UserServiceImpl">
        <!--
               要求：类中需要提供一个对应参数列表的构造函数。
               标签：constructor-arg
                       ==给谁赋值:==
				           index:指定参数在构造函数参数列表的索引位置
				           name:指定参数在构造函数中的名称
				       ==赋什么值:==
				           value:它能赋的值是基本数据类型和String类型
				           ref:它能赋的值是其他bean类型，也就是说，必须得是在配置文件中配置过的bean
        -->
        <constructor-arg name="userDao" ref="userDao"></constructor-arg>
        <constructor-arg name="name" value="张三"></constructor-arg>
        <constructor-arg name="age" value="18"></constructor-arg>
    </bean>
</beans>
```

#### 2.3.3.2.set方法注入

顾名思义，就是在类中提供需要注入成员的set方法。具体代码如下：

```java
/**
 * 业务层实现类
 */
public class UserServiceImpl implements UserService {

    private UserDao userDao;
    private String name;
    private Integer age;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public void addUser(){
        System.out.println(name+","+age);
        userDao.addUser();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--1、注意：要导入schema约束-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--2、把对象交给spring来创建-->
    <bean id="userDao" class="com.qf.dao.UserDaoImpl"></bean>
    <bean id="userService" class="com.qf.service.UserServiceImpl">
        <!--
               要求：property
               标签：constructor-arg
                       ==给谁赋值:==
				           name:找的是类中set方法后面的部分
				       ==赋什么值:==
				           value:它能赋的值是基本数据类型和String类型
				           ref:它能赋的值是其他bean类型，也就是说，必须得是在配置文件中配置过的bean
        -->
        <property name="userDao" ref="userDao"></property>
        <property name="name" value="张三"></property>
        <property name="age" value="18"></property>
    </bean>
</beans>
```

#### 2.3.3.3.自动注入

不用在配置中 指定为哪个属性赋值，由spring自动根据某个 "原则" ，在工厂中查找一个bean并为属性注入值。具体代码如下：

```java
/**
 * 业务层实现类
 */
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void addUser(){
        userDao.addUser();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--1、注意：要导入schema约束-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--2、把对象交给spring来创建-->
    <bean id="userDao" class="com.qf.dao.UserDaoImpl"></bean>
        <!--autowire="byType"：按照类型自动注入值-->
    <bean id="userService" class="com.qf.service.UserServiceImpl" autowire="byType">
    </bean>
</beans>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--1、注意：要导入schema约束-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--2、把对象交给spring来创建-->
    <bean id="userDao" class="com.qf.dao.UserDaoImpl"></bean>
    <!--autowire="byType"：按照类型自动注入值-->
    <bean id="userService" class="com.qf.service.UserServiceImpl" autowire="byName">
    </bean>
</beans>
```

#### 2.3.3.4.注入集合类型的属性

顾名思义，就是给类中的集合成员传值，它用的也是set方法注入的方式，只不过变量的数据类型都是集合。我们这里介绍注入数组，List,Set,Map。具体代码如下：

```java
/**
 * 业务层实现类
 */
public class UserServiceImpl implements UserService {

    private UserDao userDao;
    private String[] myStrs;
    private List<String> myList;
    private Set<String> mySet;
    private Map<String,String> myMap;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void setMyStrs(String[] myStrs) {
        this.myStrs = myStrs;
    }

    public void setMyList(List<String> myList) {
        this.myList = myList;
    }

    public void setMySet(Set<String> mySet) {
        this.mySet = mySet;
    }

    public void setMyMap(Map<String, String> myMap) {
        this.myMap = myMap;
    }

    public void addUser(){
        System.out.println(Arrays.toString(myStrs));
        System.out.println(myList);
        System.out.println(mySet);
        System.out.println(myMap);
        userDao.addUser();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--1、注意：要导入schema约束-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--2、把对象交给spring来创建-->
    <bean id="userDao" class="com.qf.dao.UserDaoImpl"></bean>
    <bean id="userService" class="com.qf.service.UserServiceImpl">
        <!--
                要求：property
                标签：constructor-arg
                        ==给谁赋值:==
				            name:找的是类中set方法后面的部分
				        ==赋什么值:==
				            value:它能赋的值是基本数据类型和String类型
				            ref:它能赋的值是其他bean类型，也就是说，必须得是在配置文件中配置过的bean
        -->
        <property name="userDao" ref="userDao"></property>
        <!-- 给mySet集合注入数据 -->
        <property name="mySet">
            <set>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </set>
        </property>
        <!-- 注入array数组数据 -->
        <property name="myArray">
            <array>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </array>
        </property>
        <!-- 注入list集合数据 -->
        <property name="myList">
            <list>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </list>
        </property>
        <!-- 注入Map数据 -->
        <property name="myMap">
            <map>
                <entry key="testA" value="aaa"></entry>
                <entry key="testB" value="bbb"></entry>
            </map>
        </property>
    </bean>
</beans>
```

## 2.4.Spring中的工厂类

### 2.4.1.ApplicationContext

- ApplicationContext的实现类，如下图：

  <img src="assets/wpsD6F7.tmp.jpg" alt="img" style="zoom:67%;" />	
  - ClassPathXmlApplicationContext：加载类路径下 Spring 的配置文件
  - FileSystemXmlApplicationContext：加载本地磁盘下 Spring 的配置文件

### 2.4.2.BeanFactory

- spring中工厂的类结构图

  <img src="assets/wpsB73D.tmp.png" alt="img" style="zoom:67%;" />	

  

- 区别：
  - ApplicationContext：只要一读取配置文件，默认情况下就会创建对象。

    ```java
    /**
     * 业务层实现类
     */
    public class UserServiceImpl implements UserService {
    
        private UserDao userDao;
    
        public UserServiceImpl() {
            System.out.println("UserServiceImpl对象创建了...");
        }
    
        public void setUserDao(UserDao userDao) {
            this.userDao = userDao;
        }
    
        public void addUser(){
            userDao.addUser();
        }
    }
    ```

    ```java
    /**
     * 模拟表现层
     */
    public class Client {
        public static void main(String[] args) {
            new ClassPathXmlApplicationContext("applicationContext.xml");
            System.out.println("Spring IOC容器创建好了");
        }
    }
    ```
  ```
  
  ```
  
- BeanFactory：是在 getBean 的时候才会创建对象。
  
    ```java
    /**
     * 业务层实现类
     */
    public class UserServiceImpl implements UserService {
    
        private UserDao userDao;
    
        public UserServiceImpl() {
            System.out.println("UserServiceImpl对象创建了...");
        }
    
        public void setUserDao(UserDao userDao) {
            this.userDao = userDao;
        }
    
        public void addUser(){
            userDao.addUser();
        }
    }
  ```
  
    ```java
    /**
     * 模拟表现层
     */
    public class Client {
        public static void main(String[] args) {
            new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));
            System.out.println("Spring IOC容器创建好了");
        }
    }
    ```

## 2.5.bean的作用范围

### 2.5.1.概述

- 在Spring中，bean作用域用于确定bean实例应该从哪种类型的Spring容器中返回给调用者。

### 2.5.2.五种作用域

- 目前Spring Bean的作用域或者说范围主要有五种：

  | 作用域      | 说明                                                         |
  | ----------- | ------------------------------------------------------------ |
  | singleton   | 默认值，Bean以单例方式存在spring IoC容器                     |
  | prototype   | 每次从容器中调用Bean时都返回一个新的实例，相当于执行newInstance() |
  | request     | WEB 项目中，Spring 创建一个 Bean 的对象,将对象存入到 request 域中 |
  | session     | WEB 项目中，Spring 创建一个 Bean 的对象,将对象存入到 session 域中 |
  | application | WEB 项目中，Spring 创建一个 Bean 的对象,将对象存入到 ServletContext 域中 |

- 可以通过 `<bean>` 标签的`scope` 属性控制bean的作用范围，其配置方式如下所示：

  ```xml
  <bean id="..." class="..." scope="singleton"/>
  ```

* 需要根据场景决定对象的单例、多例模式

  单例：Service、DAO、SqlSessionFactory（或者是所有的工厂）

  多例：Connection、SqlSession

## 2.6.bean的生命周期

### 2.6.1.单例bean

- 案例

  ```xml
  <bean id="userService" class="com.qf.service.UserServiceImpl"
        			scope="singleton" init-method="init" destroy-method="destroy">
  ```

  ```java
  /**
   * 业务层实现类
   */
  public class UserServiceImpl implements UserService {
  
      private UserDao userDao;
  
      public UserServiceImpl() {
          System.out.println("调用构造方法创建bean...");
      }
  
      public void setUserDao(UserDao userDao) {
          System.out.println("调用set方法注入值...");
          this.userDao = userDao;
      }
  
      public void init(){
          System.out.println("调用init方法初始化bean...");
      }
  
      public void destroy(){
          System.out.println("调用destroy方法销毁bean...");
      }
  
      public void addUser(){
          userDao.addUser();
      }
  }
  ```

  ```java
  /**
   * 模拟表现层
   */
  public class Client {
      public static void main(String[] args) {
          ClassPathXmlApplicationContext ac = 
          	new ClassPathXmlApplicationContext("applicationContext.xml");
          //关闭容器
          ac.close();
      }
  }
  ```
  
- 生命周期：

  > [容器启动]--->构造方法(实例化)--->set方法(注入)--->init方法(初始化)--->[容器关闭]--->destroy方法(销毁)

  <img src="assets/image-20210930192345026.png" alt="image-20210930192345026" style="zoom:67%;" />	

### 2.5.2.多例bean

- 案例

  ```xml
  <bean id="userService" class="com.qf.service.UserServiceImpl"
        			scope="prototype" init-method="init" destroy-method="destroy">
  ```

  ```java
  /**
   * 业务层实现类
   */
  public class UserServiceImpl implements UserService {
  
      private UserDao userDao;
  
      public UserServiceImpl() {
          System.out.println("调用构造方法创建bean...");
      }
  
      public void setUserDao(UserDao userDao) {
          System.out.println("调用set方法注入值...");
          this.userDao = userDao;
      }
  
      public void init(){
          System.out.println("调用init方法初始化bean...");
      }
  
      public void destroy(){
          System.out.println("调用destroy方法销毁bean...");
      }
  
      public void addUser(){
          userDao.addUser();
      }
  }
  ```

  ```java
  /**
   * 模拟表现层
   */
  public class Client {
      public static void main(String[] args) {
          ClassPathXmlApplicationContext ac = 
              	new ClassPathXmlApplicationContext("applicationContext.xml");
          //使用对象
          ac.getBean("userService");
      }
  }
  ```

- 生命周期：

  > [使用对象]---->构造方法(实例化)--->set方法(注入)--->init方法(初始化)--->[JVM垃圾回收]--->destroy方法(销毁)

  <img src="assets/image-20210930193104404.png" style="zoom:67%;" />	



# 3.基于注解的IOC配置

学习基于注解的IOC配置，大家脑海里首先得有一个认知，即注解配置和xml配置要实现的功能都是一样的，都是要降低程序间的耦合。只是配置的形式不一样。

## 3.1.创建工程

<img src="assets/image-20211008113601705.png" alt="image-20211008113601705" style="zoom:80%;" />	

### 3.1.1.pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.qf</groupId>
    <artifactId>Spring_IOC_Annotation</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- Spring常用依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

### 3.1.2.dao

```java
/**
 * 持久层实现类
 */
public class UserDaoImpl implements UserDao {

    @Override
    public void addUser(){
        System.out.println("insert into tb_user......");
    }
}
```

### 3.1.3.service

```java
/**
 * 业务层实现类
 */
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void addUser(){
        userDao.addUser();
    }
}
```

## 3.2.IOC

### 3.2.1.applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
      			http://www.springframework.org/schema/beans/spring-beans.xsd
				http://www.springframework.org/schema/context
      			http://www.springframework.org/schema/context/spring-context.xsd ">

    <!-- 告知spring框架,在读取配置文件，创建容器时扫描包，依据注解创建对象，并存入容器中 -->
    <context:component-scan base-package="com.qf"></context:component-scan>
</beans>
```

### 3.2.2.dao

```java
@Repository
public class UserDaoImpl implements UserDao {
	... ...
}
```

### 3.2.3.service

```java
@Service
public class UserServiceImpl implements UserService {
	... ...
}
```

## 3.3.DI

### 3.3.1.service

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    public void addUser() {
        userDao.addUser();
    }
}
```

### 3.3.2.测试

```java
/**
 * 模拟表现层
 */
public class Client {
    public static void main(String[] args) {
        ApplicationContext ac = 
            new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = ac.getBean("userServiceImpl",UserService.class);
        userService.addUser();
    }
}
```



## 3.3.常用注解

### 3.3.1.用于创建对象的

以下四个注解的作用及属性都是一模一样的，都是针对一个的衍生注解只不过是提供了更加明确的语义化。

#### 3.3.1.1.@Controller

- 作用：

  把资源交给spring来管理，相当于：`<bean id="" class="">`；一般用于表现层。

- 属性：

  value：指定bean的id；如果不指定value属性，默认bean的id是当前类的类名，首字母小写；

#### 3.3.1.2.@Service

- 作用：

  把资源交给spring来管理，相当于：`<bean id="" class="">`；一般用于业务层。

- 属性：

  value：指定bean的id；如果不指定value属性，默认bean的id是当前类的类名，首字母小写；

- 案例

  ```java
  //@Service("userService")声明bean，且id="userServiceImpl"
  @Service//声明bean，且id="userServiceImpl"
  public class UserServiceImpl implements UserService {
   	...   
  }
  ```

#### 3.3.1.3.@Repository  

- 作用：

  把资源交给spring来管理，相当于：`<bean id="" class="">`；一般用于持久层。

- 属性：

  value：指定bean的id；如果不指定value属性，默认bean的id是当前类的类名，首字母小写；

- 案例

  ```java
  //@Repository("userDaoImpl")声明bean，且id="userDaoImpl"
  @Repository//声明bean，且id="userDaoImpl"
  public class UserDaoImpl implements UserDao {
  
      @Override
      public void addUser(){
          System.out.println("insert into tb_user......");
      }
  }
  ```

#### 3.3.1.4.@Component

- 作用：

  把资源交给spring来管理，相当于：`<bean id="" class="">`；通用。

- 属性：

  value：指定bean的id；如果不指定value属性，默认bean的id是当前类的类名，首字母小写；

#### 3.3.1.5.@Scope

- 作用：

  指定bean的作用域范围。

- 属性：

  value：指定范围的值，singleton  prototype request session。

### 3.3.2.用于属性注入的

以下四个注解的作用相当于：`<property name="" ref="">`。

#### 3.3.2.1.@Autowired

- 作用：

  自动按照类型注入。<font color=red>set方法可以省略。</font>

- 案例：

  ```java
  @Service
  public class UserServiceImpl implements UserService {
  
      @Autowired //注入类型为UserDAO的bean
      private UserDao userDao;
  
      public void addUser(){
          userDao.addUser();
      }
  }
  ```

#### 3.3.2.1.@Resource

- 作用：

  自动按照名字注入。<font color=red>set方法可以省略。</font>

- 属性：

​	   name：指定bean的id。

- 案例：

  ```java
  @Service
  public class UserServiceImpl implements UserService {
  
      @Resource(name="userDaoImpl")//注入id=“userDaoImpl”的bean
      private UserDao userDao;
  
      public void addUser(){
          userDao.addUser();
      }
  }
  ```

#### 3.3.2.1.@Value 

- 作用：

  注入基本数据类型和String类型数据的

- 属性：

​	   value：用于指定值

- 案例一

  ```java
  @Service
  public class UserServiceImpl implements UserService {
  
      @Resource(name="userDaoImpl") //注入id=“userDaoImpl”的bean
      private UserDao userDao;
      @Value("张三")//注入String
      private String name;
      @Value("18")//注入Integer
      private Integer age;
  
      public void addUser(){
          System.out.println(name+","+age);
          userDao.addUser();
      }
  }
  ```

- 案例二

  1. 创建config.properties

     ```properties
     name=张三
     age=18
     ```

  2. 加载配置文件

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:context="http://www.springframework.org/schema/context"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
           		http://www.springframework.org/schema/beans/spring-beans.xsd
     			http://www.springframework.org/schema/context
           		http://www.springframework.org/schema/context/spring-context.xsd ">
         <!--加载config.properties-->
         <context:property-placeholder location="config.properties"/>
         <context:component-scan base-package="com.qf"></context:component-scan>
     </beans>
     ```

  3. 注入属性值

     ```java
     @Service
     public class UserServiceImpl implements UserService {
     
         @Autowired
         private UserDao userDao;
         @Value("${name}")//注入String
         private String name;
         @Value("${age}")//注入Integer
         private Integer age;
     
         public void addUser() {
             System.out.println(name+","+age);
             userDao.addUser();
         }
     }
     ```

# 4.Spring AOP

## 4.1.为什么要学习AOP?

- 案例：有一个接口Service有一个insert方法，在insert被调用时打印调用前的毫秒数与调用后的毫秒数，其实现为：

  ```java
  public class UserServiceImpl implements UserService {
  
      private UserDao userDao;
  
      public void setUserDao(UserDao userDao) {
          this.userDao = userDao;
      }
  
      public void addUser(){
          System.out.println("方法开始时间："+new Date());
          userDao.addUser();
          System.out.println("方法结束时间："+new Date());
      }
  }
  ```

- 问题：输出日志的逻辑还是无法复用

## 4.2.AOP概述

AOP：全称是Aspect Oriented Programming即：面向切面编程。

<img src="assets/image-20211008161951302.png" alt="image-20211008161951302" style="zoom:80%;" />

简单的说它就是把我们程序重复的代码抽取出来，在需要执行的时候，使用动态代理的技术，在不修改源码的基础上，对程序进行增强：权限校验,日志记录,性能监控,事务控制.

## 4.3.代理（Proxy）模式

### 4.3.1.创建工程

<img src="assets/image-20220103231449598.png" alt="image-20220103231449598" style="zoom:80%;" />	

### 4.3.2.代理（Proxy）模式介绍

- 作用：通过代理可以控制访问某个对象的方法，在调用这个方法前做前置处理，调用这个方法后做后置处理。（即： AOP的微观实现！）![img](assets/wps43D6.tmp.jpg)

- 核心角色

  - 抽象角色(接口)：定义公共对外方法

  - 真实角色（周杰伦）：实现抽象角色，定义真实角色所要实现的**业务逻辑**，

  - 代理角色（代理人）：实现抽象角色，是真实角色的代理，通过调用真实角色的方法来完成业务逻辑，并可以**附加自己的操作**。

    <img src="assets/wps52A7.tmp.jpg" alt="img" style="zoom: 67%;" />	

### 4.3.3.静态代理

#### 4.3.3.1.抽象角色

```java
package com.qf.proxy.StaticProxy;

public interface Star {
	/**
	 * 面谈
	 */
	void confer();
	/**
	 * 签合同
	 */
	void signContract();
	/**
	 * 订票
	 */
	void bookTicket();
	/**
	 * 唱歌
	 */
	void sing();
	/**
	 * 收钱
	 */
	void collectMoney();
}
```

#### 4.3.3.2.真正角色（周杰伦）

```java
package com.qf.proxy.StaticProxy;

public class RealStar implements Star {

	public void bookTicket() {

	}

	public void collectMoney() {

	}

	public void confer() {

	}

	public void signContract() {

	}

	public void sing() {
		System.out.println("RealStar(周杰伦本人).sing()");
	}
}
```

#### 4.3.3.3.代理角色（经纪人）

```java
package com.usian.proxy.StaticProxy;

public class ProxyStar implements Star {
	
	private Star star;
	
	public ProxyStar(Star star) {
		super();
		this.star = star;
	}

	public void bookTicket() {
		System.out.println("ProxyStar.bookTicket()");
	}

	public void collectMoney() {
		System.out.println("ProxyStar.collectMoney()");
	}

	public void confer() {
		System.out.println("ProxyStar.confer()");
	}

	public void signContract() {
		System.out.println("ProxyStar.signContract()");
	}

	public void sing() {
		star.sing();
	}
}
```

#### 4.3.3.4.测试

```java
package com.qf.proxy.StaticProxy;

public class Client {
	public static void main(String[] args) {
		Star proxy = new ProxyStar(new RealStar());
		
		proxy.confer();
		proxy.signContract();
		proxy.bookTicket();
		proxy.sing();
		
		proxy.collectMoney();
		
	}
}
```

#### 4.3.3.5.静态代理的缺点

1. 代理类和实现类实现了相同的接口，这样就出现了大量的代码重复。
2. 代理对象只服务于一种类型的对象。如果要服务多类型的对象，例如代码是只为UserService类的访问提供了代理，但是还要为其他类如DeptService类提供代理的话，就需要我们再次添加代理DeptService的代理类。

### 4.3.4.jdk动态代理

#### 4.3.4.1.抽象角色

```java
public interface Star {
    /**
     * 唱歌
     */
    void sing();
}
```

#### 4.3.4.2.真正角色（周杰伦）

```java
package com.qf.JdkProxy;

//真实角色(周杰伦)
public class RealStar implements Star {
    //优点：此时代码不再重复

    public void sing() {
        System.out.println("周杰伦：快使用双截棍,哼哼哈嘿....");

    }
}

```

#### 4.3.4.3.代理角色（经纪人）

```java
package com.qf.JdkProxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

//代理类工厂
public class ProxyFactory {

    //优点：此时可以代理任意类型的对象
    //真实角色(周杰伦)
    private Object realObj;

    public ProxyFactory(Object realObj) {
        this.realObj = realObj;
    }

    //获得代理对象
    public Object getProxyObject(){

        /**
         * Proxy：作用创建代理对象
         *      ClassLoader loader：类加载器
         *      Class<?>[] interfaces：真实角色实现的接口，根据接口生成代理类
         *      InvocationHandler h：增强的逻辑，即如何代理(宋吉吉要做的事)
         */
        return Proxy.newProxyInstance(
                realObj.getClass().getClassLoader(),
                realObj.getClass().getInterfaces(),
                new InvocationHandler() {
                    /**
                     *
                     * @param proxy：代理类，一般不用
                     * @param method：要调用的方法
                     * @param args：调用方法时的参数
                     * @return
                     * @throws Throwable
                     */
                    public Object invoke(Object proxy, Method method, Object[] args)
                        											throws Throwable {

                        System.out.println("真正的方法执行前！");
                        System.out.println("面谈，签合同，预付款，订机票");

                        Object result = method.invoke(realObj, args);

                        System.out.println("真正的方法执行后！");
                        System.out.println("收尾款");

                        return result;
                    }
                }
        );
    }
}

```

#### 4.3.4.4.测试

```java
public class Client {
    public static void main(String[] args) {
        //获得代理对象
        Star proxyObject = (Star) new ProxyFactory(new RealStar()).getProxyObject();
        System.out.println(proxyObject.getClass());
        proxyObject.sing();
    }
}
```



### 4.3.5.Cglib动态代理

cglib与动态代理最大的区别就是：

- 使用jdk动态代理的对象必须实现一个接口
- 使用cglib代理的对象则无需实现接口

CGLIB是第三方提供的包，所以需要引入jar包的坐标：

```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>2.2.2</version>
</dependency>
```

如果你已经有spring-core的jar包，则无需引入，因为spring中包含了cglib。

#### 4.3.5.1.真正角色

```java
package com.qf.proxy.CglibProxy;

public class RealStar{

	public void sing() {
		System.out.println("RealStar(周杰伦本人).sing()");
	}
}
```

#### 4.3.5.2.代理角色（经纪人）

```java
package com.qf.proxy.CglibProxy;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

//代理工厂
public class ProxyFactory implements MethodInterceptor {

    //真实角色
    private Object realObj;

    public ProxyFactory(Object realObj) {
        this.realObj = realObj;
    }

    /**'
     * 获得子类代理对象
     * @return
     */
    public Object getProxyObject() {
        //工具类
        Enhancer en = new Enhancer();
        //设置父类
        en.setSuperclass(realObj.getClass());
        //设置回调函数
        en.setCallback(this);
        //创建子类代理对象
        return en.create();
    }
    /*
        在子类中调用父类的方法
            intercept方法参数说明：
                obj ： 代理对象
                method ： 真实对象中的方法的Method实例
                args ： 实际参数
                methodProxy ：代理对象中的方法的method实例
     */
    public Object intercept(Object obj, Method method, Object[] args, 
                            	MethodProxy methodProxy)throws Throwable {
        System.out.println("真正的方法执行前！");
        System.out.println("面谈，签合同，预付款，订机票");

        Object result = method.invoke(realObj, args);

        System.out.println("真正的方法执行后！");
        System.out.println("收尾款");
        return object;
    }
}
```

#### 4.3.5.3.测试

```java
package com.qf.proxy.CglibProxy;

//测试类
public class Client {
    public static void main(String[] args) {
        //获取代理对象
        RealStar proxyObject = 
            (RealStar) new ProxyFactory(new RealStar()).getProxyObject();
        proxyObject.sing();
    }
}
```

## 4.4.AOP相关术语

1. 连接点（joinpoint）

   被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点指的就是被拦截到的方法。

2. **切入点（pointcut）**

   切入点是指我们要对哪些连接点进行拦截的定义

3. **通知（advice）**

   所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类

4. **切面（aspect）**

   是切入点和通知的结合

5. 引介（introduction）

   是一种特殊的通知，在不修改代码的前提下，引介可以在运行期为类动态地添加一些方法或字段

6. 目标对象（Target）

   要代理的目标对象（要增强的类）

7. 织入（weave）

   将增强应用到目标的过程将advice应用到target的过程

8. 代理（Proxy）

   一个类被AOP织入增强之后，就产生一个代理类

<img src="assets/wps9BB8.tmp.jpg" alt="img" style="zoom: 80%;" />

## 4.5.AOP配置

### 4.5.1.创建工程

<img src="assets/image-20211009102413999.png" alt="image-20211009103820786" style="zoom:80%;" />	

#### 4.5.1.1.pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.qf</groupId>
    <artifactId>Spring_AOP_Xml</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- Spring常用依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
         <!--支持切点表达式 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

#### 4.5.1.2.dao

```java
/**
 * 持久层实现类
 */
public class UserDaoImpl implements UserDao {

    @Override
    public void addUser(){
        System.out.println("insert into tb_user......");
    }
}
```

#### 4.5.1.3.service

```java
/**
 * 业务层实现类
 */
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void addUser(){
        userDao.addUser();
    }
}
```

#### 4.5.1.4.applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--注意：添加约束-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       			   http://www.springframework.org/schema/beans/spring-beans.xsd
       			   http://www.springframework.org/schema/aop
       			   http://www.springframework.org/schema/aop/spring-aop.xsd">
    
    <bean id="userDao" class="com.qf.dao.UserDaoImpl"></bean>
    <bean id="userService" class="com.qf.service.UserServiceImpl">
        <property name="userDao" ref="userDao"></property>
    </bean>
</beans>
```

#### 4.5.1.5.测试

```java
/**
 * 模拟表现层
 */
public class Client {
    public static void main(String[] args) {
        ApplicationContext ac = 
            new ClassPathXmlApplicationContext("applicationContext.xml");
        //使用对象
        UserService userService = ac.getBean("userService",UserService.class);
        userService.addUser();
    }
}
```

### 4.5.2.增强

1. 创建增强类

   ```java
   package com.qf.advice;
   
   import org.aspectj.lang.ProceedingJoinPoint;
   
   import java.util.Date;
   
   public class MyLogAdvice {
   
       //前置通知
       public void before(){
           System.out.println("前置通知");
       }
   
       //后置通知【try】
       public void afterReturning(){
           System.out.println("后置通知");
       }
   
       //异常通知【catch】
       public void afterThrowing(){
           System.out.println("异常通知");
       }
   
       //最终通知【finally】
       public void after(){
           System.out.println("最终通知");
       }
   
       //环绕通知
       public void around(ProceedingJoinPoint joinPoint){
           try {
               System.out.println("方法执行前的环绕通知");
               joinPoint.proceed();
               System.out.println("方法执行后的环绕通知");
           } catch (Throwable throwable) {
               throwable.printStackTrace();
           }
       }
   }
   ```

2. 配置增强类

   ```xml
   <!--增强-->
   <bean id="myLogger" class="com.qf.advice.MyLogger"></bean>
   ```

### 4.5.3.切点

1. 切点表达式

   表达式语法：

   ​		`execution([修饰符] 返回值类型 包名.类名.方法名(参数))`

   例如：

   ​		`execution(* com.qf.service.UserService.add(..))`

   ​		`execution(* com.qf.service.UserService.*(..))`

   ​		`execution(* com.usian.service.*.*(..))`

2. 配置切点

   ```xml
   <aop:config>
       <!--切点-->
       <aop:pointcut id="pointcut" expression="execution(* com.qf.service.*.*(..))"/>
   </aop:config>
   ```

### 4.5.4.切面

1. 增强的类型
   - aop:before：用于配置前置通知
   - aop:after-returning：用于配置后置【try】通知，它和异常通知只能有一个执行
   - aop:after-throwing：用于配置异常【catch】通知，它和后置通知只能执行一个
   - aop:after：用于配置最终【finally】通知
   - aop:around：用于配置环绕通知

2. 配置切面

   ```xml
   <!--切面-->
   <aop:aspect ref="myLogger">
       <!-- 用于配置前置通知：指定增强的方法在切入点方法之前执行 
   		method:用于指定通知类中的增强方法名称
   		ponitcut-ref：用于指定切入点
   	-->
   	<aop:before method="before" pointcut-ref="pointcut"/>
       <aop:after-returning method="afterReturning" pointcut-ref="pointcut"/>
       <aop:after-throwing method="afterThrowing" pointcut-ref="pointcut"/>
       <aop:after method="after" pointcut-ref="pointcut"/>
       <aop:around method="around" pointcut-ref="pointcut"/>
   </aop:aspect>
   ```

# 5.基于注解的AOP配置

## 5.1.创建工程

<img src="assets/image-20211013092131513.png" alt="image-20211013092131513" style="zoom:80%;" />	

### 5.1.1.pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.qf</groupId>
    <artifactId>Spring_AOP_Annotation</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- Spring常用依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

### 5.1.2.dao

```java
@Repository
public class UserDaoImpl implements UserDao {

    @Override
    public void addUser(){
        System.out.println("insert into tb_user......");
    }
}
```

### 5.1.3.service

```java

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    public void addUser() {
        userDao.addUser();
    }
}
```

### 5.1.4.applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        		http://www.springframework.org/schema/beans/spring-beans.xsd
        		http://www.springframework.org/schema/aop
        		http://www.springframework.org/schema/aop/spring-aop.xsd
        		http://www.springframework.org/schema/context
        		http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.qf"></context:component-scan>
</beans>
```

### 5.1.5.测试

```java
/**
 * 模拟表现层
 */
public class Client {
    public static void main(String[] args) {
        ApplicationContext ac = 
            new ClassPathXmlApplicationContext("applicationContext.xml");
        //使用对象
        UserService userService = ac.getBean("userServiceImpl",UserService.class);
        userService.addUser();
    }
}
```

## 5.2.增强

### 5.2.1.applicationContext.xml

```xml
<!-- 开启spring对注解AOP的支持 -->
<aop:aspectj-autoproxy/>
```

### 3.2.3.AOP配置

1. 常用注解

   - @Aspect：把当前类声明为切面类
   - @Before：前置通知，可以指定切入点表达式
   - @AfterReturning：后置【try】通知，可以指定切入点表达式
   - @AfterThrowing：异常【catch】通知，可以指定切入点表达式
   - @After：最终【finally】通知，可以指定切入点表达式
   - @Around：环绕通知，可以指定切入点表达式

2. 注解方式实现aop

   ```java
   @Component
   @Aspect
   public class MyLogger {
   
       @Before("execution(* com.qf.service.*.*(..))")
       public void before(){
           System.out.println("方法开始时间："+new Date());
       }
   
       @After("execution(* com.qf.service.*.*(..))")
       public void after(){
           System.out.println("方法结束时间："+new Date());
       }
   }
   ```

# 6.Spring整合MyBatis

## 6.1.创建工程

<img src="assets/image-20211009144037004.png" alt="image-20211009144037004" style="zoom:80%;" />	

### 6.1.1.pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.qf</groupId>
    <artifactId>Spring_MyBatis</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <!-- 项目源码及编译输出的编码 -->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <!-- 项目编译JDK版本 -->
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- Spring常用依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
        <!-- MySql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <!--连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.0</version>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.3</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.3</version>
        </dependency>
        <!--日志-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.26</version>
        </dependency>
        <!--junit-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>

    <build>
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

### 5.1.2.log4j.properties

```properties
log4j.rootLogger=DEBUG,A1

log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=[%t] [%c]-[%p] %m%n
```

### 6.1.3.applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    
    <context:component-scan base-package="com.qf"></context:component-scan>
</beans>
```

## 6.2.配置数据源

### 6.2.1.db.properties

```properties
jdbc.driverClass=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring?useUnicode=true&characterEncoding=UTF-8
jdbc.username=root
jdbc.password=1111
```

### 6.2.2.applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       ">

    <!--加载配置文件-->
	<context:property-placeholder location="classpath:db.properties" />

    <!-- 配置数据源 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          											destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClass}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

## 6.3.整合MyBatis

### 6.3.1.applicationContext.xml

```xml
<!--会话工厂-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
    <property name="typeAliasesPackage" value="com.qf.pojo"></property>
</bean>

<!--扫描basePackage所指定的包下的所有接口，生成代理类并交给spring管理-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!--mapper所在的包-->
    <property name="basePackage" value="com.qf.mapper"></property>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
</bean>
```

## 6.4.测试

### 6.4.1.创建表

```sql
CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `money` float DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=14 DEFAULT CHARSET=utf8;
```

### 6.4.2.pojo

```java
package com.qf.pojo;

public class User {
    private Integer id;
    private String name;
    private Float money;

    public User(String name, Float money) {
        this.name = name;
        this.money = money;
    }

    public User() {
    }

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

    public Float getMoney() {
        return money;
    }

    public void setMoney(Float money) {
        this.money = money;
    }
}
```

### 6.4.3.mapper

```java
public interface UserMapper {

    public void addUser(User user);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.qf.mapper.UserMapper">
    <insert id="addUser" parameterType="User">
		insert into t_user(name,money) values(#{name},#{money})
	</insert>
</mapper>
```

### 6.4.4.service

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper userMapper;

    @Override
    public void addUser(User user) {
        userMapper.addUser(user);
    }
}
```

### 6.4.5.junit

```java
package com.qf.test;

import com.qf.pojo.User;
import com.qf.service.UserService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")//加载配置文件
public class ServiceTest {
    
    @Autowired
    private UserService userService;

    @Test
    public void testAdd(){
        userService.addUser(new User("张三丰",4000F));

        userService.addUser(new User("宋远桥",2000F));
    }
}
```

# 7.Spring事务控制

## 7.1.事务介绍

### 7.1.1.什么是事务？

当你需要一次执行多条SQL语句时，可以使用事务。通俗一点说，如果这几条SQL语句全部执行成功，则才对数据库进行一次更新，如果有一条SQL语句执行失败，则这几条SQL语句全部不进行执行，这个时候需要用到事务。

<font color=red>刘德华《无间道》：去不了终点，回到原点</font>

回顾一下数据库事务的四大特性ACID：

```tex
原子性（Atomicity）
       要么都执行，要么都不执行

一致性（Consistency）
       事务前后的数据都是正确的

隔离性（Isolation）
      事物之间相互隔离，互不干扰（并发执行的事务彼此无法看到对方的中间状态）

持久性（Durability）
       事务一旦提交不可再回滚 
```

### 7.1.2.数据库本身控制事物

```mysql
begin transaction；
      //1.本地数据库操作：张三减少金额
      //2.本地数据库操作：李四增加金额
rollback;
或
commit transation;
```

### 7.2.3.jdbc中使用事物

 **1.获取对数据库的连接**

 **2.设置事务不自动提交（默认情况是自动提交的）** 

```java
conn.setAutoCommit(false);   //其中conn是第一步获取的随数据库的连接对象。
```

 **3.把想要一次性提交的几个sql语句用事务进行提交**

```java
try{
    Statement stmt = null; 
    stmt =conn.createStatement(); 
    stmt.executeUpdate(sql1); 
    int a=6/0;
    stmt.executeUpdate(Sql2); 
    . 
    . 
    . 
    conn.commit();   //使用commit提交事务 
}
```

 **4.捕获异常，进行数据的回滚（回滚一般写在catch块中）** 

```java
catch（Exception e） { 
   ... 
   conn.rollback(); 
}
```

## 7.2.转账案例

### 7.2.1.拷贝上一章代码

![image-20211010141139368](assets/image-20211010140920195.png)	

### 7.2.2.添加转账业务

#### 7.2.2.1.mapper

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.qf.mapper.UserMapper">
    ... ...

	<!--转账-->
	<update id="updateUserOfSub">
		update t_user set money=money-#{money} where name=#{source}
	</update>

	<update id="updateUserOfAdd">
		update t_user set money=money+#{money} where name=#{target}
	</update>
</mapper>
```

```java
public interface UserMapper {

    ... ...

    /**扣钱*/
    void updateUserOfSub(@Param("source") String source, @Param("money") Float money);
    /*加钱*/
    void updateUserOfAdd(@Param("target") String target, @Param("money") Float money);
}
```

#### 7.2.2.2.service

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper userMapper;

    /**
     * 转账
     * @param source
     * @param target
     * @param money
     */
    @Override
    public void updateUser(String source, String target, Float money) {
        userMapper.updateUserOfSub(source, money);
        int a = 6/0;
        userMapper.updateUserOfAdd(target, money);
    }
}
```

#### 7.2.2.3.测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")//加载配置文件
public class ServiceTest {
    @Autowired
    private UserService userService;
    /**
     * 转账业务
     */
    @Test
    public void testUpdate(){
        userService.updateUser("张三丰","宋远桥",1F);
    }
}
```

1. 此时我们观察数据表里面的变化情况：

   ![image-20211010144307652](assets/image-20211010144307652.png)	

   转账是成功的，但是涉及到业务的问题，如果业务层实现类有其中一个环节出问题，都会导致灾难。

2. 我们先把数据恢复到转账前。

   ![image-20211010144654927](assets/image-20211010144654927.png)	

   现在我们故意模拟转账业务出现问题

   ![image-20211010144944425](assets/image-20211010144851990.png)

   再来测试：

   <img src="assets/image-20211010144529151.png" alt="image-20211010144529151" style="zoom:80%;" />

   业务执行出错，但是！

   ![image-20211010144606528](assets/image-20211010144606528.png)	

   这是因为：不满足事务的一致性（减钱的事务提交了，加钱的事务没有提交，甚至都没有执行到）。



## 7.2.Spring中事务控制的API介绍

- 说明：
  - JavaEE体系进行分层开发，事务处理位于业务层，Spring提供了分层设计**业务层**的事务处理解决方案。

  - Spring框架为我们提供了一组事务控制的接口。具体在后面的小节介绍。这组接口是在spring-tx.RELEASE.jar中。

  - spring的事务控制都是基于AOP的，它既可以使用编程的方式实现，也可以使用配置的方式实现。我们学习的重点是使用配置的方式实现。


### 7.2.1.PlatformTransactionManager

- 此接口是spring的事务管理器，它里面提供了我们常用的操作事务的方法，源代码如下：

  ```java
  public interface PlatformTransactionManager { 
      
    	//开启事务  
      TransactionStatus getTransaction(TransactionDefinition definition) 
          						throws TransactionException; 
      //提交事务
      void commit(TransactionStatus status) throws TransactionException; 
      
    	//回滚事务
      void rollback(TransactionStatus status) throws TransactionException;   
  } 
  ```

- 真正管理事务的对象

  ![img](assets/b137cf23-1e25-312f-a568-81ae400f8ebd.png)	

  Spring为不同的orm框架提供了不同的PlatformTransactionManager接口实现类：

  - DataSourceTransactionManager：使用Spring JDBC或iBatis 进行持久化数据时使用
  - HibernateTransactionManager：使用Hibernate版本进行持久化数据时使用

### 7.2.2.TransactionDefinition 

- TransactionDefinition接口包含与事务属性相关的方法，源代码如下：

  ```java
  public interface TransactionDefinition {
      int PROPAGATION_REQUIRED = 0;
      int PROPAGATION_SUPPORTS = 1;
      int PROPAGATION_MANDATORY = 2;
      int PROPAGATION_REQUIRES_NEW = 3;
      int PROPAGATION_NOT_SUPPORTED = 4;
      int PROPAGATION_NEVER = 5;
      int PROPAGATION_NESTED = 6;
      int ISOLATION_DEFAULT = -1;
      int ISOLATION_READ_UNCOMMITTED = 1;
      int ISOLATION_READ_COMMITTED = 2;
      int ISOLATION_REPEATABLE_READ = 4;
      int ISOLATION_SERIALIZABLE = 8;
      int TIMEOUT_DEFAULT = -1;
      
      //传播行为
      int getPropagationBehavior();
  	//隔离级别
      int getIsolationLevel();
  
      int getTimeout();
  
      boolean isReadOnly();
  }
  
  ```
  
  TransactionDefinition 接口定义的事务规则包括：**事务隔离级别、事务传播行为、事务超时、事务的只读、回滚规则**属性，同时，Spring 还为我们提供了一个默认的实现类：DefaultTransactionDefinition，该类适用于大多数情况。如果该类不能满足需求，可以通过实现 TransactionDefinition 接口来实现自己的事务定义。

#### 7.2.2.1.事务隔离级别

- 事务并发时的安全问题

  | 问题       | 描述                                           | 隔离级别        |
  | ---------- | ---------------------------------------------- | --------------- |
  | 脏读       | 一个事务读取到另一个事务还未提交的数据         | read-commited   |
  | 不可重复读 | 一个事务内多次读取一行数据的内容，其结果不一致 | repeatable-read |
  | 幻读       | 一个事务内多次读取一张表中的内容，其结果不一致 | serialized-read |

- Spring事务隔离级别（比数据库事务隔离级别多一个default）由低到高为：

  | 隔离级别                   |                                                              |
  | -------------------------- | ------------------------------------------------------------ |
  | ISOLATION_DEFAULT          | 这是一个platfromtransactionmanager默认的隔离级别，使用数据库默认的事务隔离级别。 |
  | ISOLATION_READ_UNCOMMITTED | 这是事务最低的隔离级别，会产生脏读，不可重复读和幻像读。     |
  | ISOLATION_READ_COMMITTED   | 这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读。 Oracle数据库默认的隔离级别。 |
  | ISOLATION_REPEATABLE_READ  | 这种事务隔离级别可以防止脏读、不可重复读，但是可能出现幻像读。MySQL数据库默认的隔离级别。 |
  | ISOLATION_SERIALIZABLE     | 这是花费最高代价但是最可靠的事务隔离级别，事务被处理为顺序执行。除了防止脏读、不可重复读外，还避免了幻像读。 |

#### 7.2.2.2.事务的传播行为

- 什么是事务传播行为？

  事务传播行为（propagation behavior）指的就是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行。 
  例如：methodA事务方法调用methodB事务方法时，methodB是继续在调用者methodA的事务中运行呢，还是为自己开启一个新事务运行，这就是由methodB的事务传播行为决定的。

- Spring定义了七种传播行为：

  | 事务传播行为类型          | 说明                                                         |
  | ------------------------- | ------------------------------------------------------------ |
  | PROPAGATION_REQUIRED      | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。**这是最常见的选择**。 |
  | PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
  | PROPAGATION_MANDATORY     | 使用当前的事务，如果当前没有事务，就抛出异常。               |
  | PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起。                 |
  | PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。   |
  | PROPAGATION_NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常。             |
  | PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与REQUIRED类似的操作。 |

#### 7.2.2.3.事务超时

- `timeout`事务超时时间： 当前事务所需操作的数据被其他事务占用，则等待。
  - 100：自定义等待时间100（秒）。
  - -1：由数据库指定等待时间，默认值。（建议） 

#### 7.2.2.4.读写性

- `readonly` 读写性
  - true：只读，可提高查询效率，适合查询

  - false：可读可写，适合增删改

#### 7.2.2.5.回滚规则

- `rollback-for`  回滚规则，可省略或设置 rollback-for="Exception"
  - 如果事务中抛出 RuntimeException,则自动回滚

  - 如果事务中抛出 CheckException，不会自动回滚


### 7.3.3.TransactionStatus 

- PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象，该对象代表一个新的或已经存在的事务，源代码如下：

  ```java
  public  interface TransactionStatus{
     boolean isNewTransaction();
     void setRollbackOnly();
     boolean isRollbackOnly();
  }
  ```

## 7.4.改造转账案例

### 7.4.1.applicationContext.xml

```xml
 	<!--配置事物管理器-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--配置事物属性-->
    <bean class="org.springframework.transaction.support.DefaultTransactionDefinition">
        <property name="propagationBehaviorName" value="PROPAGATION_REQUIRED"/>
        <property name="readOnly" value="false"></property>
    </bean>
```

### 7.4.2.service

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper userMapper;
    @Autowired
    private TransactionDefinition txDefinition;
    @Autowired
    private PlatformTransactionManager txManager;
    /**
     * 转账
     * @param source
     * @param target
     * @param money
     */
    @Override
    public void updateUser(String source, String target, Float money) {
        // 获取一个事务
        TransactionStatus txStatus = txManager.getTransaction(txDefinition);
        try {

            userMapper.updateUserOfSub(source, money);
            int a = 6/0;
            userMapper.updateUserOfAdd(target, money);
            //提交事务
            txManager.commit(txStatus);
        }catch (Exception e){
            //回滚事务
            txManager.rollback(txStatus);
            e.printStackTrace();
        }
    }
}
```

### 7.4.3.测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")//加载配置文件
public class ServiceTest {
    @Autowired
    private UserService userService;
    /**
     * 转账业务
     */
    @Test
    public void testUpdate(){
        userService.updateUser("张三丰","宋远桥",1F);
    }
}
```

- 事务回滚：

  ![image-20211010181201057](assets/image-20211010181201057.png)

- 满足执行：

  ![image-20211010144654927](assets/image-20211010144654927.png)	

- 我们现在虽然实现了事务控制，但是代码非常的臃肿，我们可以使用动态代理简化代码

## 7.3.动态代理控制事务

### 7.3.1.factory

我们创建一个工厂，专门用来给 Service 创建代理对象，如下：

```java
package com.qf.factory;

import com.qf.service.UserService;
import com.qf.service.UserServiceImpl;
import org.hamcrest.Factory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * bean工厂
 */
@Component
public class BeanFactory {
    @Autowired
    private UserService userService;
    @Autowired
    private TransactionDefinition txDefinition;
    @Autowired
    private PlatformTransactionManager txManager;

    /**
     * 获得UserServiceImpl对象
     *
     * @return
     */
    public UserService getUserService() {
        return (UserService) Proxy.newProxyInstance(
            userService.getClass().getClassLoader(),
            userService.getClass().getInterfaces(),
            new InvocationHandler() {
                    public Object invoke(Object proxy, Method method, Object[] args)
                        											throws Throwable {
                        //开启事务
                        TransactionStatus txStatus = 
                            txManager.getTransaction(txDefinition);
                        try {
                            method.invoke(userService, args);
                            //提交事务
                            txManager.commit(txStatus);
                        } catch (Exception e) {
                            //回滚事务
                            txManager.rollback(txStatus);
                            e.printStackTrace();
                        }
                        return null;
                    }
                });
    }
}
```

### 7.3.2.applicationContext.xml

```xml
<!--配置service代理对象-->
<bean id="proxyService" factory-bean="beanFactory" factory-method="getUserService"></bean>
```

### 7.2.3.service

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper userMapper;

    /**
     * 转账
     * @param source
     * @param target
     * @param money
     */
    @Override
    public void updateUser(String source, String target, Float money) {
        userMapper.updateUserOfSub(source, money);
        int a = 6/0;
        userMapper.updateUserOfAdd(target, money);
    }
}
```

### 7.2.4.测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class ServiceTest {
    @Autowired
    @Qualifier("proxyService")//注入代理对象
    private UserService userService;

    @Test
    public void testUpdate(){
        userService.updateUser("张三丰","宋远桥",1F);
    }
}
```

- 事务回滚：

![image-20211010181201057](assets/image-20211010181201057.png)

## 7.4.Spring AOP控制事务

### 7.4.1.导入schema约束

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
    
    <!--配置事物属性
    <bean class="org.springframework.transaction.support.DefaultTransactionDefinition">
        <property name="propagationBehaviorName" value="PROPAGATION_REQUIRED"/>
        <property name="readOnly" value="false"></property>
    </bean>

    配置service代理对象
    <bean id="proxyService" factory-bean="beanFactory" factory-method="getUserService"> 
    </bean>-->
</beans>    
```

### 7.4.2.配置增强

```xml
    <!-- 1、增强 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <!--事务属性-->
        <tx:attributes>
        <!-- 指定方法名称：是业务核心方法
            read-only：是否是只读事务。默认false，不只读。
            isolation：指定事务的隔离级别。默认值是使用数据库的默认隔离级别。
            propagation：指定事务的传播行为。
            timeout：指定超时时间。默认值为：-1。永不超时。
            rollback-for：用于指定一个异常，当执行产生该异常时，事务回滚。产生其他异常，事务不回滚。
						 省略时任何异常都回滚。
            -->
        <tx:method name="*" read-only="false" propagation="REQUIRED"/>
        <tx:method name="select*" read-only="true" propagation="SUPPORTS"/>
        <tx:method name="get*" read-only="true" propagation="SUPPORTS"/>
       </tx:attributes>
    </tx:advice>
```

### 7.4.3.配置切点

```xml
    <aop:config>
        <!--2、切点-->
        <aop:pointcut expression="execution(* com.qf.service.*.*(..))" id="pointcut"/>
    </aop:config>
```

### 7.4.4.配置切面

```xml
    <aop:config>
        <!--3、切面-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"></aop:advisor>
    </aop:config>
```

### 7.4.5.factory

删除bean工程

### 7.4.6.测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")//加载配置文件
public class ServiceTest {
    @Autowired
    private UserService userService;
    /**
     * 转账业务
     */
    @Test
    public void testUpdate(){
        userService.updateUser("张三丰","宋远桥",1F);
    }
}
```

- 事务回滚：

  ![image-20211010181201057](assets/image-20211010181201057.png)

- 问题一：如果我们把方法名称改了，那么事务还会回滚吗？

  - 修改applicationContext.xml：

  <img src="assets/image-20211010193859811.png" alt="image-20211010193859811" style="zoom: 67%;" />	

  - 修改service：

  <img src="assets/image-20211010193749978.png" alt="image-20211010193749978" style="zoom: 67%;" />	

- 问题二：如果我们把异常捕捉了，那么事务还会回滚吗？

  <img src="assets/image-20211010192502117.png" alt="image-20211010193526921" style="zoom:67%;" />	

# 8.基于注解的AOP控制事务

## 8.1.拷贝上一章代码

<img src="assets/image-20211010191228365.png" alt="image-20211010191228365" style="zoom:80%;" />	

## 8.2.applicationContext.xml

```xml
<!-- 开启spring对注解事务的支持 -->
<tx:annotation-driven transaction-manager="transactionManager"/> 
```

## 8.3.service

```java
@Service
@Transactional(readOnly=true,propagation= Propagation.SUPPORTS)
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper userMapper;
    /**
     * 转账
     * @param source
     * @param target
     * @param money
     */
    @Override
    @Transactional(readOnly=false,propagation=Propagation.REQUIRED)
    public void updateUser(String source, String target, Float money) {
        userMapper.updateUserOfSub(source, money);
        int a = 6/0;
        userMapper.updateUserOfAdd(target, money);
    }
}
```

## 8.4.测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")//加载配置文件
public class ServiceTest {
    @Autowired
    private UserService userService;
    /**
     * 转账业务
     */
    @Test
    public void testUpdate(){
        userService.updateUser("张三丰","宋远桥",1F);
    }
}
```

- 事务回滚：

  ![image-20211010181201057](assets/image-20211010181201057.png)



