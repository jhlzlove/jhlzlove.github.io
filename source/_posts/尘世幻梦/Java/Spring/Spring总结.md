---
title: Spring使用总结
categories:
  - Java
  - Spring
abbrlink: ece26077
---

Spring使用总结

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

<!-- code_chunk_output -->
- [一、Spring](#一spring)
- [二、Spring的核心API：ApplicationContext](#二spring的核心apiapplicationcontext)
- [三、Spring工厂创建复杂对象](#三spring工厂创建复杂对象)
- [四、Spring配置文件的细节](#四spring配置文件的细节)
  - [1. 只配置bean的class属性](#1-只配置bean的class属性)
  - [2. name别名的使用](#2-name别名的使用)
  - [3. ref标签](#3-ref标签)
- [五、Spring注入方式](#五spring注入方式)
  - [1. set注入的简化写法](#1-set注入的简化写法)
  - [2. 构造注入](#2-构造注入)
- [六、Spring工厂创建复杂对象的三种方式](#六spring工厂创建复杂对象的三种方式)
  - [1. FactoryBean 接口](#1-factorybean-接口)
  - [2. 实例工厂](#2-实例工厂)
  - [3. 静态工厂](#3-静态工厂)
- [七、如何控制Spring工厂创建对象的次数](#七如何控制spring工厂创建对象的次数)
  - [1. 控制简单对象的创建次数](#1-控制简单对象的创建次数)
  - [2. 控制复杂对象的创建次数](#2-控制复杂对象的创建次数)
- [八、对象的生命周期](#八对象的生命周期)
  - [1. 创建阶段](#1-创建阶段)
  - [2. 初始化阶段](#2-初始化阶段)
  - [3. 销毁阶段](#3-销毁阶段)
- [九、配置文件参数化](#九配置文件参数化)
- [十、类型转换器](#十类型转换器)
  - [1. 自定义类型转换器实现](#1-自定义类型转换器实现)
- [十一、后置处理Bean](#十一后置处理bean)
- [十二、静态、动态代理的概念](#十二静态动态代理的概念)
  - [1. 静态代理](#1-静态代理)
  - [2. 动态代理](#2-动态代理)
- [十三、Spring AOP（Aspect Oriented Programing） 编程](#十三spring-aopaspect-oriented-programing-编程)
  - [1. 开发步骤](#1-开发步骤)
  - [2. AOP的底层实现原理](#2-aop的底层实现原理)
    - [2.1 JDK动态代理](#21-jdk动态代理)
    - [2.2 CGlib 动态代理](#22-cglib-动态代理)
  - [3. 基于注解的AOP开发](#3-基于注解的aop开发)
  - [4. AOP开发过程中的坑](#4-aop开发过程中的坑)
  - [5. AOP总结](#5-aop总结)
- [十四、Spring的事务管理](#十四spring的事务管理)
  - [1. 事务并发产生的问题](#1-事务并发产生的问题)
    - [1.1 脏读](#11-脏读)
    - [1.2 不可重复读](#12-不可重复读)
    - [1.3 幻影读](#13-幻影读)
  - [2. 传播属性](#2-传播属性)
    - [2.1 只读属性(readOnly)](#21-只读属性readonly)
    - [2.2 超时属性(timeout)](#22-超时属性timeout)
    - [2.3 异常属性(rollbackFor)](#23-异常属性rollbackfor)
    - [2.4 事务属性常见配置总结](#24-事务属性常见配置总结)
- [十五、Spring MVC](#十五spring-mvc)
  - [1. 为什么要整合MVC框架](#1-为什么要整合mvc框架)
  - [2. Spring 可以整合那些 MVC 框架](#2-spring-可以整合那些-mvc-框架)
  - [3. Spring整合MVC框架的核心思路](#3-spring整合mvc框架的核心思路)
    - [3.1 准备工厂](#31-准备工厂)
  - [4. Spring工厂创建对象的优先级](#4-spring工厂创建对象的优先级)

<!-- /code_chunk_output -->

## 一、Spring

Spring 是一个轻量级的解决方案，它有两大核心内容：AOP 和反转控制。

1. 反转控制（Inverse of Control）
   - 反转： 赋值交给 Spring，解耦合
   - 把对于成员变量赋值的控制权，从代码反转到 Spring 工厂和配置文件中完成。
   - 底层实现： 工厂设计模式。

2. 依赖注入DI（Dependency Injection）

   - 注入： 通过 Spring 的工厂及配置文件，为对象（bean、组件）的成员变量赋值。
   - 依赖注入： 当以一个类需要另一个类时，就产生了依赖，一旦出现依赖，就可以把另一个类作为本类的成员变量，最终通过 Spring 配置文件进行注入（赋值）。

## 二、Spring的核心API：ApplicationContext

ApplicationContext 是 Spring 的核心接口，它用于对象的创建，可以把它当作一个 Bean 工厂。使用这个接口可以屏蔽实现的差异，从而解耦合。

ApplicationContext 包括：

1. ClassPathXmlApplicationContext (非 Web 环境：main junit)
2. XmlWebApplicationContext（Web 环境）

另外，Application 是一个重量级资源，对于这类资源，我们不应去频繁的创建。

## 三、Spring工厂创建复杂对象

1. 创建工厂类型；
2. 配置文件的配置 ApplicationContext.xml
3. 通过工厂类获得对象
  ApplicationContext
    |-ClassPathXmlApplicationContext
    |-WebXmlApplicationContext

什么是复杂对象？
不能通过 **new** 关键字的构造方法创建的对象。例如，jdbc 的 **Connection** 对象，Mybatis 中的 **SqlSessionFactory** 等。

什么是简单对象？
可以直接通过 **new** 构造方法创建对象，这样的对象叫做简单对象。

接口加反射，什么都能做。Spring 工厂是可以调用对象私有的构造方法创建对象，其中大量使用反射来获取信息帮助我们创建对象，这就是 Spring 工厂比我们自己创造的简易工厂强大的地方。

## 四、Spring配置文件的细节

### 1. 只配置bean的class属性

```xml{.line-numbers}
<!-- 该配置不含 id 属性 -->
<bean class="com.xxx.User"></bean>
```

> 应用场景：如果这个 bean 只需要使用一次，那么就可以省略 id 值。如果 bean 会使用多次，或者被其他 bean 引用则需要设置 id 值。

### 2. name别名的使用

```xml{.line-numbers}
<!-- name 就是一个别名，使用 getBean(name) 方法同样也可以创建 bean 对象，与 getBean(id) 是等效的。 -->
<bean id="user" name="a,b,c" class="com.xxx.User">
```

id 和 name 的不同：

1. 别名可以定义多个，id 只能定义一个（比如，人的大名只有一个，小名可以有多个）
2. XML 对于 id 属性的值，命名要求：必须以字母开头，后面是字母、数字、下划线、连字符。不能以特殊字符开头。name 属性值没有要求。因此，name 可以用于比较特殊命名的场景下 **^[注①]^**。
3. 代码
  containBeanDefinition(id) 只能判断 id 是否存在，不能判断 name；containBean() 可以判断 id，也可以判断 name。

> **注①：** XML 发展到了今天：ID 属性值的限制已经不存在了。这就是语言不断更新发展的好处。

### 3. ref标签

```xml{.line-numbers}
<bean id="userDao" class="com.xxx.UserDaoImpl">
</bean>
<bean id="userService" class="com.xxx.UserServiceImpl">
  <property name="userDao">
    <ref bean="userDao" />
  </property>
</bean>
```

> Spring 4.x 废除了 `<ref local=""/>`，它和 `<ref bean=""/>` 基本等效。但是前者只能引用本配置文件；后者除了可以引用本配置文件，还可以引用父配置文件。

## 五、Spring注入方式

|名称|举例|所属|说明|
|--|--|--|--|
|setter注入|setxxx()|
|自动注入|@Autowired|Spring提供|默认根据类型注入|
|自动注入|@Resource|JavaEE规范|默认根据名称注入|
|构造方法注入|public xxxConstruct()||

对于 @Autowired 和 @Resource，如果按照默认的类型找不到目标类的话，会自动使用另一种方式去查找。

属性注入：@Value 注入 map 集合时，文件中必须使用 json 格式赋值，使用 `#{${属性}}` 取值，map的键如果相同，后面的值会覆盖前面的值。

### 1. set注入的简化写法

1. 基于属性简化

    ```xml{.line-numbers}
    <!-- jdk 类型 -->
    <!-- 原始写法 -->
    <property name="id">
      <value>11</value>
    </property>
    <!-- 简化后写法 -->
    <property name="id" value="11" />
    <!-- value 属性只能简化 8 种基本类型 + String 标签 -->

    <!-- 用户自定义类型 -->
    <!-- 原始写法 -->
    <bean id="userService" class="com.xxx.UserServiceImpl">
      <property name="userDao">
        <ref bean="userDao" />
      </property>
    </bean>

    <!-- 简化后写法 -->
    <bean id="userService" class="com.xxx.UserServiceImpl">
      <property name="userDao" ref="userDao"/>
    </bean>
    ```

2. 基于命名空间 p 简化写法

    使用命名空间需要在 xml 的头声明中导入对应的 xsd 模板，否则会报错。

    ```xml{.line-numbers}
    <!-- jdk 类型简化写法 -->
    <bean id="person" class="" p:name="xiaoming" p:id="100"/>

    <!-- 自定义类型简化写法 -->
    <bean id="" class="" p:userDao-ref="userDao"/>
    ```

### 2. 构造注入

1. 提供有参构造方法

    ```java{.line-numbers}
    public class User {
        private String name;
        private Integer age;

        public User(String name, Integer age) {
          this.name = name;
          this.age = age;
        }
      }
    ```

2. Spring 的配置文件

    ```xml{.line-numbers}
      <bean id="" class="">
      <!-- 一个标签对应一个参数 -->
        <constructor-arg type="">
          <value>小明</value>
        </constructor-arg>
        <constructor-arg type="">
          <value>20</value>
        </constructor-arg>
      </bean>
    ```

    > 如果构造方法有重载，并且参数的个数相同，这个时候需要 `<constructor-arg type="">` 指明参数的类型才能完成注入。

## 六、Spring工厂创建复杂对象的三种方式

### 1. FactoryBean 接口

1. 实现 FactoryBean 接口的三个方法： getObject()，书写创建复杂对象的代码并返回复杂对象、getObjectType()，返回创建的复杂对象的 Class 对象、isSinglrton()，return true 只创建一个复杂对象，return false 每一次调用，都生成一个复杂对象。

2. Spring 配置文件的配置

```xml{.line-numbers}
<!-- 虽然配置是和简单对象是一样的，但是通过 id 获取的是这个类创建的复杂对象 Connection -->
  <bean id="conn" class="com.xxx.ConnectionFactoryBean">
```

如果想要获取 ConnectionFactoryBean 对象，需要 getBean("&conn")，就是在 id 的前面加上 `&`。

### 2. 实例工厂

```xml{.line-numbers}
<!-- 先声明实例对象，再引用 -->
<beann id="connFactory" class="com.xxx.ConnectionFactory"/>
<bean id="conn" factory-bean="connFactory" factory-method="getConnection"/>
```

### 3. 静态工厂

```xml
<!-- 直接使用 -->
<bean id="conn" class="com.xxx.xxxFactoryBean" factory-method="getConnection"/>
```

## 七、如何控制Spring工厂创建对象的次数

Q：为什么要控制对象的创建次数?
A：节省不必要的内存浪费。

- 什么样的对象只创建一次？
  - SqlSessionFactory
  - DAO
  - Service

- 什么样的对象每一次都要创建
  - Connection
  - SqlSession | Session
  - Controller | Struts2 Action

总之一句话：如果可以共用，并且是线程安全的，可以只创建一次。

### 1. 控制简单对象的创建次数

```xml{.line-numbers}
<bean id="" scope="singleton|prototype" class="">
```

singleton: 只会创建一次；
prototype: 每一次都会创建新的对象。
默认值 singleton。

### 2. 控制复杂对象的创建次数

实现 FactoryBean 的接口的复杂对象：
isSingleton() {
  return true;只会创建一次
  return false;每一次都会创建新的
}
如果没有 isSingleton() 方法，还是配置 scope 属性。

## 八、对象的生命周期

### 1. 创建阶段

`scope="singleton"`: Spring 工厂创建的同时（new ClassPathXmlApplicationContext()），对象被创建。
如果想要在 **singleton** 的情况下，使用 `getBean()` 时才能创建对象，可以加入 **lazy-init** 属性，`lazy-init="true"`

`scope="prototype"`: 在调用 getBean() 的时候创建对象。

### 2. 初始化阶段

Spring 工厂在创建完对象后，调用对象的初始化方法，完成对应的初始化操作。

1. 初始化方法的提供：程序员根据需求，提供初始化方法，最终完成初始化操作。
2. 初始化调用：Spring 工厂调用。

初始化实现方式：

1. 实现 InitializingBean 接口，重写 afterPropertiesSet() 方法。
2. 对象中提供一个普通的方法
  在配置文件中添加 init-method 属性，值为我们定义的普通的方法名称，注意不需要 `()`。

**Q:** 如果一个对象实现了 InitializingBean 接口，也提供了普通的方法，那么执行的顺序是什么样子的呢？
**A:** 先执行 InitializingBean，再执行普通方法。

### 3. 销毁阶段

Spring 销毁对象前，会调用对象的销毁方法，完成销毁。销毁方法的操作只适用于 `scope="singleton"` 的对象。销毁操作主要指资源的释放操作。例如 io，connection 等的关闭。

销毁方法：程序员根据自己的需求，定义销毁方法。
调用：Spring 工厂完成调用

实现方式：

- 实现 DisposableBean 接口，重写 destroy() 方法；
- 自定义销毁方法，在配置文件的 destroy-method 属性中写自定义的方法名。

**Q:** Spring 什么时候销毁对象？
**A:** ctx.close();

**Q:** 如果一个对象实现了 DisposableBean 接口，也提供了普通的销毁方法，那么执行的顺序是什么样子的呢？
**A:** 执行顺序：先执行 DisposableBean 接口的实现方法，再执行普通方法。

## 九、配置文件参数化

把Spring配置文件中需要经常修改的字符串信息，转移到一个更小的配置文件中。例如，数据库的连接可能在后期维护中会更换账户和密码，这时就可以把这些内容放到另一个文件中，这个文件专门存储变化的数据。

- 提供一个小的配置文件（.properties 文件），名字、位置任意。
- Spring 的配置文件和小配置文件整合。

`db.properties` 文件：

```properties{.line-numbers}
jdbc.driverClassName = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/jhlz?useSSL=false
jdbc.username = root
jdbc.password = root
```

`spring-config.xml` 文件：

```xml{.line-numbers}
<!-- 配置小文件的路径 -->
<context:property-placeholder location="classpath:/db.properties"/>

<bean id="conn" class="com.xxx.ConnectionFactoryBean">
  <property name="driverClassName" value="${jdbc.driverClassName}"/>
  <property name="url" value="${jdbc.url}"/>
  <property name="username" value="${jdbc.username}"/>
  <property name="password" value="${jdbc.password}"/>
</bean>
```

## 十、类型转换器

### 1. 自定义类型转换器实现

1. 实现 Converter 接口，定义转换的类型与实现。
2. 配置文件注册配置

```xml{.line-numbers}
<!-- 创建自定义转换对象 -->
<bean id="myDateConverter" class="com.xxx.MyDateConverter">
<!-- 注册类型转换器 -->
<!-- 目的：告诉 Spring 框架，开发的是一个类型转换器。 -->
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
  <property name="converters">
    <set>
      <ref bean="myDateConverter"/>
    </set>>
  </property>
</bean>
```

> 注册 ConversionServiceFactoryBean 时的 id 必须为 conversionService，大小写也需要一样。
> Spring 框架其实内置了日期类型的转换器，但只是针对 `2021/12/12` 这种格式，至于其它格式依然需要程序员手动实现。

## 十一、后置处理Bean

BeanPostProcess 作用： 对 Spring 工厂所创建的对象，进行再加工。

实现 BeanPostProcessor 接口，重写其中的方法。最好是在 after 的方法中写需要的逻辑。

> BeanPostProcessor 会对 Spring 工厂创建的所有对象都生效。

## 十二、静态、动态代理的概念

### 1. 静态代理

静态代理：为每一个原始类，手工编写一个代理类（有.java 和 .class文件）

由此我们可以知道静态代理存在的问题：

1. 静态文件数量过多，不利于项目管理；
2. 额外功能的维护性差（代理类中，额外功能修改复杂）。

### 2. 动态代理

1. JDK 动态代理 Proxy.newProxyInstance() 通过接口创建代理的实现类。
2. CGlib 动态代理 Enhancer 通过继承父类创建的代理类。

## 十三、Spring AOP（Aspect Oriented Programing） 编程

概念：基于 Spring 的动态代理开发，通过代理类为原始类增加额外功能。
好处：利于原始类的维护。
本质：JDK 动态代理。

思考：

1. AOP 如何创建动态代理类？（动态字节码技术）
2. Spring 的工厂是如何加工创建代理对象？（通过原始对象的id值，获得代理对象）（BeanPostProcessor）

Spring AOP： 默认使用 JDK 代理
SpringBoot AOP： 默认使用 CGlib 代理

### 1. 开发步骤

1. 原始对象
2. 额外功能（MethodInterCeptor）
3. 切入点
4. 组装切面（额外功能 + 切入点）

### 2. AOP的底层实现原理

#### 2.1 JDK动态代理

1. 类加载器的作用
通过了类加载器把对应类的字节码文件加载到JVM
通过类加载器创建类的 Class 对象，进而创建这个类的对象。

2. 如何获得类加载器？
每一个类的 `.class` 文件自动分配与之对应的 ClassLoader.

3. 动态代理创建的过程中，需要 ClassLoader 创建代理类的 Class 对象，可是因为动态代理类没有对应的 `.class` 文件，JVM 也就不会为其分配 ClassLoader，如何创建？
借用一个 ClassLoader，任意的 ClassLoader 都可以。

#### 2.2 CGlib 动态代理

CGlib创建动态代理的原理：父子继承关系创建代理对象，原始类作为父类，代理类作为子类，这样既可以保证两者方法一致，同时在代理类中提供新的实现（额外功能+原始方法）。

![CGlib创建代理对象](https://cdn.jsdelivr.net/gh/prettywinter/dist/images/doc/CGlib创建代理对象.png)

![CGlib代理](https://cdn.jsdelivr.net/gh/prettywinter/dist/images/doc/CGlib代理.png)

### 3. 基于注解的AOP开发

Spring 配置文件加入：

```xml{.line-numbers}
<!-- 告知Spring是基于注解的AOP -->
<aop:aspect-autoproxy/>
```

在默认情况下：AOP 编程底层使用 JDK 动态代理创建代理对象。那么如何切换到 CGlib 动态代理呢？有两种方式，一种是针对 **注解式** 开发，另一种是针对 **传统** 开发。

1. 注解式 AOP 开发：修改 Spring 的配置文件，在开启动态代理的标签中添加属性 `proxy-target-class="true"`

    ```xml{.line-numbers}
    <!-- 告知Spring是基于注解的AOP -->
    <aop:aspect-autoproxy proxy-target-class="true"/>
    ```

2. 传统 AOP 开发切换到 CGlib 配置

    ```xml{.line-numbers}
    <aop:config proxy-target-class="true">
      <aop:pointcut id="pc" expression="execution(* login(..)) or execution(* register(..))"/>
      <aop:advisor advice-ref="arround" pointcut-ref="pc"/>
    </aop:config>
    ```

### 4. AOP开发过程中的坑

在同一个的业务类中，进行业务方法间的相互调用，只有最外层方法才加入了额外功能，内部的方法通过普通的方式调用，调用的都是原始方法。如果想让内层的方法也通过代理对象调用，必须实现 ApplicationContextAware 接口获得工厂，进而获得代理对象。

### 5. AOP总结

![AOP 总结](https://cdn.jsdelivr.net/gh/prettywinter/dist/images/doc/AOP总结.png)

## 十四、Spring的事务管理

### 1. 事务并发产生的问题

#### 1.1 脏读

一个事务读取了另外一个事务没有提交的数据。
解决方案：@Transactional(isolation=Isolation.READ_COMMITTED)

#### 1.2 不可重复读

在同一个事务中，多次查询同一条数据，但是读取的结果不同。
解决方案：@Transactional(isolation=Isolation.REPEATABLE_READ)
本质：一把行锁

#### 1.3 幻影读

在一个事务中，对整表数据查询，显示的结果不一样。注意：是整表。会在本事务中出现数据不一致。
解决方案：@Transactional(isolation=Isolation.SERIALIZABLE)
本质：表级锁。

隔离属性在实战中的建议：推荐使用 Spring 指定的 ISOLATION_DEFAULT

  1. MySQL repeatable_read
  2. Oracle read_commited

> 在真正的项目中，并发访问情况是很低的。如果真的遇到并发问题，那么可以使用乐观锁解决，基于系统层面的，不会对系统的性能有太大的影响。
比如：Hibernate（JPA） Version、MyBatis 通过拦截器自定义开发。

### 2. 传播属性

概念：它描述了事务解决嵌套问题的特征。
什么是事务的嵌套：它指的是一个大的事务中，包含了若干个小的事务。
问题：大事务中融入了很多小的事务，它们彼此影响，最终就会导致外部大的事务失败，丧失了事务的原子性。
|属性名称|外部不存在事务|外部存在事务|用法|备注|
|--|--|--|--|--|
|REQUIRED|开启事务|融合到外部事务中|@Transactional(Propagation.REQUIRED)|增删改|
|SUPPORTS|不开启事务|融合到外部事务中|@Transactional(Propagation.SUPPORTS)|查询|
|REQUIRES_NEW|开启新的事务|挂起外部事务，开启新的事务|@Transactional(propagation.REQUIRES_NEW)|日志记录|
|NOT_SUPPORTED|不开启事务|挂起外部事务|@Transactional(Propagation.NOT_SUPPORTED)|极其不常用|
|NEVER|不开启事务|抛出异常|@Transactional(Propagation.NEVER)|极其不常用|
|MANDATORY|抛出异常|融合到外部事务|@Transactional(Propagation.MANDATORY)|极其不常用|

#### 2.1 只读属性(readOnly)

默认为 false，不开启。针对只查询的操作，可以设为 true 开启可以提高系统的运行效率。

#### 2.2 超时属性(timeout)

timeout 以秒为单位，默认值为 -1，最终由对应的数据库来指定。

#### 2.3 异常属性(rollbackFor)

Spring 的事务处理过程中，默认对于 RuntimeException 及其子类，采用的回滚的策略；
对于 Exception 及其子类，采用的是提交的策略。

手动指定：
rollbackFor = Exception.class
noRollbackFor = RuntimeException.class

> 建议在实战中使用 RuntimeException 及其子类，使用事务异常属性的默认值。

#### 2.4 事务属性常见配置总结

1. 隔离属性   默认值
2. 传播属性   Required（默认值）  增删改  Supports 查询操作
3. 只读属性   readOnly false  增删改  true  查询操作
4. 超时属性   默认值 -1
5. 异常属性   默认值

增删改操作  @Transaction
查询操作  @Transaction(propagation=Propagation.SUPPORTS, readOnly=true)

## 十五、Spring MVC

### 1. 为什么要整合MVC框架

1. MVC 框架提供了控制器调用 Service 请求响应的处理，
2. 接收请求参数 request.getParameter()
3. 控制程序的运行流程
4. 视图解析（jsp JSON Freemarker Thyemeleaf）

### 2. Spring 可以整合那些 MVC 框架

1. struts 1
2. webwork
3. jsf
4. struts 2
5. springMVC

### 3. Spring整合MVC框架的核心思路

#### 3.1 准备工厂

1. Web 开发过程中如何创建工厂
`ApplicationContext ctx = new new WebXmlApplicationContext("/applicationContext.xml");`

2. 如何保证工厂唯一同时被共用
工厂存储在 ServletContext 这个作用域中 ServletContext.setAttribute("xxxx", ctx);

唯一：
WebXmlApplicationContext

@Configuration的使用：
使用这个注解可以让 Spring 工厂创建 bean，可以不用在 applicationContext.xml 中编写 bean 标签。

创建工厂的代码改变了，需要使用 AnnotationConfigApplicationContext。指定配置 bean 的 Class。
`ApplicationContext ctx = new AnnotationConfigApplicationContext(xxxConfig.class);`
指定配置 bean 所在的路径。
`ApplicationContext ctx = new AnnotationConfigApplicationContext("com.config");`

### 4. Spring工厂创建对象的优先级

bean 标签 > @Bean > @Component 及其衍生注解。

- Spring 读取Properties文件：
PropertiesPlaceholderConfigurer
- 读取yml文件转换为Proterties：YamlPropertiesFactoryBean#setResources()YamlPropertiesFactoryBean#getObject()

Spring与YML集成依赖（最低所需版本：1.18）

```xml{.line-numbers}
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.30</version>
</dependency>
```
