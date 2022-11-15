## 10.reflection-review



### 10.1 分析方法四要素

我们先来看一下，不使用反射机制调用一个方法需要几个要素的参与。

调用一个方法，四要素：

- 调用哪个对象的
- 哪个方法
- 传什么参数
- 返回什么值



```java
public class SomeService {
    public void doSome() {
        System.out.println("public void doSome()执行了...");
    }

    public String doSome(String s) {
        System.out.println("public String doSome(String s)执行了...");
        return s;
    }

    public String doSome(String s, int i) {
        System.out.println("public String doSome(String s, int i) 执行了...");
        return s + i;
    }
}
```



测试程序：

```java
@Test
public void test1(){
    //不使用反射机制
    SomeService someService  = new SomeService();
    someService.doSome();
    String s1 = someService.doSome("张三");
    String s2 = someService.doSome("张三",666);
}
```

output：

```java
public void doSome()执行了...
public String doSome(String s)执行了...
public String doSome(String s, int i) 执行了...
```



### 10.2 反射调用方法



```java
@Test
public void test2() throws Exception {
    //使用反射机制

    //获取类
    Class<?> clazz = Class.forName("codermartinn.reflect.SomeService");

    //获取方法
    Method doSome = clazz.getDeclaredMethod("doSome", String.class, int.class);

    //调用方法
    //四要素：调用哪个对象？哪个方法？传什么参数？返回什么值？
    //obj:调用哪个对象？---> 传入对象
    //哪个方法？---> doSome
    //传什么参数？---> args
    //返回什么值 --->  retVal
        //如何创建对象 newInstance(过时) or 无参数构造方法
        //Object o = clazz.newInstance();
    Constructor<?> constructor = clazz.getDeclaredConstructor();
    Object obj = constructor.newInstance();
    Object retVal = doSome.invoke(obj, "李四", 666);
}
```

ouput:

```java
public String doSome(String s, int i) 执行了...
```







### 10.3 使用反射给属性赋值

```java
public class User {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

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

知道以下这几条信息：

- 类名是：com.powernode.reflect.User

- 该类中有String类型的name属性和int类型的age属性。
- 另外你也知道该类的设计符合javabean规范。（也就是说属性私有化，对外提供setter和getter方法）

你如何通过反射机制给User对象的name属性赋值zhangsan，给age属性赋值20岁。

自己编写：

```java
@Test
public void test3() throws Exception{
    //获取类
    Class<?> clazz = Class.forName("codermartinn.reflect.User");

    //初始化
    Constructor<?> constructor = clazz.getDeclaredConstructor();
    Object obj = constructor.newInstance();

    //给属性赋值
    Field name = clazz.getDeclaredField("name");
    Field age = clazz.getDeclaredField("age");

    //设置私有成员变量可以访问
    name.setAccessible(true);
    age.setAccessible(true);

    name.set(obj, "张三");
    age.set(obj, 20);

    System.out.println(obj);
}
```

output:

```java
User{name='张三', age=20}
```



调用set方法给属性赋值:

```java
    @Test
    public void test4() throws Exception {
        //利用反射机制，使用set给age赋值


        String className = "codermartinn.reflect.User";
        String propertyName = "age";

        //获取类
        Class<?> clazz = Class.forName(className);

        //获取方法名
        String setMethod = "set" + propertyName.toUpperCase().charAt(0) + propertyName.substring(1);
        
        //获取属性类型
        Field field = clazz.getDeclaredField(propertyName);

        //获取方法
        Method method = clazz.getDeclaredMethod(setMethod, field.getType());

        //实例化对象
        Object obj = clazz.newInstance();

        //调用setAge方法
        method.invoke(obj, 20);

        System.out.println(obj);
    }
```

output:

```java
User{name='null', age=20}
```

