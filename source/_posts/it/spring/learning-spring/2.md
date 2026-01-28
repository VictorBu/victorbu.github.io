---
title: 学习 Spring (二) Spring 注入
date: 2019-02-21 22:00:00
updated: 2019-02-22 21:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

# 常用的两种注入方式

1. 设值注入
1. 构造注入

# 示例准备工作

添加 InjectionDAO：

```
public interface InjectionDAO {
	
	void save(String arg);
	
}
```

添加 InjectionDAOImpl：

```
public class InjectionDAOImpl implements InjectionDAO {
	
	public void save(String arg) {
		System.out.println("保存数据：" + arg);
	}

}
```

添加 InjectionService：

```
public interface InjectionService {
	
	void save(String arg);
	
}
```

# 设值注入

添加 InjectionServicePropertyImpl：

```
public class InjectionServicePropertyImpl implements InjectionService {

	private InjectionDAO injectionDAO;
	
	public void setInjectionDAO(InjectionDAO injectionDAO) {
		this.injectionDAO = injectionDAO;
	}

	public void save(String arg) {
		System.out.println("Service(Property)接收参数：" + arg);
		arg = arg + ":" + this.hashCode();
		injectionDAO.save(arg);
	}
	
}
```

添加配置文件 classpath:spring-injection-property.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >
        
     <bean id="injectionService" class="com.karonda.ioc.injection.service.InjectionServicePropertyImpl">
        <property name="injectionDAO" ref="injectionDAO"></property>
     </bean>

    <bean id="injectionDAO" class="com.karonda.ioc.injection.dao.InjectionDAOImpl"></bean>
 </beans>
```

添加测试 TestInjectionProperty：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestInjectionProperty extends UnitTestBase {

	public TestInjectionProperty() {
		super("classpath:spring-injection-property.xml");
	}
	
	@Test
	public void testSetter() {
		InjectionService service = super.getBean("injectionService");
		service.save("这是要保存的数据");
	}
	
}
```

# 构造注入

添加 InjectionServiceConstructorImpl：

```
public class InjectionServiceConstructorImpl implements InjectionService {
	
	private InjectionDAO injectionDAO;
	
	//构造器注入
	public InjectionServiceConstructorImpl(InjectionDAO injectionDAO) {
		this.injectionDAO = injectionDAO;
	}

	public void save(String arg) {
		//模拟业务操作
		System.out.println("Service(Constructor)接收参数：" + arg);
		arg = arg + ":" + this.hashCode();
		injectionDAO.save(arg);
	}
	
}
```

添加配置文件 spring-injection-constructor.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >

		<bean id="injectionService" class="com.karonda.ioc.injection.service.InjectionServiceConstructorImpl">
        	<constructor-arg name="injectionDAO" ref="injectionDAO"></constructor-arg>
        </bean>
        
        <bean id="injectionDAO" class="com.karonda.ioc.injection.dao.InjectionDAOImpl"></bean>
	
 </beans>
```

添加测试 TestInjectionConstructor

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestInjectionConstructor extends UnitTestBase {
	
	public TestInjectionConstructor() {
		super("classpath:spring-injection-constructor.xml");
	}

	
	@Test
	public void testCons() {
		InjectionService service = super.getBean("injectionService");
		service.save("这是要保存的数据");
	}
	
}
```

# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
