---
layout: mypost
title: SpringBoot下项目搭建
categories: [Activiti]
---

#### **1.新建一个springboot项目，选中Web，MySQL**
![这里写图片描述](https://wx3.sinaimg.cn/mw690/becbe214gy1fq6js8wt8dj20r00hzgmw.jpg)

> 添加activiti的maven依赖

```xml
<!--Activiti基础JAR包-->
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-spring-boot-starter-basic</artifactId>
    <version>5.22.0</version>
</dependency>
```
> springboot的版本
```
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.5.8.RELEASE</version>
	<relativePath/> <!-- lookup parent from repository -->
</parent>
```
#### **2.配置数据库和activiti基础设置**
```
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver
  activiti:
    # 自动部署验证设置:true-开启（默认）、false-关闭
    check-process-definitions: false
    #自动部署文件路径后缀
#   process-definition-location-prefix: classpath:/processes/
#   process-definition-location-suffixes:
#      - **.bpmn
#      - **.bpmn20.xml
```
- **如果不关闭自动部署，且没有配置自动部署文件的路径，则会报出如下错误**

```java
org.springframework.beans.factory.BeanCreationException:
Error creating bean with name 'springProcessEngineConfiguration' defined in class path  resource [org/activiti/spring/boot/DataSourceProcessEngineAutoConfiguration$DataSourceProcessEngineConfiguration.class]:
Bean instantiation via factory method failed;
nested exception is org.springframework.beans.BeanInstantiationException:
Failed to instantiate [org.activiti.spring.SpringProcessEngineConfiguration]:
Factory method 'springProcessEngineConfiguration' threw exception;
nested exception is java.io.FileNotFoundException:
class path resource [processes/] cannot be resolved to URL because it does not exist
```
#### **3.运行项目，查看数据库**

![这里写代码片](https://wx3.sinaimg.cn/mw690/becbe214ly1fq7azng5roj20ke0f9dhe.jpg)
