---
title: Spring Security总结
categories:
  - Java
  - Spring Security
music:
  server: netease
  type: song
  id: 430793674
abbrlink: 95588d77
---

Spring Security总结

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=true} -->

## 核心类

### 认证常用三个类

- AuthenticationManager（authenticate()，返回 Authentication 表示认证成功，返回 AuthenticationException 认证失败）
    > AuthenticationManager 主要实现类为 ProviderManager，在 ProviderManager 中管理了众多的 AuthenticationProvider 实例，在一次完整的认证流程中，允许存在多个 AuthenticationProvider，用来实现多种认证方式，这些 AuthenticationProvider 实例都是由 ProviderManager 统一管理的。
- Authentication（保存认证以及认证成功的信息）
- SecurityContextHolder（取出用户信息）
  > 基于 threadLocal 线程绑定，请求时把 session 中的用户信息放入 SecurityContextHolder 中，请求结束后清空信息；之后的请求同理。

### 授权常用三个类

AccessDeclsionManager（访问决策管理器）此次请求是否放行资源
AccessDecislonVoter（访问决定投票器）此次请求是否具有访问资源的权限
ConfigAttribute（保存授权时的角色信息）

ProviderManager 是一个接口
AuthenticationProvider 是它的一个实现。
DaoAuthenticationProvider 通过调用 retrieveUser() 认证。

思考：

- 为什么在引入 Spring Security 之后没有任何配置所有的请求就要认证呢？
  SpringBootWebSecurityConfiguration，自动配置类
- 在项目中明明没有登录界面，登录界面怎么来的呢？
  访问接口时，在引入 spring security 之后会先经过一系列过滤器。在请求到达 FilterSecurityInterceptor 时，发现请求并未认证，请求拦截下来，并抛出 AccessDeniedException 异常。抛出的异常会被 ExceptionTranslationFilter 捕获，这个 Filter 中会调用 LoginUrlAuthenticationEntryPoint#commence 方法给客户端返回 302，要求客户端进行重定向到 /login 页面。客户端发送 /login 请求。/login 请求会再次被拦截器中 DefaultLoginPageGeneratingFilter 拦截到，并在拦截器中返回生成登录页面。
- 为什么使用 user 和控制台打印的密码能登录，登录时验证数据源存在哪里呢？
  Spring Security中有一个基于内存（InMemoryUserDetailsManager）的默认用户实现。

### 总结

#### AuthenticationManager ProviderManager AuthencationProvider关系

AuthenticationManager 全局的父接口，只有一个 authenticate 方法。需要 ProviderManager 实现。ProviderManager 遍历下面的 AuthencationProvider 认证，只要有一个认证通过即可。

##### WebSecurityConfigurerAdapter

WebSecurityConfigurerAdapter 是 Spring Security 为我们提供的扩展类，方便我们重写默认配置，实现定制。

##### UserDetailsService 用来修改默认认证的数据源信息

UserDetailsService接口下有许多的实现。同时，此接口也方便了我们以后的自定义数据源的扩展。

## 配置AuthenticationManager的两种方式

1. 继承 WebSecurityConfigurerAdapter，springboot 对 security 默认配置中 在工厂默认创建 AuthenticationManager。

    ```java{.line-numbres}
    // 默认配置会自动发现创建的 UserDetailService 的 Bean
    @Autowired
    public void initialize(AuthenticationManagerBuilder builder) {
        System.out.println("spring boot 默认配置");
    }
    ```

2. 自定义全局认证数据源

    ```java{.line-numbres}
    // 自定义配置
    @Override
    public void configure(AuthenticationManagerBuilder builder) {
        
    }
    ```

**总结：**

1. 默认自动配置全局 AuthenticationManager 默认找当前项目中是否存在自定义 UserDetailService 实例，自动将当前项目的 UserDetailService 实例设置为数据源。
2. 默认自动配置全局 AuthenticationManager 在工厂中使用时直接在代码中注入即可。
3. 一旦通过自定义的方式配置后，会自动覆盖默认的实现；并且需要在实现中指定自定义的数据源。

## 密码加密

常见加密方案：Hash算法+盐
单向自适应函数（占用大量系统资源，每个认证请求都会大大降低应用程序的性能），开发者可以通过bcrypt、PBKDF2、sCrypt以及argon2来体验这种自适应单向函数加密。

密码的认证是由 PasswordEncoder 进行的，他可以根据不同的加密实现不同的认证方式，所以非常的灵活。

自定义密码加密有两种方式，一种是直接指定密码加密的方式（比如Bcrypt），另一种是使用自定义升级的方式。

## RememberMe

RememberMe是一种服务端的行为，并非是把用户名密码保存在Cookie中的信息。传统的登录方式基于Session会话，一旦用户的会话过期，就要再次登录，这样就太过于繁琐。如果有一种机制，会话过期后，还能继续保持认证状态，就会方便很多,RememberMe就是为了解决这一需求而生的。

具体的实现思路就是通过Cookie来记录当前用户身份，用户登录成功之后，会通过一定算法，将用户信息时间戳等进行贾母，加密完成后，通过响应头带回前端存储再Cookie中，当浏览器会话过期之后，如果再次访问网站，会自动将Cookie中的信息发送给服务器，服务器对Cookie中的信息进行校验分析，进而确定出用户的身份，Cookie中所保存的用户信息也是有失效的，例如三天、一周等。

认证成功后写一段信息在Cookie中

## 会话管理(SessionManagementFilter)

会话并发管理：简单来说，就是多个客户端使用同一账户登录。默认情况下，同一账户可以再多少设备上登录并没有限制，我们可以在 Spring Security 中进行配置。

开启会话管理

```java{.line-numbers}
// 自定义配置
@Override
public void configure(HttpSecurity http) throw Exception {
    http...
        .sessionManagement() // 开启会话管理
        // 最大并发会话为 1
        .maximumSessions(1);
}
```

Spring Security 开启会话管理默认的是挤掉另一个客户端登录；我们可以设置为禁止其它客户端登录（当前用户登录成功，其它客户端无法使用当前的账户登录，除非当前用户注销退出）。集群下的会话管理可以使用 Redis 的Session 共享。
