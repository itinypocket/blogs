# Spring Boot教程(十八)：Spring Boot集成shiro ehcache



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
```

# 三、创建ehcache配置文件

在`resources`目录下，创`ehcache.xml`配置文件，内容如下:

```
<?xml version="1.0" encoding="UTF-8"?>
<ehcache updateCheck="false" dynamicConfig="false">
    <diskStore path="java.io.tmpdir"/>

    <cache name="authorizationCache"
           maxEntriesLocalHeap="2000"
           eternal="false"
           timeToIdleSeconds="1800"
           timeToLiveSeconds="1800"
           overflowToDisk="false"
           statistics="true">
    </cache>

    <cache name="authenticationCache"
           maxEntriesLocalHeap="2000"
           eternal="false"
           timeToIdleSeconds="1800"
           timeToLiveSeconds="1800"
           overflowToDisk="false"
           statistics="true">
    </cache>

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

# 四、修改UserRealm类，添加构造函数

```
package com.songguoliang.springboot.shiro;

import com.songguoliang.springboot.entity.SysUser;
import com.songguoliang.springboot.service.SysPermissionService;
import com.songguoliang.springboot.service.SysUserService;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authc.credential.CredentialsMatcher;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.cache.CacheManager;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.util.ByteSource;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.List;

/**
 * @Description
 * @Author sgl
 * @Date 2018-06-11 17:07
 */
public class UserRealm extends AuthorizingRealm {

	private static final Logger LOGGER = LoggerFactory.getLogger(UserRealm.class);
	
	/**
     * 注意此处需要添加@Lazy注解，否则SysUserService缓存注解、事务注解不生效
     */
    @Autowired
    @Lazy
    private SysUserService sysUserService;

    @Autowired
    @Lazy
    private SysPermissionService sysPermissionService;
	
    public UserRealm(CacheManager cacheManager, CredentialsMatcher matcher) {
        super(cacheManager, matcher);
    }

	//省略代码……

}

```


# 五、创建自定义CacheManager类

由于Spring和Shiro都各自维护了自己的Cache抽象，为防止UserRealm注入的service里缓存注解和事务注解失效，所以定义自己的CacheManager处理缓存。

ShiroSpringCacheManager类内容如下：

```
package com.songguoliang.springboot.shiro;

import net.sf.ehcache.Ehcache;
import net.sf.ehcache.Element;
import org.apache.shiro.cache.Cache;
import org.apache.shiro.cache.CacheException;
import org.apache.shiro.cache.CacheManager;

import java.util.ArrayList;
import java.util.Collection;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;

/**
 * @Description
 * @Author sgl
 * @Date 2018-06-05 11:56
 */
public class ShiroSpringCacheManager implements CacheManager {

    private org.springframework.cache.CacheManager cacheManager;

    public ShiroSpringCacheManager(org.springframework.cache.CacheManager cacheManager) {
        this.cacheManager = cacheManager;
    }

    @Override
    public <K, V> Cache<K, V> getCache(String name) throws CacheException {
        org.springframework.cache.Cache cache = cacheManager.getCache(name);
        return new ShiroSpringCache<>(cache);
    }

    class ShiroSpringCache<K, V> implements Cache<K, V> {
        private org.springframework.cache.Cache springCache;

        public ShiroSpringCache(org.springframework.cache.Cache springCache) {
            if (springCache == null) {
                throw new IllegalArgumentException("Cache argument cannot be null.");
            }
            this.springCache = springCache;
        }

        @Override
        public V get(K key) throws CacheException {
            org.springframework.cache.Cache.ValueWrapper valueWrapper = springCache.get(key);
            if (valueWrapper != null) {
                return (V) valueWrapper.get();
            }
            return null;
        }

        @Override
        public V put(K key, V value) throws CacheException {
            V previous = get(key);
            springCache.put(key, value);
            return previous;
        }

        @Override
        public V remove(K key) throws CacheException {
            V previous = get(key);
            springCache.evict(key);
            return previous;
        }

        @Override
        public void clear() throws CacheException {
            springCache.clear();
        }

        @Override
        public int size() {
            Object nativeCache = springCache.getNativeCache();
            if (nativeCache instanceof Ehcache) {
                Ehcache ehcache = (Ehcache) nativeCache;
                return ehcache.getSize();
            }
            throw new UnsupportedOperationException("invoke spring cache abstract size method not supported");
        }

        @Override
        public Set<K> keys() {
            Object nativeCache = springCache.getNativeCache();
            if (nativeCache instanceof Ehcache) {
                Ehcache ehcache = (Ehcache) nativeCache;
                return new HashSet<>(ehcache.getKeys());
            }
            throw new UnsupportedOperationException("invoke spring cache abstract keys method not supported");
        }

        @Override
        public Collection<V> values() {
            Object nativeCache = springCache.getNativeCache();
            if (nativeCache instanceof Ehcache) {
                Ehcache ehcache = (Ehcache) nativeCache;
                List keys = ehcache.getKeys();
                Map<Object, Element> elementMap = ehcache.getAll(keys);
                List<Object> values = new ArrayList<>();
                for (Element element : elementMap.values()) {
                    values.add(element.getObjectValue());
                }
                return (Collection<V>) values;
            }
            throw new UnsupportedOperationException("invoke spring cache abstract values method not supported");
        }
    }

}

```


