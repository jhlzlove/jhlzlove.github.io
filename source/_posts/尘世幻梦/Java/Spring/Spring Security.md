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
img:
---

Spring Security总结

<!-- more -->

# Spring Security总结
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

##### WebSecurityConfigurerAdapter 扩展 Spring Security 所有的默认配置

##### UserDetailsService 用来修改默认认证的数据源信息

UserDetailsService接口下有许多的实现。同时，此接口也方便了我们以后的自定义数据源的扩展。

# 配置 AuthenticationManager 的两种方式
1. 继承 WebSecurityConfigurerAdapter，
   springboot 对 security 默认配置中 在工厂默认创建 AuthenticationManager。
```java{.line-numbres}
    // 默认配置会自动发现创建的 UserDetailService 的 Bean
    @Autowired
    public void initialize(AuthenticationManagerBuilder builder) {
        System.out.println("spring boot 默认配置");
    }
```
2. 自定义全局数据源配置
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

