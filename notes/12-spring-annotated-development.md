## 12.spring-annotated-development

### 12.1 回顾注解

注解的存在主要是为了简化XML的配置。**Spring6倡导全注解开发**。

我们来回顾一下：

- 第一：注解怎么定义，注解中的属性怎么定义？
- 第二：注解怎么使用？
- 第三：通过反射机制怎么读取注解？



自定义注解:

```java
/**
 * 自定义注解
 */
//标注注解的注解，称为元注解。@Target修饰@Component能出现的位置
//以下表示@component注解可以出现在类上，属性上。
@Target(value = {ElementType.TYPE, ElementType.FIELD})

//@Retention也是一个元注解。用来标注@Component注解最终保留在class文件当中,并且可以被反射机制读取
@Retention(RetentionPolicy.RUNTIME)
public @interface Component {
    //定义注解的属性
    String value();//String是属性类型，value是属性名
}
```



使用注解：

```java
@Component(value = "userBean")
public class User {
}
```



通过「反射机制」获取`User`类上注解的值：

```java
@Test
public void test1() throws Exception {
    //通过反射机制，读取注解
    Class<?> clazz = Class.forName("com.codermartinn.bean.User");

    //判断类上有没有注解
    if (clazz.isAnnotationPresent(Component.class)) {
        //获取类上的注解
        Component annotation = clazz.getAnnotation(Component.class);
        //访问注解属性
        String value = annotation.value();
        System.out.println("@Component注解的值：" + value);
    }
}
```

output:

```java
@Component注解的值：userBean
```



![image-20221116093110515](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221116093110515.png)





需求：只知道一个包的名字，扫描包下所有的类，当这个类上有@Component注解，实例化对象，然后放入Map集合

```java
@Test
public void ComponentScan() throws Exception {
    /**
     * 需求：只知道一个包的名字，扫描包下所有的类，当这个类上有@Component注解，实例化对象，然后放入Map集合
     */
    String packageName = "com.codermartinn.bean";
    Map<String, Object> beanMap = new HashMap<>();

    // 获取包下所有的类
    String packagePath = packageName.replaceAll("\\.", "/");
    URL url = ClassLoader.getSystemClassLoader().getResource(packagePath);
    
    // 获取到绝对路径
    assert url != null;
    String path = url.getPath();
    File file = new File(path);
    
    //拿到所有子文件
    File[] files = file.listFiles();
    Arrays.stream(files).forEach(f -> {
        //System.out.println(f.getName().split("\\.")[0]);
        String className = packageName + "." + f.getName().split("\\.")[0];
        try {
            Class<?> clazz = Class.forName(className);
            if (clazz.isAnnotationPresent(Component.class)) {
                // 获取注解
                Component annotation = clazz.getAnnotation(Component.class);
                String id = annotation.value();
                // 创建对象
                Object obj = clazz.newInstance();
                beanMap.put(id, obj);
            }
        } catch (ClassNotFoundException | InstantiationException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    });

    System.out.println(beanMap);
}
```

output:

```java
{orderBean=com.codermartinn.bean.Order@2ff4f00f, userBean=com.codermartinn.bean.User@c818063}
```



### 12.2 声明Bean的注解

负责声明Bean的注解，常见的包括四个：

- @Component 老大
- @Controller     别名
- @Service           别名
- @Repository      别名

通过源码可以看到，@Controller、@Service、@Repository这三个注解都是@Component注解的别名。

也就是说：这四个注解的功能都一样。用哪个都可以。

只是为了增强程序的可读性，建议：

- 控制器类上使用：Controller
- service类上使用：Service
- dao类上使用：Repository

他们都是只有一个value属性。value属性用来指定bean的id，也就是bean的名字。



### 12.3 Spring注解的使用



如何使用以上的注解呢？

- 第一步：加入aop的依赖
- 第二步：在配置文件中添加context命名空间
- 第三步：在配置文件中指定扫描的包
- 第四步：在Bean类上使用注解



**第一步：加入aop的依赖**

我们可以看到当加入spring-context依赖之后，会关联加入aop的依赖。所以这一步不用做。

![image-20221116094435555](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221116094435555.png)

**第二步：在配置文件中添加context命名空间**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

</beans>
```



**第三步：在配置文件中指定要扫描的包**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
     
    <!--给spring指定要扫描哪些包-->
    <context:component-scan base-package="com.codermartinn.spring.bean"/>
</beans>
```



**第四步：在Bean类上使用注解**

```java
@Component("userBean")
public class User {
}
```



测试程序：

```java
@Test
public void test1(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    User userBean = applicationContext.getBean("userBean", User.class);
    System.out.println(userBean);
}
```

output:

```java
com.codermartinn.spring.bean.User@51dcb805
```



**如果把value属性彻底去掉，spring会被Bean自动取名吗？会的。并且默认名字的规律是：Bean类名首字母小写即可。**

BankDao的bean的名字为：bankDao



