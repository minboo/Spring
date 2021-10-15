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
## 2.4.3 p名称空间注入
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
      
          <bean id="book" class="com.atguigu.spring5.Book" p:bname="九阳神功" p:bauthor="无名氏">
    </bean>
```

就可以重新进行改造 `xmlns:p="http://www.springframework.org/schema/p"`

多了个p操作的不同 注意区别

## 2.4.4 xml注入其他属性
将value值设置一个null值

不是给value设置一个null而是使用一个null标签<null/>
```xml
<property name="address">
            <null/>
        </property>
```

属性值包含其他符号

如果value中包含《》会识别不出则需要转义

使用`CDATA<value><! [ CDATA [ 所需要的值 ] ]></value>`
```xml
<property name="address">
            <value><![CDATA[<<南京>>]]></value>
        </property>
```
## 2.4.5 注入属性外部bean

建立两个类
```java
public class UserService {//service类

    //创建UserDao类型属性，生成set方法
    private UserDao userDao;
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void add() {
        System.out.println("service add...............");
        userDao.update();//调用dao方法
    }
}

public class UserDaoImpl implements UserDao {//dao类

    @Override
    public void update() {
        System.out.println("dao update...........");
    }
}

```

spring的注入

此时属性值是对象，不能用value 需要用ref
```xml
<bean id="userService" class="com.atguigu.spring5.service.UserService">
    <!--注入userDao对象
        name属性：类里面属性名称
        ref属性：创建userDao对象bean标签id值
    -->
    <property name="userDao" ref="userDaoImpl"></property>
</bean>

<bean id="userDaoImpl" class="com.atguigu.spring5.dao.UserDaoImpl"></bean>

```

## 2.4.6 注入属性内部bean

设置两个类
```java
//部门类
public class Dept {
    private String dname;
    public void setDname(String dname) {
        this.dname = dname;
    }
}

//员工类
public class Emp {
    private String ename;
    private String gender;
    //员工属于某一个部门，使用对象形式表示
    private Dept dept;
    
    public void setDept(Dept dept) {
        this.dept = dept;
    }
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setGender(String gender) {
        this.gender = gender;
    }
}

```

通过spring的配置文件进行配置
```xml
<bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="lucy"></property>
        <property name="gender" value="女"></property>
        <!--设置对象类型属性-->
        <property name="dept">
            <bean id="dept" class="com.atguigu.spring5.bean.Dept">
                <property name="dname" value="安保部"></property>
            </bean>
        </property>
    </bean>
```
## 2.4.7注入属性级联赋值

只是xml格式不同

注意区分
```xml
<bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="lucy"></property>
        <property name="gender" value="女"></property>
        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
       
    </bean>
    <bean id="dept" class="com.atguigu.spring5.bean.Dept">
        <property name="dname" value="财务部"></property>
    </bean>
```
或者是使用对象调用

注意其中的区别`<property name="dept.dname" value="技术部"></property>`

但是在java类中需要生成name的get方法
```xml
<bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="lucy"></property>
        <property name="gender" value="女"></property>
        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
      
    </bean>
    <bean id="dept" class="com.atguigu.spring5.bean.Dept">
        <property name="dname" value="财务部"></property>
    </bean>
```

## 2.4.8注入数组/集合属性（含模板）
```xml
<bean id="对象名" class="包名"></bean>
```

在bean里面定义对象值使用porperty

由于name中的属性 value值只能是一个

而数组，集合的属性值为多个

标签值一一对应
```xml
<property name="courses">
            <array>
                <value>java课程</value>
                <value>数据库课程</value>
            </array>
        </property>
```
```xml
<!--list类型属性注入-->
        <property name="list">
            <list>
                <value>张三</value>
                <value>小三</value>
            </list>
        </property>
```
```xml
!--set类型属性注入-->
        <property name="sets">
            <set>
                <value>MySQL</value>
                <value>Redis</value>
            </set>
        </property>
```

注意区分map属性的value不同，使用的是entry属性

因为map哈希有key和value值
```xml
<!--map类型属性注入-->
        <property name="maps">
            <map>
                <entry key="JAVA" value="java"></entry>
                <entry key="PHP" value="php"></entry>
            </map>
        </property>
```
**以下为类名的文件**
```java
public class Stu {
    //1 数组类型属性
    private String[] courses;
    //2 list集合类型属性
    private List<String> list;
    //3 map集合类型属性
    private Map<String,String> maps;
    //4 set集合类型属性
    private Set<String> sets;

    //学生所学多门课程
    private List<Course> courseList;
    public void setCourseList(List<Course> courseList) {
        this.courseList = courseList;
    }

