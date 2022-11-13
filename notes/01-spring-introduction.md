## 01.spring-introduction



### analyze the case

**struct:**

```java
├─main
│  ├─java
│  │  └─com
│  │      └─codermartinn    
│  │          └─spring      
│  │              ├─client  
│  │              ├─dao     
│  │              │  └─impl 
│  │              ├─service 
│  │              │  └─impl 
│  │              └─web     
│  └─resources              
└─test                      
    └─java  
```



**client:**

```java
/**
 * @Description: 测试类
 * @author: coderMartin
 * @date: 2022-11-12
 */
public class Test {
    public static void main(String[] args) {
        UserAction userAction = new UserAction();
        userAction.deleteRequest();
    }
}
```



**controller:**

```java
/**
 * @Description: 表示层
 * @author: coderMartin
 * @date: 2022-11-12
 */
public class UserAction {
    private UserService userService = new UserServiceImpl();


    /**
     * 删除用户信息的请求
     */
    public void deleteRequest(){
        userService.deleteUserInfo();
    }
}
```



**service:**

```java
/**
 * @Description: 业务层
 * @author: coderMartin
 * @date: 2022-11-12
 */
public interface UserService {
    /**
     * 删除用户信息
     */
    void deleteUserInfo();
}
```

```java
/**
 * @Description:
 * @author: coderMartin
 * @date: 2022-11-12
 */
public class UserServiceImpl implements UserService {
    UserDao userDao = new UserDaoImpl();

    @Override
    public void deleteUserInfo() {
        // 删除用户信息，是业务层调用持久层删除信息
        userDao.deleteById();
        // 处理业务的代码
        // ...
    }

}
```



**dao:**

```java
/**
 * @Description: 持久层
 * @author: coderMartin
 * @date: 2022-11-12
 */
public interface UserDao {

    /**
     * 根据id删除用户信息
     */
    void deleteById();
}
```

```java
/**
 * @Description:
 * @author: coderMartin
 * @date: 2022-11-12
 */
public class UserDaoImplForMySQL implements UserDao {
    @Override
    public void deleteById() {
        System.out.println("MySQL is deleting user information...");
    }
}
```



**output:**

```java
MySQL is deleting user information...
```



上面的案例存在什么问题呢？

**需求：**

如果，不想用`MySQL`数据库了，想换`Oracle`数据库，这时怎么办呢？

![image-20221112221453274](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221112221453274.png)

添加一个实现类，`UserDaoImplForOracle.java`

```java
/**
 * @Description: Oracle
 * @author: coderMartin
 * @date: 2022-11-12
 */

public class UserDaoImplForOracle implements UserDao {
    @Override
    public void deleteById() {
        System.out.println("Oracle is deleting user information...");    }
}
```

运行程序，发现业务层还是调用`MySQL`数据库。

```java
MySQL is deleting user information...
```

为什么呢？因为在业务层实现类中，固定写了`MySQL`数据库。

`UserDao userDao = new UserDaoImplForMySQL();`

```java
/**
 * @Description:
 * @author: coderMartin
 * @date: 2022-11-12
 */
public class UserServiceImpl implements UserService {
    UserDao userDao = new UserDaoImplForMySQL();

    @Override
    public void deleteUserInfo() {
        // 删除用户信息，是业务层调用持久层删除信息
        userDao.deleteById();
        // 处理业务的代码
        // ...
    }

}
```



如果要满足需求的话，就要修改旧的代码`UserDao userDao = new UserDaoImplForMySQL();`

不符合『设计模式』中的「开闭原则」！！！



### OCP开闭原则

```markdown
* 什么是OCP？
    OCP是软件七大开发原则当中最基本的一个原则：开闭原则
    对什么开？对扩展开放。
    对什么闭？对修改关闭。
* OCP原则是最核心的，最基本的，其他的六个原则都是为这个原则服务的。
* OCP开闭原则的核心是什么？
    只要你在扩展系统功能的时候，没有修改以前写好的代码，那么你就是符合OCP原则的。
    反之，如果在扩展系统功能的时候，你修改了之前的代码，那么这个设计是失败的，违背OCP原则。
* 当进行系统功能扩展的时候，如果动了之前稳定的程序，修改了之前的程序，之前所有程序都需要进行重新测试。这是不想看到的，因为非常麻烦。
```



### DIP原则

![image-20221112223034668](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221112223034668.png)

