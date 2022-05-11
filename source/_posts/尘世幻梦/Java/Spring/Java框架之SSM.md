---
title: Java框架之SSM
categories:
  - Java
music:
  server: netease
  type: song
  id: 440403990
abbrlink: fa647c74
img:
---

Java框架之SSM整理。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

<!-- code_chunk_output -->

- [Sping+Spring MVC+Mybatis](#spingspring-mvcmybatis)
  - [Spring](#spring)
  - [Spring MVC](#spring-mvc)
    - [Spring MVC异常处理](#spring-mvc异常处理)
  - [Mybatis/Mybatis Plus](#mybatismybatis-plus)
    - [分页插件](#分页插件)
    - [saveBatch() 插入的性能问题](#savebatch-插入的性能问题)
    - [Mybatis plus中不能插入或更新null值](#mybatis-plus中不能插入或更新null值)
  - [Spring AOP](#spring-aop)
  - [文件的上传下载](#文件的上传下载)
  - [拦截器(一般用于用户认证，其它不常用)](#拦截器一般用于用户认证其它不常用)

<!-- /code_chunk_output -->

## Sping+Spring MVC+Mybatis

### Spring

Spring 管理事物的方式：

- 编程式事务，在代码中硬编码。(不推荐使用)
- 声明式事务，在配置文件中配置(推荐使用)
  1. 基于XML的声明式事务
  2. 基于注解的声明式事务

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

{% link Mybatis官网::https://mybatis.org/mybatis-3/zh/index.html %}
{% link Mybatis Plus官网::https://baomidou.com/guide/ %}

#### 分页插件

Mybatis Plus自带的分页：IPage、Page
IPage是一个接口，Page是IPage的实现类，
在使用时，可以使用Page创建一个对象实例，设置分页的一些参数；IPage对象用来存放自定义的分页规则和查询条件参数，比如以下简单例子：

1. UserMapper.java

    ```java{.line-numbers}
    //mapper中也可以使用 Page ，效果一样的
    IPage<User> getUserList(Page<User> page,@param("id") Integer id,@Param("startTime") String startTime,@Param("endTime") String endTime);
    ```

2. UserService.java

    ```java{.line-numbers}
    IPage<User> selectUserList(Integer id, String startTime, String endTime);
    ```

3. UserServiceImpl.java

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

4. UserController.java

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

Mybatis框架中 jdbcType 支持两种时间类型，分别是：`jdbcType=DATE`、`jdbcType=TIMESTAMP`，第一种格式是 `年-月-日`，后一种格式是 `年-月-日 时:分:秒`，分别对应数据库的 date 类型和 datetime 类型。

Mybatis Plus自带的 LambdaQueryWrapper 可以用来查询对象信息，如下代码：根据openId 查找对应的 customer。在不涉及多表查询时，用此方法比较 nice，个人感觉很好玩。

```java{.line-numbers}
// 注意，只有指定了 LambdaQueryWrapper<?> 中的 ? 类型后才可以使用函数式接口
LambdaQueryWrapper<Customer> query = Wrappers.lambdaQuery();
query.eq(Customer::getOpenId, openId);
return customerMapper.selectOne(query);
```

Mybatis 的 mapper.xml 文件中，在插入数据的标签中可以添加 `useGeneratedKeys="true" keyProperty="id"` 这两个属性，插入之后会返会新插入数据的主键 id，之后我们可以使用的这个主键与其他表进行关联。不过需要注意的是，这个 `id` 必须是自增的。

#### saveBatch() 插入的性能问题

Mybatis Plus 中有批量插入的方法：saveBatch()，它可以保存一个集合。使用这个方法，不用写 sql 语句，但是性能比较差，网上看文章说在数据库（指MySQL）连接（url）后面加入一个配置：`rewriteBatchedStatements=true`，这样可以解决大批量插入时的性能问题，当我第一次看到这个方法查资料时看到一篇文章写的特别好，但是后来浪子找不到了，等到以后找到再补上链接。

#### Mybatis plus中不能插入或更新null值

当使用 Mybatis Plus 时，更新数据为 null 值时，会发现，即使数据库的字段可以设置为 null，但是更新或插入时还是数据更新失败。这个就涉及到字段验证策略了。对于这个问题，官网 [常见问题](https://www.mybatis-plus.com/guide/faq.html) 中也有解决方法，[点击查看官网说明](https://www.mybatis-plus.com/guide/faq.html#%E6%8F%92%E5%85%A5%E6%88%96%E6%9B%B4%E6%96%B0%E7%9A%84%E5%AD%97%E6%AE%B5%E6%9C%89-%E7%A9%BA%E5%AD%97%E7%AC%A6%E4%B8%B2-%E6%88%96%E8%80%85-null)

1. 局部配置（在更改的字段加上特定的注解）

    ```java{.line-numbers}
    // 修改策略，最好是再后面加上该字段在数据库中的类型
    @TableField(updateStrategy = FieldStrategy.IGNORED, jdbcType = JdbcType.INTEGER)
    ```

2. 可以开全局配置，在 `xxx.properties` 或者 `xxx.yml` 文件中注入配置 GlobalConfiguration 属性 fieldStrategy。

### Spring AOP

编写一个 Aop 配置类：
导入 aop 依赖；在 Aop 配置类上加上 @Aspect、@Configuration 注解；编写切面增强方法。

1. 切入点表达式
    - 方法级别的切入点表达式：`execution(* com.XXX.*.*(..))`，第一个 * 号代表可以返回任意类型值。
    - 类级别的切入点表达式：`within(com.XXX.*)`
    - 自定义拦截器：`@annotation(com.XXX)`
2. 增强方式
    - 前置增强、后置增强、环绕增强
    - 前置和后置都没有返回值，方法参数都是JointPoint
    - 环绕增强中，需要调用proceed()才能继续处理业务逻辑(类似拦截器)，该方法返回值为业务的返回值，因此环绕增强的返回类型设置为 Object 比较推荐。
    - 环绕增强的方法参数是ProceedingJointPoint

### 文件的上传下载

要求表单提交必须是post，encType 必须为 multiPart/form-data; `multipart file` 默认的上传文件大小(max-request-size)最大为10M.

文件的下载在响应之前要设置以附件的形式，否则点击下载时会在浏览器打开。
`response.setHeader("content-disposition", "attachment;filename=" + URLEncoder.encode(fileName, "UTF-8"))`

将文件传到指定的目录：transferTo()，文件io可以使用 spring 框架自带的 copy 方法：`FileCopyUtils.copy(is, os)`

### 拦截器(一般用于用户认证，其它不常用)

编写拦截器：

1. 实现 HandlerInterceptor 接口；
2. 重写 preHandle、postHandle、afterCompletion 方法，其中 preHandle 方法中返回 true 代表放行，返回 false 代表中断。
3. 配置拦截器：实现 WebMvcConfigurer 接口；重写 addInterceptors 方法，添加编写的拦截器。另外，在实现的这个接口中，还有另外一个方法可以解决跨域问题，是一个全局跨域配置。

preHandle 返回值为 true 时，执行控制器中的方法，当控制器方法执行完成后会返回拦截器中执行拦截器中的 postHandle 方法，postHandle 执行完成之后响应请求。在响应请求完成后会执行 afterCompletion 方法，该方法无论执行成功或者失败都会执行。

拦截器只能拦截 controller 相关请求，不能拦截 jsp 静态资源文件；拦截器可以中断请求轨迹；请求之前如果该请求配置了拦截器，请求会先经过拦截器，放行之后执行请求的 controller，controller 执行完成后会回到拦截器继续执行拦截器代码。
如果配置了多个拦截器，默认执行的顺序和栈结构是一样的；但是也可以通过 `order()` 方法修改，里面填 int 类型的数字，数字大的优先执行。
