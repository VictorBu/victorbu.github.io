---
title: 学习 Spring (一) Spring 介绍
date: 2019-02-21 21:00:00
updated: 2019-02-21 21:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

# Spring 是什么

Spring 是一个轻量级的 IoC (控制反转)和 AOP (面向切面)的容器框架

![](https://oss.x8y.cc/blog-img/2019/spring-framework-runtime.png)

# 框架与类库的区别

1. 框架一般是封装了逻辑、高内聚的，类库则是松散的工具组合
1. 框架专注于某一领域，类库则是更通用的

# IoC 与 DI 的关系

+ IoC: 控制权的转移，应用程序本身不负责依赖对象的创建和维护，而是由外部容器负责创建和维护(获得依赖对象的过程被反转了)
+ DI: 由 IoC 容器在运行期间，动态地将某种依赖关系注入到对象之中

IoC 是一种设计思想，DI 是这种思想的一种实现

# 示例

## 添加接口

```
public interface OneInterface {
    String hello(String word);
}
```

## 实现接口

```
public class OneInterfaceImpl implements OneInterface {

    public String hello(String word){
        return "Word from interface \"OneInterface\": " + word;
    }
}
```

## 添加配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >
        
	<bean id="oneInterface" class="com.karonda.ioc.interfaces.OneInterfaceImpl"></bean>
	
 </beans>
```

## 添加测试

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestOneInterface extends UnitTestBase {

    public TestOneInterface(){
        super("classpath*:spring-ioc.xml");
    }

    @Test
    public void testHello(){
        OneInterface oif = super.getBean("oneInterface");
        System.out.println(oif.hello("world"));
    }
}
```

其中 UnitTestBase 代码：

```
public class UnitTestBase {
	
	private ClassPathXmlApplicationContext context;
	
	private String springXmlpath;
	
	public UnitTestBase() {}
	
	public UnitTestBase(String springXmlpath) {
		this.springXmlpath = springXmlpath;
	}
	
	@Before
	public void before() {
		if (StringUtils.isEmpty(springXmlpath)) {
			springXmlpath = "classpath*:spring-*.xml";
		}
		try {
			context = new ClassPathXmlApplicationContext(springXmlpath.split("[,\\s]+"));
			context.start();
		} catch (BeansException e) {
			e.printStackTrace();
		}
	}
	
	@After
	public void after() {
		context.destroy();
	}
	
	@SuppressWarnings("unchecked")
	protected <T extends Object> T getBean(String beanId) {
		try {
			return (T)context.getBean(beanId);
		} catch (BeansException e) {
			e.printStackTrace();
			return null;
		}
	}
	
	protected <T extends Object> T getBean(Class<T> clazz) {
		try {
			return context.getBean(clazz);
		} catch (BeansException e) {
			e.printStackTrace();
			return null;
		}
	}

}
```

# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)


# Bean 容器初始化

## 两基础个包
1. org.springframework.beans：BeanFactory 提供配置结构和基本功能，加载并初始化 Bean
1. org.springframework.context：ApplicationContext 保存了 Bean 对象并在 Spring 中被广泛使用

## 初始化 ApplicationContext 方式：

1. 本地文件
1. Classpath
1. Web 应用中依赖 servlet 或 Listener

# 备忘：在 IDEA 中构建 Maven Spring 项目

File --> New --> Project

![](https://oss.x8y.cc/blog-img/2019/idea-maven-spring-1.png)

![](https://oss.x8y.cc/blog-img/2019/idea-maven-spring-2.png)

![](https://oss.x8y.cc/blog-img/2019/idea-maven-spring-3.png)

![](https://oss.x8y.cc/blog-img/2019/idea-maven-spring-4.png)

![](https://oss.x8y.cc/blog-img/2019/idea-maven-spring-5.png)

![](https://oss.x8y.cc/blog-img/2019/idea-maven-spring-6.png)

![](https://oss.x8y.cc/blog-img/2019/idea-maven-spring-7.png)

![](https://oss.x8y.cc/blog-img/2019/idea-maven-spring-8.png)

在 pom.xml 中添加依赖包：

```
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>4.3.18.RELEASE</version>
</dependency>

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-test</artifactId>
  <version>4.3.18.RELEASE</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>
```


