## Spring IoC

1. 由spring管理,生成的类

	>Interface
	>:	生成出来的类实现的接口列表
	>	- `org.springframework.aop.SpringProxy`
	>	- `org.springframework.aop.framework.Advised`
	>	- `package.user_interfaces`(用户自定义实现的接口)
	>CgLib
	>:

2. BeanDefination
   :	解析applicationContext.xml
	
	   instantiateBean
	   :	bean实例化


## 设置接口

```java
public static Set<Class> getAllInterfacesForClassAsSet(Class clazz, ClassLoader classLoader) {
		Assert.notNull(clazz, "Class must not be null");
		if (clazz.isInterface() && isVisible(clazz, classLoader)) {
			return Collections.singleton(clazz);
		}
		Set<Class> interfaces = new LinkedHashSet<Class>();
		while (clazz != null) {
			Class<?>[] ifcs = clazz.getInterfaces();
			for (Class<?> ifc : ifcs) {
				interfaces.addAll(getAllInterfacesForClassAsSet(ifc, classLoader));
			}
			clazz = clazz.getSuperclass();
		}
		return interfaces;
	}
```

### 动态代理

1. JdkDynamicAopProxy


### spring注解解析

* TransactionAnnotationParser
	- SpringTransactionAnnotationParser


# AspectJ/Aop

### `Interface_base.java`
	
```java
public interface Interface_base {
			
	public void base();
}
```

### `Interface_A.java`

```java
public interface Interface_A extends Interface_base{

	public void save();
	public void find();
	public void update();
}
```

### `Test_advice.java`
- *在applicationContext.xml中配置*

```java
@Aspect
public class Test_advice{
	@Before(value = "execution(* cn.itcast.erp.test.Interface_A.*(..))")
	public void fn1(){
		System.out.println("前置增强.....");
	}
	@After(value = "execution(* cn.itcast.erp.test.Interface_base.*(..))")
	public void fn2(){
		System.out.println("后置增强.....");
	}
}
```

### `Test_impl.java`
- *在applicationContext.xml中配置*

```java
public class Test_impl implements Target_inter{
	//输出:
	//save/find/update
	//前置增强.....
	public void save() {
		System.out.println("save");
	}

	public void find() {
		System.out.println("find");
	}

	public void update() {
		System.out.println("update");
	}
	//--------------------------------------------
	//输出:
	//base
	//后置增强.....	   
	public void base() {
		System.out.println("base");
	}

}
```

### 结论

>**Aop方法增强**:

	- 接口继承,不会继承增强
	- 运行时,实现类调用方法时,会去判断该方法所属接口是否配置了advice(增强)
