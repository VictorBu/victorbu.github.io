---
title: 学习 Spring (十一) 注解之 Spring 对 JSR 支持
date: 2019-03-03 17:00:00
updated: 2019-03-03 17:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

# @Resource

Spring 还支持使用 JSR-250 中的 @Resource 注解的变量或 setter 方法

@Resource 有一个 name 属性，并且默认 Spring 解释该值作为被注入 bean 的名称

```
public class SimpleMovieLister{
    private MovieFinder movieFinder;
	
	@Resource(name="myMovieFinder")
	public void setMovieFinder(MovieFinder movieFinder){
	    this.movieFinder = movieFinder;
	}
}
```

如果没有显式指定 @Resource 的 name，默认名称 是从属性名或 setter 方法得出

注解提供的名字被解析为一个 bean 的名称，这是由 ApplicationContext 中的 CommonAnnotationBeanPostProcessor 发现并处理的

CommonAnnotationBeanPostProcessor 不仅能识别 JSR-250 中的生命周期注解 @Resource，在Spring 2.5 中引入支持初始化回调和销毁回调，前提是 CommonAnnotationBeanPostProcessor 是在 Spring 的 ApplicationContext 中注册的

```
public class CachingMovieLister{
    
	@PostConstruct
	public void popularMovieCache(){
	
	}
	
	@PreDestory
	public void clearMovieCache(){
	
	}

}
```

## 示例

添加类：

```
@Repository
public class JsrDAO {
	
	public void save() {
		System.out.println("JsrDAO invoked.");
	}
	
}

@Service
public class JsrServie {
	
	@Resource
	private JsrDAO jsrDAO;
	
//	@Resource
	public void setJsrDAO(JsrDAO jsrDAO) {
		this.jsrDAO = jsrDAO;
	}

	@PostConstruct
	public void init() {
		System.out.println("JsrServie init.");
	}

	@PreDestroy
	public void destroy() {
		System.out.println("JsrServie destroy.");
	}

	public void save() {
		jsrDAO.save();
	}
	
}
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestJsr extends UnitTestBase {
	
	public TestJsr() {
		super("classpath*:spring-beanannotation.xml");
	}
	
	@Test
	public void testSave() {
		JsrServie service = getBean("jsrServie");
		service.save();
	}
	
}
```

# JSR330 标准注解

从 Spring 3.0 开始支持 JSR330 标准注解(依赖注入注解)，其扫描方式与 Spring 注解一致

使用 JSR330 需要依赖 javax.inject 包

## @Inject

@Inject 等效于 @Autowired，可以使用于类、属性、方法、构造器

## @Named

+ 如果想使用特定名称 进行依赖注入，使用 @Named
+ @Named 与 @Component 是等效的

### 示例

添加 Maven 引用：

```
<dependency>
  <groupId>javax.inject</groupId>
  <artifactId>javax.inject</artifactId>
  <version>1</version>
</dependency>
```

修改 JsrServie 类：

```
//@Service
@Named
public class JsrServie {
	
//	@Resource
//	@Inject
	private JsrDAO jsrDAO;
	
//	@Resource
	@Inject
	public void setJsrDAO(@Named("jsrDAO") JsrDAO jsrDAO) {
		this.jsrDAO = jsrDAO;
	}

	@PostConstruct
	public void init() {
		System.out.println("JsrServie init.");
	}

	@PreDestroy
	public void destroy() {
		System.out.println("JsrServie destroy.");
	}

	public void save() {
		jsrDAO.save();
	}
	
}
```



# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