**如果是多个包怎么办？有两种解决方案：**

- **第一种：在配置文件中指定多个包，用逗号隔开。**
- **第二种：指定多个包的共同父包。**







### 12.4 选择性实例化Bean

假设在某个包下有很多Bean，有的Bean上标注了Component，有的标注了Controller，有的标注了Service，有的标注了Repository，现在由于某种特殊业务的需要，只允许其中所有的Controller参与Bean管理，其他的都不实例化。这应该怎么办呢？

```java
@Component
public class A {
    public A() {
        System.out.println("Class A 无参数构造方法执行了...@Component");
    }
}

@Controller
class B {
    public B() {
        System.out.println("Class B 无参数构造方法执行了...@Controller");
    }
}

@Service
class C {
    public C() {
        System.out.println("Class C 无参数构造方法执行了...@Service");
    }
}

@Repository
class D {
    public D() {
        System.out.println("Class D 无参数构造方法执行了...@Repository");
    }
}

@Controller
class E {
    public E() {
        System.out.println("Class E 无参数构造方法执行了...@Controller");
    }
}
```



第一种解决方法：
        use-default-filters="false":让全部带声明bean的注解失效



配置文件：

```xml
<!--
    第一种解决方法：
        use-default-filters="false":让全部带声明bean的注解失效
-->

<!--给spring指定要扫描哪些包-->
<context:component-scan base-package="com.codermartinn.spring.bean" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```



测试程序：

```java
@Test
public void test2(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-choose.xml");
}
```

output：

```java
Class B 无参数构造方法执行了...@Controller
Class E 无参数构造方法执行了...@Controller
```



第二种解决方法：
    use-default-filters="true":让全部带声明bean的注解生效
    然后排除掉不需要的

```xml
<!--
    第二种解决方法：
        use-default-filters="true":让全部带声明bean的注解生效
        然后排除掉不需要的
-->

<!--给spring指定要扫描哪些包-->
<context:component-scan base-package="com.codermartinn.spring.bean" use-default-filters="true">
    <context:exclude-filter type="annotation" expression=""/>
</context:component-scan>
```





### 12.5 负责注入的注解 **

@Component @Controller @Service @Repository 这四个注解是用来声明Bean的，声明后这些Bean将被实例化。接下来我们看一下，如何给Bean的属性赋值。给Bean属性赋值需要用到这些注解：

- @Value
- @Autowired
- @Qualifier
- @Resource



#### 12.5.1 @Value

当属性的类型是简单类型时，可以使用@Value注解进行注入。

可以不提供set方法。

```java
@Component
public class MyDataSource implements DataSource {

    @Value("com.mysql.cj.jdbc.Driver")
    private String driver;

    @Value("jdbc:mysql://localhost:3306/spring")
    private String url;

    @Value("root")
    private String username;

    @Value("123456")
    private String password;


    @Override
    public String toString() {
        return "MyDataSource{" +
                "driver='" + driver + '\'' +
                ", url='" + url + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--给spring指定要扫描哪些包-->
    <context:component-scan base-package="com.codermartinn.spring.bean2"/>

</beans>
```



测试程序：

```java
@Test
public void test3(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring2.xml");
    MyDataSource myDataSource = applicationContext.getBean("myDataSource", MyDataSource.class);
    System.out.println(myDataSource);
}
```

output:

```java
MyDataSource{driver='com.mysql.cj.jdbc.Driver', url='jdbc:mysql://localhost:3306/spring', username='root', password='123456'}
```



@Value注解可以直接使用在属性上，也可以使用在setter方法上。都是可以的。都可以完成属性的赋值:

```java
@Component
public class User {
    
    private String name;

    private int age;

    @Value("李四")
    public void setName(String name) {
        this.name = name;
    }

    @Value("30")
    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```



@Value注解可以直接使用在构造方法完成注入:

```java
@Component
public class User {

    private String name;

    private int age;

    public User(@Value("隔壁老王") String name, @Value("33") int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```



**@Value注解可以出现在属性上、setter方法上、以及构造方法的形参上。可见Spring给我们提供了多样化的注入。太灵活了。**





#### 12.5.2 @Autowired与@Qualifier

@Autowired注解可以用来注入**非简单类型**。被翻译为：自动连线的，或者自动装配。

单独使用@Autowired注解，**默认根据类型装配**。【默认是byType】



源码中有两处需要注意：

- 第一处：该注解可以标注在哪里？

- - 构造方法上
  - 方法上
  - 形参上
  - 属性上
  - 注解上

- 第二处：该注解有一个required属性，默认值是true，表示在注入的时候要求被注入的Bean必须是存在的，如果不存在则报错。如果required属性设置为false，表示注入的Bean存在或者不存在都没关系，存在的话就注入，不存在的话，也不报错。



dao：

```java
public interface OrderDao {
    void insert();
}
```



