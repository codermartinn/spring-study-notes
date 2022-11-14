## 05.Bean-scope

### 5.1 singleton

默认情况下，Spring的IoC容器创建的Bean对象是单例的。Bean在spring上下文初始化的时候实例化的。



bean：

```jav
public class SpringBean {
    public SpringBean() {
        System.out.println("SpringBean构造方法执行了...");
    }
}
```



spring-scope.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="springBean" class="com.codermartinn.spring.bean.SpringBean"/>
</beans>
```



测试类：

```java
@Test
public void testBeanScope(){

    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-scope.xml");
    SpringBean springBean = applicationContext.getBean("springBean", SpringBean.class);
    System.out.println(springBean);

    SpringBean springBean2 = applicationContext.getBean("springBean", SpringBean.class);
    System.out.println(springBean2);

    SpringBean springBean3 = applicationContext.getBean("springBean", SpringBean.class);
    System.out.println(springBean3);
}
```

output:

```java
SpringBean构造方法执行了...
com.codermartinn.spring.bean.SpringBean@5876a9af
com.codermartinn.spring.bean.SpringBean@5876a9af
com.codermartinn.spring.bean.SpringBean@5876a9af
```

通过测试得知，默认情况下，Bean对象的创建是在初始化Spring上下文的时候就完成的。



### 5.2 prototype

如果想让Spring的Bean对象以多例的形式存在，可以在bean标签中指定scope属性的值为：**prototype**，这样Spring会在每一次执行getBean()方法的时候创建Bean对象，调用几次则创建几次。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="springBean" class="com.codermartinn.spring.bean.SpringBean" scope="prototype"/>
</beans>
```

运行测试程序：

```java
SpringBean构造方法执行了...
com.codermartinn.spring.bean.SpringBean@4cc451f2
SpringBean构造方法执行了...
com.codermartinn.spring.bean.SpringBean@6379eb
SpringBean构造方法执行了...
com.codermartinn.spring.bean.SpringBean@294425a7
```



当把测试代码中的getBean()方法所在行代码注释掉：

```java
    @Test
    public void testBeanScope(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-scope.xml");
    }
```

![image-20221114112512216](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221114112512216.png)

并不会像单例一样，在上下文初始化时就把对象创建出来。只有在调用`getBean()`方法的时候，才会创建新的对象。

没有指定scope属性时，默认是singleton单例的。



### 5.3 其它scope

**scope属性的值不止两个，它一共包括8个选项：**

- singleton：默认的，单例。
- prototype：原型。每调用一次getBean()方法则获取一个新的Bean对象。或每次注入的时候都是新对象。
- request：一个请求对应一个Bean。**仅限于在WEB应用中使用**。
- session：一个会话对应一个Bean。**仅限于在WEB应用中使用**。
- global session：**portlet应用中专用的**。如果在Servlet的WEB应用中使用global session的话，和session一个效果。（portlet和servlet都是规范。servlet运行在servlet容器中，例如Tomcat。portlet运行在portlet容器中。）
- application：一个应用对应一个Bean。**仅限于在WEB应用中使用。**
- websocket：一个websocket生命周期对应一个Bean。**仅限于在WEB应用中使用。**
- 自定义scope：很少使用。