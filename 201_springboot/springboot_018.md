# Spring Boot教程(十七)：Spring Boot导出war包部署到外部Tomcat

对于创建的jsp的web项目，有时想把项目打成war包部署到外部的Tomcat上，要达到这种目的，需要修改一些东西。

# 一、修改Maven的打包方式

Maven默认的`packing`为`jar`，所以要将其改为`war`:

```
<packaging>war</packaging>
```

# 二、修改内迁容器的依赖范围

将spring-boot-starter-tomcat的要构建可执行并可部署到外部容器中的war文件，需要将嵌入容器依赖项标记为“provided”，如以下示例所示：

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-tomcat</artifactId>
	<scope>provided</scope>
</dependency>
```



<br><br>

参考：

https://spring.io/guides/gs/convert-jar-to-war/


https://docs.spring.io/spring-boot/docs/2.0.2.RELEASE/reference/htmlsingle/#build-tool-plugins-maven-packaging


https://docs.spring.io/spring-boot/docs/2.0.2.RELEASE/reference/htmlsingle/#howto-create-a-deployable-war-file