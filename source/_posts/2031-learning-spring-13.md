---
title: 学习 Spring (十三) AOP 配置
date: 2019-03-09 07:00:00
updated: 2019-03-09 07:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

Spring 所有的切面和通知器都必须放在一个 <aop:config> 内(可以配置包含多个 <aop:config> 元素)，每一个 <aop:config> 可以包含 pointcut, advisor 和 aspect 元素(它们必须按照这个顺序进行声明)

<aop:config> 风格的配置大量使用了 Spring 的自动代理机制

# 配置 Aspect

新建切面类：

```
public class MoocAspect {
	
}
```

添加配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

	<bean id="moocAspect" class="com.karonda.aop.schema.advice.MoocAspect"></bean>
	
	<aop:config>
		<aop:aspect id="moocAspectAOP" ref="moocAspect">

		</aop:aspect>
	</aop:config>

 </beans>
```

# 配置 Pointcut

pointcut 类型说明详见：[pointcut expressions](https://docs.spring.io/spring-framework/docs/5.1.5.RELEASE/spring-framework-reference/core.html#aop-pointcuts-examples)

新建类：

```
public class AspectBiz {

}
```

修改配置文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

	<bean id="moocAspect" class="com.karonda.aop.schema.advice.MoocAspect"></bean>

	<bean id="aspectBiz" class="com.karonda.aop.schema.advice.biz.AspectBiz"></bean>
	
	<aop:config>
		<aop:aspect id="moocAspectAOP" ref="moocAspect">
			<aop:pointcut id="moocPointcut" expression="execution(* com.karonda.aop.schema.advice.biz.*Biz.*(..))"/>

		</aop:aspect>
	</aop:config>

 </beans>

```

# Advice

添加依赖包 aspectjweaver：

```
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.13</version>
    </dependency>
```

修改 MoocAspect：

```
public class MoocAspect {

	public void before(){
	    System.out.println("MoocAspect before.");
    }

    public void afterReturning(){
		System.out.println("MoocAspect afterReturning.");
	}

	public void afterThrowing(){
		System.out.println("MoocAspect afterThrowing.");
	}

	public void after(){
		System.out.println("MoocAspect after.");
	}

	// 环绕通知方法的第一个参数必须是 ProceedingJoinPoint 类型
	public Object around(ProceedingJoinPoint pjp){
		Object obj = null;
		try{
			System.out.println("MoocAspect around 1.");
			obj = pjp.proceed();
			System.out.println("MoocAspect around 2.");
		}catch (Throwable e){
			e.printStackTrace();
		}

		return obj;
	}

	public Object aroundInit(ProceedingJoinPoint pjp, String bizName, int times){
		System.out.println(bizName + " " + times);
		Object obj = null;
		try{
			System.out.println("MoocAspect aroundInit 1.");
			obj = pjp.proceed();
			System.out.println("MoocAspect aroundInit 2.");
		}catch (Throwable e){
			e.printStackTrace();
		}

		return obj;
	}
}
```

修改 AspectBiz：

```
public class AspectBiz {
	
	public void biz() {
		System.out.println("AspectBiz biz.");
//		throw new RuntimeException();
	}

	public void init(String bizName, int times){
		System.out.println("AspectBiz init: " + bizName + " " + times);
	}

}
```

修改配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

	<bean id="moocAspect" class="com.karonda.aop.schema.advice.MoocAspect"></bean>

	<bean id="aspectBiz" class="com.karonda.aop.schema.advice.biz.AspectBiz"></bean>

	<aop:config>
		<aop:aspect id="moocAspectAOP" ref="moocAspect">
			<aop:pointcut expression="execution(* com.karonda.aop.schema.advice.biz.*Biz.*(..))" id="moocPiontcut"/>
			<!--前置通知-->
			<aop:before method="before" pointcut-ref="moocPiontcut"/>
			<!--返回后通知-->
			<aop:after-returning method="afterReturning" pointcut-ref="moocPiontcut"/>
			<!--抛出异常通知-->
			<aop:after-throwing method="afterThrowing" pointcut-ref="moocPiontcut" />
			<!--后通知-->
			<aop:after method="after" pointcut-ref="moocPiontcut"/>
			<!--环绕通知-->
			<aop:around method="around" pointcut-ref="moocPiontcut"/>
			<!--带参数-->
			<aop:around method="aroundInit" pointcut="execution(* com.karonda.aop.schema.advice.biz.AspectBiz.init(String, int))
				and args(bizName, times)"/>

		</aop:aspect>
	</aop:config>

 </beans>

```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestAOPSchemaAdvice extends UnitTestBase {
    public TestAOPSchemaAdvice(){
        super("classpath:spring-aop-schema-advice.xml");
    }

    @Test
    public void testBiz(){
        AspectBiz biz = super.getBean("aspectBiz");
        biz.biz();
    }

    @Test
    public void testInit(){
        AspectBiz biz = super.getBean("aspectBiz");
        biz.init("moocService", 3);
    }
}
```


# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
