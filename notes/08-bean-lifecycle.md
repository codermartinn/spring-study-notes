## 08.bean-lifecycle **



### 8.1 什么是Bean的生命周期

Spring其实就是一个管理Bean对象的工厂。它负责对象的创建，对象的销毁等。
所谓的生命周期就是：**对象从创建开始到最终销毁的整个过程。**
什么时候创建Bean对象？
创建Bean对象的前后会调用什么方法？
Bean对象什么时候销毁？
Bean对象的销毁前后调用什么方法？



### 8.2 为什么要知道Bean的生命周期

其实生命周期的本质是：在哪个时间节点上调用了哪个类的哪个方法。
我们需要充分的了解在这个生命线上，都有哪些特殊的时间节点。
只有我们知道了特殊的时间节点都在哪，到时我们才可以确定代码写到哪。
我们可能需要在某个特殊的时间点上执行一段特定的代码，这段代码就可以放到这个节点上。当生命线走到这里的时候，自然会被调用。



### 8.3 Bean的生命周期之5步

Bean生命周期的管理，可以参考Spring的源码：**AbstractAutowireCapableBeanFactory类的doCreateBean()方法****。**

Bean生命周期可以粗略的划分为五大步：

- 第一步：实例化Bean (调用无参构造方法)
- 第二步：给Bean属性赋值(调用set方法)
- 第三步：初始化Bean(会调用Bean的init方法，注意：这个init方法需要自己写，自己配置)
- 第四步：使用Bean
- 第五步：销毁Bean(会调用Bean的destroy方法，注意：这个destroy方法需要自己写)

![image-20221114164223213](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221114164223213.png)



```java
public class User {

    private String name;

    public void setName(String name) {
        System.out.println("step2:给对象的属性赋值...");
        this.name = name;
    }

    public User() {
        System.out.println("step1:无参构造方法执行...");
    }

    //这个方法自己写，方法名随意
    public void initBean() {
        System.out.println("step3:初始化Bean...");
    }

    //这个方法自己写，方法名随意
    public void destroyBean() {
        System.out.println("step5:销毁Bean...");
    }
}
```



配置文件：

```xml
<!--需要手动指定初始化方法和销毁方法-->
<bean id="userBean" class="com.codermartinn.spring.bean.User" init-method="initBean" destroy-method="destroyBean">
    <property name="name" value="张三"/>
</bean>
```



测试程序：

```java
@Test
public void test1() {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    User userBean = applicationContext.getBean("userBean", User.class);
    System.out.println("step4:使用bean " + userBean);
    //注意，必须手动关闭Spring container，这样Spring container才会销毁Bean
    ClassPathXmlApplicationContext context = (ClassPathXmlApplicationContext) applicationContext;
    context.close();
}
```

output:

```java
step1:无参构造方法执行...
step2:给对象的属性赋值...
step3:初始化Bean...
step4:使用bean com.codermartinn.spring.bean.User@1d296da
step5:销毁Bean...
```



###  8.4 Bean的生命周期之7步

在以上的5步中，第3步是初始化Bean，如果你还想在初始化前和初始化后添加代码，可以加入“Bean后处理器”。



Bean生命周期划分为七大步：

- 第一步：实例化Bean (调用无参构造方法)
- 第二步：给Bean属性赋值(调用set方法)
- 第三步：执行`Bean后处理器`的before方法
- 第四步：初始化Bean(会调用Bean的init方法，注意：这个init方法需要自己写，自己配置)
- 第五步：执行`Bean后处理器`的after方法
- 第六步：使用Bean
- 第七步：销毁Bean(会调用Bean的destroy方法，注意：这个destroy方法需要自己写)





编写一个类实现BeanPostProcessor类，并且重写before和after方法：

```java
public class LogBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("执行Bean后处理器的before方法");
        return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("执行Bean后处理器的after方法");
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }
}
```

在spring.xml文件中配置“Bean后处理器”：

```xml
<!--配置Bean后处理器-->
<!--注意：这个Bean后处理器作用于当前xml文件所有的bean-->
<bean class="com.codermartinn.spring.bean.LogBeanPostProcessor"/>
```

**一定要注意：在spring.xml文件中配置的Bean后处理器将作用于当前配置文件中所有的Bean。**



运行测试程序：

```java
@Test
public void test1() {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    User userBean = applicationContext.getBean("userBean", User.class);
    System.out.println("step4:使用bean " + userBean);
    //注意，必须手动关闭Spring container，这样Spring container才会销毁Bean
    ClassPathXmlApplicationContext context = (ClassPathXmlApplicationContext) applicationContext;
    context.close();
}
```