```
UserAction依赖了具体的UserServiceImpl
UserServiceImpl依赖了具体的UserDaoImplForMySQL
目前来说：上是依赖下的。
凡是上依赖下的，都违背了依赖倒置原则。

什么叫符合依赖倒置原则？
上 不再依赖  下 ，符合依赖倒置原则

什么是依赖倒置原则？
     面向接口编程，面向抽象编程，不要面向具体编程。
```



**summary**：

    * 什么是依赖倒置原则？
        面向接口编程，面向抽象编程，不要面向具体编程。
    * 依赖倒置原则的目的？
        降低程序的耦合度，提高扩展力。
    * 什么叫做符合依赖倒置？
        上 不依赖 下，就是符合。
    * 什么叫做违背依赖倒置？
        上 依赖 下，就是违背。
        只要“下”一改动，“上”就受到牵连。

![image-20221112224147638](https://codermartinn.oss-cn-guangzhou.aliyuncs.com/img/image-20221112224147638.png)

`UserDao userDao = new UserDaoImplForMySQL();`改为

`UserDao userDao;`



如何解决？

**controller**:

```java
/**
 * @Description: 表示层
 * @author: coderMartin
 * @date: 2022-11-12
 */
public class UserAction {
    //private UserService userService = new UserServiceImpl();
    
    // 面向接口
    private UserService userService;

    /**
     * 删除用户信息的请求
     */
    public void deleteRequest(){
        userService.deleteUserInfo();
    }
}
```

**service**:

```java
/**
 * @Description:
 * @author: coderMartin
 * @date: 2022-11-12
 */
public class UserServiceImpl implements UserService {
    //UserDao userDao = new UserDaoImplForMySQL();
    //UserDao userDao = new UserDaoImplForMySQL();

    // 面向接口
    UserDao userDao;

    @Override
    public void deleteUserInfo() {
        // 删除用户信息，是业务层调用持久层删除信息
        userDao.deleteById();
        // 处理业务的代码
        // ...
    }
}
```

如果代码是这样编写的，才算是完全面向接口编程，才符合依赖倒置原则。那你可能会问，这样`userDao`是`null`，在执行的时候就会出现空指针异常呀。你说的有道理，确实是这样的，所以我们要解决这个问题。解决空指针异常的问题，其实就是解决两个核心的问题：

- 第一个问题：谁来负责对象的创建。【也就是说谁来：new UserDaoImplForOracle()/new UserDaoImplForMySQL()】
- 第二个问题：谁来负责把创建的对象赋到这个属性上。【也就是说谁来把上面创建的对象赋给userDao属性】

如果我们把以上两个核心问题解决了，就可以做到既符合OCP开闭原则，又符合依赖倒置原则。





### IoC控制反转

    控制反转：IoC（Inversion of Control）
    反转是什么呢？
        反转的是两件事：
            第一件事：我不在程序中采用硬编码的方式来new对象了。（new对象我不管了，new对象的权利交出去了。）
            第二件事：我不在程序中采用硬编码的方式来维护对象的关系了。（对象之间关系的维护权，我也不管了，交出去了。）
    
    控制反转：是一种编程思想。或者叫做一种新型的设计模式。由于出现的比较新，没有被纳入GoF23种设计模式范围内。



控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计思想，可以用来降低代码之间的耦合度，符合依赖倒置原则。

控制反转的核心是：**将对象的创建权交出去，将对象和对象之间关系的管理权交出去，由第三方容器来负责创建与维护**。

控制反转常见的实现方式：依赖注入（Dependency Injection，简称DI）

通常，依赖注入的实现由包括两种方式：

- set方法注入

- 构造方法注入

而Spring框架就是一个实现了IoC思想的框架。

IoC可以认为是一种**全新的设计模式**，但是理论和时间成熟相对较晚，并没有包含在GoF中。（GoF指的是23种设计模式）



### Spring Framework

    * Spring框架实现了控制反转IoC这种思想
        Spring框架可以帮你new对象。
        Spring框架可以帮你维护对象和对象之间的关系。
    * Spring是一个实现了IoC思想的容器。
    * 控制反转的实现方式有多种，其中比较重要的叫做：依赖注入(Dependency Injection，简称DI)。
    * 控制反转是思想。依赖注入是这种思想的具体实现。
    * 依赖注入DI，又包括常见的两种方式：
        第一种：set注入（执行set方法给属性赋值）
        第二种：构造方法注入（执行构造方法给属性赋值）
    * 依赖注入 中 “依赖”是什么意思？ “注入”是什么意思？
        依赖：A对象和B对象的关系。
        注入：是一种手段，通过这种手段，可以让A对象和B对象产生关系。
        依赖注入：对象A和对象B之间的关系，靠注入的手段来维护。而注入包括：set注入和构造注入。