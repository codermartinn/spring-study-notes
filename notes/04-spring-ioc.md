## 04.spring-ioc

### 4.1 IoC 控制反转

- 控制反转是一种思想。
- 控制反转是为了降低程序耦合度，提高程序扩展力，达到OCP原则，达到DIP原则。
- 控制反转，反转的是什么？

- - 将对象的创建权利交出去，交给第三方容器负责。
  - 将对象和对象之间关系的维护权交出去，交给第三方容器负责。

- 控制反转这种思想如何实现呢？

- - DI（Dependency Injection）：依赖注入



### 4.2 DI 依赖注入

依赖注入实现了控制反转的思想。

**Spring通过依赖注入的方式来完成Bean管理的。**

**Bean管理说的是：Bean对象的创建，以及Bean对象中属性的赋值（或者叫做Bean对象之间关系的维护）。**

依赖注入：

- 依赖指的是对象和对象之间的关联关系。
- 注入指的是一种数据传递行为，通过注入行为来让对象和对象产生关系。

依赖注入常见的实现方式包括两种：

- 第一种：set注入
- 第二种：构造注入



#### set方法注入

```java
public class UserDao {

    public static final Logger logger = LoggerFactory.getLogger(UserDao.class);


    public void insert(){
        logger.info("数据库正在保存信息...");
    }
}
```



```java
public class UserService {
    private UserDao userDao;

    public void saveUser(){
        userDao.insert();
    }
}
```



配置spring xml文件

![image-20221113165149779](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221113165149779.png)





编写测试程序：

```java
public class SpringDITest {

    @Test
    public void testSetDI() {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        UserService userService = applicationContext.getBean("userServiceBean", UserService.class);
        userService.saveUser();
    }
}
```



output:

```java
java.lang.NullPointerException: Cannot invoke "com.codermartinn.spring.dao.UserDao.insert()" because "this.userDao" is null
```



```java
public class UserService {
    private UserDao userDao;

    public void saveUser(){
        userDao.insert();
    }
}
```

- userDao没有被赋值



提供了set方法注入：

```java
public class UserService {
    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void saveUser(){
        userDao.insert();
    }
}
```

再次运行测试程序：

```java
java.lang.NullPointerException: Cannot invoke "com.codermartinn.spring.dao.UserDao.insert()" because "this.userDao" is null
```

还是userDao没有被赋值，`setUserDao(UserDao userDao)`方法没有被调用。

该如何让spring调用`serDao userDao(UserDao userDao)`方法呢？



```java
public class UserService {
    private UserDao userDao;

    // 使用set方式注入，必须提供set方法。
    // 反射机制要调用这个方法给属性赋值的。
    public void setUserDaoXX(UserDao userDao) {
        this.userDao = userDao;
    }

    public void saveUser(){
        userDao.insert();
    }
}
```



修改spring xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置dao-->
    <bean id="userDaoBean" class="com.codermartinn.spring.dao.UserDao"/>

    <!--配置service-->
    <bean id="userServiceBean" class="com.codermartinn.spring.service.UserService">
        <!--name属性：set方法的方法名，去掉set，然后把剩下的单词的首字母变小写-->
        <!--ref属性：ref赋值为要注入的bean的id-->
        <property name="userDaoXX" ref="userDaoBean"/>
    </bean>

</beans>
```



运行测试程序：

```java
[main] INFO com.codermartinn.spring.dao.UserDao - 数据库正在保存信息...
```



实现原理：

通过property标签获取到属性名：userDao

通过属性名推断出set方法名：setUserDao

通过反射机制调用setUserDao()方法给属性赋值

property标签的name是属性名。

property标签的ref是要注入的bean对象的id。**(通过ref属性来完成bean的装配，这是bean最简单的一种装配方式。装配指的是：创建系统组件之间关联的动作)**



**总结：set注入的核心实现原理：通过反射机制调用set方法来给属性赋值，让两个对象之间产生关系。**



#### 构造注入

核心原理：通过调用构造方法来给属性赋值。



dao：

```java
public class UserDao {

    public static final Logger logger = LoggerFactory.getLogger(UserDao.class);


    public void insert(){
        logger.info("数据库正在保存信息...");
    }
}
```



```java
public class VipDao {