output：

```java
step1:无参构造方法执行...
step2:给对象的属性赋值...
执行Bean后处理器的before方法
step3:初始化Bean...
执行Bean后处理器的after方法
step4:使用bean com.codermartinn.spring.bean.User@34e9fd99
step5:销毁Bean...
```

如果加上Bean后处理器的话，Bean的生命周期就是7步了：

![image-20221115143815966](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221115143815966.png)



### 8.5 Bean生命周期之10步

如果根据源码跟踪，可以划分更细粒度的步骤，10步：

![image](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image.png)

上图中检查Bean是否实现了Aware的相关接口是什么意思？

Aware相关的接口包括：BeanNameAware、BeanClassLoaderAware、BeanFactoryAware

- 当Bean实现了BeanNameAware，Spring会将Bean的名字传递给Bean。
- 当Bean实现了BeanClassLoaderAware，Spring会将加载该Bean的类加载器传递给Bean。
- 当Bean实现了BeanFactoryAware，Spring会将Bean工厂对象传递给Bean。

测试以上10步，可以让User类实现5个接口，并实现所有方法：

- BeanNameAware
- BeanClassLoaderAware
- BeanFactoryAware
- InitializingBean
- DisposableBean



```java
public class User implements BeanNameAware, BeanClassLoaderAware, BeanFactoryAware, InitializingBean,DisposableBean {

    private String name;

    public void setName(String name) {
        System.out.println("step2:给对象的属性赋值...");
        this.name = name;
    }

    public User() {
        System.out.println("step1:无参构造方法执行...");
    }

    //这个方法自己写，方法名随意
    public void initBean() {
        System.out.println("step4:初始化Bean...");
    }

    //这个方法自己写，方法名随意
    public void destroyBean() {
        System.out.println("step7:销毁Bean...");
    }

    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println("这个bean的类加载器：" + classLoader);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("这个bean的工厂对象：" + beanFactory);
    }

    @Override
    public void setBeanName(String s) {
        System.out.println("这个bean的名字：" + s);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean的afterPropertiesSet()执行了...");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean的destroy()执行了...");
    }
}
```



运行测试程序：

```java
@Test
public void test1() {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    User userBean = applicationContext.getBean("userBean", User.class);
    System.out.println("step6:使用bean " + userBean);
    //注意，必须手动关闭Spring container，这样Spring container才会销毁Bean
    ClassPathXmlApplicationContext context = (ClassPathXmlApplicationContext) applicationContext;
    context.close();
}
```

output:

```java
step1:无参构造方法执行...
step2:给对象的属性赋值...
这个bean的名字：userBean
这个bean的类加载器：jdk.internal.loader.ClassLoaders$AppClassLoader@63947c6b
这个bean的工厂对象：org.springframework.beans.factory.support.DefaultListableBeanFactory@147ed70f: defining beans [userBean,com.codermartinn.spring.bean.LogBeanPostProcessor#0]; root of factory hierarchy
step3:执行Bean后处理器的before方法
InitializingBean的afterPropertiesSet()执行了...
step4:初始化Bean...
step5:执行Bean后处理器的after方法
step6:使用bean com.codermartinn.spring.bean.User@702657cc
DisposableBean的destroy()执行了...
step7:销毁Bean...
```

**通过测试可以看出来：**

- **InitializingBean的方法早于init-method的执行。**
- **DisposableBean的方法早于destroy-method的执行。**

对于SpringBean的生命周期，掌握之前的7步即可。够用。



### 8.6 Bean的作用域不同，管理方式不同

Spring container只对`singleton`的bean进行完整的生命周期管理。

如果是`prototype`的bean，Spring container 只负责将该bean初始化完毕。等客户端一旦获取到该bean之后，Spring container 不再管理该bean的生命周期了。



### 8.7 自己new的对象如何让Spring管理

有些时候可能会遇到这样的需求，某个java对象是我们自己new的，然后我们希望这个对象被Spring容器管理，怎么实现？

```java
@Test
public void test2() {
    // 自己new 对象
    Student student = new Student();
    System.out.println(student);

    // 将自己new 出来的对象交给Spring container管理
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    factory.registerSingleton("studentBean", student);


    //从Spring container中获取
    Object studentBean = factory.getBean("studentBean");
    System.out.println(studentBean);
}
```

output:

```java
com.codermartinn.spring.bean.Student@1134affc
com.codermartinn.spring.bean.Student@1134affc
```

