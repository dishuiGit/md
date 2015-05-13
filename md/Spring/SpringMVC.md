[TOC]

### `springmvc.xml`

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
		http://www.springframework.org/schema/mvc 
		http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd 
		http://www.springframework.org/schema/context 
		http://www.springframework.org/schema/context/spring-context-3.0.xsd 
		http://www.springframework.org/schema/aop 
		http://www.springframework.org/schema/aop/spring-aop-3.0.xsd 
		http://www.springframework.org/schema/tx 
		http://www.springframework.org/schema/tx/spring-tx-3.0.xsd ">

	<!-- 配置式开发开始=========================================== -->
	<!-- 处理器 映射器 
	<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
	-->
	<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
		<!-- 集中管理Url -->
		<property name="mappings">
			<props>
				<prop key="/items1.action ">itemsController</prop>
				<prop key="/items2.action ">itemsController</prop>
				<prop key="/items3.action ">items1Controller</prop>
			</props>
		</property>
	</bean>
	
	<!-- 处理器 适配器  要求处理器实现Controller接口-->
	<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>
	<!-- 处理器 适配器  要求处理器实现HttpRequestHandler接口-->
	<bean class="org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter"></bean>
	
	<!-- 视图解释器     jspViewResolver 默认支持 jstl -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<!-- 配置前缀 -->
		<property name="prefix" value="/WEB-INF/jsp/"/>
		<!-- 配置后缀 -->
		<property name="suffix" value=".jsp"/>
	</bean>
	
	<!-- 处理器  
	<bean name="/items1.action" class="cn.itcast.core.controller.ItemsController"></bean>
	-->
	<bean id="itemsController" class="cn.itcast.core.controller.ItemsController"></bean>
	<bean id="items1Controller" class="cn.itcast.core.controller.Items1Controller"></bean>
	<!-- 配置式开发结束=========================================== -->
	
	
	<!-- 注解开发 开始============================================ -->
	<!-- 配置处理器映射器 -->
	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"></bean>
	<!-- 配置处理器适配器 -->
	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
		<!-- 配置类型转换 -->
		<!-- 配置日期转换 Converter -->
		<property name="webBindingInitializer" ref="customBinding"/>
		<!-- 支持JSON转换 -->
		<property name="messageConverters">
			<list>
				<bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"/>
			</list>
		</property>
	</bean>
	
	<!-- 自定义参数绑定  spring提供-->
	<bean id="customBinding" class="org.springframework.web.bind.support.ConfigurableWebBindingInitializer">
		<!-- 类型转换 -->
		<property name="conversionService" ref="conversionService"/>
		<!-- 校验JavaBean合法性 -->
		<property name="validator" ref="validator"/>
	</bean>
	<!-- 校验器 -->
	<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
		<!-- 指定由谁来完成此次校验 -->
		<property name="providerClass" value="org.hibernate.validator.HibernateValidator"/>
		<!-- 指定把提示信息放在配置文件中 -->
		<property name="validationMessageSource" ref="messageSource"></property>
	</bean>
	<!-- 校验错误信息配置文件 -->
	<bean id="messageSource"
		class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
		<!-- 资源文件名-->
		<property name="basenames">   
       	 <list>    
            <value>classpath:CustomValidationMessages</value> 
       	 </list>   
    	</property>
		<!-- 资源文件编码格式 -->
		<property name="fileEncodings" value="utf-8" />
		<!-- 对资源文件内容缓存时间，单位秒 -->
		<property name="cacheSeconds" value="120" />
	</bean>
	
	
	
	<!-- 参数格式  字符串类型  yyyy-MM-dd HH-mm-ss  格式化  -->
	<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
		<property name="converters">
			<list>
				<!-- 日期转换 -->
				<bean class="cn.itcast.common.converter.CustomDateConverter"/>
				<!-- 日期转换 -->
<!-- 				<bean class="cn.itcast.common.converter.CustomDate1Converter"/> -->
				<!-- 参数 传递 前后去掉空格 -->
				<bean class="cn.itcast.common.converter.CustomTrimConverter"/>
			</list>
		</property>
	</bean>
	
	<!-- 注解开发第二种配置文式  
	<mvc:annotation-driven />
	 -->
	 <mvc:interceptors>
	 	<mvc:interceptor>
	 		<mvc:mapping path="/**"/>
	 		<bean class="cn.itcast.common.inteceptor.LoginInteceptor"></bean>
	 	</mvc:interceptor>
	 </mvc:interceptors>
	 <!-- 拦截器 
	 <mvc:interceptors>
	 	<mvc:interceptor>
	 		<mvc:mapping path="/**"/>
	 		<bean class="cn.itcast.common.inteceptor.Inteceptor1"></bean>
	 	</mvc:interceptor>
	 	<mvc:interceptor>
	 		<mvc:mapping path="/**"/>
	 		<bean class="cn.itcast.common.inteceptor.Inteceptor2"></bean>
	 	</mvc:interceptor>
	 </mvc:interceptors>
	-->
	<!-- 配置 处理器  本身不用配置这里   配置扫描处理器-->
	<context:component-scan base-package="cn.itcast.core.controller"/>
	
	<!-- 实例化此异常处理类 -->
	<bean class="cn.itcast.common.exception.CustomExceptionResolver"/>
	
	<!-- 静态资源不拦截 location : 资源位置-->
	<mvc:resources location="/js/" mapping="/js/**"/>
	
	
	<!-- 配置Springmvc支持上传图片 -->
	<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<!-- 指定上传大小  上传的图片不能超过1M    默认是 B -->
		<property name="maxUploadSize" value="1048576"/>
	</bean>
	
	<!-- 注解开发 结束============================================ -->
		
</beans>
```

### 处理器 映射器 (注解解析类)
--------

```xml
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"></bean>
```

+ __`RequestMappingHandlerMapping.class`__

	![](../../pic/mybatis/HandlerMapping.png)

+ __spring生命周期图__

	![](../../pic/Spring生命周期.jpg)

### 处理器 适配器 (注解解析类)
------------

	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
	
+ __`RequestMappingHandlerAdapter.class`__

	![](../../pic/mybatis/HandlerAdapter.png)
