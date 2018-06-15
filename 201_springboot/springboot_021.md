# Spring Boot整合shiro后导致@Cacheable、@Transactional等注解失效的问题


# 一、问题描述

- Springboot整合shiro前，service里的@Cacheable、@Transactional等注解都正常使用。
- 整合shiro之后，UserRealm类里自动注入的service中的注解失效

UserRealm代码如下:

```
public class UserRealm extends AuthorizingRealm {
    private static final Logger LOGGER = LoggerFactory.getLogger(UserRealm.class);
    /**
     * 这种情况下SysUserService里缓存、事务等注解失效
     */
    @Autowired
    private SysUserService sysUserService;
    
    //省略其他代码
}

```

SysUserService代码如下：

```
@Service
public class SysUserService {
    @Autowired
    private SysUserMapper userMapper;

    @Cacheable(cacheNames = "users", key = "'sysuser:'+#userName")
    public SysUser findByUserName(String userName) {
        return userMapper.findByUserName(userName);
    }
}

```

# 二、解决方法

**在UserRealm自动注入service时，添加@Lazy注解延迟注入**，如下：

```
public class UserRealm extends AuthorizingRealm {
    private static final Logger LOGGER = LoggerFactory.getLogger(UserRealm.class);
    /**
     * 添加@Lazy注解延迟注入
     */
    @Autowired
    @Lazy
    private SysUserService sysUserService;

	/**
     * 添加@Lazy注解延迟注入
     */
    @Autowired
    @Lazy
    private SysPermissionService sysPermissionService;
    
    //省略其他代码
}
```








<br><br><br><br>

参考：
https://stackoverflow.com/questions/21512791/spring-service-with-cacheable-methods-gets-initialized-without-cache-when-autowi