```java
@Repository("orderDaoImplForMySQL")
public class OrderDaoImplForMySQL implements OrderDao {
    @Override
    public void insert() {
        System.out.println("MySQL数据库正在保存订单信息...");
    }
}
```



service:

```java
@Service("orderService")
public class OrderService {
    @Autowired
    private OrderDao orderDao;


    public void generate(){

        orderDao.insert();
    }
}
```



配置文件,开启组件扫描：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--给spring指定要扫描哪些包-->
    <context:component-scan base-package="com.codermartinn.spring.dao"/>
    <context:component-scan base-package="com.codermartinn.spring.service"/>

</beans>
```



测试类：

```java
@Test
public void test4() {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring3.xml");
    OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
    orderService.generate();
}
```

output:

```java
MySQL数据库正在保存订单信息...
```



@Autowired注解默认是byType进行注入的，也就是说根据类型注入的，如果以上程序中，UserDao接口还有另外一个实现类，会出现问题吗？

```java
@Repository("orderDaoImplForOracle")
public class OrderDaoImplForOracle implements OrderDao {
    @Override
    public void insert() {
        System.out.println("正在向Oracle数据库插入User数据");
    }
}
```

不能装配，UserDao这个Bean的数量大于1.

怎么解决这个问题呢？**当然要byName，根据名称进行装配了。**

@Autowired注解和@Qualifier注解联合起来才可以根据名称进行装配，在@Qualifier注解中指定Bean名称。

```java
@Service("orderService")
public class OrderService {
    @Autowired
    @Qualifier("orderDaoImplForOracle")
    private OrderDao orderDao;

    public void generate(){
        orderDao.insert();
    }
}
```

测试程序，运行结果：

```java
正在向Oracle数据库插入User数据
```





总结：

- @Autowired注解可以出现在：属性上、构造方法上、构造方法的参数上、setter方法上。
- 当带参数的构造方法只有一个，@Autowired注解可以省略。
- @Autowired注解默认根据类型注入。如果要根据名称注入的话，需要配合@Qualifier注解一起使用。



#### 12.5.3 @Resource **

官方建议使用。

@Resource注解也可以完成非简单类型注入。

那它和@Autowired注解有什么区别？

- @Resource注解是JDK扩展包中的，也就是说属于JDK的一部分。所以该注解是标准注解，更加具有通用性。(JSR-250标准中制定的注解类型。JSR是Java规范提案。)
- @Autowired注解是Spring框架自己的。
- **@Resource注解默认根据名称装配byName，未指定name时，使用属性名作为name。通过name找不到的话会自动启动通过类型byType装配。**
- **@Autowired注解默认根据类型装配byType，如果想根据名称装配，需要配合@Qualifier注解一起用。**
- @Resource注解用在属性上、setter方法上。
- @Autowired注解用在属性上、setter方法上、构造方法上、构造方法参数上。



@Resource注解属于JDK扩展包，所以不在JDK当中，需要额外引入以下依赖：【**如果是JDK8的话不需要额外引入依赖。高于JDK11或低于JDK8需要引入以下依赖。**】

如果你是Spring6+版本请使用这个依赖：

```java
<dependency>
  <groupId>jakarta.annotation</groupId>
  <artifactId>jakarta.annotation-api</artifactId>
  <version>2.1.1</version>
</dependency>
```

一定要注意：**如果你用Spring6，要知道Spring6不再支持JavaEE，它支持的是JakartaEE9。（Oracle把JavaEE贡献给Apache了，Apache把JavaEE的名字改成JakartaEE了，大家之前所接触的所有的  javax.\*  包名统一修改为  jakarta.\*包名了。）**



如果你是spring5-版本请使用这个依赖:

```java
<dependency>
  <groupId>javax.annotation</groupId>
  <artifactId>javax.annotation-api</artifactId>
  <version>1.3.2</version>
</dependency>
```

![image-20221116114405930](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221116114405930.png)



原来的代码使用` @Resource`注解：

```java
@Service("orderService")
public class OrderService {
    //根据Bean的name进行注入
    @Resource(name = "orderDaoImplForOracle")
    private OrderDao orderDao;

    public void generate(){
        orderDao.insert();
    }
}
```



一句话总结：

> @Resource注解：默认byName注入，没有指定name时把属性名当做name，根据name找不到时，才会byType注入。byType注入时，某种类型的Bean只能有一个。





### 12.6 全注解式开发

所谓的全注解开发就是不再使用spring配置文件了。写一个配置类来代替配置文件。

配置类代替spring配置文件:

```java
@Configuration
@ComponentScan({"com.codermartinn.spring.dao","com.codermartinn.spring.service"})
public class SpringConfig {
}
```



测试程序,不再new ClassPathXmlApplicationContext()对象了。：

```
@Test
public void test5() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
    OrderService orderService = context.getBean("orderService", OrderService.class);
    orderService.generate();
}
```

output：

```java
MySQL数据库正在保存订单信息...
```

