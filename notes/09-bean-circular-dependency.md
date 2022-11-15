## 09.bean-circular-dependency

### 9.1 什么是Bean的循环依赖

A对象中有B属性。B对象中有A属性。这就是循环依赖。我依赖你，你也依赖我。

比如：丈夫类Husband，妻子类Wife。Husband中有Wife的引用。Wife中有Husband的引用。

![image-20221115154507890](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221115154507890.png)



Husband：

```java
public class Husband {
    private String name;
    private Wife wife;

    public void setName(String name) {
        this.name = name;
    }

    public void setWife(Wife wife) {
        this.wife = wife;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Husband{" +
                "name='" + name + '\'' +
                ", wife=" + wife.getName() +
                '}';
    }
}
```

Wife:

```java
public class Wife {
    private String name;
    private Husband husband;

    public void setName(String name) {
        this.name = name;
    }

    public void setHusband(Husband husband) {
        this.husband = husband;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Wife{" +
                "name='" + name + '\'' +
                ", husband=" + husband.getName() +
                '}';
    }
}
```





### 9.2 singleton下的set注入产生的循环依赖

测试一下在singleton+setter的模式下产生的循环依赖，Spring是否能够解决？

spring.xml 文件

```xml
<bean id="husbandBean" class="com.codermartinn.spring.bean.Husband" scope="singleton">
    <property name="name" value="张三"/>
    <property name="wife" ref="wifeBean"/>
</bean>

<bean id="wifeBean" class="com.codermartinn.spring.bean.Wife" scope="singleton">
    <property name="name" value="小花"/>
    <property name="husband" ref="husbandBean"/>
</bean>
```



测试程序：

```java
@Test
public void test1(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");

    Husband husbandBean = applicationContext.getBean("husbandBean", Husband.class);
    System.out.println(husbandBean);

    Wife wifeBean = applicationContext.getBean("wifeBean", Wife.class);
    System.out.println(wifeBean);
}
```

output:

```java
Husband{name='张三', wife=小花}
Wife{name='小花', husband=张三}
```

**singleton + setter注入模式下的循环依赖是没有任何问题的。**

singleton表示在整个Spring容器当中是单例的，独一无二的对象。

```markdown
在singleton + setter模式下，为什么循环依赖不会出现问题，Spring是如何应对的？
    主要的原因是，在这种模式下Spring对Bean的管理主要分为清晰的两个阶段：
        第一个阶段：在Spring容器加载的时候，实例化Bean，只要其中任意一个Bean实例化之后，马上进行 “曝光”【不等属性赋值就曝光】
        第二个阶段：Bean“曝光”之后，再进行属性的赋值(调用set方法。)。

    核心解决方案是：实例化对象和对象的属性赋值分为两个阶段来完成的。

注意：只有在scope是singleton的情况下，Bean才会采取提前“曝光”的措施。
```





### 9.3 prototype下的set注入产生的循环依赖

prototype+set注入的方式下，循环依赖会不会出现问题？

```xml
<bean id="husbandBean" class="com.codermartinn.spring.bean.Husband" scope="prototype">
    <property name="name" value="张三"/>
    <property name="wife" ref="wifeBean"/>
</bean>

<bean id="wifeBean" class="com.codermartinn.spring.bean.Wife" scope="prototype">
    <property name="name" value="小花"/>
    <property name="husband" ref="husbandBean"/>
</bean>
```



执行测试程序：发生了异常，异常信息如下：

Caused by: org.springframework.beans.factory.**BeanCurrentlyInCreationException**: Error creating bean with name 'husbandBean': Requested bean is currently in creation: Is there an unresolvable circular reference?

​	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:265)

​	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199)

​	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:325)

​	... 44 more



翻译为：创建名为“husbandBean”的bean时出错：请求的bean当前正在创建中：是否存在无法解析的循环引用？

通过测试得知，当循环依赖的**所有Bean**的scope="prototype"的时候，产生的循环依赖，Spring是无法解决的，会出现**BeanCurrentlyInCreationException**异常。

大家可以测试一下，以上两个Bean，如果其中一个是singleton，另一个是prototype，是没有问题的。

**注意：当两个bean的scope都是prototype的时候，才会出现异常。如果其中任意一个是singleton的，就不会出现异常。**