    public static final Logger logger = LoggerFactory.getLogger(UserDao.class);


    public void insert(){
        logger.info("数据库正在保存VIP信息...");
    }
}
```



service：

```java
public class CustomerService {

    private UserDao userDao;
    private VipDao vipDao;

    public CustomerService(UserDao userDao, VipDao vipDao) {
        this.userDao = userDao;
        this.vipDao = vipDao;
    }

    public void save() {
        userDao.insert();
        vipDao.insert();
    }
}
```



配置 bean xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置dao-->
    <bean id="userDaoBean" class="com.codermartinn.spring.dao.UserDao"/>
    <bean id="vipDaoBean" class="com.codermartinn.spring.dao.VipDao"/>

    <!--配置service-->
    <bean id="customerServiceBean" class="com.codermartinn.spring.service.CustomerService">
        <!--构造注入-->
        <!--
            index:构造方法的参数位置
        -->
        <constructor-arg index="0" ref="userDaoBean"/>
        <constructor-arg index="1" ref="vipDaoBean"/>
    </bean>
</beans>
```



测试程序:

```java
@Test
public void testConstructDI() {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
    CustomerService customerService = applicationContext.getBean("customerServiceBean", CustomerService.class);
    customerService.save();
}
```

output：

```java
2022-11-13 17:20:40 754 [main] INFO com.codermartinn.spring.dao.UserDao - 数据库正在保存信息...
2022-11-13 17:20:40 754 [main] INFO com.codermartinn.spring.dao.UserDao - 数据库正在保存VIP信息...
```

构造注入成功。

也可以不用`index`，用`name`属性：

![image-20221113172420815](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221113172420815.png)





### 4.3 set注入专题

#### 注入内部bean和外部bean

dao:

```java
public class OrderDao {
    public static final Logger logger = LoggerFactory.getLogger(OrderDao.class);


    public void insert(){
        logger.info("订单正在生成...");
    }
}
```

service:

```java
public class OrderService {
    private OrderDao orderDao;

    //set方法注入
    public void setOrderDao(OrderDao orderDao) {
        this.orderDao = orderDao;
    }

    /**
     * 生成订单的业务方法
     */
    public void generate() {
        orderDao.insert();
    }
}
```



set-di.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--声明/定义bean-->
    <bean id="orderDaoBean" class="com.codermartinn.spring.dao.OrderDao" />


    <bean id="orderServiceBean" class="com.codermartinn.spring.service.OrderService">
        <!--注入外部bean-->
        <property name="orderDao" ref="orderDaoBean"/>
    </bean>


    <bean id="orderServiceBean2" class="com.codermartinn.spring.service.OrderService">
        <!--注入内部bean-->
        <property name="orderDao">
            <!--在property标签中嵌套bean标签，这就是内部bean-->
            <bean class="com.codermartinn.spring.dao.OrderDao"/>
        </property>
    </bean>
</beans>
```



测试程序：

```java
@Test
public void testSetDI2(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("set-di.xml");
    OrderService orderService = applicationContext.getBean("orderServiceBean", OrderService.class);
    orderService.generate();

    OrderService orderService2 = applicationContext.getBean("orderServiceBean2", OrderService.class);
    orderService2.generate();
}
```

output:

```java
2022-11-13 17:39:14 734 [main] INFO com.codermartinn.spring.dao.OrderDao - 订单正在生成...
2022-11-13 17:39:14 734 [main] INFO com.codermartinn.spring.dao.OrderDao - 订单正在生成...
```



#### 注入简单类型



service:

```java
public class OrderService {
    private OrderDao orderDao;

    //set方法注入
    public void setOrderDao(OrderDao orderDao) {
        this.orderDao = orderDao;
    }

    /**
     * 生成订单的业务方法
     */
    public void generate() {
        orderDao.insert();
    }
}
```

上面注入的是「引用类型」，假如要注入的是「基本数据类型」呢？



定义bean：

```java
public class User {
    private String username;//String是简单类型
    private String password;
    private int age;//int是简单类型

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", age=" + age +
                '}';
    }
}
```



配置xml文件：

```xml
    <!--注入简单类型-->
    <bean id="userBean" class="com.codermartinn.spring.bean.User">
        <!--给简单类型注入，不能使用ref，而是使用value-->
        <property name="username" value="zhangsan"/>
        <property name="password" value="123456"/>
        <property name="age" value="20"/>
    </bean>
