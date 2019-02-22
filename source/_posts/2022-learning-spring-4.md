---
title: 学习 Spring (四) Bean 的生命周期
date: 2019-02-22 16:00:00
updated: 2019-02-22 16:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

定义 --> 初始化 --> 使用 --> 销毁

# 初始化

1. 实现 org.springframework.beans.factory.InitializingBean 接口，覆盖 afterPropertiesSet 方法

	```
	public class ExampleBean implements InitializingBean{
		public void afterPropertiesSet() throws Exception{
		// do some initialization work
		}
		
	}
	```

1. 配置 init-method:

	```
	<bean id="exampleInitBean" class="example.ExampleBean" init-method="start"/>
	```

	```
	public class ExampleBean{
		public void start(){
		// do some initialization work
		}
		
	}
	```

# 销毁

1. 实现 org.springframework.beans.factory.DisposableBean 接口，覆盖 destory 方法

	```
	public class ExampleBean implements DisposableBean{
		public void destory() throws Exception{
		// do some destruction work
		}
		
	}
	```

1. 配置 destory-method

	```
	<bean id="exampleInitBean" class="example.ExampleBean" destory-method="stop"/>
	```

	```
	public class ExampleBean{
		public void stop(){
		// do some destruction work
		}
		
	}
	```

# 配置全局默认初始化、销毁方法

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
		default-init-method="init" default-destory-method="destory">
	
 </beans>
```

# 多种方式同时使用执行顺序

1. afterPropertiesSet()
1. init-method / default-init-method
1. destory()
1. destory-method / default-destory-method

+ 如果同时配置了 init-method 和 default-init-method 或 destory-method 和 default-destory-method，default-init-method 或 default-destory-method 不执行
+ default-init-method 和 default-destory-method 可以不在 Bean 中声明


# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
