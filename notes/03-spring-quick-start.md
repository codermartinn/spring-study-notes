## 03.spring-quick-start



### 创建模块

![image-20221113103810345](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221113103810345.png)



### 配置pom.xml

添加spring context的依赖，pom.xml配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codermartinn</groupId>
    <artifactId>spring-02-bean</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!--配置仓库-->
    <repositories>
        <repository>
            <id>repository.spring.milestone</id>
            <name>Spring Milestone Repository</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>

    <!--配置依赖-->
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>6.0.0-M2</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```



### 定义bean

定义bean：User:

```java
/**
 * @Description: 这是一个bean，封装了用户信息，Spring可以帮助我们创建User对象吗？
 * @author: coderMartin
 * @date: 2022-11-13
 */
public class User {
}
```



### 配置 Spring xml

bean的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--Spring的配置文件-->
    <!--放在resource文件夹下，就相当于放在ClassPath下-->

    <!--
        bean标签的两个重要属性：
            id:这个是bean的身份证号，不能重复
            class:必须填写类的全路径，全限定类名(带包名的类名)
    -->
    <bean id="userBean" class="com.codermartinn.bean.User">

    </bean>
</beans>
```



bean的id和class属性：

- **id属性：代表对象的唯一标识。可以看做一个人的身份证号。**
- **class属性：用来指定要创建的java对象的类名，这个类名必须是全限定类名（带包名）。**





### 编写测试程序

```java
public class FirstSpringTest {
    @Test
    public void testBean() {
        //step1:获取Spring的container
        /*
         * ApplicationContext是一个interface，ClassPathXmlApplicationContext是一个implementation
         * ClassPathXmlApplicationContext： 专门从class path中加载Spring config中的一个Spring context object
         * 这行代码作用：启动了Spring container，解析spring config xml文件，并且实例化所有bean对象，放在Spring container中
         */
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");


        //step2:根据bean的id，从Spring的container中获取object
        Object userBean = applicationContext.getBean("userBean");
        System.out.println(userBean);
    }
}
```

**output:**

```java
com.codermartinn.spring.bean.User@6572421
```



### Details



**底层是怎么创建对象的，是通过反射机制调用无参数构造方法吗？**

默认情况下，Spring会通过「反射机制」，调用类的无参构造方法来实例化对象。

```java
public class User {
    public User() {
        System.out.println("User的无参构造方法执行了...");
    }
}
```

执行测试类:

```java
User的无参构造方法执行了...
com.codermartinn.spring.bean.User@6572421
```



如果没有写无参构造函数，就会报错：

```java
public class User {
//    public User() {
//        System.out.println("User的无参构造方法执行了...");
//    }

    public User(String s) {
        System.out.println("User的有参构造方法执行了...");
    }
}
```

![image-20221113114615420](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221113114615420.png)



**把创建好的对象存储到一个什么样的数据结构当中了呢？**

![image-20221113114742187](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221113114742187.png)

**像这样的spring.xml文件可以有多个吗？**

可以

![image-20221113114946281](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221113114946281.png)



**在配置文件中配置的类必须是自定义的吗，可以使用JDK中的类吗，例如：java.util.Date？**

xml文件中配置：

```xml
<bean id="nowTime" class="java.util.Date"/>
```

![image-20221113115350210](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221113115350210.png)



**getBean()方法返回的类型是Object，如果访问子类的特有属性和方法时，还需要向下转型，有其它办法可以解决这个问题吗？**

不想强制转换。getBean(?,指定类型)

```java
Date nowTime = applicationContext.getBean("nowTime", Date.class);
System.out.println(nowTime);
```





 **ClassPathXmlApplicationContext是从类路径中加载配置文件，如果没有在类路径当中，又应该如何加载配置文件呢？**

- `FileSystemXmlApplicationContext()`

```java
@Test
public void testXmlPath(){
    ApplicationContext applicationContext = new FileSystemXmlApplicationContext();
    applicationContext.getBean();
}
```



**ApplicationContext的超级父接口BeanFactory**

```java
BeanFactory beanFactory = new ClassPathXmlApplicationContext("spring.xml");
Object vipBean = beanFactory.getBean("vipBean");
System.out.println(vipBean);
```

BeanFactory是IOC的顶级接口,ApplicationContext是BeanFactory的子接口。

Spring的IOC底层实际使用了：工厂模式

Spring的IOC底层怎么实现的？XML解析+工厂模式+反射机制



**并不是在调用getBean()方法时创建对象，而是执行以下代码时就会实例化对象**

```java
@Test
public void testInitBean(){
    new ClassPathXmlApplicationContext("spring.xml");
}
```

**output**:

```java
User的无参构造方法执行了...
```



### 集成Log4j2

从Spring5之后，Spring框架支持集成的日志框架是Log4j2.

如何启用日志框架：

第一步：引入Log4j2的依赖

```xml
<!--log4j2的依赖-->
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-core</artifactId>
  <version>2.19.0</version>
</dependency>
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-slf4j2-impl</artifactId>
  <version>2.19.0</version>
</dependency>
```

第二步：在类的根路径下提供log4j2.xml配置文件（文件名固定为：log4j2.xml，文件必须放到类根路径下。）

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>

    <loggers>
        <!--
            level指定日志级别，从低到高的优先级：
                ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < OFF
        -->
        <root level="DEBUG">
            <appender-ref ref="spring6log"/>
        </root>
    </loggers>

    <appenders>
        <!--输出日志信息到控制台-->
        <console name="spring6log" target="SYSTEM_OUT">
            <!--控制日志输出的格式-->
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss SSS} [%t] %-3level %logger{1024} - %msg%n"/>
        </console>
    </appenders>

</configuration>
```

第三步：使用日志框架

```java
Logger logger = LoggerFactory.getLogger(FirstSpringTest.class);
logger.info("我是一条日志消息");
```