```



编写测试程序：

```java
@Test
public void testSimpleType() {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("set-di.xml");
    User user = applicationContext.getBean("userBean", User.class);
    System.out.println(user);
}
```

output：

```java
User{username='zhangsan', password='123456', age=20}
```



简单类型包括哪些呢？可以通过Spring的源码来分析一下：BeanUtils类

**通过源码分析得知，简单类型包括：**

- **基本数据类型**
- **基本数据类型对应的包装类**
- **String或其他的CharSequence子类**
- **Number子类**
- **Date子类**
- **Enum子类**
- **URI**
- **URL**
- **Temporal子类**
- **Locale**
- **Class**
- **另外还包括以上简单值类型对应的数组类型。**



#### 级联属性赋值（了解）

配置文件xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="studentBean" class="com.codermartinn.spring.bean.Student">
        <property name="name" value="张三"/>
        <property name="clazz" ref="clazzBean"/>
        <!--级联属性赋值,必须提供get方法-->
        <property name="clazz.name" value="高三一班"/>
    </bean>
    <bean id="clazzBean" class="com.codermartinn.spring.bean.Clazz"/>

<!--    <bean id="clazzBean" class="com.codermartinn.spring.bean.Clazz">-->
<!--        <property name="name" value="高三一班"/>-->
<!--    </bean>-->
</beans>
```



![image-20221113221319983](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221113221319983.png)



编写测试程序：

```java
@Test
public void testCascade(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("cascade.xml");

    Student student = applicationContext.getBean("studentBean", Student.class);
    System.out.println(student);

    Clazz clazz = applicationContext.getBean("clazzBean", Clazz.class);
    System.out.println(clazz);
}
```

output:

```java
Student{name='张三', clazz=Clazz{name='高三一班'}}
Clazz{name='高三一班'}
```



**要点：**

- **在spring配置文件中，如上，注意顺序。**
- **在spring配置文件中，clazz属性必须提供getter方法。**



#### 注入数组



person:

```java
public class Person {
    private String[] hobbies;

    public void setHobbies(String[] hobbies) {
        this.hobbies = hobbies;
    }

    @Override
    public String toString() {
        return "Person{" +
                "hobbies=" + Arrays.toString(hobbies) +
                '}';
    }
}
```



spring-array.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="personBean" class="com.codermartinn.spring.bean.Person">
        <!--这个数组中的元素是String简单类型-->
        <property name="hobbies">
            <array>
                <value>football</value>
                <value>code</value>
                <value>math</value>
                <value>food</value>
            </array>
        </property>
    </bean>
</beans>
```



编写测试程序：

```java
    @Test
    public void testArray() {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-array.xml");

        Person person = applicationContext.getBean("personBean", Person.class);
        System.out.println(person);
    }
```

output:

```java
Person{hobbies=[football, code, math, food]}
```



如果数组中的元素不是简单类型呢？

```java
public class Phone {
    private String name;

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Phone{" +
                "name='" + name + '\'' +
                '}';
    }
}
```



```java
public class Person {
    private String[] hobbies;

    private Phone[] phones;

    public void setHobbies(String[] hobbies) {
        this.hobbies = hobbies;
    }

    public void setPhones(Phone[] phones) {
        this.phones = phones;
    }

