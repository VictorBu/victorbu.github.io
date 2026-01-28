---
title: 学习 Spring (八) 注解之 Bean 的定义及作用域
date: 2019-02-26 07:00:00
updated: 2019-02-26 07:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

# Classpath 扫描与组件管理

从 Spring 3.0 开始，Spring JavaConfig 项目提供了很多特性，包括使用 java 而不是 XML 定义 bean，比如 @Configuration, @Bean, @Import, @DependsOn

@Component 是一个通用注解，可用于任何 bean；@Repository, @Service, @Controller 是更具有针对性的注解：

+ @Repository 通常用于注解 DAO 类，即持久层
+ @Service 通常用于注解 Service 类，即服务层
+ @Controller 通常用于 Controller 类，即控制层 (MVC)

# 元注解 (Meta-annotations)

许多 Spring 提供的注解可以作为自己的代码，即“元数据注解“，元注解是一个简单的注解，可以应用到另一个注解(即注解的注解)

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // 定义 @Service 注解时继承了 @Component 的所有特性 
public @interface Service{
    // ...
}
```

除了 value(), 元注解还可以有其他的属性，允许定制：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope("session")
public @interface SessionScope{
    ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT
}
```


# 类的自动检测与 Bean 的注册

Spring 可以自动检测类并注册 Bean 到 ApplicationContext 中

为了能够检测这些类并注册相应的 Bean，需要下面内容：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd 
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context.xsd">
        
        <context:component-scan base-package="org.example"/>
	
 </beans>
```

<context:component-scan> 包含 <context:annotation-config>, 通常在使用前者后，不再使用后者

# 使用过滤器进行自定义扫描

默认情况下，类被自动发现并注册 bean 的条件是：使用 @Component, @Repository, @Service, @Controller 注解或者使用 @Component 的自定义注解

可以通过过滤器修改上面的行为，如：下面例子的 XML 配置忽略所有的 @Repository 注解并用 Stub 代替：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
        
        <context:component-scan base-package="org.example">
            <context:include-filter type="regex" expression=".*Stub.*Repository"/>
            <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
        </context:component-scan>
	
 </beans>
```

还可使用 use-default-filters= "false" 禁用自动发现与注册

# 定义 Bean

扫描过程中组件被自动检测，Bean 名称是由 BeanNameGenerator 生成的(@Component, @Repository, @Service, @Controller 都会有个 name 属性用于显示设置 Bean Name)：

```
@Service("myMovieLister")
public class SimpleMovieLister{
		// ...
}

@Repository // 没有设置 name 将使用默认 name(类名首字母小写)
public class MovieFinderImpl implements MovieFinder{
    // ...
}
```

可以自定义 bean 命名策略：实现 BeanNameGenerator 接口，并一定要包含一个无参数构造函数

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
        
        <context:component-scan base-package="org.example"
                                name-generator="org.example.MyNameGenerrator"/>
	
 </beans>
```

# 作用域

通常情况下自动查找的 Spring 组建，其 scope 是 singleton，Spring 2.5 提供了一个 标识 scope 的注解 @Scope

```
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder{
    // ...
} 
```

也可以自定义 scope 策略，实现 ScopeMetadataResolver 接口并提供一个无参构造函数

```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
        
        <context:component-scan base-package="org.example"
                                scope-resolver="org.example.MyScopeResolver"/>
	
 </beans>
```

# 代理方式 

可以使用 scoped-proxy 属性指定代理，有三个值可选：no, interfaces, targetClass

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
        
        <context:component-scan base-package="org.example"
                                scoped-proxy="interfaces"/>
	
 </beans>
```

# 示例

新建类：

```
@Component
// @Component("bean")
@Scope
// @Scope("prototype")
public class BeanAnnotation {
	
	public void say(String arg) {
		System.out.println("BeanAnnotation : " + arg);
	}
	
	public void myHashCode() {
		System.out.println("BeanAnnotation : " + this.hashCode());
	}
	
}
```

添加配置文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd" >
        
        <context:component-scan base-package="com.karonda.beanannotation"></context:component-scan>
        
 </beans>
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestBeanAnnotation extends UnitTestBase {
	
	public TestBeanAnnotation() {
		super("classpath*:spring-beanannotation.xml");
	}
	
	@Test
	public void testSay() {
		// BeanAnnotation bean = super.getBean("bean");
		BeanAnnotation bean = super.getBean("beanAnnotation");
		bean.say("This is test.");
	}
	
	@Test
	public void testScpoe() {
		BeanAnnotation bean = super.getBean("beanAnnotation");
		bean.myHashCode();
		
		bean = super.getBean("beanAnnotation");
		bean.myHashCode();
	}
	
}
```


# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
