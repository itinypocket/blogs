# 同一个类里@Cacheable缓存不起作用


# 一、问题描述

环境：

- springboot 2.1.2.RELEASE
- ehcache 2.10.6

如下，`selectAll()`方法通过`@Cacheable`设置了缓存，在`get(String paramKey)`方法里面，调用`selectAll()`时不会使用缓存。但其他类调用selectAll()方法时，缓存有效。

```java

@Service
public class SystemConfigService{
	@Autowired
	private SystemConfigMapper systemConfigMapper;

	public SystemConfig get(String paramKey) {
		//此处调用selectAll方法，selectAll方法不会使用缓存
		List<SystemConfig> list =selectAll();
		
	}

	@Cacheable(cacheNames = "dict")
	public List<SystemConfig> selectAll() {
		return systemConfigMapper.selectAll();
	}
}

```



# 二、参考解决方法

问题原因：

注解@Cacheable是使用AOP代理实现的 ，通过创建内部类来代理缓存方法，类内部的方法调用类内部的缓存方法不会走代理，所以就不能正常创建缓存，所以每次都需要去调用数据库。

参考解决方法:

方法1：将缓存的方法单独放一个类里，与调用的方法分开，不放在同一个类里。
方法2：从ApplicationContext里获取当前类的代理对象。

方法1不贴代码了，方法2参考代码如下：

```

@Service
public class SystemConfigService{
	@Autowired
	private SystemConfigMapper systemConfigMapper;
	//注入当前对象的代理对象
	@Autowired
	private SystemConfigService _this;


	public SystemConfig get(String paramKey) {
		//通过代理对象访问方法
		List<SystemConfig> list = _this.selectAll();
	}

	@Cacheable(cacheNames = CacheConstant.ONE_HOUR)
	public List<SystemConfig> selectAll() {
		return systemConfigMapper.selectAll();
	}
}

```