    @Override
    public String toString() {
        return "Person{" +
                "hobbies=" + Arrays.toString(hobbies) +
                ", phones=" + Arrays.toString(phones) +
                '}';
    }
}
```



配置文件xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="personBean" class="com.codermartinn.spring.bean.Person">
        <!--这个数组中的元素是String简单类型-->
        <property name="hobbies">
            <array>
                <value>football</value>
                <value>code</value>
                <value>math</value>
                <value>food</value>
            </array>
        </property>

        <!--这个数组中的元素不是简单类型-->
        <property name="phones">
            <array>
                <ref bean="huawei"/>
                <ref bean="oppo"/>
                <ref bean="vivo"/>
                <ref bean="xiaomi"/>
            </array>
        </property>
    </bean>

    <bean id="huawei" class="com.codermartinn.spring.bean.Phone">
        <property name="name" value="huawei"/>
    </bean>
    <bean id="vivo" class="com.codermartinn.spring.bean.Phone">
        <property name="name" value="vivo"/>
    </bean>
    <bean id="oppo" class="com.codermartinn.spring.bean.Phone">
        <property name="name" value="oppo"/>
    </bean>
    <bean id="xiaomi" class="com.codermartinn.spring.bean.Phone">
        <property name="name" value="xiaomi"/>
    </bean>

</beans>
```



还是运行原来的测试程序：

```java
Person{hobbies=[football, code, math, food], phones=[Phone{name='huawei'}, Phone{name='oppo'}, Phone{name='vivo'}, Phone{name='xiaomi'}]}
```



**要点：**

- **如果数组中是简单类型，使用value标签。**
- **如果数组中是非简单类型，使用ref标签。**



#### 注入List集合和Set集合

person:

```java
public class Person {
    //注入List集合
    private List<String> friends;

    //注入Set集合
    private Set<String> address;

    public void setFriends(List<String> friends) {
        this.friends = friends;
    }

    public void setAddress(Set<String> address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "Person{" +
                "friends=" + friends +
                ", address=" + address +
                '}';
    }
}
```



xml配置文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="personBean" class="com.codermartinn.spring.DTO.Person">
        <property name="friends">
            <!--List集合元素有序可重复-->
            <list>
                <value>张三</value>
                <value>张三</value>
                <value>李四</value>
                <value>王五</value>
            </list>
        </property>

        <property name="address">
            <!--Set集合元素无序不可重复-->
            <set>
                <value>北京</value>
                <value>北京</value>
                <value>南京</value>
                <value>西安</value>
            </set>
        </property>
    </bean>
</beans>
```



测试程序：

```java
@Test
public void testCollection(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-collection.xml");
    Person person = applicationContext.getBean("personBean", Person.class);
    System.out.println(person);
}
```

output:

```java
Person{friends=[张三, 张三, 李四, 王五], address=[北京, 南京, 西安]}
```

**注意:**

- **注入List集合的时候使用list标签，如果List集合中是简单类型使用value标签，反之使用ref标签。**

- **注入Set集合使用<set>标签**
- **set集合中元素是简单类型的使用value标签，反之使用ref标签。**





#### 注入Map集合

person:

```java
public class Person {
    //注入List集合
    private List<String> friends;

    //注入Set集合
    private Set<String> address;

    //注入Map集合
    private Map<Integer,String> phones;

    public void setFriends(List<String> friends) {
        this.friends = friends;
    }

    public void setAddress(Set<String> address) {
        this.address = address;
    }

    public void setPhones(Map<Integer, String> phones) {
        this.phones = phones;
    }

    @Override
    public String toString() {
        return "Person{" +
                "friends=" + friends +
                ", address=" + address +
                ", phones=" + phones +
                '}';
    }
}
```



xml文件配置：

```xml
//....省略其他部分
<property name="phones">
    <!--注入Map集合-->
    <map>
        <entry key="0" value="1234546" />
        <entry key="1" value="78484848" />
        <entry key="2" value="15616516514" />
        <!--如果不是简单类型-->
        <!--<entry key-ref="" value-ref=""/>-->
    </map>
</property>
```



测试程序：

```java
    @Test
    public void testCollection(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-collection.xml");
        Person person = applicationContext.getBean("personBean", Person.class);
        System.out.println(person);
    }
