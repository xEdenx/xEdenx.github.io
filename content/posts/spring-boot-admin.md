---
title: 使用 SpringBootAdmin 监控 SpringBoot 应用
description: 使用 Spring Boot Admin 监控 Spring Boot 应用。
publishDate: 2018-09-01 12:48:49
tags:
  - spring boot
  - SpringBootAdmin
  - DevOps
---

### [ 在线文档 ](http://codecentric.github.io/spring-boot-admin/current/#logfile)

### Spring Boot Admin Server Config

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.tneciv</groupId>
    <artifactId>adminserver</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <name>adminserver</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.15.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-boot-admin.version>1.5.7</spring-boot-admin.version>
        <spring-cloud.version>Edgware.SR4</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>de.codecentric</groupId>
                <artifactId>spring-boot-admin-dependencies</artifactId>
                <version>${spring-boot-admin.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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

#### Application.java

```java
package com.tneciv.adminserver;

import de.codecentric.boot.admin.config.EnableAdminServer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@EnableAdminServer
@EnableEurekaClient
@SpringBootApplication
public class AdminserverApplication {

    public static void main(String[] args) {
        SpringApplication.run(AdminserverApplication.class, args);
    }
}
```

#### application.properties

```
spring.application.name=admin-server
server.port=8888
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
eureka.instance.preferIpAddress=true
eureka.instance.instance-id=${spring.cloud.client.ipAddress}:${server.port}
```

### Spring Boot Admin Client Config

基于Eureka的配置下，client无需引入spring-boot-admin-client，只需将admin-server与应用注册于同一个注册中心下

client端需要关闭安全检测，配置logfilepath后才能使用全部功能

#### application.properties

```
management.security.enabled = false
# log file path 用于开启logs tab
logging.file = /data/project/logs/projecta-%d{yyyy-MM-dd}.log
logging.pattern.file = %date [%level] [%thread] %logger{80} [%file : %line] %msg%n
```