# 六、ShiroConfig类里修改

1、 在ShiroConfig类里添加以下配置：


```
/**
 * 缓存管理器
 * @param cacheManager
 * @return
 */
@Bean
public ShiroSpringCacheManager shiroSpringCacheManager(org.springframework.cache.CacheManager cacheManager) {
    return new ShiroSpringCacheManager(cacheManager);
}

@Bean
@DependsOn("lifecycleBeanPostProcessor")
public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
    DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
    defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
    return defaultAdvisorAutoProxyCreator;
}
```

2、修改原有配置如下：

```
 /**
 * 自定义realm
 *
 * @return
 */
@Bean
public UserRealm userRealm(CacheManager cacheManager) {
    UserRealm userRealm = new UserRealm(cacheManager, hashedCredentialsMatcher());
    userRealm.setAuthenticationCacheName("authenticationCache");
    userRealm.setAuthorizationCacheName("authorizationCache");
    return userRealm;
}

/**
 * 安全管理器
 * 注：使用shiro-spring-boot-starter 1.4时，返回类型是SecurityManager会报错，直接引用shiro-spring则不报错
 *
 * @return
 */
@Bean
public DefaultWebSecurityManager securityManager(CacheManager cacheManager) {
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    securityManager.setRealm(userRealm(cacheManager));
    securityManager.setCacheManager(cacheManager);
    return securityManager;
}
```

3、完整的ShiroConfig类内容：

```
package com.songguoliang.springboot.shiro;

import org.apache.shiro.authc.credential.HashedCredentialsMatcher;
import org.apache.shiro.cache.CacheManager;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.DependsOn;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * @Description
 * @Author sgl
 * @Date 2018-06-11 17:23
 */
@Configuration
public class ShiroConfig {

    /**
     * 缓存管理器
     * @param cacheManager
     * @return
     */
    @Bean
    public ShiroSpringCacheManager shiroSpringCacheManager(org.springframework.cache.CacheManager cacheManager) {
        return new ShiroSpringCacheManager(cacheManager);
    }

    /**
     * 凭证匹配器
     *
     * @return
     */
    @Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher() {
        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
        //md5加密1次
        hashedCredentialsMatcher.setHashAlgorithmName("md5");
        hashedCredentialsMatcher.setHashIterations(1);
        return hashedCredentialsMatcher;
    }

    /**
     * 自定义realm
     *
     * @return
     */
    @Bean
    public UserRealm userRealm(CacheManager cacheManager) {
        UserRealm userRealm = new UserRealm(cacheManager, hashedCredentialsMatcher());
        userRealm.setAuthenticationCacheName("authenticationCache");
        userRealm.setAuthorizationCacheName("authorizationCache");
        return userRealm;
    }

    /**
     * 安全管理器
     * 注：使用shiro-spring-boot-starter 1.4时，返回类型是SecurityManager会报错，直接引用shiro-spring则不报错
     *
     * @return
     */
    @Bean
    public DefaultWebSecurityManager securityManager(CacheManager cacheManager) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(userRealm(cacheManager));
        securityManager.setCacheManager(cacheManager);
        return securityManager;
    }


    /**
     * 设置过滤规则
     *
     * @param securityManager
     * @return
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        shiroFilterFactoryBean.setLoginUrl("/login");
        shiroFilterFactoryBean.setSuccessUrl("/");
        shiroFilterFactoryBean.setUnauthorizedUrl("/unauth");

        //注意此处使用的是LinkedHashMap，是有顺序的，shiro会按从上到下的顺序匹配验证，匹配了就不再继续验证
        //所以上面的url要苛刻，宽松的url要放在下面，尤其是"/**"要放到最下面，如果放前面的话其后的验证规则就没作用了。
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        filterChainDefinitionMap.put("/static/**", "anon");
        filterChainDefinitionMap.put("/login", "anon");
        filterChainDefinitionMap.put("/captcha.jpg", "anon");
        filterChainDefinitionMap.put("/favicon.ico", "anon");
        filterChainDefinitionMap.put("/**", "authc");

        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }


    @Bean
    @DependsOn("lifecycleBeanPostProcessor")
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
        return defaultAdvisorAutoProxyCreator;
    }


}

```


至此，我们以将shiro和ehcache集成完毕，有个需要特别注意的是，UserRealm里注入的SysUserService等service，需要延迟注入，所以都要添加@Lazy注解(如果不加需要自己延迟注入)，否则会导致该service里的@Cacheable缓存注解、@Transactional事务注解等失效。













<br><br><br><br>

源码： 
[github](https://github.com/itinypocket/spring-boot-study/tree/master/spring-boot-shiro-ehcache) 
[码云](https://gitee.com/itinypocket/spring-boot-study/tree/master/spring-boot-shiro-ehcache)









