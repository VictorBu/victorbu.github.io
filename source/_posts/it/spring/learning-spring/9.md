---
title: 学习 Spring (九) 注解之 @Required, @Autowired, @Qualifier
date: 2019-02-28 07:00:00
updated: 2019-02-28 07:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

# @Required

@Required 注解适用于 bean 属性的 setter 方法

这个注解仅仅表示，受影响的 bean 属性必须在配置时被填充，通过在 bean 定义或通过自动装配一个明确的属性值：

```
public class SimpleMovieLister{
    private MovieFinder movieFinder;
	
	@Required
	public void SetMovieFinder(MovieFinder movieFinder){
	    this.movieFinder = movieFinder;
	}
}
```

@Required 使用比较少，一般使用 @Autowired

# @Autowired: setter, 构造器, 成员变量自动注入

可以将 @Autowired 理解为“传统”的 setter 方法：

```
public class SimpleMovieLister{
    private MovieFinder movieFinder;
	
	@Autowired
	public void SetMovieFinder(MovieFinder movieFinder){
	    this.movieFinder = movieFinder;
	}
}
```

可用于构造器或成员变量：

```
@Autowired
private MovieCatalog movieCatalog;


private CustomerPreferenceDao customerPreferenceDao;

@Autowired
public MovieRecommender(CustomerPreferenceDao customerPreferenceDao){
    this.customerPreferenceDao = customerPreferenceDao;
}

```

默认情况下，如果因找不到合适的 bean 将会导致 autowiring 失败抛出异常，可以通过下面的方式避免：

```
public class SimpleMovieLister{
    private MovieFinder movieFinder;
	
	@Autowired(required=false)
	public void SetMovieFinder(MovieFinder movieFinder){
	    this.movieFinder = movieFinder;
	}
}

```

+ 每个类只能有一个构造器被标记为 required=true
+ @Autowired 的必要属性，建议使用 @Required 注解

## 示例

添加类：

```
public interface InjectionDAO {
	
	void save(String arg);

}

@Repository
public class InjectionDAOImpl implements InjectionDAO {
	
	public void save(String arg) {
		//模拟数据库保存操作
		System.out.println("保存数据：" + arg);
	}

}

public interface InjectionService {
	
	void save(String arg);
	
}

@Service
public class InjectionServiceImpl implements InjectionService {

	// @Autowired
	private InjectionDAO injectionDAO;

	// @Autowired
	public void setInjectionDAO(InjectionDAO injectionDAO) {
		this.injectionDAO = injectionDAO;
	}

	@Autowired
	public InjectionServiceImpl(InjectionDAO injectionDAO) {
		this.injectionDAO = injectionDAO;
	}


	public void save(String arg) {
		//模拟业务操作
		System.out.println("Service(Property)接收参数：" + arg);
		arg = arg + ":" + this.hashCode();
		injectionDAO.save(arg);
	}
	
}
```

添加测试类：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestInjection extends UnitTestBase {

    public TestInjection(){
        super("classpath:spring-beanannotation.xml");
    }

    @Test
    public void testAutowired(){
        InjectionService service = super.getBean("injectionServiceImpl");

        service.save("This is autowired.");
    }

}
```

# @Autowired: 数组, Map 自动注入

可以使用 @Autowired 注解那些众所周知的解析依赖性接口，比如：BeanFactory, ApplicationContext, Environment, ResourceLoader, ApplicationEventPublisher, MessageSource

```
public class MovieRecommender{
    
	@Autowired
	private ApplicationContext context;
	
	public MovieRecommender(){
	}
}

```

可以通过添加注解给需要该类型的数组的字段或方法，以提供 ApplicationContext 中的所有特定类型的 bean

```
private Set<MovieCatalog> movieCatalog;

@Autowired
public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs){
    this.movieCatalogs = movieCatalogs;
}
```

可以用于装配 key 为 String 的 Map

```
private Map<String, MovieCatalog> movieCatalogs;

@Autowired
public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs){
    this.movieCatalogs = movieCatalogs;
}
```

如果希望数组有序，可以让 bean 实现 org.springframework.core.Ordered 接口或使用 @Order注解

@Autowired 是由 BeanPostProcessor 处理的，所以不能在自己的 BeanPostProcessor 或 BeanFactoryPostProcessor 类型应用这些注解，这些类型必须通过 XML 或者 @Bean 注解加载

# 示例

新建类：

```
public interface BeanInterface {

}

@Order(value = 2)
@Component
public class BeanImplOne implements BeanInterface {

}

@Order(value = 1)
@Component
public class BeanImplTwo implements BeanInterface {

}

