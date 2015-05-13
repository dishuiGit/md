
[TOC]

## struts总结

### 将常用字符串值设置为常量

```java  
//将字符串值设置为常量"loginEmp" 
public static final String LOGIN_EMP_INFO = "loginEmp";
```

### action scope设置
	 `action`设置`scope="prototype"`,否则爆`ConversionErrorInterceptor.java`错误!

### href表单提交方式:
	<a href="javascript:document.forms[0].submit()"></a>;

### 在登录方法中添加获取IP地址的操作

```java

	HttpServletRequest request = ServletActionContext.getRequest();
	String loginIp = request.getHeader("x-forwarded-for"); 
	if(loginIp == null || loginIp.length() == 0 || "unknown".equalsIgnoreCase(loginIp)) { 
		loginIp = request.getHeader("Proxy-Client-IP"); 
	} 
	if(loginIp == null || loginIp.length() == 0 || "unknown".equalsIgnoreCase(loginIp)) { 
		loginIp = request.getHeader("WL-Proxy-Client-IP"); 
	} 
	if(loginIp == null || loginIp.length() == 0 || "unknown".equalsIgnoreCase(loginIp)) { 
		loginIp = request.getRemoteAddr(); 
	}

```

### `checkboxlist` names,list,listkey,listvalue的应用


### struts json 配置

```xml
<result name="getAllGT" type="json">
	<!-- 设置数据来源 -->
	<param name="root">action</param>
	<param name="includePorperties">
		gtList\[\d+\]\.uuid,
		gtList\[\d+\]\.name
	</param>
</result>
```

提交表单:首先要确认

### struts 标签 
iterator标签取不到值  试着加#号
