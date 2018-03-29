---
title: spring学习一-spring容器
date: 2018-03-12 11:08:57
tags: spring ioc
categories: spring
---

# 前言

学习spring官网上的文档，虽然都是英文，看起来会有点吃力，但是希望能够坚持下去，加油！！！
首先从spring的核心技术开始，spring的核心技术包括ioc容器和aop切面编程，下面就从ioc容器开始

# ioc容器和beans

## spring ioc 和 beans 简介

Inversion of Control (IoC 控制反转)还有一种叫法是dependency injection (DI 依赖注入)，叫法不同但其实表示的是同一个意思。要理解它们的意思就得看看它们和传统的初始化对象的方式有何不同。

* 传统方式：控制权在对象，我们在对象内部通过new创建对象，但无论怎么说控制权在对象中

* ioc方式：对象由容器创建，对象的依赖可以通过以下几种方式注入。

        1.创建类实例的时候将通过类的含参构造方法将依赖通过参数注入
        2.利用工厂方法创建类实例的时候通过工厂方法的参数将依赖注入实例
        3.创建实例之后通过属性的set方法将依赖注入实例

综上，我们可以看到，首先对象的创建从对象转移到了容器中，其次依赖的初始化也从对象中转移到了容器，这就是控制反转。

在spring中`org.springframework.beans`和`org.springframework.context`这两个包组成了spring框架的ioc容器。`BeanFactory`接口提供了一套对象配置机制来管理任何类型的对象。`ApplicationContext`接口继承了`BeanFactory`，除了`BeanFactory`的基本功能之外，它还增加了企业所需要的功能。

在spring中被ioc容器管理的对象就叫beans。一个bean就是一个被spring容器创建，装配和管理的类实例。

## 容器概述

`org.springframework.context.ApplicationContext`接口用来代表spring中的ioc容器,用来装配beans。容器根据configuration metadata来决定装配哪些beans。configuration metadata可以用xml、java注解、java代码三种方式来描述。

常用的`ApplicationContext`接口实现类有:

* **FileSystemXmlApplicationContext**:该容器从 XML 文件中加载已被定义的 bean。在这里，你需要提供给构造器 XML 文件的完整路径
* **ClassPathXmlApplicationContext**：该容器从 XML 文件中加载已被定义的 bean。在这里，你不需要提供 XML 文件的完整路径，只需正确配置 CLASSPATH 环境变量即可，因为，容器会从 CLASSPATH 中搜索 bean 配置文件。
* **WebXmlApplicationContext**：该容器会在一个 web 应用程序的范围内加载在 XML 文件中已被定义的 bean。

<div align="left">
    <img src="/images/spring学习一-spring容器/container-magic.png" alt="container-magic"/>
</div>

***Figure 1. The Spring IoC container***

### Configuration metadata

配置Configuration metadata的三种方式：

* xml:传统的配置方式，将<bean/>元素配置在顶层的<beans/>元素之中

* Annotation-based configuration:spring 2.5引入

* Java-based configuration:spring 3.0开始引入，关于使用java code配置的相关特性后面我们学习@Configuration, @Bean, @Import 和 @DependsOn等注解的时候再深入

### 初始化容器

直接上例子

初始化容器,例子使用的是`ClassPathXmlApplicationContext`实现类，直接传入xml配置文件的路径，这里的路径是相对于`CLASSPATH`的路径(如果是WebXmlApplicationContext的话，还需要在web.xml中配置一些初始化容器的参数，这个后面再说，这里先mark一下)

``` java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

`(services.xml)` configuration file:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```
`daos.xml` file:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

#### Composing XML-based configuration metadata

我们的项目中通常会有多个xml配置文件用来表示不同模块的beans。这种做法是非常有用的。那么我们如何加载这多个xml呢。第一种方式就是上面例子中的方法，直接给容器的构造方法传多个参数，还有一种方法就是用`<import/>`元素在xml文件中加载其他xml文件,例如：

``` xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```
>这里用的路径都是相对于`CLASSPATH`的路径，第一个`/`有没有都一样，推荐省略。当然你也可以使用绝对路径，不过这样做也不推荐，因为在不同的操作系统上会出问题。同样也不推荐在相对路径使用`../`。

