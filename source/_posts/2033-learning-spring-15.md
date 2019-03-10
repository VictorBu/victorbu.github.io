---
title: 学习 Spring (十五) Advisor
date: 2019-03-10 08:00:00
updated: 2019-03-10 08:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

advisor 就像一个小的自包含的方面，只有一个 advice

切面自身通过一个 bean 表示，并且必须实现某个 advice 接口，同时 advisor 也可以很好的利用 AspectJ 的切入点表达式

Spring 通过配置文件中 <aop:advisor> 元素支持 advisor，实际使用中，大多数情况下它会和 transactional advice 配合使用

为了定义一个 advisor 的优先级以便让 advice 可以有序，可以使用 order 属性来定义 advisor 的顺序

# 示例

添加切面：

```
public class ConcurrentOperationExecutor implements Ordered {

	private static final int DEFAULT_MAX_RETRIES = 2;

	private int maxRetries = DEFAULT_MAX_RETRIES;
	
	private int order = 1;

	public void setMaxRetries(int maxRetries) {
		this.maxRetries = maxRetries;
	}

	public int getOrder() {
		return this.order;
	}

	public void setOrder(int order) {
		this.order = order;
	}

	public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
		int numAttempts = 0;
		PessimisticLockingFailureException lockFailureException;
		do {
			numAttempts++;
			System.out.println("Try times : " + numAttempts);
			try {
				return pjp.proceed();
			} catch (PessimisticLockingFailureException ex) {
				lockFailureException = ex;
			}
		} while (numAttempts <= this.maxRetries);
		System.out.println("Try error : " + numAttempts);
		throw lockFailureException;
	}
}
```

添加类：

```
@Service
public class InvokeService {
	
	public void invoke() {
		System.out.println("InvokeService ......");
	}
	
	public void invokeException() {
		throw new PessimisticLockingFailureException("");
	}

}
```

添加配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop.xsd">

	<context:component-scan base-package="com.karonda.aop.schema"></context:component-scan>

	<aop:config>
		<aop:aspect id="concurrentOperationRetry" ref="concurrentOperationExecutor">
			<aop:pointcut id="idempotentOperation"
				expression="execution(* com.karonda.aop.schema.advisors.service.*.*(..)) " />
			<aop:around pointcut-ref="idempotentOperation" method="doConcurrentOperation" />
		</aop:aspect>
	</aop:config>
	
	<bean id="concurrentOperationExecutor" class="com.karonda.aop.schema.advisors.ConcurrentOperationExecutor">
		<property name="maxRetries" value="3" />
		<property name="order" value="100" />
	</bean>

 </beans>
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestAOPSchemaAdvisors extends UnitTestBase {
	
	public TestAOPSchemaAdvisors() {
		super("classpath:spring-aop-schema-advisors.xml");
	}
	
	@Test
	public void testSave() {
		InvokeService service = super.getBean("invokeService");
		service.invoke();
		
		System.out.println();
		service.invokeException();
 	}

}
```

# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
