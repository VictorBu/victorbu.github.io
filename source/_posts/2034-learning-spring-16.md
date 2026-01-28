---
title: 学习 Spring (十六) AOP API
date: 2019-03-10 19:00:00
updated: 2019-03-10 19:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

Spring AOP API 是 Spring 1.2 历史用法，现在仍然支持

这是 Spring AOP 基础，现在的用法也是基于历史的，只是更简便了

# Pointcut

## 实现之一：NameMatchMethodPointcut

根据方法名字进行匹配

成员变量：mappedNames，匹配的方法名集合

```
<bean id="pointcutBean" class="org.springframework.aop.support.NameMatchMethodPointcut"> 
	<property name="mappedNames"> 
		<list> 
			<value>sa*</value> 
		</list> 
	</property> 
</bean>
```

# Before advice

只是在进入方法之前被调用，不需要 MethodInvocation 对象

前置通知可以在连接点执行之前插入自定义行为，但不能改变返回值

```
public class MoocBeforeAdvice implements MethodBeforeAdvice {

	@Override
	public void before(Method method, Object[] args, Object target)
			throws Throwable {
		System.out.println("MoocBeforeAdvice : " + method.getName() + ", " +
				 target.getClass().getName());
	}

}
```

# Throws advice

如果连接点抛出异常，throws advice 在连接点返回后被调用

如果 throws-advice 的方法抛出异常，那么它将覆盖原有异常

接口org.springframework.aop.ThrowsAdvice 不包含任何方法，仅仅是一个声明，实现类需要实现类似下面的方法：void afterThrowing([Method, args, target], ThrowableSubclass)

```
public class MoocThrowsAdvice implements ThrowsAdvice {
	
	public void afterThrowing(Exception ex) throws Throwable {
		System.out.println("MoocThrowsAdvice afterThrowing 1");
	}
	
	public void afterThrowing(Method method, Object[] args, Object target, Exception ex) throws Throwable {
		System.out.println("MoocThrowsAdvice afterThrowing 2 : " + method.getName() + ", " + 
				target.getClass().getName());
	}

}
```

# After Returning advice

可以访问返回值(但不能进行修改)、被调用的方法、方法的参数和目标

如果抛出异常，将会抛出拦截器链，替代返回值

```
public class MoocAfterReturningAdvice implements AfterReturningAdvice {

	@Override
	public void afterReturning(Object returnValue, Method method,
			Object[] args, Object target) throws Throwable {
		System.out.println("MoocAfterReturningAdvice : " + method.getName() + ", " + 
			target.getClass().getName() + ", " + returnValue);
	}

}
```

# Interception around advice

Spring 的切入点模型使得切入点可以独立与 advice 重用，以针对不同的 advice 可以使用相同的切入点

```
public class MoocMethodInterceptor implements MethodInterceptor {

	@Override
	public Object invoke(MethodInvocation invocation) throws Throwable {
		System.out.println("MoocMethodInterceptor 1 : " + invocation.getMethod().getName() + ", " + 
				invocation.getStaticPart().getClass().getName());
		 Object obj = invocation.proceed();
		 System.out.println("MoocMethodInterceptor 2 : " + obj);
		 return obj;
	}

}
```

# Introduction advice

Spring 把引入通知作为一种特殊的拦截通知

仅使用于类，不能和任何切入点一起使用

需要同时实现 IntroductionAdvisor 和 IntroductionInterceptor

```
public interface Lockable {
	
	void lock();

	void unlock();

	boolean locked();

}

// 通常不是直接去实现 IntroductionInterceptor 接口，而是继承 DelegatingIntroductionInterceptor
public class LockMixin extends DelegatingIntroductionInterceptor implements Lockable {

	private boolean locked;

    public void lock() {
        this.locked = true;
    }

    public void unlock() {
        this.locked = false;
    }

    public boolean locked() {
        return this.locked;
    }

    public Object invoke(MethodInvocation invocation) throws Throwable {
        if (locked() && invocation.getMethod().getName().indexOf("set") == 0) {
            throw new RuntimeException();
        }
        return super.invoke(invocation);
    }

}

// 持有独立的 LockMixin 实例
public class LockMixinAdvisor extends DefaultIntroductionAdvisor {

	public LockMixinAdvisor() {
        super(new LockMixin(), Lockable.class);
    }
}
```

# Advisor

Advisor 是仅包含一个切入点表达式关联的单个通知的方面

除了 introductions, advisor 可以用于任何通知

org.springframework.aop.support.DefaultPointcutAdvisor 是最常用的 advisor 类，它可以与 MethodInterceptor, BeforeAdvice 或者 ThrowsAdvice 一起使用

它可以混合在 Spring 同一个 AOP 代理的 advisor 和 advice

# ProxyFactoryBean

创建 Spring AOP 代理的基本方法是使用 ProxyFactoryBean，这样可以完全控制切入点和通知以及它们的顺序