### 使用容器

`T getBean(String name, Class<T> requiredType)`

``` java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

## Bean 概述

定义一个bean需要的metadata：

* 完整类路径名
* bean的作用域、生命周期回调等
* bean的依赖关系

***Table 1. The bean definition***

属性 | 描述
---|---
class |	这个属性是强制性的，并且指定用来创建 bean 的 bean 类。
name |	这个属性指定唯一的 bean 标识符。在基于 XML 的配置元数据中，你可以使用 ID 和/或 name 属性来指定 bean 标识符。
scope |	这个属性指定由特定的 bean 定义创建的对象的作用域，它将会在 bean 作用域的章节中进行讨论。
constructor-arg | 它是用来注入依赖关系的，并会在接下来的章节中进行讨论。
properties | 它是用来注入依赖关系的，并会在接下来的章节中进行讨论。
autowiring mode | 它是用来注入依赖关系的，并会在接下来的章节中进行讨论。
lazy-initialization mode |	延迟初始化的 bean 告诉 IoC 容器在它第一次被请求时，而不是在启动时去创建一个 bean 实例。
initialization 方法 |	在 bean 的所有必需的属性被容器设置之后，调用回调方法。它将会在 bean 的生命周期章节中进行讨论。
destruction 方法 | 当包含该 bean 的容器被销毁时，使用回调方法。它将会在 bean 的生命周期章节中进行讨论。


可以在应用程序运行期间向容器注册用户创建的对象，但尽量不要这么做

### 命名beans

* 用id属性定义唯一标识符：
``` xml
<bean id="id" class="className"/>
```

* 用name属性定义一个或多个标识符,多个标识符之间可以用`,` `;`或者`空格`分割
``` xml
<bean name="name1,name2,name3" class="className"/>
```

* 定义别名,如下所示三个标识符其实表示同一个bean
``` xml
<alias name="subsystemA-dataSource" alias="subsystemB-dataSource"/>
<alias name="subsystemA-dataSource" alias="myApp-dataSource" />
```

### 初始化beans

* 使用构造方法初始化bean

``` xml
<bean id="exampleBean" class="examples.ExampleBean"/>
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

`class`属性定义了类的全路径名，spring通过反射获得它的构造器来创建对象。这里用的是默认的无参构造器，后面在讲依赖的时候会提到如何用有参的构造器

* 使用静态工厂方法初始化bean

``` xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

``` java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

`class`属性定义了包含静态工厂方法的类的全路径名，`factory-method`定义了静态工厂方法的方法名。可以看到我们在xml中并没有定义静态工厂方法返回的类型。

* 使用实例工厂方法初始化bean

``` xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

``` java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```
`class`属性不用定义；`factory-bean`定义工厂类对象的名字，它指向容器中定义的工厂类bean；`factory-method`定义了实例工厂方法名

## 依赖

### 依赖注入

* 基于构造方法的依赖注入

当多个参数的类型之间不存在歧义的时候我们可以用下面的方式通过构造方法注入。
``` xml
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>

    <bean id="bar" class="x.y.Bar"/>

    <bean id="baz" class="x.y.Baz"/>
</beans>
```
**ps:这儿xml中构造方法参数的顺序有没有要求不太清楚，以后遇到了可以测试以下。**

当多个参数的类型之间存在歧义的时候怎么办呢？先看下面的例子。

``` java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

可以看到years和ultimateAnswer分别是int和String类型，像这种简单类型的值我们都可以传入字面量,例如`<value>true</value>`，这时候spring就不知道传入值的确切类型。碰到这种情况我们有以下几种方法。

1.明确参数类型
``` xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

2.明确参数顺序
``` xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

3.明确参数名
``` xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```
这里要注意的是java有debug和release两种编译方式，这两种编译方式会导致编译出来的参数名不一样，用debug编译方式的话上述配置没有问题，但是release编译方式的话就会导致spring反射失败从而无法注入。这时候我们就要用JDK的`@ConstructorProperties`注解显示得指明参数的名字(java的编译方式我们就不去深究了，如果想用这种方式注入，记得用`@ConstructorProperties`注解就行了)。
``` java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```  

