---
title: 学习 Spring (五) Aware 接口
date: 2019-02-22 17:00:00
updated: 2019-02-22 17:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

Spring 中提供了一些以 Aware 结尾的接口，实现了 Aware 接口的 bean 在被初始化之后可以获取相应资源

+ 通过 Aware 接口，可以对 Spring 相应资源进行操作(一定要慎重)
+ 为对 Spring 进行简单的扩展提供了方便的入口

# 示例

## ApplicationContextAware

添加配置文件 spring-aware-applicationcontext.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >
        
         <bean id="moocApplicationContext" class="com.karonda.aware.MoocApplicationContext" ></bean>
        
 </beans>
```

实现接口：

```
public class MoocApplicationContext implements ApplicationContextAware  {
	
	@Override
	public void setApplicationContext(ApplicationContext applicationContext)
			throws BeansException {
		System.out.println("MoocApplicationContext : " + applicationContext.getBean("moocApplicationContext").hashCode());
	}
	
}
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestApplicationContextAware extends UnitTestBase {
	
	public TestApplicationContextAware() {
		super("classpath:spring-aware-applicationcontext.xml");
	}
	
	@Test
	public void testMoocApplicationContext() {
		System.out.println("testMoocApplicationContext : " + super.getBean("moocApplicationContext").hashCode());
	}
	
}
```

## MoocBeanName

添加配置文件 classpath:spring-aware-beanname.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >

	<bean id="moocBeanName" class="com.karonda.aware.MoocBeanName" ></bean>
        
 </beans>
```

实现接口：

```
public class MoocBeanName implements BeanNameAware {
	
	@Override
	public void setBeanName(String name) {
		System.out.println("MoocBeanName : " + name);
	}

}
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestBeanNameAware extends UnitTestBase {

	public TestBeanNameAware() {
		super("classpath:spring-aware-beanname.xml");
	}
	
	@Test
	public void textMoocBeanName() {
		System.out.println("textMoocBeanName : " + super.getBean("moocBeanName"));
	}
	
}
```

## 同时实现 ApplicationContextAware 和 MoocBeanName

修改 MoocBeanName：

```
public class MoocBeanName implements BeanNameAware, ApplicationContextAware {

	private String beanName;
	
	@Override
	public void setBeanName(String name) {
		this.beanName = name;
		System.out.println("MoocBeanName : " + name);
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext)
			throws BeansException {
		System.out.println("setApplicationContext : " + applicationContext.getBean(this.beanName).hashCode());
	}

}
```

修改 TestBeanNameAware：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestBeanNameAware extends UnitTestBase {

	public TestBeanNameAware() {
		super("classpath:spring-aware-beanname.xml");
	}
	
	@Test
	public void textMoocBeanName() {
		System.out.println("textMoocBeanName : " + super.getBean("moocBeanName").hashCode());
	}
	
}
```

# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
