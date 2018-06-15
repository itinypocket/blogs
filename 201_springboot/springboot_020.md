# Spring Boot教程(十九)：Spring Boot集成shiro ehcache(使用shiro的缓存管理)



# 一、项目准备

为了方便，这里直接使用[Spring Boot教程(十六)：Spring Boot集成shiro](springboot_017.md)章节的源码。


# 二、添加依赖

```
<!-- cache -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>
<!-- shiro-ehcache-->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <version>1.4.0</version>
</dependency>
```


# 三、创建ehcache配置文件

在`resources`目录下，创`ehcache.xml`配置文件，内容如下:

```
<?xml version="1.0" encoding="UTF-8"?>
<ehcache updateCheck="false" dynamicConfig="false">
    <diskStore path="java.io.tmpdir"/>

    <cache name="users"
           timeToLiveSeconds="300"
           maxEntriesLocalHeap="1000"/>

    <!--
        name:缓存名称。
        maxElementsInMemory：缓存最大个数。
        eternal:对象是否永久有效，一但设置了，timeout将不起作用。
        timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
        timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
        overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。
        diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
        maxElementsOnDisk：硬盘最大缓存个数。
        diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
        diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
        memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
        clearOnFlush：内存数量最大时是否清除。
    -->
    <defaultCache name="defaultCache"
                  maxElementsInMemory="10000"
                  eternal="false"
                  timeToIdleSeconds="120"
                  timeToLiveSeconds="120"
                  overflowToDisk="false"
                  maxElementsOnDisk="100000"
                  diskPersistent="false"
                  diskExpiryThreadIntervalSeconds="120"
                  memoryStoreEvictionPolicy="LRU"/>
</ehcache>
```


# 四、修改ShiroConfig类

1、添加以下代码：

```
/**
 * 缓存管理器
 * @return
 */
@Bean
public EhCacheManager ehCacheManager(){
    EhCacheManager cacheManager = new EhCacheManager();
    cacheManager.setCacheManagerConfigFile("classpath:ehcache.xml");
    return cacheManager;
}

/**
 * aop代理
 * @return
 */
@Bean
@DependsOn("lifecycleBeanPostProcessor")
public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
    DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
    defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
    return defaultAdvisorAutoProxyCreator;
}
```

2、修改userRealm()和securityManager()，添加缓存支持

```
/**
 * 自定义realm
 *
 * @return
 */
@Bean
public UserRealm userRealm() {
    UserRealm userRealm = new UserRealm();
    userRealm.setCredentialsMatcher(hashedCredentialsMatcher());
    userRealm.setCacheManager(ehCacheManager());
    return userRealm;
}

/**
 * 安全管理器
 * 注：使用shiro-spring-boot-starter 1.4时，返回类型是SecurityManager会报错，直接引用shiro-spring则不报错
 *
 * @return
 */
@Bean
public DefaultWebSecurityManager securityManager() {
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    securityManager.setRealm(userRealm());
    securityManager.setCacheManager(ehCacheManager());
    return securityManager;
}
```


# 五、添加EhcacheConfig类

在com.songguoliang.springboot.configuration包下添加EhcacheConfig配置类：

```
package com.songguoliang.springboot.configuration;

import org.springframework.cache.ehcache.EhCacheManagerFactoryBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Description
 * @Author sgl
 * @Date 2018-06-15 11:45
 */
@Configuration
public class EhcacheConfig {
	 /**
     * 设置为共享模式
     * @return
     */
    @Bean
    public EhCacheManagerFactoryBean ehCacheManagerFactoryBean() {
        EhCacheManagerFactoryBean cacheManagerFactoryBean = new EhCacheManagerFactoryBean();
        cacheManagerFactoryBean.setShared(true);
        return cacheManagerFactoryBean;
    }
}

```

# 六、修改UserRealm类，添加@Lazy注解延迟注入service

```
//省略代码……

/**
 * 注意此处需要添加@Lazy注解，否则SysUserService缓存注解、事务注解不生效
 */
@Autowired
@Lazy
private SysUserService sysUserService;

@Autowired
@Lazy
private SysPermissionService sysPermissionService;

//省略代码……

```


# 七、注意

至此，我们以将shiro和ehcache集成完毕，使用的是shiro框架提供的缓存管理器，有个需要特别注意的是，**UserRealm里注入的SysUserService等service，需要延迟注入，所以都要添加@Lazy注解(如果不加需要自己延迟注入)，否则会导致该service里的@Cacheable缓存注解、@Transactional事务注解等失效**。













<br><br><br><br>

源码： 
[github](https://github.com/itinypocket/spring-boot-study/tree/master/spring-boot-shiro-ehcache-shiro) 
[码云](https://gitee.com/itinypocket/spring-boot-study/tree/master/spring-boot-shiro-ehcache-shiro)

