### 9.4 singleton下的构造注入产生的循环依赖

Husband:

```java
public class Husband {
    private String name;
    private Wife wife;

    public Husband(String name, Wife wife) {
        this.name = name;
        this.wife = wife;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Husband{" +
                "name='" + name + '\'' +
                ", wife=" + wife.getName() +
                '}';
    }
}
```

Wife:

```java
public class Wife {
    private String name;
    private Husband husband;

    public Wife(String name, Husband husband) {
        this.name = name;
        this.husband = husband;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Wife{" +
                "name='" + name + '\'' +
                ", husband=" + husband.getName() +
                '}';
    }
}
```

配置文件：

```xml
<!--构造注入，这种循环依赖有没有问题？-->
<!--注意：基于构造注入的方式下产生的循环依赖也是无法解决的，所以编写代码时一定要注意。-->
<bean id="h" scope="singleton" class="com.codermartinn.spring.bean2.Husband">
    <constructor-arg index="0" value="张三"/>
    <constructor-arg index="1" ref="w"/>
</bean>

<bean id="w" scope="singleton" class="com.codermartinn.spring.bean2.Wife">
    <constructor-arg index="0" value="小花"/>
    <constructor-arg index="1" ref="h"/>
</bean>
```



测试程序：

```java
@Test
public void test2(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring2.xml");

    Husband husbandBean = applicationContext.getBean("h", Husband.class);
    System.out.println(husbandBean);

    Wife wifeBean = applicationContext.getBean("w", Wife.class);
    System.out.println(wifeBean);
}
```

运行报错：

```java
警告: Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'h' defined in class path resource [spring2.xml]: Cannot resolve reference to bean 'w' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'w' defined in class path resource [spring2.xml]: Cannot resolve reference to bean 'h' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'h': Requested bean is currently in creation: Is there an unresolvable circular reference?

org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'h' defined in class path resource [spring2.xml]: Cannot resolve reference to bean 'w' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'w' defined in class path resource [spring2.xml]: Cannot resolve reference to bean 'h' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'h': Requested bean is currently in creation: Is there an unresolvable circular reference?
```

和上一个测试结果相同，都是提示产生了循环依赖，并且Spring是无法解决这种循环依赖的。
为什么呢？
**主要原因是因为通过构造方法注入导致的：因为构造方法注入会导致实例化对象的过程和对象属性赋值的过程没有分离开，必须在一起完成导致的。**



### 9.5 Spring解决循环依赖的机理 **

Spring为什么可以解决set + singleton模式下循环依赖？

根本的原因在于：这种方式可以做到将“实例化Bean”和“给Bean属性赋值”这两个动作分开去完成。

实例化Bean的时候：调用无参数构造方法来完成。**此时可以先不给属性赋值，可以提前将该Bean对象“曝光”给外界。**

给Bean属性赋值的时候：调用setter方法来完成。

**两个步骤是完全可以分离开去完成的，并且这两步不要求在同一个时间点上完成。**

也就是说，Bean都是单例的，我们可以先把所有的单例Bean实例化出来，放到一个集合当中（我们可以称之为缓存），所有的单例Bean全部实例化完成之后，以后我们再慢慢的调用setter方法给属性赋值。这样就解决了循环依赖的问题。



```markdown
源码分析：
DefaultSingletonBeanRegistry类中有三个比较重要的缓存：
    private final Map<String, Object> singletonObjects                  一级缓存
    private final Map<String, Object> earlySingletonObjects             二级缓存
    private final Map<String, ObjectFactory<?>> singletonFactories      三级缓存

    这三个缓存都是Map集合。
    Map集合的key存储的都是bean的name（bean id）。

    一级缓存存储的是：单例Bean对象。完整的单例Bean对象，也就是说这个缓存中的Bean对象的属性都已经赋值了。是一个完整的Bean对象。
    二级缓存存储的是：早期的单例Bean对象。这个缓存中的单例Bean对象的属性没有赋值。只是一个早期的实例对象。
    三级缓存存储的是：单例工厂对象。这个里面存储了大量的“工厂对象”，每一个单例Bean对象都会对应一个单例工厂对象。
                    这个集合中存储的是，创建该单例对象时对应的那个单例工厂对象。

```

