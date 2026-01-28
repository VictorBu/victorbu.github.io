---
title: 学习 Spring (六) 自动装配
date: 2019-02-25 15:30:00
updated: 2019-02-25 15:30:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

1. No: (默认)不做任何操作
1. byName: 根据属性名自动装配。此选项将检查容器并根据名字查找与属性完全一致的 bean，并将其与属性自动装配
1. byType: 如果容器中存在一个与指定属性类型相同的 bean，那么将与该属性自动装配；如果存在多个该类型的 bean，那么抛出异常，并指出不能使用 byType 方式进行自动装配；如果没有找到相匹配的 bean，则什么事都不发生
1. constructor: 与 byType 方式类似，不同之处在于它应用于构造器参数。如果容器中没有找到与构造器参数类型一致的 bean 则抛出异常

准备工作，添加 AutoWiringDAO：

```
public class AutoWiringDAO {
	
	public void say(String word) {
		System.out.println("AutoWiringDAO : " + word);
	}

}
```

# byName

添加 AutoWiringServiceByName：

```
public class AutoWiringServiceByName {
	
	private AutoWiringDAO autoWiringDAO;

	public void setAutoWiringDAO(AutoWiringDAO autoWiringDAO) {
		this.autoWiringDAO = autoWiringDAO;
	}
	
	public void say(String word) {
		this.autoWiringDAO.say(word);
	}

}
```

添加配置文件 spring-autowiring-byname.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
        default-autowire="byName">
        
        <bean id="autoWiringService" class="com.karonda.autowiring.AutoWiringServiceByName" ></bean>
        
        <bean id="autoWiringDAO" class="com.karonda.autowiring.AutoWiringDAO" ></bean>
	
 </beans>
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestAutoWiringByName extends UnitTestBase {
	
	public TestAutoWiringByName() {
		super("classpath:spring-autowiring-byname.xml");
	}
	
	@Test
	public void testSay() {
		AutoWiringServiceByName service = super.getBean("autoWiringService");
		service.say(" this is a test");
	}

}
```

# byType

添加 AutoWiringServiceByType：

```
public class AutoWiringServiceByType {
	
	private AutoWiringDAO autoWiringDAO;

	public void setAutoWiringDAO(AutoWiringDAO autoWiringDAO) {
		this.autoWiringDAO = autoWiringDAO;
	}
	
	public void say(String word) {
		this.autoWiringDAO.say(word);
	}

}
```

添加配置文件 spring-autowiring-bytype.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
        default-autowire="byType">
        
        <bean id="autoWiringService" class="com.karonda.autowiring.AutoWiringServiceByType" ></bean>
        
        <bean class="com.karonda.autowiring.AutoWiringDAO" ></bean>
	
 </beans>
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestAutoWiringByType extends UnitTestBase {

	public TestAutoWiringByType() {
		super("classpath:spring-autowiring-bytype.xml");
	}
	
	@Test
	public void testSay() {
		AutoWiringServiceByType service = super.getBean("autoWiringService");
		service.say(" this is a test");
	}

}
```

# constructor

添加 AutoWiringServiceConstructor

```
public class AutoWiringServiceConstructor {
	
	private AutoWiringDAO autoWiringDAO;

	public AutoWiringServiceConstructor(AutoWiringDAO autoWiringDAO){
		this.autoWiringDAO = autoWiringDAO;
	}
	public void say(String word) {
		this.autoWiringDAO.say(word);
	}

}
```

添加配置文件 spring-autowiring-constructor.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
        default-autowire="constructor">
        
        <bean id="autoWiringService" class="com.karonda.autowiring.AutoWiringServiceConstructor" ></bean>
        
        <bean class="com.karonda.autowiring.AutoWiringDAO" ></bean>
	
 </beans>
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestAutoWiringConstructor extends UnitTestBase {

	public TestAutoWiringConstructor() {
		super("classpath:spring-autowiring-constructor.xml");
	}
	
	@Test
	public void testSay() {
		AutoWiringServiceConstructor service = super.getBean("autoWiringService");
		service.say(" this is a test");
	}

}
```

 

# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
