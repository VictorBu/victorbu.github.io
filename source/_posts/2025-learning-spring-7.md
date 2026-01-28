---
title: 学习 Spring (七) Resource
date: 2019-02-25 16:00:00
updated: 2019-02-25 16:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记


Resource: Spring 针对资源文件的统一接口

+ UrlResource: URL 对应的资源，根据一个 URL 地址即可构建
+ ClassPathResource: 获取类路径下的资源文件
+ FileSystemResource: 获取文件系统里面的资源
+ ServletContextResource: ServletContext 封装的资源，用于访问 ServletContext 环境下的资源
+ InputStreamResource: 针对于输入流封装的资源
+ ByteArrayResource: 针对于字节数组封装的资源

ResourceLoader: 所有的 application context 都实现了 ResourceLoader 接口

```
public interface ResourceLoader{
    Resource getResource(String location);
}
```

ResourceLoader 注入参数前缀：

前缀|例子|解释
-|-|-
classpath:|classpath:com/myapp/config.xml|从 classpath 加载
file:|file:/data/config.xml|从文件系统加载
http:|http://myserver/logo.png|从 URL 加载
无|/data/config.xml|依赖于 ApplicationContext

# 示例

新建类：

```
public class MoocResource implements ApplicationContextAware  {
	
	private ApplicationContext applicationContext;
	
	@Override
	public void setApplicationContext(ApplicationContext applicationContext)
			throws BeansException {
		this.applicationContext = applicationContext;
	}
	
	public void resource() throws IOException {
		//Resource resource = applicationContext.getResource("classpath:config.txt");
		//Resource resource = applicationContext.getResource("file:E:\\project\\java\\demo\\learningspring\\src\\main\\resources\\config.txt");
		//Resource resource = applicationContext.getResource("https://www.cnblogs.com/victorbu/p/10430698.html");
		Resource resource = applicationContext.getResource("config.txt");
		System.out.println(resource.getFilename());
		System.out.println(resource.contentLength());
	}

}
```

添加配置文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >
        
        <bean  id="moocResource" class="com.karonda.resource.MoocResource" ></bean>
	
 </beans>
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestResource extends UnitTestBase {
	
	public TestResource() {
		super("classpath:spring-resource.xml");
	}
	
	@Test
	public void testResource() {
		MoocResource resource = super.getBean("moocResource");
		try {
			resource.resource();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	
}
```


# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
