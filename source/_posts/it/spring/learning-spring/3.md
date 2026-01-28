---
title: 学习 Spring (三) Bean 的配置项 & 作用域
date: 2019-02-21 23:00:00
updated: 2019-02-21 23:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

# 配置项

+ Id: 整个 IoC 容器中的唯一标识
+ Class: 具体实例化的类(必须配置项)
+ Scope: 作用域
+ Constructor arguments: 构造器参数
+ Properties: 属性
+ Autowiring mode: 自动装配模式
+ lazy-initialization mode: 懒加载模式
+ Initialization/destruction method: 初始化/销毁 方法

# 作用域

+ singleton: 单例(默认模式)，指一个 Bean 容器中只存在一份
+ prototype: 每次请求(每次使用创建新的实例)，destory 方式不生效
+ request: 每次 http 请求创建一个实例且仅在当前 request 內有效
+ session: 同上，每次 http 请求创建，当前 session 内有效
+ global session: 基于 portlet 的 web 中有效(portlet 定义了 global session)，如果是在 web 中同 session

# 作用域示例

添加 BeanScope：

```
public class BeanScope {
	
	public void say() {
		System.out.println("BeanScope say : " + this.hashCode());
	}
	
}
```

## singleton

添加配置文件 spring-beanscope-singleton.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >
        
        <bean id="beanScope" class="com.karonda.bean.BeanScope" scope="singleton"></bean>
        
 </beans>
```

添加测试 TestBeanScopeSingleton：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestBeanScopeSingleton extends UnitTestBase {
	
	public TestBeanScopeSingleton() {
		super("classpath*:spring-beanscope-singleton.xml");
	}
	
	@Test
	public void testSay() {
		BeanScope beanScope = super.getBean("beanScope");
		beanScope.say();
		
		BeanScope beanScope2 = super.getBean("beanScope");
		beanScope2.say();
	}

}
```

## prototype

添加配置文件 spring-beanscope-prototype.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >
        
        <bean id="beanScope" class="com.karonda.bean.BeanScope" scope="prototype"></bean>
        
 </beans>
```

添加测试 TestBeanScopePrototype：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestBeanScopePrototype extends UnitTestBase {

	public TestBeanScopePrototype() {
		super("classpath*:spring-beanscope-prototype.xml");
	}
	
	@Test
	public void testSay() {
		BeanScope beanScope = super.getBean("beanScope");
		beanScope.say();

		beanScope = super.getBean("beanScope");
		beanScope.say();
	}

}
```


# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
