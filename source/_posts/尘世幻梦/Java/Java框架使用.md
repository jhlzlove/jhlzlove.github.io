---
title: Java框架使用总结
categories:
  - Java
music:
  server: netease
  type: song
  id: 440403990
abbrlink: fa647c74
img:
---

Java框架使用总结，主要涉及SSM。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

<!-- code_chunk_output -->

- [Java框架使用总结](#java框架使用总结)
  - [SSM](#ssm)
    - [Spring](#spring)
    - [Spring MVC](#spring-mvc)
      - [Spring MVC异常处理](#spring-mvc异常处理)
    - [Mybatis/Mybatis Plus](#mybatismybatis-plus)
      - [Mybatis Plus 多数据源配置使用](#mybatis-plus-多数据源配置使用)
      - [分页插件 使用](#分页插件-使用)
      - [saveBatch() 插入的性能问题](#savebatch-插入的性能问题)
      - [Mybatis plus 中不能插入或更新null值](#mybatis-plus-中不能插入或更新null值)
    - [Spring Boot](#spring-boot)
      - [注解](#注解)
      - [AOP：](#aop)
      - [文件的上传下载：](#文件的上传下载)
      - [拦截器(一般用于用户认证，其它不常用)](#拦截器一般用于用户认证其它不常用)
      - [打包(打包之前先清理(clean)再打包(package))](#打包打包之前先清理clean再打包package)
      - [SpringBoot解决跨域问题](#springboot解决跨域问题)
      - [Jasypt 加密](#jasypt-加密)
      - [SpringBoot 处理异常](#springboot-处理异常)
  - [SSH](#ssh)

<!-- /code_chunk_output -->

# Java框架使用总结
## SSM

### Spring
**Spring 管理事务的方式**
- 编程式事务，在代码中硬编码。(不推荐使用)
- 声明式事务，在配置文件中配置(推荐使用)
   > 声明式事务又分为两种：
   > - 基于XML的声明式事务
   > - 基于注解的声明式事务


### Spring MVC
1. 跳转到页面
   - 转发方式：直接 return 逻辑视图名
   - 重定向方式：重定向(redirect)不会经过视图解析器 redirect:/视图全名
2. 跳转到方法
   - 转发跳转相同 controller 的不同方法使用 springMVC 提供的 `forward:` 关键字
   - 重定向使用 `redirect:/controller的RequestMapping/方法的RequestMapping` 
不同 controller 的方法跳转，和相同的一样。

ResponseBody 底层使用的 Jackson
重定向到登录页面：`response.sendRedirect(request.getContextPath() + "/login.jsp")`

#### Spring MVC异常处理

实现 HandlerExceptionResolver 的接口，重写里面的 resolveException 方法。


### Mybatis/Mybatis Plus

没什么官网文档更详细的了：
- [Mybatis](https://mybatis.org/mybatis-3/zh/index.html)
- [Mybatis Plus](https://baomidou.com/guide/)

#### Mybatis Plus 多数据源配置使用

保障主库的数据安全，完成数据的同步。
读写分离：

引入依赖：
```xml{.line-numbers}
<groupId>com.baomidou</groupId>
<artifactId>dynamic-datasource-spring-boot-starter</artifactId>
<verwsion>3.4.5</verwsion>
```

配置：略


#### 分页插件 使用
Mybatis Plus 自带的分页：IPage、Page
IPage是一个接口，Page是IPage的实现类，
在使用时，可以使用Page创建一个对象实例，设置分页的一些参数；IPage对象用来存放自定义的分页规则和查询条件参数，比如以下简单例子：
`UserMapper.java`
```java{.line-numbers}
    //mapper中也可以使用 Page ，效果一样的
    IPage<User> getUserList(Page<User> page,@param("id") Integer id,@Param("startTime") String startTime,@Param("endTime") String endTime);
```
`UserService.java`
```java{.line-numbers}
    IPage<User> selectUserList(Integer id, String startTime, String endTime);
```
`UserServiceImpl.java`
```java{.line-numbers}
    @Autowired
    private UserMapper userMapper;
    @Override
    public IPage<User> selectUserList(Integer id, String startTime, String endTime) {
        Page<User> page = new Page<>();
        // page.setXXX 设置自定义分页属性

        IPage<User> ipage = userMapper.getUserList(page, id, startTime, endTime);
        return ipage;
    }
```
`UserController.java`
```java{.line-numbers}
    @Autowired
    private UserService userService;
    
    @GetMapping("/userList")
    public IPage<User> selectUserList(Integer id, String startTime, String endTime) {
       return userService.selectUserList(id, startTime, endTime);
    }
```

上面的Page分页参数有两种设置方式如下：

{% gallery::1::two::center %}
![Page参数](https://cdn.jsdelivr.net/gh/prettywinter/dist/images/doc/Page参数.png "Page参数")

![Page可设置参数](https://cdn.jsdelivr.net/gh/prettywinter/dist/images/doc/Page可设置参数.png "Page可设置参数")
{% endgallery %}


Mybatis框架中 jdbcType 支持两种时间类型，分别是：jdbcType=DATE、jdbcType=TIMESTAMP
前一种格式是 年-月-日，后一种格式是 年-月-日 时:分:秒，分别对应数据库的 date 类型和 datetime 类型。

Mybatis Plus 自带的 LambdaQueryWrapper 可以用来查询对象信息，如下代码：根据openId 查找对应的 customer。在不涉及多表查询时，用此方法比较 nice，个人感觉很好玩
```java{.line-numbers}
    // 注意，只有指定了 LambdaQueryWrapper<?> 中的 ? 类型后才可以使用函数式接口
    LambdaQueryWrapper<Customer> query = Wrappers.lambdaQuery();
    query.eq(Customer::getOpenId, openId);
    return customerMapper.selectOne(query);
```

Mybatis 的 mapper.xml 文件中，在插入数据的标签中可以添加 `useGeneratedKeys="true" keyProperty="id"` 这两个属性，插入之后会返会主键 id，之后我们可以使用的这个主键与其他表进行关联。不过需要注意的是，这个 `id` 必须是自增的。

#### saveBatch() 插入的性能问题

Mybatis Plus 中有批量插入的方法：saveBatch()，它可以保存一个集合。使用这个方法，不用写 sql 语句，但是性能比较差，网上看文章说在数据库（指MySQL）连接（url）后面加入一个配置：`rewriteBatchedStatements=true`，这样可以解决大批量插入时的性能问题，当我第一次看到这个方法查资料时看到一篇文章写的特别好，但是后来浪子找不到了，等到以后找到再补上链接。

#### Mybatis plus 中不能插入或更新null值

当用户更新数据为 null 值时，会发现，即使数据库的字段可以为 null 但是更新或插入时还是数据更新失败。这个就涉及到字段验证策略了。对于这个问题，官网也有说明，[点击查看官网说明](https://mp.baomidou.com/guide/faq.html#%E6%8F%92%E5%85%A5%E6%88%96%E6%9B%B4%E6%96%B0%E7%9A%84%E5%AD%97%E6%AE%B5%E6%9C%89-%E7%A9%BA%E5%AD%97%E7%AC%A6%E4%B8%B2-%E6%88%96%E8%80%85-null)
可以开全局配置或者局部配置（在更改的字段加上特定的注解）
```java{.line-numbers}
    // 最好是再后面加上该字段在数据库中的类型
    @TableField(updateStrategy = FieldStrategy.IGNORED, jdbcType = JdbcType.INTEGER)
```

### Spring Boot

使用 IDEA 工具创建 SpringBoot 项目时，可以使用阿里云的模板：`https://start.aliyun.com/`

#### 注解
1. @SpringBootApplication注解：是一个组合注解，包含多个注解；
特定注解：修饰注解的注解，@Target:指定注解的范围；@Retention:指定注解什么时候有效。
   > @SpringBootConfiguration：自动配置 spring springmvc 相关环境
    @EnableAutoConfiguration：开启自动配置，自动配置核心注解，自动配置 spring 相关环境，引入第三方技术自动配置其环境，mybatis-springboot、redis-springboot 等等
    @ComponentScan: 组件扫描，默认扫描当前包及子包，根据注解发挥注解作用。

2. @Configuration 可以创建多个对象
3. @Component 只能创建单个对象
4. @MapperScan 扫描Dao层接口，交给Spring工厂去创建对象

**注入方式：**
setter注入、构造注入(推荐使用)、自动注入
Autowired：Spring框架 默认根据类型注入
Resource：JavaEE规范 默认根据名称注入 自动根据类型注入
属性注入：@Value注入map集合时，文件中必须使用json格式赋值，使用"#{${属性}}"取值，map的键如果相同，后面的值会覆盖前面的值。

Spring Boot自带日志插件可以用来输出SQL语句;Spring Boot默认是 info 级别的日志。可以使用 `logging.level` 修改日志级别。
```yml
    # yml文件写法，properties也可以；
    # 以下写法的包也可以按照yml风格来写
    logging:
      level:
        com.XXX.XXX: debug
      # 日志输出到文件
      file: 
        name: log.log
        # 日志输出路径
        path: XXX/
```

#### AOP：
编写一个 Aop 配置类：
导入 aop 依赖；在 Aop 配置类上加上 @Aspect、@Configuration 注解；编写切面增强方法。
1. 切入点表达式：
    - 方法级别的切入点表达式：`execution(* com.XXX.*.*(..))`，第一个 * 号代表可以返回任意类型值。
    - 类级别的切入点表达式：`within(com.XXX.*)`
    - 自定义拦截器：`@annotation(com.XXX)`
2. 增强方式：
    - 前置增强、后置增强、环绕增强
    - 前置和后置都没有返回值，方法参数都是JointPoint
    - 环绕增强中，需要调用proceed()才能继续处理业务逻辑(类似拦截器)，该方法返回值为业务的返回值，因此环绕增强的返回类型设置为 Object 比较推荐。
    - 环绕增强的方法参数是ProceedingJointPoint

#### 文件的上传下载：
要求表单提交必须是post，encType 必须为 multiPart/form-data; `multipart file` 默认的上传文件大小(max-request-size)最大为10M.

将文件传到指定的目录：transferTo()

文件的下载在响应之前要设置以附件的形式，否则点击下载时会在浏览器打开。
`response.setHeader("content-disposition", "attachment;filename=" + URLEncoder.encode(fileName, "UTF-8"))`
文件io可以使用 spring 框架自带的 copy 方法：`FileCopyUtils.copy(is, os)`

#### 拦截器(一般用于用户认证，其它不常用)
编写拦截器：实现 HandlerInterceptor 接口；重写 preHandle、postHandle、afterCompletion 方法，其中 preHandle 方法中返回 true 代表放行，返回 false 代表中断。
配置拦截器：实现 WebMvcConfigurer 接口；重写 addInterceptors 方法，添加编写的拦截器。
另外，在实现的这个接口中，还有另外一个方法可以解决跨域问题，是一个全局跨域配置。

preHandle 返回值为 true 时，执行控制器中的方法，当控制器方法执行完成后会返回拦截器中执行拦截器中的 postHandle 方法，postHandle 执行完成之后响应请求，在响应请求完成后会执行 afterCompletion 方法。afterCompletion 无论执行成功或者失败都会执行。
**注意：** 只能拦截 controller 相关请求，不能拦截 jsp 静态资源文件；拦截器可以中断请求轨迹；请求之前如果该请求配置了拦截器，请求会先经过拦截器，放行之后执行请求的 controller，controller 执行完成后会回到拦截器继续执行拦截器代码。
如果配置了多个拦截器，默认执行的顺序和栈结构是一样的；但是也可以通过 `order()` 方法修改，里面填 int 类型的数字，数字大的先执行。

#### 打包(打包之前先清理(clean)再打包(package))
1. jar
    Spring Boot默认的打包方式就是 jar 包，使用maven的方式打包即可。
2. war
    1. 修改 `pom.xml` 文件：`<packaging>war</packaging>`
    2. 去掉内嵌的 tomcat 容器的依赖，可以在pom文件中找到对应的依赖，在依赖中加入 `<scope>provided</scope>` 此依赖就不参与打包
    3. 如果整合的是 jsp，那么使用打包的插件时，打包插件必须是 `1.4.2` 的版本，另外，在pom文件中加入以下内容，指定 jsp 文件的打包位置：
    ```xml
        <resources>
            <!-- 打包时将 jsp 文件拷贝到 META-INF 目录下 -->
            <resource>
                <!-- 指定 resources 插件处理那个目录下的资源文件 -->
                <directory>src/main/webapp</directory>
                <!-- 指定必须要放在此目录下才能被访问到 -->
                <targetPath>META-INF/resources</targetPath>
                <includes>
                    <include>**/**</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/**</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
            <directory>src/main/java</directory>
                <excludes>
                    <exclude>**/*.java</exclude>
                </excludes>
            </resource>
        </resources>
    ```
    在插件中指定入口类：
    ```
        <configuration>
            <fork>true</fork>
            <jvmArgument>-Dfile.encoding=UTF-8</jvmArgument>
            <mainClass>com.XXX.Application.class</mainClass>
        </configuration>
    ```
    4. 在启动类中，继承 `SpringBootServletInitiallizer` 类，重写 `configure()` 方法进行配置:
    return builder.sources(Application.class)

**注意：** 打成的 war 包使用外部 Tomcat 部署时，项目的配置文件中配置的端口和项目名称会失效。

以后台的方式启动：
`java -jar jar包名称 nohub`

#### SpringBoot解决跨域问题
1. @CrossOrigin 注解用在 Controller 类中，所有方法允许其它域中进行访问
2. 全局配置：写一个配置类
```java
@Configuration
public class CorsConfig {
 
    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();

        config.setAllowedOrigins("*");
        config.setAllowedHeaders("*");
        config.setAllowedMethods("*");
        // 处理所有请求的跨域配置
        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }
}
```

#### Jasypt 加密
引入依赖；
编写加密配置；
```properties{.line-numbers}
    <!-- 指定加密算法 -->
    jasypt.encryptor.algorithm=
    <!-- 指定密钥 -->
    jasypt.encryptor.password=
```
配置使用时使用 `ENC()` 括号里填写生成的密钥
例如，加密服务器主机地址，数据库用户名、密码


#### SpringBoot 处理异常

在自定义的全局异常类上加入 @ControllerAdvice，配合 @ExceptionHandler 处理指定的异常。



## SSH
在Hibernate3中，引入了Restrictions类作为Expression的替代，以后的版本，不再推荐使用Expression。
```java{.line-numbers}
    // 推荐使用
    Criteria cr = session.createCriteria(ChargeRecord.class).add(Restrictions.eq("chargeStatus", "01")).setProjection(Projections.sum("chargeAmount"));
    // 不再推荐
    TbItemParamExample example = new TbItemParamExample();
    Criteria criteria = example.createCriteria();
    criteria.andItemCatIdEqualTo(cid);
    Criteria ct= session.createCriteria(TUser.class);
    //Criteria中可以增加查询条件
    ct.add(Expression.eq("name","Erica"));
    ct.add(Expression.eq("sex",new Integer(1)));
    //Criteria中增加的查询条件可以由表达式对象创建
```