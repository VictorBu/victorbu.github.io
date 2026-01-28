---
title: 学习 Spring (十) 注解之 @Bean, @ImportResource, @Value, 基于泛型的自动装配
date: 2019-03-01 07:00:00
updated: 2019-03-01 07:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

# @Bean

@Bean 标识一个用于配置和初始化一个由 Spring IoC 容器管理的新对象的方法，类似于 XML 配置文件的 <bean/>

可以在 Spring 的 @Configuration 注解的类中使用 @Bean 注解任何方法，在方法里面创建对象返回

```
@Configuration
public class AppConfig{
    @Bean
	public MyService myService(){
	    return new MyServiceImpl();
	}
}
```

## 示例

新建类：

```
public interface Store<T> {

}


public class StringStore implements Store<String> {
	
	public void init() {
		System.out.println("This is init.");
	}
	
	public void destroy() {
		System.out.println("This is destroy.");
	}
	
}

@Configuration
public class StoreConfig {

	@Bean(initMethod = "init", destroyMethod = "destroy")// 如果没有指定 name，那么name 为方法的名称
	public StringStore stringStore() {
		return new StringStore();
	}

}
```

添加测试：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestJavabased extends UnitTestBase {
	
	public TestJavabased() {
		super("classpath*:spring-beanannotation.xml");
	}
	
	@Test
	public void test() {
		Store store = super.getBean("stringStore");
		System.out.println(store.getClass().getName());
	}
	
}
```

@Bean 默认是单例的，如果要改变作用域范围，可以再添加 @Scope 注解

# @ImportResource

```
<beans>
        
    <context:annotation-config/>
    <!--加载资源文件-->
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <!--引用资源文件中的配置内容-->
    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.username}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
        
 </beans>
```

使用 @ImportResource 可以代替上面 XML 配置：

```
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig{
    @Value("${jdbc.url")
	private String url;
	
	@Value("${jdbc.username}")
	private String username;
	
	@Value("${jdbc.password}")
	private String password;
	
	@Bean
	public DataSource dataSource(){
	    return new DriverManagerDataSource(url, username, password);
	}
}
```

## 示例：

添加类：

```
public class MyDriverManager {
	
	public MyDriverManager(String url, String userName, String password) {
		System.out.println("url : " + url);
		System.out.println("userName: " + userName);
		System.out.println("password: " + password);
	}

}
```

添加配置文件 config.properties：

```
jdbc.username=root
jdbc.password=root
jdbc.url=127.0.0.1
```

添加配置文件 config.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd" >
        
    <context:property-placeholder location="classpath:/config.properties"/>
    
</beans>
```

修改类 StoreConfig：

```
@Configuration
@ImportResource("classpath:config.xml")
public class StoreConfig {

	@Value("${jdbc.url}")
	private String url;

	@Value("${jdbc.username}")
	private String username;

	@Value("${jdbc.password}")
	private String password;

	@Bean
	public MyDriverManager myDriverManager(){
		return new MyDriverManager(url, username, password);
	}


//	@Bean(initMethod = "init", destroyMethod = "destroy")// 如果没有指定 name，那么name 为 方法的名称
//	public StringStore stringStore() {
//		return new StringStore();
//	}

}
```

添加测试：

```
@Test
public void testMyDriverManager() {
	MyDriverManager manager = super.getBean("myDriverManager");
	System.out.println(manager.getClass().getName());
}
```

# 基于泛型的自动装配

## 示例

新建类：

```
public class IntegerStore implements Store<Integer> {

}
```

修改类：

```
@Configuration
@ImportResource("classpath:config.xml")
public class StoreConfig {

	@Autowired
	@Qualifier(value="stringStore")
	private Store<String> s1;

	@Autowired
	@Qualifier(value="integerStore")
	private Store<Integer> s2;


	@Bean
	public StringStore stringStore() {
		return new StringStore();
	}
	@Bean
	public IntegerStore integerStore() {
		return new IntegerStore();
	}

	@Bean
	public StringStore stringStoreTest(){
		System.out.println("s1: " + s1.getClass().getName());
		System.out.println("s2: " + s2.getClass().getName());
		return new StringStore();
	}
}
```

添加测试：

```
@Test
public void testG() {
	Store store = super.getBean("stringStoreTest");
}
```



# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