@Component
public class BeanInvoker {
	
	@Autowired
	private List<BeanInterface> list;
	
	@Autowired
	private Map<String, BeanInterface> map;
	
	public void say() {
		if (null != list && 0 != list.size()) {
			System.out.println("list...");
			for (BeanInterface bean : list) {
				System.out.println(bean.getClass().getName());
			}
		} else {
			System.out.println("List<BeanInterface> list is null !!!!!!!!!!");
		}
		
		System.out.println();
		
		if (null != map && 0 != map.size()) {
			System.out.println("map...");
			for (Map.Entry<String, BeanInterface> entry : map.entrySet()) {
				System.out.println(entry.getKey() + "      " + entry.getValue().getClass().getName());
			}
		} else {
			System.out.println("Map<String, BeanInterface> map is null !!!!!!!!!!");
		}

	}

}
```

在 TestInjection 中添加测试方法：

```
@Test
public void testMultiBean(){
	BeanInvoker invoker = super.getBean("beanInvoker");
	invoker.say();
}
```

# @Qualifier

按类型自动装配可能多个 bean 实例的情况，可以使用 @Qualifier 注解缩小范围(或指定唯一)，也可以用于指定单独的构造器参数或方法参数

可用于注解集合类型变量

```
public class MovieRecommender{

    @Autowired
	@Qualifier("main")
	private MovieCatalog movieCatalog;
}


public class MovieRecommender{
    private MovieCatalog movieCatalog;
	private CustomerPreferenceDao customerPreferenceDao;
	
	@Autowired
	public void prepare(@Qualifier("main")MovieCatalog movieCatalog
	    , CustomerPreferenceDao customerPreferenceDao){
		this.movieCatalog = movieCatalog;
		this.customerPreferenceDao = customerPreferenceDao;
	}
}
```

在 XML 文件中实现 Qualifier：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd" >
        
        <context:annotation-config />

        <bean class="example.SimpleMovieCatalog">
            <qualifier value="main"/>
        </bean>

        <bean class="example.SimpleMovieCatalog">
            <qualifier value="action"/>
        </bean>

        <bean id="movieRecommender" class="example.MovieRecommender" />
        
 </beans>
```

如果通过名字进行注解注入，主要使用的不是 @Autowired (即使在技术上能够通过 @Qualifier 指定 bean 的名字)，替代方式是使用 JSR-250 @Resource 注解，它是通过其独特的名称定义来识别特定的目标(这是一个与所声明的类型无关的匹配过程)

因语义差异，集合或 Map 类型的 bean 无法通过 @Autowired 来注入时，因为没有类型匹配到这样的 bean，为这些 bean 使用 @Resource 注解，通过唯一名称引用集合或 Map 的 bean


@Autowired 适用于 fields, constructors, multi-argument methods 这些允许在参数级别使用 @Qualifier 注解缩小范围的情况

@Resource 适用于成员变量、只有一个参数的 setter 方法，所以在目标时构造器或一个多参数方法时，最好的方式时使用 @Qualifier

可以定义自己的 qualifier 注解：

```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre{
    String value();
}

public class MovieRecommender{
    @Autowired
	@Genre("Action")
	private MovieCatalog actionCatalog;
	private MovieCatalog comedyCatalog;
	
	@Autowired
	public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog){
	    this.comedyCatalog = comedyCatalog;
	}

}
```

## 示例

修改 BeanInvoker：

```
@Component
public class BeanInvoker {
	
	@Autowired
	private List<BeanInterface> list;
	
	@Autowired
	private Map<String, BeanInterface> map;

	@Autowired
	@Qualifier("beanImplTwo")
	private BeanInterface beanInterface;
	
	public void say() {
		if (null != list && 0 != list.size()) {
			System.out.println("list...");
			for (BeanInterface bean : list) {
				System.out.println(bean.getClass().getName());
			}
		} else {
			System.out.println("List<BeanInterface> list is null !!!!!!!!!!");
		}
		
		System.out.println();
		
		if (null != map && 0 != map.size()) {
			System.out.println("map...");
			for (Map.Entry<String, BeanInterface> entry : map.entrySet()) {
				System.out.println(entry.getKey() + "      " + entry.getValue().getClass().getName());
			}
		} else {
			System.out.println("Map<String, BeanInterface> map is null !!!!!!!!!!");
		}

		System.out.println();

		if (null != beanInterface) {
			System.out.println(beanInterface.getClass().getName());
		} else {
			System.out.println("beanInterface is null...");
		}

	}

}
```



# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
