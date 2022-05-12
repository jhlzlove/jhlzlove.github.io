---
title: Java项目整合配置
categories:
  - Java
  - 配置
music:
  server: netease
  type: song
  id: 440403990
abbrlink: ff45fa2
---

来一部二手秘籍罩你闯荡四方。基于 SpringBoot 的两种常用整合配置，注释含混不清的、缺少的以后有空整理。

<!-- more -->

## 配置示例

### 1. properties

```properties{.line-numbers}
# 应用名称
spring.application.name=mytest
# 应用服务 WEB 访问端口
server.port=8080
# spring 框架在指定包下的类打印 SQL 日志信息级别
logging.level.com.jhlz.mapper=debug
# 数据库驱动（8.0之后）：
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据源名称
spring.datasource.name=defaultDataSource
# 数据库连接地址
spring.datasource.url=jdbc:mysql://localhost:3306/blue?useSSL=false&useUnicode=true&characterEncoding=utf-8&mysqlEncoding=utf-8&serverTimezone=Asia/Shanghai
# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=
# Mybatis全局设置插入策略，可以插入 null 或 空字符串
mybatis-plus.global-config.db-config.insert-strategy=ignored
# Mybatis全局设置更新策略，可以更新为 null 或 空字符串
mybatis-plus.global-config.db-config.update-strategy=ignored
# 关闭 mybatis plus 的 banner
mybatis-plus.global-config.banner=false
# 设置实体类别名
mybatis-plus.type-aliases-package=com.jhlz.model
# 打印 sql
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl

```

### 2. yml

```yml{.line-numbers}
logging:
    level:
        com:
        jhlz:
            mapper: debug
mybatis-plus:
    configuration:
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    global-config:
        banner: false
        db-config:
        insert-strategy: ignored
        update-strategy: ignored
    type-aliases-package: com.jhlz.model
server:
    port: 8080
spring:
    application:
        name: mytest
    datasource:
        driver-class-name: com.mysql.cj.jdbc.Driver
        name: defaultDataSource
        # rewriteBatchedStatements=true 开启该选项需要 MySQL 驱动版本 > 5.1.13，可以解决Myatis plus批量插入的性能问题
        url: jdbc:mysql://localhost:3306/blue?useSSL=false&useUnicode=true&characterEncoding=utf-8&mysqlEncoding=utf-8&serverTimezone=Asia/Shanghai&rewriteBatchedStatements=true
        username: root
        password: 
```

## 附

```bash{.line-numbers}
# sqlserver 默认端口号为：1433
URL:"jdbc:microsoft:sqlserver://localhost:1433;DatabaseName=dbname"
DRIVERNAME:"com.microsoft.jdbc.sqlserver.SQLServerDriver";

# mysql 默认端口号为：3306
URL:"jdbc:mysql://localhost:3306/test?username=root&password=&useUnicode=true&characterEncoding=utf-8&mysqlEncoding=utf-8&serverTimezone=Asia/Shanghai"
DRIVERNAME:"com.mysql.jdbc.Driver";

# oracle 默认端口号为：1521
URL:"jdbc:oracle:thin:@localhost:1521:orcl";
DRIVERNAME:"oracle.jdbc.driver.OracleDriver";

# mongodb 默认端口：27017
URL:"mongodb://root:123456@localhost:27017/test"
```
