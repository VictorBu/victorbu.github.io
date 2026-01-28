---
title: 学习 Spring (十四) Introduction
date: 2019-03-10 07:00:00
updated: 2019-03-10 07:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

Introduction 允许一个切面声明一个实现指定接口的通知对象，并且提供了一个接口实现类来代表这些对象

由 <aop:aspect> 中的 <aop:declare-parents> 元素声明该元素用于声明所匹配的类型拥有一个新的 parents

# 示例

新增接口：

```
public interface Fit {
	
	void filter();

}
```

添加实现：

```
public class FitImpl implements Fit {

	@Override
	public void filter() {
		System.out.println("FitImpl filter.");
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
			<!--Introduction-->
			<aop:declare-parents types-matching="com.karonda.aop.schema.advice.biz.*(+)"
								 implement-interface="com.karonda.aop.schema.advice.Fit"
								 default-impl="com.karonda.aop.schema.advice.FitImpl"/>

		</aop:aspect>
	</aop:config>

 </beans>
```

添加测试：

```
    @Test
    public void testFit(){
        Fit fit = (Fit)super.getBean("aspectBiz");
        fit.filter();
    }
```

schema-defined (基于配置文件) aspects 只支持 singleton model


# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
