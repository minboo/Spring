# 2.IOC
## 2.1 IOC的处理过程

(1)IOC底层原理

(2) IOC接口(BeanFactory)

(3) IOC操作Bean管理（基于xml)

(4)IOC操作Bean管理(基于注解)

## 2.2 IOC底层原理
**1.xml解析**

xml配置文件，配置创建的对象
```xml
<bean id=" " class=" "></bean>
```
**2.工厂模式**

定义一个中间静态函数，降低其耦合度

**3.反射**

通过反射创建对象
```xml
Class xx=Class.forName(classValue);
xx.newInstance();
```
## 2.3 IOC接口
**IOC基于IOC容器，底层就是对象工厂**

1.`BeanFactory`，Spring内部使用接口，不提供给开发人员，加载配置xml解析不会创建，只有创建对象才会getBean

2.`ApplicationContext`是BeanFactory子接口，功能强大，开发人员可使用，加载配置就会创建对象

3.`ApplicationContext`有一些特别的实现类

特别是`ClassPathXmlApplicationContext`，在src目录下可以写文件名

如果是`FileSystemXmlApplicationContext`，在src目录下，必须写绝对路径

在idea下点击ctrl+H进入


![image](https://user-images.githubusercontent.com/69302396/137127686-f9dbaa72-a2dc-451c-8c07-9c1ced9515da.png)

**示例代码**

```java
//1 加载spring配置文件
ApplicationContext context =new ClassPathXmlApplicationContext("bean2.xml");

//2 获取配置创建的对象
UserService userService = context.getBean("userService", UserService.class);
```

## 2.4 IOC操作Bean管理（普通bean)

**什么是Bean管理？**

> Spring创建对象
>
> Spring注入属性


**有两种方式可以实现**

1.基于xml配置方式实现

2.基于注解方式实现


**在spring配置文件中，使用bean标签，在标签内加对应属性**

>id属性 取一个别名
>
>class属性 创建类中包的路径

在xml中配置文件<bean id="userService" class="com.atguigu.spring5.service.UserService">
  
在java文件中
  
```java
  //1 加载spring配置文件
ApplicationContext context =new ClassPathXmlApplicationContext("bean2.xml");

//2 获取配置创建的对象
UserService userService = context.getBean("userService", UserService.class);
```
创建完对象后，默认调用的是无参数构造函数完成对象创建
  
## 2.4.1 传统注入属性
  
常规思路常规代码的用法：
创建类，创造属性和方法

> 第一种使用set方式进行注入（原始做法）
>
> 第二种是有参函数进行构造
  
```java
public String getAddress() {
        return address;
    }
private String address;

Book book=new Book();//无参set注入
book.setAddress("123");

Book book=new Book("123");//调用有参数函数构造注入
```
## 2.4.2 xml注入属性
  
在spring配置文件中进行注入
  
set方法的注入属性
  
在xml文件下注入使用bean
  
用poperty完成属性注入

> poperty name是属性 value是值
```xml
<bean id="book" class="com.atguigu.spring5.Book">
<property name="码农研究僧" value="99999"></property>
</bean>
```
在java中加载配置文件以及创建对象
```java 
public void testBook1() {
        //1 加载spring配置文件
        ApplicationContext context =
                new ClassPathXmlApplicationContext("bean1.xml");

        //2 获取配置创建的对象
        Book book = context.getBean("book", Book.class);

        System.out.println(book);
        book.testDemo();
    }
```
  
定义一个book类
```java
public class Book {


    public void setBname(String bname) {
        this.bname = bname;
    }

    //创建属性
    private String bname;
    private String bauthor;


    private String address;

    //创建属性对应的set方法


    public void setBauthor(String bauthor) {
        this.bauthor = bauthor;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public void testDemo() {
        System.out.println(bname+"::"+bauthor+"::"+address);
    }

}
```
  
**第二种：有参函数进行构造注入属性区别只是在xml以及java的方法构造而已其他都一样**
```xml
<bean id="orders" class="com.atguigu.spring5.Orders">
        <constructor-arg name="oname" value="电脑"></constructor-arg>
        <constructor-arg name="address" value="China"></constructor-arg>
    </bean>
```
`constructor-arg`是有参构造函数的属性