    public void setSets(Set<String> sets) {
        this.sets = sets;
    }
    public void setCourses(String[] courses) {
        this.courses = courses;
    }
    public void setList(List<String> list) {
        this.list = list;
    }
    public void setMaps(Map<String, String> maps) {
        this.maps = maps;
    }

    public void test() {
        System.out.println(Arrays.toString(courses));
        System.out.println(list);
        System.out.println(maps);
        System.out.println(sets);
        System.out.println(courseList);
    }
}
```
在测试类中

>加载xml配置以及调用
>
>加载配置文件的创建对象 调用文件路径
>
>然后获取对象的类名 最后的输出结果给类名的对象
```java
    @Test
    public void testCollection1() {
        ApplicationContext context =
                new ClassPathXmlApplicationContext("bean1.xml");
        类名 对象名 = context.getBean("对象名", 类名.class);
        对象名.test();
    }
```
## 2.4.9集合注入对象属性
```java
<bean id......>
<!--注入list集合类型，值是对象-->
        <property name="courseList">
            <list>
                <ref bean="course1"></ref>
                <ref bean="course2"></ref>
            </list>
        </property>
</bean>

<!--创建多个course对象-->
    <bean id="course1" class="com.atguigu.spring5.collectiontype.Course">
        <property name="cname" value="Spring5框架"></property>
    </bean>
    <bean id="course2" class="com.atguigu.spring5.collectiontype.Course">
        <property name="cname" value="MyBatis框架"></property>
    </bean>
```

## 2.4.10集合注入属性的公共部提取
使用util的名称空间注入

回顾p的注入
`xmlns:p="http://www.springframework.org/schema/p"`

所以util的注入类似

但xsi的位置也要多一个，把bean的名称都改为util

```xml
xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
```

```xml
<!--1 提取list集合类型属性注入-->
    <util:list id="bookList">
        <value>易筋经</value>
        <value>九阴真经</value>
        <value>九阳神功</value>
    </util:list>

    <!--2 提取list集合类型属性注入使用-->
    <bean id="book" class="com.atguigu.spring5.collectiontype.Book" scope="prototype">
        <property name="list" ref="bookList"></property>
    </bean>
```
## 2.5 IOC操作Bean管理（工厂bean)

> spring有两种类型，一种是上面的普通bean，另一种是工厂bean
> 
> 普通bean，在配置文件中，定义的bean类型就是返回类型
> 
> 而工厂bean返回类型可以不同
> 
> 1.创建类，让这个类作为工厂bean，实现接口FactoryBean
> 
> 2.实现接口内的方法，定义返回的bean类型
```java
public class MyBean implements FactoryBean<Course> {

    //定义返回bean
    @Override
    public Course getObject() throws Exception {
        Course course = new Course();
        course.setCname("abc");
        return course;
    }
}
```

```xml
<bean id="myBean" class="com.atguigu.spring5.factorybean.MyBean"></bean>
```
## 2.6 IOC操作Bean管理（bean作用域）

**1.spring本身默认的时候是单实例**

**测试代码**

```java
@Test
    public void testCollection2() {
        ApplicationContext context =
                new ClassPathXmlApplicationContext("bean2.xml");
        Book book1 = context.getBean("book", Book.class);
        Book book2 = context.getBean("book", Book.class);
       // book.test();
        System.out.println(book1);
        System.out.println(book2);
    }
```

**2.spring 配置文件下 bean 标签设置单实例还是多实例里面有属性（scope）用于设置单实例还是多实例参数设置以及区别**
```java
@Test
    public void testCollection2() {
        ApplicationContext context =
                new ClassPathXmlApplicationContext("bean2.xml");
        Book book1 = context.getBean("book", Book.class);
        Book book2 = context.getBean("book", Book.class);
       // book.test();
        System.out.println(book1);
        System.out.println(book2);
    }
```

**2.spring 配置文件下 bean 标签设置单实例还是多实例里面有属性（scope）用于设置单实例还是多实例参数设置以及区别**

> singleton 单实例对象，默认是这个。加载spring配置的时候就会创建对象
> 
> prototype 多实例对象。调用getBean方法的时候就会创建对象。
> 
> 测试代码用如上那个

**3.其他参数 request 以及session**

## 2.7 IOC操作Bean管理（bean生命周期）
生命周期：从创建对象到对象销毁的过程

过程：

> 1.为构造器创建bean的实列（无参构造）
> 
> 2.为bean的属性设置和对其他bean引用（调用set方法）
> 
> 3.bean初始化配置
> 
> 4.获取bean
> 
> 5.销毁bean
代码过程：
```xml
<bean id="orders" class="com.atguigu.spring5.bean.Orders" init-method="initMethod" destroy-method="destroyMethod">
        <property name="oname" value="手机"></property>
