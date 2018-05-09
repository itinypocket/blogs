# Spring Boot教程(十二)：Spring Boot集成热部署插件devtools

# 一、介绍

在开发工程中，修改一点儿代码，想看效果就需要重新启动服务，这样会花费大量时间在重启服务上，通过devtools热部署可以大大减少重启服务的时间。

之所以能减少时间，是因为Spring Boot自动重启的原理在于使用两个classloader：不改变的类（如第三方jar）由base类加载器加载，正在开发的类由restart类加载器加载。应用重启时，restart类加载器被扔掉重建，而base类加载器不变，这种方法意味着应用程序重新启动通常比“冷启动”快得多，因为base类加载器已经可用并已填充。

**注意：不同的IDE效果不一样，Eclipse中保存文件即可引起classpath更新(需要打开自动编译)，从而触发重启。而IDEA则需要自己手动command+F9(Mac OS X 10.5+版本的快捷键，其他版本的可能有所不同，根据自己的情况而定)重新编译一下**


# 二、项目准备

创建一个最简单的包含web依赖的springboot项目：

pom.xml文件：

```
<!-- Spring Boot启动器父类 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.1.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
</properties>

<dependencies>
    <!-- Spring Boot web启动器 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional> <!-- 防止将devtools依赖传递到其他模块中 -->
    </dependency>
</dependencies>
```

启动类Application：

```
package com.songguoliang.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @Description
 * @Author sgl
 * @Date 2018-05-02 14:51
 */
@SpringBootApplication
public class Application{
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }


}

```


# 三、添加依赖

集成devtools只需要添加下面的依赖即可，通过`<optional>true</optional>`可防止依赖传递。


```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional> <!-- 防止将devtools依赖传递到其他模块中 -->
</dependency>
```

# 四、测试

我们先启动服务，然后再创建`HelloController`，代码如下：

```
package com.songguoliang.springboot.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Description
 * @Author sgl
 * @Date 2018-05-09 10:21
 */
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
    
}

```

此时我们无需重启服务，便可以访问`http://localhost:8080/hello`,（注：eclipse需开启自动编译，idea需要重新编译）

![](images/13_01.png)












<br><br><br><br>

源码： 
[github](https://github.com/itinypocket/spring-boot-study/tree/master/spring-boot-devtools) 
[码云](https://gitee.com/itinypocket/spring-boot-study/tree/master/spring-boot-devtools)