```

ouotput:

```java
Person{friends=[张三, 张三, 李四, 王五], address=[北京, 南京, 西安], phones={0=1234546, 1=78484848, 2=15616516514}}
```

**要点：**

- **使用<map>标签**
- **如果key是简单类型，使用 key 属性，反之使用 key-ref 属性。**
- **如果value是简单类型，使用 value 属性，反之使用 value-ref 属性。**



#### 注入Properties

java.util.Properties继承java.util.Hashtable，所以Properties也是一个Map集合。

**要点：**

- **使用<props>标签嵌套<prop>标签完成。**
- **key与value只能是String**



#### 注入null和空字符串

注入空字符串使用：<value/> 或者 value=""

注入null使用：<null/> 或者 不为该属性赋值

```xml
<bean id="catBean" class="com.codermartinn.spring.DTO.Cat">
    <!--不给属性赋值，属性的默认值就为null-->
    <!--<property name="name" value="tom"/>-->
    
    <!--手动注入null-->
    <!--
        <property name="name">
            <null/>
        </property>
    -->
    
    <!--注入空字符串-->
    <property name="name" value=""/>
    
    <property name="age" value="12"/>
</bean>
```



#### 注入的值含有特殊符号

XML中有5个特殊字符，分别是：<、>、'、"、&

以上5个特殊符号在XML中会被特殊对待，会被当做XML语法的一部分进行解析，如果这些特殊符号直接出现在注入的字符串当中，会报错。

解决方案包括两种：

- 第一种：特殊符号使用转义字符代替。
- 第二种：将含有特殊符号的字符串放到：<![CDATA[]]> 当中。因为放在CDATA区中的数据不会被XML文件解析器解析。

5个特殊字符对应的转义字符分别是：

| **特殊字符** | **转义字符** |
| ------------ | ------------ |
| >            | &gt;         |
| <            | &lt;         |
| '            | &apos;       |
| "            | &quot;       |
| &            | &amp;        |



### 4.4 p命名空间注入

目的：简化配置。

p命名空间注入底层还是set注入，只不过p命名空间注入可以让spring配置更简单





使用p命名空间注入的前提条件包括两个：

- 第一：在XML头部信息中添加p命名空间的配置信息：xmlns:p="http://www.springframework.org/schema/p"
- 第二：p命名空间注入是基于setter方法的，所以需要对应的属性提供setter方法。



Dog:

```java
public class Dog {

    private String name;
    private int age;

    private Date birthday;

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "Dog{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", birthday=" + birthday +
                '}';
    }
}
```



xml配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dogBean" class="com.codermartinn.spring.DTO.Dog" p:name="大黄" p:age="6" p:birthday-ref="birthDayBean"/>

    <!--获取当前系统时间-->
    <bean id="birthDayBean" class="java.util.Date"/>
</beans>
```



测试程序：

```java
@Test
public void testP(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-p.xml");
    Dog dog = applicationContext.getBean("dogBean", Dog.class);
    System.out.println(dog);
}
```

output：

```java
Dog{name='大黄', age=6, birthday=Mon Nov 14 10:11:35 CST 2022}
```





### 4.5 c命名空间注入

c命名空间是简化构造方法注入的。



使用c命名空间的两个前提条件：

第一：需要在xml配置文件头部添加信息：xmlns:c="http://www.springframework.org/schema/c"

第二：需要提供构造方法。

```java
package com.powernode.spring6.beans;

/**
 * @author 动力节点
 * @version 1.0
 * @className MyTime
 * @since 1.0
 **/
public class MyTime {
    private int year;
    private int month;
    private int day;

    public MyTime(int year, int month, int day) {
        this.year = year;
        this.month = month;
        this.day = day;
    }

    @Override
    public String toString() {
        return "MyTime{" +
                "year=" + year +
                ", month=" + month +
                ", day=" + day +
                '}';
    }
}

```



spring-c.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--<bean id="myTimeBean" class="com.powernode.spring6.beans.MyTime" c:year="1970" c:month="1" c:day="1"/>-->

    <bean id="myTimeBean" class="com.powernode.spring6.beans.MyTime" c:_0="2008" c:_1="8" c:_2="8"/>

</beans>
```

所以，c命名空间是依靠构造方法的。

**注意：不管是p命名空间还是c命名空间，注入的时候都可以注入简单类型以及非简单类型。**



### 4.6 util命名空间

使用util命名空间可以让**配置复用**。

使用util命名空间的前提是：在spring配置文件头部添加配置信息。如下：

![image-20221114102128479](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221114102128479.png)



MyDataSource1:

```java
package com.powernode.spring6.beans;

import java.util.Properties;

/**
 * @author 动力节点
 * @version 1.0
 * @className MyDataSource1
 * @since 1.0
 **/
