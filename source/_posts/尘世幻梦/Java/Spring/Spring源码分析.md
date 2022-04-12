---
title: Spring源码分析笔记
categories:
  - Spring
  - Java
abbrlink: fd4a5dd8
---

Spring源码分析笔记

<!-- more -->

# Spring源码分析笔记
<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=true} -->

工厂的分类：
BeanFactory接口：
1. ConfigurableBeanFactory
2. AutowireCapableBeanFactory
3. ListableBeanFactory
    DefaultListableBeanFactory

XmlBeanFactory（Spring3.1之后过期）：
读取XML配置文件，创建对应的对象
```java
BeanFactory beanFactory = new XmlBeanFactory(Resource->xml)
beanFactory.getBean();
```
1. 怎么读取配置文件获得IO资源
2. 读取配置文件后，如何在Spring中以对象的形式进行封装。
3. 根据配置信息创建对象
4. 所创建对象的生命周期


Spring MVC 开发中同时存在 2 个 Spring 工厂。

Spring 定时任务和事务管理配置
<task:annotation >
<tx:annotation-manager >

属性编辑器(PropertyEditor) 老版的自定义转换器，由 jdk 提供。
Spring 提供了 Convertor 接口供我们实现自定义转换器。

各种Aware不在工厂启动的时候操作，而是在创建对象的第三步初始化中通过 BeanPostProcessor 完成。不要在工厂启动的时候注入，而是在对象创建的时候注入。

默认情况下 ConfigurationClassPostProcessor 处理顶级注解（注册）：
@Component（@Service @Repository @Controller）
@Configuration
@PropertySource
@Import
@ComponentScan

git@github.com:jhlzlove/hexo-theme-volantis.git

@Configuration 注解的类中，@Bean 注解的配置bean通过代理增加额外的scope功能。