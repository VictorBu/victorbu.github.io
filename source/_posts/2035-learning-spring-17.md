---
title: 学习 Spring (十七) Spring 对 AspectJ 的支持 (完结)
date: 2019-03-14 07:00:00
updated: 2019-03-14 07:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

@AspectJ 的风格类似纯 java 注解的普通 java 类

Spring 可以使用 AspectJ 来做切入点解析

AOP 的运行时仍旧是纯的 Spring AOP, 对 AspectJ 的编译器或者织入无依赖性

# Spring 中配置 @AspectJ

对 @AspectJ 支持可以使用 XML 或 Java 风格的配置

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
        
        <context:component-scan base-package="com.karonda.aop.aspectj"/>
     	<aop:aspectj-autoproxy></aop:aspectj-autoproxy>

 </beans>
```

确保 AspectJ 的 aspectjweaver.jar (1.6.8或更高版本) 库包含在应用程序的 classpath 中

# aspect

@AspectJ 切面使用 @Aspect 注解配置，拥有 @Aspect 的任何 bean 将被 Spring 自动识别并应用

用 @Aspect 注解的类可以有方法和字段，他们也可能包括切入点、通知和引入的声明

@Aspect 注解是不能通过类路径自动检测发现的，所以需要配合使用 @Component 注释或者在 XML 配置 bean

一个类中的 @Aspect 注解标识它为一个切面，并且将自己从自动代理中排除

```
@Component
@Aspect
public class MoocAspect {
	
}
```

# pointcut

一个切入点通过一个普通的方法定义来提供，并且切入点表达式使用 @Pointcut 注解，方法返回类型必须为 void

```
@Component
@Aspect
public class MoocAspect {
	
	@Pointcut("execution(* com.karonda.aop.aspectj.biz.*Biz.*(..))")
	public void pointcut() {}
	
	@Pointcut("within(com.karonda.aop.aspectj.biz.*)")
	public void bizPointcut() {}

}
```

## 组合 pointcut

切入点表达式可以通过 &&, || 和 ! 进行组合，也可以通过名字引用切入点表达式

通过组合，可以建立更加复杂的切入点表达式

## 定义良好的 pointcut

AspectJ 是编译期的 AOP

检查代码并匹配连接点与切入点的代价是昂贵的

一个好的切入点应该包括以下几点：

+ 选择特定类型的连接点，如：execution, get, set, call, handler
+ 确定连接点范围，如：within, withincode
+ 匹配上下文信息，如：this, target, @annotation

# advice
添加类：

```
@Service
public class MoocBiz {

	public String save(String arg) {
		System.out.println("MoocBiz save : " + arg);
//		throw new RuntimeException(" Save failed!");
		return " Save success!";
	}

}
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestAspectJ extends UnitTestBase {
	
	public TestAspectJ() {
		super("classpath:spring-aop-aspectj.xml");
	}
	
	@Test
	public void test() {
		MoocBiz biz = getBean("moocBiz");
		biz.save("This is test.");
	}
	
}
```


## Before advice

在 MoocAspect 类添加方法：

```
//	@Before("execution(* com.karonda.aop.aspectj.biz.*Biz.*(..))")
	@Before("pointcut()")
	public void before() {
		System.out.println("Before.");
	}
```

## After returning advice

```
	@AfterReturning(pointcut="bizPointcut()", returning="returnValue")
	public void afterReturning(Object returnValue) {
		System.out.println("AfterReturning : " + returnValue);
	}
```

## After throwing advice

```
	@AfterThrowing(pointcut="pointcut()", throwing="e")
	public void afterThrowing(RuntimeException e) {
		System.out.println("AfterThrowing : " + e.getMessage());
	}
```

## After (finally) advice

最终通知必须准备处理正常和异常两种返回情况，它通常用于释放资源

```
	@After("pointcut()")
	public void after() {
		System.out.println("After.");
	}
```

## Around advice

```
	@Around("pointcut()")
	public Object around(ProceedingJoinPoint pjp) throws Throwable {
		System.out.println("Around 1.");
		Object obj = pjp.proceed();
		System.out.println("Around 2.");
		System.out.println("Around : " + obj);
		return obj;
	}
```

# advice 扩展

## 给 advice 传递参数

### args

```
	@Before("pointcut() && args(arg)")
	public void beforeWithParam(String arg) {
		System.out.println("BeforeWithParam." + arg);
	}
```

### annotation

可以用来判断方法上是否加了某注解或者方法上加的注解对应的值

定义一个注解：

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MoocMethod {
	
	String value();

}
```

添加注解；

```
@Service
public class MoocBiz {

	@MoocMethod("MoocBiz save MoocMethod.")
	public String save(String arg) {
		System.out.println("MoocBiz save : " + arg);
//		throw new RuntimeException(" Save failed!");
		return " Save success!";
	}

}
```

添加通知：

```
@Before("pointcut() && @annotation(moocMethod)")
	public void beforeWithAnnotaion(MoocMethod moocMethod) {
		System.out.println("BeforeWithAnnotation." + moocMethod.value());
	}
```

## advice 的参数及泛型

```
public interface Sample<T>{
    void sampleGenericMethod (T param);
	void sampleGenericCollectionMethod (Collection<T> param);
}


@Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
public void beforeSampleMethod(MyType param){

}

@Before("execution(* ..Sample+.sampleGenericCollectionMethod(*)) && args(param)")
public void beforeSampleMethod(Collection<MyType> param){
}
```

## advice 参数名称

通知和切入点注解有一个额外的 argNames 属性，它可以用来指定所注解的方法的参数名

```
@Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && annotation(auditable)", argNames="bean,auditable")
public void audit(Object bean, Auditable a)
```

如果第一个参数是 JoinPoint, ProceedingJoinPoint, JoinPoint.StaticPart 那么可以忽略它

```
@Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && annotation(auditable)", argNames="bean,auditable")
public void audit(JoinPoint jp, Object bean, Auditable a)
``` 

## Introduction

允许一个切面声明一个通知对象实现指定接口，并且提供了一个接口实现类来代表这些对象

introduction 使用 @DeclareParents 进行注解，这个注解用来定义匹配的类型拥有一个新的 parent

```
@Aspect
public class UsageTracking{
    @DeclareParents(value="com.xyz.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
	public class UsageTracked mixin;
	
	@Before("com.xyz.myapp.SystemArchitecture.businessService() && this(usageTracked)")
	public void recordUsage(UsageTracked usageTracked) {
	    usageTracked.incrementUseCount();
	}
}
```

## 切面实例化模型

perthis 切面通过指定 @Aspect 注解 perthis 子句实现

每个独立的 service 对象执行时都会创建一个切面实例

service 对象的每个方法在第一次执行的时候创建切面实例，切面在 service 对象失效的同时失效

```
@Aspect("perthis(com.xyz.myapp.SystemArchitecture.businessService())")
public class MyAspect{
    private int someState;
	
	@Before(com.xyz.myapp.SystemArchitecture.businessService())
	public void recordServiceUsage(){
	
	}
	
}
```

# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