使用 ProxyFactoryBean 或者其他 IoC 相关类来创建 AOP 代理最重要的好处是通知和切入点也可以由 IoC 来管理

如果被代理类没有实现任何接口，使用 CGLIB 代理，否则使用 JDK 代理

通过设置 proxyTargetClass 为 true，可强制使用 CGLIB

如果目标类实现了一个(或者多个)接口，那么创建代理的类型将依赖 ProxyFactoryBean 的配置

如果 ProxyFactoryBean 的 proxyInterfaces 属性被设置为一个或者多个全限定接口名，基于 JDK 的代理将被创建

如果 ProxyFactoryBean 的 proxyInterfaces 属性没有被设置，但是目标类实现了一个(或者更多)接口，那么 ProxyFactoryBean 将自动检测到这个目标类已经实现了至少一个接口，创建一个基于 JDK 的代理

```
	<!--对应的是 ProxyFactoryBean，target 指向的才是目标类-->
 	<bean id="bizLogicImpl" class="org.springframework.aop.framework.ProxyFactoryBean">
 		<property name="target">
 			<ref bean="bizLogicImplTarget"/>
 		</property>
 		<property name="interceptorNames">
 			<list>
 				<value>defaultAdvisor</value>
 				<value>moocAfterReturningAdvice</value>
 				<value>moocMethodInterceptor</value>
 				<value>moocThrowsAdvice</value>
 			</list>
 		</property>
 	</bean>
```

# 实例

## 使用 Pointcut

添加：

```
public interface BizLogic {
	
	String save();

}


public class BizLogicImpl implements BizLogic {
	
	public String save() {
		System.out.println("BizLogicImpl : BizLogicImpl save.");
		return "BizLogicImpl save.";
//		throw new RuntimeException();
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
        
     <bean id="moocBeforeAdvice" class="com.karonda.aop.api.MoocBeforeAdvice"></bean>
     
     <bean id="moocAfterReturningAdvice" class="com.karonda.aop.api.MoocAfterReturningAdvice"></bean>
     
     <bean id="moocMethodInterceptor" class="com.karonda.aop.api.MoocMethodInterceptor"></bean>
     
     <bean id="moocThrowsAdvice" class="com.karonda.aop.api.MoocThrowsAdvice"></bean>
     

     <bean id="bizLogicImplTarget" class="com.karonda.aop.api.BizLogicImpl"></bean>

 	<bean id="pointcutBean" class="org.springframework.aop.support.NameMatchMethodPointcut">
 		<property name="mappedNames">
 			<list>
 				<value>sa*</value>
 			</list>
 		</property>
 	</bean>
	
 	<bean id="defaultAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
 		<property name="advice" ref="moocBeforeAdvice" />
 		<property name="pointcut" ref="pointcutBean" />
 	</bean>

	<!--对应的是 ProxyFactoryBean，target 指向的才是目标类-->
 	<bean id="bizLogicImpl" class="org.springframework.aop.framework.ProxyFactoryBean">
 		<property name="target">
 			<ref bean="bizLogicImplTarget"/>
 		</property>
 		<property name="interceptorNames">
 			<list>
 				<value>defaultAdvisor</value>
 				<value>moocAfterReturningAdvice</value>
 				<value>moocMethodInterceptor</value>
 				<value>moocThrowsAdvice</value>
 			</list>
 		</property>
 	</bean>

 </beans>
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestAOPAPI extends UnitTestBase {
	
	public TestAOPAPI() {
		super("classpath:spring-aop-api.xml");
	}
	
	@Test
	public void testSave() {
		BizLogic logic = (BizLogic)super.getBean("bizLogicImpl");
		logic.save();
	}

}
```

## 不使用 Pointcut

修改配置：

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
        
     <bean id="moocBeforeAdvice" class="com.karonda.aop.api.MoocBeforeAdvice"></bean>
     
     <bean id="moocAfterReturningAdvice" class="com.karonda.aop.api.MoocAfterReturningAdvice"></bean>
     
     <bean id="moocMethodInterceptor" class="com.karonda.aop.api.MoocMethodInterceptor"></bean>
     
     <bean id="moocThrowsAdvice" class="com.karonda.aop.api.MoocThrowsAdvice"></bean>
     

     <bean id="bizLogicImplTarget" class="com.karonda.aop.api.BizLogicImpl"></bean>


 	<bean id="bizLogicImpl" class="org.springframework.aop.framework.ProxyFactoryBean">
 		<property name="proxyInterfaces">
 			<value>com.karonda.aop.api.BizLogic</value>
 		</property>
 		<property name="target">
 			<ref bean="bizLogicImplTarget"/>
 		</property>
 		<property name="interceptorNames">
 			<list>
 				<value>moocBeforeAdvice</value>
 				<value>moocAfterReturningAdvice</value>
 				<value>moocMethodInterceptor</value>
 				<value>moocThrowsAdvice</value>
 			</list>
 		</property>
 	</bean>

 </beans>

