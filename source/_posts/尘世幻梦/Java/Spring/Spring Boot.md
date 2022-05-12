---
title: Spring Boot
categories:
  - Java
  - Spring
  - Spring Boot
music:
  server: netease
  type: song
  id: 440403990
abbrlink: a541262a
---


<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

<!-- code_chunk_output -->

- [Spring Boot](#spring-boot)
  - [1. 注解](#1-注解)
  - [2. SpringBoot 处理异常](#2-springboot-处理异常)
  - [3. 日志打印](#3-日志打印)
  - [4. Maven打包(打包之前先清理(clean)再打包(package))](#4-maven打包打包之前先清理clean再打包package)
  - [5. SpringBoot解决跨域问题](#5-springboot解决跨域问题)
  - [6. Jasypt 加密](#6-jasypt-加密)

<!-- /code_chunk_output -->

## Spring Boot

使用 IDEA 工具创建 SpringBoot 项目时，可以使用阿里云的快速构建模板：`https://start.aliyun.com/`，速度还可以。

### 1. 注解

1. @SpringBootApplication注解：是一个组合注解，包含多个注解；
特定注解：修饰注解的注解，@Target:指定注解的范围；@Retention:指定注解什么时候有效。

   - @SpringBootConfiguration：自动配置 spring springmvc 相关环境
   - @EnableAutoConfiguration：开启自动配置，自动配置核心注解，自动配置 spring 相关环境，引入第三方技术自动配置其环境，mybatis-springboot、redis-springboot 等等
   - @ComponentScan: 组件扫描，默认扫描当前包及子包，根据注解发挥注解作用。

2. @Configuration 可以创建多个对象
3. @Component 只能创建单个对象
4. @MapperScan 扫描Dao层接口，交给Spring工厂去创建对象

### 2. SpringBoot 处理异常

在自定义的全局异常类上加入 @ControllerAdvice，配合 @ExceptionHandler 处理指定的异常。

### 3. 日志打印

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

### 4. Maven打包(打包之前先清理(clean)再打包(package))

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

    4. 在插件中指定入口类：

        ```xml
        <configuration>
            <fork>true</fork>
            <jvmArgument>-Dfile.encoding=UTF-8</jvmArgument>
            <mainClass>com.XXX.Application.class</mainClass>
        </configuration>
        ```

    5. 在启动类中，继承 `SpringBootServletInitiallizer` 类，重写 `configure()` 方法进行配置:
    return builder.sources(Application.class)

**注意：** 打成的 war 包使用外部 Tomcat 部署时，项目的配置文件中配置的端口和项目名称会失效。

后台方式启动 jar 包：`nohup java -jar jar包名称 &`

### 5. SpringBoot解决跨域问题

1. @CrossOrigin 注解用在 Controller 类中，所有方法允许其它域中进行访问
2. 全局配置：写一个配置类（正式项目一般都是此方式）

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

### 6. Jasypt 加密

引入相关依赖，编写加密配置；

```properties{.line-numbers}
<!-- 指定加密算法 -->
jasypt.encryptor.algorithm=
<!-- 指定密钥 -->
jasypt.encryptor.password=
```
