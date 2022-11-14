## 07.bean-instantiation

Spring为Bean提供了多种实例化方式，通常包括4种方式。（也就是说在Spring中为Bean对象的创建准备了多种方案，目的是：更加灵活）

- 第一种：通过构造方法实例化
- 第二种：通过简单工厂模式实例化
- 第三种：通过工厂方法模式实例化
- 第四种：通过FactoryBean接口实例化



### 7.1 通过构造方法实例化

默认情况下，会调用Bean的无参数构造方法。

```java
public class SpringBean {
    public SpringBean() {
        System.out.println("SpringBean的无参构造方法执行...");
    }
}
```



spring.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--
        Spring提供的Bean的实例化方式：
            第一种：在Spring配置文件中直接配置全路径，Spring会自动调用该类的无参构造方法来实例化Bean.
    -->
    <bean id="bean1" class="com.codermartinn.spring.bean.SpringBean"/>
</beans>
```



测试程序：

```java
@Test
public void test1(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    SpringBean bean1 = applicationContext.getBean("bean1", SpringBean.class);
    System.out.println(bean1);
}
```

output：

```java
SpringBean的无参构造方法执行...
com.codermartinn.spring.bean.SpringBean@6572421
```



### 7.2 通过简单工厂模式实例化

调用工厂类的静态方法。

Star类：

```java
public class Star {
    public Star() {
        System.out.println("Star的无参构造方法执行了...");
    }
}
```



简单工厂类：

```java
public class StarFactory {

    public static Star getStar(){
        return  new Star();
    }
}
```



配置文件:

```xml
    <!--
        Spring提供的Bean的实例化方式：
            第二种：通过简单工厂模式，告诉Spring框架，调用哪个类获取Bean
    -->
    <bean id="bean2" class="com.codermartinn.spring.bean.StarFactory" factory-method="getStar"/>
```



测试程序：

```java
@Test
public void test2(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    Star bean2 = applicationContext.getBean("bean2", Star.class);
    System.out.println(bean2);
}
```

output:

```java
Star的无参构造方法执行了...
com.codermartinn.spring.bean.Star@79e2c065
```



### 7.3 通过工厂方法模式实例化

这种方式本质上是：通过工厂方法模式进行实例化。

第一步：定义一个Bean：

```java
public class Order {
}
```

第二步：定义具体工厂类，工厂类中定义实例方法:

```java
public class OrderFactory {
    public Order get(){
        return new Order();
    }
}

```

第三步：在Spring配置文件中指定factory-bean以及factory-method:

```xml
<bean id="orderFactory" class="com.powernode.spring6.bean.OrderFactory"/>
<bean id="orderBean" factory-bean="orderFactory" factory-method="get"/>
```

第四步：编写测试程序:

```java
@Test
public void testSelfFactoryBean(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    Order orderBean = applicationContext.getBean("orderBean", Order.class);
    System.out.println(orderBean);
}
```



### 7.4 通过FactoryBean接口实例化

以上的第三种方式中，factory-bean是我们自定义的，factory-method也是我们自己定义的。

在Spring中，当你编写的类直接实现FactoryBean接口之后，factory-bean不需要指定了，factory-method也不需要指定了。

factory-bean会自动指向实现FactoryBean接口的类，factory-method会自动指向getObject()方法。

第一步：定义一个Bean

```java
public class Person {
}
```

第二步：编写一个类实现FactoryBean接口

```java
public class PersonFactoryBean implements FactoryBean<Person> {

    @Override
    public Person getObject() throws Exception {
        return new Person();
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }

    @Override
    public boolean isSingleton() {
        // true表示单例
        // false表示原型
        return true;
    }
}
```

第三步：在Spring配置文件中配置FactoryBean

```xml
<bean id="personBean" class="com.powernode.spring6.bean.PersonFactoryBean"/>
```

测试程序：

```java
@Test
public void testFactoryBean(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    Person personBean = applicationContext.getBean("personBean", Person.class);
    System.out.println(personBean);

    Person personBean2 = applicationContext.getBean("personBean", Person.class);
    System.out.println(personBean2);
}
```



### 7.5 BeanFactory和FactoryBean的区别

#### 7.5.1 BeanFactory

Spring IoC容器的顶级对象，BeanFactory被翻译为“Bean工厂”，在Spring的IoC容器中，“Bean工厂”负责创建Bean对象。

BeanFactory是工厂。

#### 7.5.2 FactoryBean

FactoryBean：它是一个Bean，是一个能够**辅助Spring**实例化其它Bean对象的一个Bean。

在Spring中，Bean可以分为两类：

- 第一类：普通Bean
- 第二类：工厂Bean（记住：工厂Bean也是一种Bean，只不过这种Bean比较特殊，它可以辅助Spring实例化其它Bean对象。）



### 7.6 注入自定义Date

java.util.Date在Spring中被当做简单类型，简单类型在注入的时候可以直接使用value属性或value标签来完成。但我们之前已经测试过了，对于Date类型来说，采用value属性或value标签赋值的时候，对日期字符串的格式要求非常严格，必须是这种格式的：Mon Oct 10 14:30:26 CST 2022。其他格式是不会被识别的。

```java
public class Student {

    private Date birthday;

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "Student{" +
                "birthday=" + birthday +
                '}';
    }
}
```



xml:

```xml
    <bean id="nowTime" class="java.util.Date" />
    <bean id="studentBean" class="com.codermartinn.spring.bean.Student">
        <!--把日期类型当作简单类型-->
<!--        <property name="birthday" value="Mon Oct 10 14:30:26 CST 2022"/>-->
        <!--把日期当作非简单类型-->
        <property name="birthday"  ref="nowTime"/>
    </bean>
```

这种只能以系统时间作为生日，不符合需求。



那该如何做呢？

```java
public class DateFactoryBean implements FactoryBean<Date> {

    private String strDate;

    public void setStrDate(String strDate) {
        this.strDate = strDate;
    }

    @Override
    public Date getObject() throws Exception {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        Date date = sdf.parse(strDate);
        return date;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }
}
```

DateFactoryBean这个工厂Bean协助Spring创建这个普通Bean: Date。



在配置文件中进行配置：

```xml
    <bean id="dateBean" class="com.codermartinn.spring.bean.DateFactoryBean">
        <constructor-arg index="0" value="1999-10-11"/>
    </bean>

    <bean id="studentBean" class="com.codermartinn.spring.bean.Student">
        <property name="birthday" ref="dateBean"/>
    </bean>
```



测试程序：

```java
@Test
public void test3(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    Student student = applicationContext.getBean("studentBean", Student.class);
    System.out.println(student);
}
```

output：

```java
Student{birthday=Mon Oct 11 00:00:00 CST 1999}
```