```

可以使用匿名内部 bean 来隐藏目标和代理之间的区别(推荐做法，可以避免直接使用 getBean 获取原始对象绕过代理而不会执行 advice)，上述配置修改：

1. 移除： 	`<bean id="moocThrowsAdvice" class="com.karonda.aop.api.MoocThrowsAdvice"></bean>`
1. `<property name="target">` 修改为：`<bean class="com.karonda.aop.api.BizLogicImpl"></bean>`

```
 	<bean id="bizLogicImpl" class="org.springframework.aop.framework.ProxyFactoryBean">
 		<property name="proxyInterfaces">
 			<value>com.karonda.aop.api.BizLogic</value>
 		</property>
 		<property name="target">
			<bean class="com.karonda.aop.api.BizLogicImpl"></bean>
 		</property>
 		<property name="interceptorNames">
 			<list>
 				<value>moocBeforeAdvice</value>
 				<value>moocAfterReturningAdvice</value>
 				<value>moocMethodInterceptor</value>
 				<value>moocThrowsAdvice</value>
 			</list>
 		</property>
 	</bean>
```

# Proxying class

CGLIB 代理的工作原理是在运行时生成目标类的子类，Spring 配置这个生成的子类委托方法调用到原来的目标

子类是用 Decorator 模式，织入通知

CGLIB 的代理对用户是透明的，需要注意：
+ final 方法不能被通知，因为它们不能被覆盖
+ 不用把 CGLIB 添加到 classpath 中，在 Spring 3.2 之后 CGLIB 被重新包装并包含在 Spring 核心的JAR (即基于 CGLIB 的 AOP 就像 JDK 动态代理一样“开箱即用”)

# global advisor

用 * 做通配，匹配所有拦截器加入通知链(实现了 MethodInterceptor 接口的)

## 示例：

moocMethodInterceptor 可以修改为 mooc*：
```
	<property name="interceptorNames">
		<list>
			<value>moocBeforeAdvice</value>
			<value>moocAfterReturningAdvice</value>
			<!--<value>moocMethodInterceptor</value>-->
			<value>mooc*</value>
			<value>moocThrowsAdvice</value>
		</list>
	</property>
```

# 简化的 proxy 定义

使用父子 bean 定义以及内部 bean 定义，可能会带来更清洁和更简洁的代理定义(抽象属性标记父 bean 定义为 abstract，这样它不能被实例化)

## 示例：

修改配置文件：

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
        
     <bean id="moocBeforeAdvice" class="com.karonda.aop.api.MoocBeforeAdvice"></bean>
     
     <bean id="moocAfterReturningAdvice" class="com.karonda.aop.api.MoocAfterReturningAdvice"></bean>
     
     <bean id="moocMethodInterceptor" class="com.karonda.aop.api.MoocMethodInterceptor"></bean>
     
     <bean id="moocThrowsAdvice" class="com.karonda.aop.api.MoocThrowsAdvice"></bean>


	<bean id="baseProxyBean" class="org.springframework.aop.framework.ProxyFactoryBean"
		  lazy-init="true" abstract="true"></bean>

	<bean id="bizLogicImpl"  parent="baseProxyBean">
		<property name="target">
			<bean class="com.karonda.aop.api.BizLogicImpl"></bean>
		</property>
		<property name="proxyInterfaces">
			<value>com.karonda.aop.api.BizLogic</value>
		</property>
		<property name="interceptorNames">
			<list>
				<value>moocBeforeAdvice</value>
				<value>moocAfterReturningAdvice</value>
				<value>moocMethodInterceptor</value>
				<value>moocThrowsAdvice</value>
			</list>
		</property>
	</bean>

 </beans>

```

# 使用 ProxyFactory

使用 Spring AOP 而不必依赖于 Spring IoC：

```
ProxyFactory factory = new ProxyFactory(myBusinessInterfaceImpl);
factory.addAdvice(myMethodInterceptor);
factory.addAdvisor(myAdvisor);
MyBusinessInterface tb = (MyBusinessInterface)factory.getProxy();
```

大多数情况下最佳实践是使用 IoC 容器创建 AOP 代理

虽然可以硬编码方式实现，但是 Spring 推荐使用配置或注解方式实现

# 使用 "auto-proxy"

Spring 也允许使用”自动代理“的 bean 定义，它可以自动代理选定的 bean，这是建立在 Spring 的 "bean post processor" 功能基础上的(在加载 bean 的时候就可以修改)

```
<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <property name="beanNames" value="jdk*,onlyJdk"/>
	<property name="interceptorNames">
	    <list>
		    <value>myInterceptor</value>
		</list>
	</property>
</bean>
```

使用 DefaultAdvisorAutoPorxyCreator，当前 IoC 容器中自动应用，不用显示声明引用 advisor 的 bean 定义

```
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoPorxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="customAdvisor" class="com.mycompany.MyAdvisor"/>
<bean id="bussinessObject1" class="com.mycompany.BusinessObject1">
    <!-- 可以省略 Properties 定义-->
</bean>
```



# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
