# 升级fastJson版本报错：java.lang.IllegalArgumentException: Content-Type cannot contain wildcard type '*' 


# 一、问题描述

在springboot项目中，将fastJson版本升级到`1.2.59`，然后用postman工具访问get请求方法时报错java.lang.IllegalArgumentException: Content-Type cannot contain wildcard type '*'，详细错误信息如下：

```
2019-09-03 11:49:28.276 ERROR 32168 --- [nio-8080-exec-3] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.IllegalArgumentException: Content-Type cannot contain wildcard type '*'] with root cause

java.lang.IllegalArgumentException: Content-Type cannot contain wildcard type '*'
	at org.springframework.util.Assert.isTrue(Assert.java:118) ~[spring-core-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.http.HttpHeaders.setContentType(HttpHeaders.java:921) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.http.converter.AbstractHttpMessageConverter.addDefaultHeaders(AbstractHttpMessageConverter.java:256) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.http.converter.AbstractHttpMessageConverter.write(AbstractHttpMessageConverter.java:211) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter.write(FastJsonHttpMessageConverter.java:244) ~[fastjson-1.2.59.jar:na]
	at org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodProcessor.writeWithMessageConverters(AbstractMessageConverterMethodProcessor.java:290) ~[spring-webmvc-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor.handleReturnValue(RequestResponseBodyMethodProcessor.java:180) ~[spring-webmvc-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.method.support.HandlerMethodReturnValueHandlerComposite.handleReturnValue(HandlerMethodReturnValueHandlerComposite.java:82) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:122) ~[spring-webmvc-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:892) ~[spring-webmvc-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:797) ~[spring-webmvc-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) ~[spring-webmvc-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1039) ~[spring-webmvc-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:942) ~[spring-webmvc-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1005) ~[spring-webmvc-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:897) ~[spring-webmvc-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:634) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:882) ~[spring-webmvc-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:741) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53) ~[tomcat-embed-websocket-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:92) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:93) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118) ~[spring-web-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202) ~[tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:490) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:408) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:853) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1587) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_162]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_162]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.22.jar:9.0.22]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_162]

```

# 解决方法

原因是为了安全增加了保护机制，Content-Type不能含有通配符，强制要求用户自己配置MediaType，如下：

```

//升级最新版本需加
List<MediaType> supportedMediaTypes = new ArrayList<>();
supportedMediaTypes.add(MediaType.APPLICATION_JSON);
supportedMediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
supportedMediaTypes.add(MediaType.APPLICATION_ATOM_XML);
supportedMediaTypes.add(MediaType.APPLICATION_FORM_URLENCODED);
supportedMediaTypes.add(MediaType.APPLICATION_OCTET_STREAM);
supportedMediaTypes.add(MediaType.APPLICATION_PDF);
supportedMediaTypes.add(MediaType.APPLICATION_RSS_XML);
supportedMediaTypes.add(MediaType.APPLICATION_XHTML_XML);
supportedMediaTypes.add(MediaType.APPLICATION_XML);
supportedMediaTypes.add(MediaType.IMAGE_GIF);
supportedMediaTypes.add(MediaType.IMAGE_JPEG);
supportedMediaTypes.add(MediaType.IMAGE_PNG);
supportedMediaTypes.add(MediaType.TEXT_EVENT_STREAM);
supportedMediaTypes.add(MediaType.TEXT_HTML);
supportedMediaTypes.add(MediaType.TEXT_MARKDOWN);
supportedMediaTypes.add(MediaType.TEXT_PLAIN);
supportedMediaTypes.add(MediaType.TEXT_XML);
fastConverter.setSupportedMediaTypes(supportedMediaTypes);

```

完整代码如下：

```

@Bean
public HttpMessageConverters fastJsonHttpMessageConverters() {
    //1、定义一个convert转换消息的对象
    FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();

    //升级最新版本需加
    List<MediaType> supportedMediaTypes = new ArrayList<>();
    supportedMediaTypes.add(MediaType.APPLICATION_JSON);
    supportedMediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
    supportedMediaTypes.add(MediaType.APPLICATION_ATOM_XML);
    supportedMediaTypes.add(MediaType.APPLICATION_FORM_URLENCODED);
    supportedMediaTypes.add(MediaType.APPLICATION_OCTET_STREAM);
    supportedMediaTypes.add(MediaType.APPLICATION_PDF);
    supportedMediaTypes.add(MediaType.APPLICATION_RSS_XML);
    supportedMediaTypes.add(MediaType.APPLICATION_XHTML_XML);
    supportedMediaTypes.add(MediaType.APPLICATION_XML);
    supportedMediaTypes.add(MediaType.IMAGE_GIF);
    supportedMediaTypes.add(MediaType.IMAGE_JPEG);
    supportedMediaTypes.add(MediaType.IMAGE_PNG);
    supportedMediaTypes.add(MediaType.TEXT_EVENT_STREAM);
    supportedMediaTypes.add(MediaType.TEXT_HTML);
    supportedMediaTypes.add(MediaType.TEXT_MARKDOWN);
    supportedMediaTypes.add(MediaType.TEXT_PLAIN);
    supportedMediaTypes.add(MediaType.TEXT_XML);
    fastConverter.setSupportedMediaTypes(supportedMediaTypes);

    //2、添加fastjson的配置信息
    FastJsonConfig fastJsonConfig = new FastJsonConfig();
    fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
    //3、在convert中添加配置信息
    fastConverter.setFastJsonConfig(fastJsonConfig);
    //4、将convert添加到converters中
    HttpMessageConverter<?> converter = fastConverter;
    return new HttpMessageConverters(converter);
}

```