</bean>
```
**调用方法的时候要在bean 显示初始化以及销毁**

```java
public class Orders {

    //无参数构造
    public Orders() {
        System.out.println("第一步 执行无参数构造创建bean实例");
    }

    private String oname;
    public void setOname(String oname) {
        this.oname = oname;
        System.out.println("第二步 调用set方法设置属性值");
    }

    //创建执行的初始化的方法
    public void initMethod() {
        System.out.println("第三步 执行初始化的方法");
    }

    //创建执行的销毁的方法
    public void destroyMethod() {
        System.out.println("第五步 执行销毁的方法");
    }
}

```
```java
 @Test
    public void testBean3() {
//        ApplicationContext context =
//                new ClassPathXmlApplicationContext("bean4.xml");
        ClassPathXmlApplicationContext context =
                new ClassPathXmlApplicationContext("bean4.xml");
        Orders orders = context.getBean("orders", Orders.class);
        System.out.println("第四步 获取创建bean实例对象");
        System.out.println(orders);

        //手动让bean实例销毁
        context.close();
    }
```

**还有加上额外的另外两步骤在初始化之前和初始化之后**

> 把bean实例传递给bean后置处理器方法 放置在初始化之前和初始化之后 （放置在第3步骤前后）
> 
> 创建类，实现接口BeanPostProcessor，创建后置处理器

```java
public class MyBeanPost implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之前执行的方法");
        return bean;
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之后执行的方法");
        return bean;
    }
}

```

**在xml配置后置处理器**
```xml
<!--配置后置处理器-->
    <bean id="myBeanPost" class="com.atguigu.spring5.bean.MyBeanPost"></bean>
```

结果截图：

![image](https://user-images.githubusercontent.com/69302396/137337868-96910cf1-f144-465b-81dc-2c15014931b3.png)

## 2.8 IOC操作Bean管理（xml自动装配）
根据指定装配规则（名称或者类型），spring自动将其注入

手动注入，为外部bean的实现过程，可参考如上过程

自动注入 实现自动装配

> bean标签属性autowire，配置自动装配
> 
> autowire属性常用两个值：
> 
> byName根据属性名称注入 ，注入值bean的id值和类属性名称一样
> 
> byType根据属性类型注入


byName 如果外部bean有多个bean的话，只需要有个配对上了就可以
但ByType如果外部有多个bean的话，会出错

```xml
<bean id="emp" class="com.atguigu.spring5.autowire.Emp" autowire="byType">
        <!--<property name="dept" ref="dept"></property>-->
    </bean>
    <bean id="dept" class="com.atguigu.spring5.autowire.Dept"></bean>
```
以上为一个外部bean 但如果为多个外部bean，识别不上是哪一个
```xml
<bean id="emp" class="com.atguigu.spring5.autowire.Emp" autowire="byType">
        <!--<property name="dept" ref="dept"></property>-->
    </bean>
    <bean id="dept" class="com.atguigu.spring5.autowire.Dept"></bean>
    <bean id="dept1" class="com.atguigu.spring5.autowire.Dept"></bean>
```
## 2.9 IOC操作Bean管理（外部属性文件）
**1.通过配置数据库信息**
- 配置德鲁伊连接池
- 引入德鲁伊连接池的jar包
- 具体引入过程是 先复制jar包到工程lib下，在project structure的module中点加号添加进入
**2.引入外部属性文件配置数据库连接池**
而不是都放在xml文件，太多的name和value要修改

补充

1.直接配置连接池的配置信息
```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/userDb"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
```
2.创建尾部属性文件，properties格式文件，写数据库的信息
在src下创建一个jdbc.properties的文件

    prop.driverClass=com.mysql.jdbc.Driver
    prop.url=jdbc:mysql://localhost:3306/userDb
    prop.userName=root
    prop.password=123123

具体前面的名字可以随便写，只不过为了区分其他文件driverClass，以免混淆
后引入spring配置文件

**引入context名称空间**
```xml
xmlns:context="http://www.springframework.org/schema/context"
```
![image](https://user-images.githubusercontent.com/69302396/137338929-706e7ccf-17a6-449e-b72c-04c985ba0a50.png)

类似前面所讲的p名称空间
在spring配置文件使用标签引入外部属性文件

```xml
<!--引入外部属性文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!--配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${prop.driverClass}"></property>
        <property name="url" value="${prop.url}"></property>
        <property name="username" value="${prop.userName}"></property>
        <property name="password" value="${prop.password}"></property>
    </bean>
