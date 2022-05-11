---
title: 使用maven-assembly-plugin打包项目
categories: 
    - Java
    - maven
abbrlink: 2d0c3e0e
---

本示例插件以 `3.3.0` 为例，结合官方文档进行简单的配置。推荐直接访问 [官网](https://maven.apache.org/plugins/maven-assembly-plugin/usage.html)，介绍的很详细。

<!-- more -->

pom.xml 文件配置：

```xml{.line-numbers}
<plugins>
    <plugin>
        <!-- NOTE: We don't need a groupId specification because the group is
        org.apache.maven.plugins ...which is assumed by default.-->
        <!-- 不需要指定 groupId，因为这个插件默认就是 apache 的,指定 artifactId 和版本就行 -->
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.3.0</version>

        <configuration>
            <!-- 打包说明文件的位置 -->
            <descriptors>
                <descriptor>src/main/resources/assembly.xml</descriptor>
            </descriptors>
            <!-- 描述 -->
            <descriptorRefs>
                <descriptorRef>jar-with-dependencies</descriptorRef>
            </descriptorRefs>

            <!-- 指定程序启动的入口类（主类）,注意：指明主类只支持 jar 和 war -->
            <archive>
                <manifest>
                    <mainClass>cn.paimeng.bot.RabbitbotApplication</mainClass>
                </manifest>
            </archive>
        </configuration>

        <executions>
            <execution>
                <!-- this is used for inheritance merges -->
                <id>make-assembly</id>
                <!-- bind to the packaging phase -->
                <phase>package</phase>
                <goals>
                    <goal>single</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
</plugins>
```
