# Simple SpringBoot Application
本篇采用 Maven 构建 SpringBoot Application

* 推荐使用 [Spring Initializr](https://start.spring.io/) 快速创建 APP
* **或** 可以通过IDE创建Maven Project

### 结构 
```
project
   |
   | src
      | main
         | java
             | com.xxxx.kkk
             :
             :
         | resources
             | application.yml
      | test
   | target
   | pom.xml
        :
        :
```
### 常用文件:

1) **pox.xml**: 项目文件的配置, 如 groupId, artifactId, 依赖包及其版本号等.
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.xxxx.xxxxx</groupId>
    <artifactId>xxx-xxx</artifactId>
    <version>0.1</version>
    <name>xxx-xxx</name>
    <description>a project for Spring Boot</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>10</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```
2) **application.yml / application.properties**: 程序的配置,以及环境变量的设置. 如 database 的 url, username, password 等配置
```
spring:
    application:
    name: xxxx
    datasource:
    url: jdbc:mysql://localhost:3306
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
    jpa:
    database: mysql
    properties:
    hibernate:
        show_sql: false
        format_sql: true

server:
    port: 8080

logging:
    file: logs/xxx-log.txt
    pattern:
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
    level:
    org.springframework: ERROR
    org.hibernate: ERROR
```
3) **main class**: Java 程序必备的main class
```
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```
4) **target 文件夹**: 项目打包成jar存放的目录