public class MyDataSource1 {
    private Properties properties;

    public void setProperties(Properties properties) {
        this.properties = properties;
    }

    @Override
    public String toString() {
        return "MyDataSource1{" +
                "properties=" + properties +
                '}';
    }
}
```



MyDataSource2:

```java
package com.powernode.spring6.beans;

import java.util.Properties;

/**
 * @author 动力节点
 * @version 1.0
 * @className MyDataSource2
 * @since 1.0
 **/
public class MyDataSource2 {
    private Properties properties;

    public void setProperties(Properties properties) {
        this.properties = properties;
    }

    @Override
    public String toString() {
        return "MyDataSource2{" +
                "properties=" + properties +
                '}';
    }
}
```



spring-util.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">

    <util:properties id="prop">
        <prop key="driver">com.mysql.cj.jdbc.Driver</prop>
        <prop key="url">jdbc:mysql://localhost:3306/spring</prop>
        <prop key="username">root</prop>
        <prop key="password">123456</prop>
    </util:properties>

    <bean id="dataSource1" class="com.powernode.spring6.beans.MyDataSource1">
        <property name="properties" ref="prop"/>
    </bean>

    <bean id="dataSource2" class="com.powernode.spring6.beans.MyDataSource2">
        <property name="properties" ref="prop"/>
    </bean>
</beans>
```



### 4.7 基于XML的自动装配

#### 根据名称自动装配

Spring的配置文件这样配置：

spring-autowire.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService" class="com.powernode.spring6.service.UserService" autowire="byName"/>
    
    <bean id="aaa" class="com.powernode.spring6.dao.UserDao"/>

</beans>
```

这个配置起到关键作用：

- UserService Bean中需要添加autowire="byName"，表示通过名称进行装配。
- UserService类中有一个UserDao属性，而UserDao属性的名字是aaa，**对应的set方法是setAaa()**，正好和UserDao Bean的id是一样的。这就是根据名称自动装配。



#### 根据类型自动装配



spring-autowire.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--byType表示根据类型自动装配-->
    <bean id="accountService" class="com.powernode.spring6.service.AccountService" autowire="byType"/>

    <bean class="com.powernode.spring6.dao.AccountDao"/>

</beans>
```

可以看到无论是byName还是byType，在装配的时候都是基于set方法的。所以set方法是必须要提供的。提供构造方法是不行的



如果byType，根据类型装配时，如果配置文件中有两个类型一样的bean会出现什么问题呢？

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountService" class="com.powernode.spring6.service.AccountService" autowire="byType"/>

    <bean id="x" class="com.powernode.spring6.dao.AccountDao"/>
    <bean id="y" class="com.powernode.spring6.dao.AccountDao"/>

</beans>
```

当byType进行自动装配的时候，配置文件中某种类型的Bean必须是唯一的，不能出现多个。



### 4.8 Spring引入外部属性配置文件

我们都知道编写数据源的时候是需要连接数据库的信息的，例如：driver url username password等信息。这些信息可以单独写到一个属性配置文件中吗，这样用户修改起来会更加的方便。当然可以。

第一步：写一个数据源类，提供相关属性。

```java
package com.powernode.spring6.beans;

import javax.sql.DataSource;
import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.SQLFeatureNotSupportedException;
import java.util.logging.Logger;

/**
 * @author 动力节点
 * @version 1.0
 * @className MyDataSource
 * @since 1.0
 **/
public class MyDataSource implements DataSource {
    @Override
    public String toString() {
        return "MyDataSource{" +
                "driver='" + driver + '\'' +
                ", url='" + url + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }

    private String driver;
    private String url;
    private String username;
    private String password;

    public void setDriver(String driver) {
        this.driver = driver;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    //......
}
```



第二步：在类路径下新建jdbc.properties文件，并配置信息。

```properties
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/spring
username=root
password=root123
```



第三步：在spring配置文件中引入context命名空间。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

</beans>
```

第四步：在spring中配置使用jdbc.properties文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="jdbc.properties"/>
    
    <bean id="dataSource" class="com.powernode.spring6.beans.MyDataSource">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
    </bean>
</beans>
```