```
## 2.10 IOC操作Bean管理（基于注解方式）
**注解是什么？**

> 注解是代码特殊标记，格式：@注解名称（属性名称=属性值。。）
> 
> 可作用在类， 方法，属性上面
> 
> 注解目的：简化xml配置

**spring针对Bean管理中创建对象提供注解**

`@Component`： 游戏中普通的注解

`@Service`：业务逻辑层以及Service层

`@Controller`： 外部层

`@Repository`：dao层即持久层

功能是一样的，都可以用来创建对象，只不过把每个对象用在不同地方，以便查看

基于注解方式实现对象创建

**1.引入额外的aop依赖jar包**

![image](https://user-images.githubusercontent.com/69302396/137339395-4be6713f-864d-4773-8152-049039266621.png)

**2.开启组件扫描，使用context的名称空间**

这个context要通过名称空间引入
```xml
<!--开启组件扫描
 1 如果扫描多个包，多个包使用逗号隔开
 2 扫描包上层目录
-->
<context:component-scan base-package="com.atguigu"></context:component-scan>
```

base-package如果定义多个包，可以加全路径，分别用逗号隔开
或者是放置上层目录

**3.创建类，再类上面添加注解，注解可以选择上面四种**

（value等于之前bean的id值，默认是类名，而且首字母小写）

（value默认不写的话，直接@注解 即可）

```xml
//在注解里面value属性值可以省略不写，
//默认值是类名称，首字母小写
//UserService -- userService
//@Component(value = "userService")  //<bean id="userService" class=".."/>
@Service
public class UserService {
	public void add() {
        System.out.println("service add......."+name);
        userDao.add();
    }
}
```
**4.具体讲解开启组件扫描的详细配置**

如果引入的包中不加说明，默认会被所有包都扫描了

其实可以加一些说明，过滤或者添加说明只扫描一些包

自定义一个过滤，定义扫描的注解类

     use-default-filters="false" 表示现在不使用默认 filter，自己配置 filter
     context:include-filter ，设置扫描哪些内容
     type只扫描这种注解类
     expression表示扫描的为该注解类

```xml
<!--示例 1

-->
<context:component-scan base-package="com.atguigu" use-defaultfilters="false">
 <context:include-filter type="annotation"

expression="org.springframework.stereotype.Controller"/><!--代表只扫描Controller注解的类-->
</context:component-scan>
```
不定义一个过滤，扫面的所有内容，但可设置内容不扫描

    context:exclude-filter： 设置哪些内容不进行扫描
    

```xml
<!--示例 2

-->
<context:component-scan base-package="com.atguigu">
 <context:exclude-filter type="annotation"

expression="org.springframework.stereotype.Controller"/><!--表示Controller注解的类之外一切都进行扫描-->
</context:component-scan>
```

**5.基于注解方式实现属性注入**

定义两个类，一个接口实现类，一个类写函数
接口实现类
```java
@Repository(value = "userDaoImpl1")
public class UserDaoImpl implements UserDao {
    @Override
    public void add() {
        System.out.println("dao add.....");
    }
}
```
`@Autowired`：根据属性类型自动装配
```java
@Autowired  //根据类型进行注入
private UserDao userDao;
```
`@Qualifier(value=" ")`：根据属性名称自动注入
```java
@Autowired  //根据类型进行注入
@Qualifier(value = "userDaoImpl1") //根据名称进行注入
private UserDao userDao;
```
`@Resource`：可根据属性类型或者名称注入
```java
@Resource(name = "userDaoImpl1")  //根据名称进行注入
private UserDao userDao;
```
`@Value`：注入普通类型的注入

注解不是对象类型的定义，可以是字符串等其他
```java
@Value(value = "abc")
private String name;
```
**完全注解开发**

**主要是引用两个注解：**

`@Configuration` 作为配置类的提示

`@ComponentScan`扫描配置类的注解

创建配置类，替代xml配置文件
```java
@Configuration  //作为配置类，替代xml配置文件
@ComponentScan(basePackages = {"com.atguigu"})
public class SpringConfig {

}
```
测试类和之前不一样

之前是加载配置文件xml

现在是new一个对象，对象为注解的配置类，即加载这个配置类AnnotationConfigApplicationContext
```java
@Test
public void testService2() {
    //加载配置类
    ApplicationContext context
            = new AnnotationConfigApplicationContext(SpringConfig.class);
    UserService userService = context.getBean("userService", UserService.class);
    System.out.println(userService);
    userService.add();
}
```
