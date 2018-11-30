# 使用新版本mysql的jdbc驱动时报时区问题

java.sql.SQLException: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.


# 一、问题
今天将mysql的驱动升级到最新版8.0.13后，项目报以下错误：

```

java.sql.SQLException: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:129) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:97) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:89) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:63) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:73) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:76) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:835) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:455) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:240) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:207) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.zaxxer.hikari.util.DriverDataSource.getConnection(DriverDataSource.java:136) ~[HikariCP-3.2.0.jar:na]
	at com.zaxxer.hikari.pool.PoolBase.newConnection(PoolBase.java:369) ~[HikariCP-3.2.0.jar:na]
	at com.zaxxer.hikari.pool.PoolBase.newPoolEntry(PoolBase.java:198) ~[HikariCP-3.2.0.jar:na]
	at com.zaxxer.hikari.pool.HikariPool.createPoolEntry(HikariPool.java:467) [HikariCP-3.2.0.jar:na]
	at com.zaxxer.hikari.pool.HikariPool.checkFailFast(HikariPool.java:541) [HikariCP-3.2.0.jar:na]
	at com.zaxxer.hikari.pool.HikariPool.<init>(HikariPool.java:115) [HikariCP-3.2.0.jar:na]
	at com.zaxxer.hikari.HikariDataSource.getConnection(HikariDataSource.java:112) [HikariCP-3.2.0.jar:na]
	at org.springframework.jdbc.datasource.DataSourceUtils.fetchConnection(DataSourceUtils.java:151) [spring-jdbc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.jdbc.datasource.DataSourceUtils.doGetConnection(DataSourceUtils.java:115) [spring-jdbc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.jdbc.datasource.DataSourceUtils.getConnection(DataSourceUtils.java:78) [spring-jdbc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.mybatis.spring.transaction.SpringManagedTransaction.openConnection(SpringManagedTransaction.java:82) [mybatis-spring-1.3.2.jar:1.3.2]
	at org.mybatis.spring.transaction.SpringManagedTransaction.getConnection(SpringManagedTransaction.java:68) [mybatis-spring-1.3.2.jar:1.3.2]
	at org.apache.ibatis.executor.BaseExecutor.getConnection(BaseExecutor.java:338) [mybatis-3.4.6.jar:3.4.6]
	at org.apache.ibatis.executor.SimpleExecutor.prepareStatement(SimpleExecutor.java:84) [mybatis-3.4.6.jar:3.4.6]
	at org.apache.ibatis.executor.SimpleExecutor.doQuery(SimpleExecutor.java:62) [mybatis-3.4.6.jar:3.4.6]
	at org.apache.ibatis.executor.BaseExecutor.queryFromDatabase(BaseExecutor.java:326) [mybatis-3.4.6.jar:3.4.6]
	at org.apache.ibatis.executor.BaseExecutor.query(BaseExecutor.java:156) [mybatis-3.4.6.jar:3.4.6]
	at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:109) [mybatis-3.4.6.jar:3.4.6]
	at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:83) [mybatis-3.4.6.jar:3.4.6]
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:148) [mybatis-3.4.6.jar:3.4.6]
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:141) [mybatis-3.4.6.jar:3.4.6]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_152]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_152]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_152]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_152]
	at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:433) [mybatis-spring-1.3.2.jar:1.3.2]
	at com.sun.proxy.$Proxy57.selectList(Unknown Source) [na:na]
	at org.mybatis.spring.SqlSessionTemplate.selectList(SqlSessionTemplate.java:230) [mybatis-spring-1.3.2.jar:1.3.2]
	at org.apache.ibatis.binding.MapperMethod.executeForMany(MapperMethod.java:139) [mybatis-3.4.6.jar:3.4.6]
	at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:76) [mybatis-3.4.6.jar:3.4.6]
	at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:59) [mybatis-3.4.6.jar:3.4.6]
	at com.sun.proxy.$Proxy58.getUsers(Unknown Source) [na:na]
	at com.joydotech.gfk.service.UserService.getUsers(UserService.java:20) [classes/:na]
	at com.joydotech.gfk.controller.UserController.list(UserController.java:22) [classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_152]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_152]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_152]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_152]
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:215) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:142) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:102) [spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:895) [spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:800) [spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) [spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1038) [spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:942) [spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:998) [spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:890) [spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:634) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:875) [spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:741) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53) [tomcat-embed-websocket-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:92) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:93) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:199) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:490) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:408) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:770) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1415) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_152]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_152]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.12.jar:9.0.12]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_152]
Caused by: com.mysql.cj.exceptions.InvalidConnectionAttributeException: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method) ~[na:1.8.0_152]
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62) ~[na:1.8.0_152]
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45) ~[na:1.8.0_152]
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423) ~[na:1.8.0_152]
	at com.mysql.cj.exceptions.ExceptionFactory.createException(ExceptionFactory.java:61) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.exceptions.ExceptionFactory.createException(ExceptionFactory.java:85) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.util.TimeUtil.getCanonicalTimezone(TimeUtil.java:132) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.protocol.a.NativeProtocol.configureTimezone(NativeProtocol.java:2234) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.protocol.a.NativeProtocol.initServerSession(NativeProtocol.java:2258) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.ConnectionImpl.initializePropsFromServer(ConnectionImpl.java:1319) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.ConnectionImpl.connectOneTryOnly(ConnectionImpl.java:966) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	at com.mysql.cj.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:825) ~[mysql-connector-java-8.0.13.jar:8.0.13]
	... 91 common frames omitted
	
```

# 二、问题原因

新版本mysql驱动需要加时区。

# 三、解决方法

在jdbc连接的url后面加上`serverTimezone=Asia/Shanghai` (中国标准时间)即可，如下：

```
spring.datasource.url=jdbc:mysql://localhost:3306/demo?serverTimezone=Asia/Shanghai

```

如果需要使用统一标准世界时间，添加`serverTimezone=UTC`即可，UTC表示世界标准时间

# 四、java代码获取时区的方法

```java
Calendar calendar = Calendar.getInstance();
TimeZone timeZone = calendar.getTimeZone();
System.out.println(timeZone.getID());
System.out.println(timeZone.getDisplayName());
```

# 五、避免中文乱码

添加`characterEncoding=utf8`，如下：

```
spring.datasource.url=jdbc:mysql://localhost:3306/demo?characterEncoding=utf8&serverTimezone=Asia/Shanghai
```

# 六、避免useSSL警告

添加`useSSL=true`，如下：

```
spring.datasource.url=jdbc:mysql://localhost:3306/demo?characterEncoding=utf8&useSSL=true&serverTimezone=Asia/Shanghai
```





<br/><br/><br/><br/>

