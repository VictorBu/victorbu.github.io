---
title: Spring Boot 2.x 多数据源配置之 JPA 篇
date: 2019-05-30 11:20:00
updated: 2019-05-30 11:20:00
categories: [IT]
tags: [Spring Boot, JPA]
---

场景假设：现有电商业务，商品和库存分别放在不同的库

# 配置数据库连接

```
app:
  datasource:

    first:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://127.0.0.1/product?useSSL=false
      username: root
      password: root
      configuration:
        maximum-pool-size: 10

    second:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://127.0.0.1/stock?useSSL=false
      username: root
      password: root
      configuration:
        maximum-pool-size: 10
```

# 添加配置类

## FirstConfig

```
@Configuration
@EnableJpaRepositories(
        basePackages = {"com.karonda.springboot2datasourcesjpa.dao.first"},// 1. dao 层所在的包
        entityManagerFactoryRef = "firstEntityManagerFactory",
        transactionManagerRef = "firstTransactionManager")
@EnableTransactionManagement
public class FirstConfig {

    @Bean
    @Primary
    public PlatformTransactionManager firstTransactionManager() {
        return new JpaTransactionManager(firstEntityManagerFactory().getObject());
    }

    @Bean
    @Primary
    public LocalContainerEntityManagerFactoryBean firstEntityManagerFactory() {

        HibernateJpaVendorAdapter jpaVendorAdapter = new HibernateJpaVendorAdapter();

        LocalContainerEntityManagerFactoryBean factoryBean = new LocalContainerEntityManagerFactoryBean();
        factoryBean.setDataSource(firstDataSource());
        factoryBean.setJpaVendorAdapter(jpaVendorAdapter);
        factoryBean.setJpaProperties(hibernateProperties());

        factoryBean.setPackagesToScan("com.karonda.springboot2datasourcesjpa.entity.first");// 2. 实体类所在的包

        return factoryBean;
    }

    @Bean
    @Primary
    @ConfigurationProperties("app.datasource.first")
    public DataSourceProperties firstDataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean
    @Primary
    @ConfigurationProperties("app.datasource.first.configuration")
    public DataSource firstDataSource() {
        return firstDataSourceProperties()
                .initializeDataSourceBuilder()
                .type(HikariDataSource.class) // 3. 可以显示指定连接池，也可以不显示指定；即此行代码可以注释掉
                .build();
    }

    private Properties hibernateProperties() {
        final Properties hibernateProperties = new Properties();
        hibernateProperties.setProperty("hibernate.dialect", "org.hibernate.dialect.MySQL5Dialect");
//        hibernateProperties.setProperty("hibernate.hbm2ddl.auto", "create");
        return hibernateProperties;
    }

}
```

## SecondConfig

```
@Configuration
@EnableJpaRepositories(
        basePackages = {"com.karonda.springboot2datasourcesjpa.dao.second"},
        entityManagerFactoryRef = "secondEntityManagerFactory",
        transactionManagerRef = "secondTransactionManager")
@EnableTransactionManagement
public class SecondConfig {

    @Bean
    public PlatformTransactionManager secondTransactionManager() {
        return new JpaTransactionManager(secondEntityManagerFactory().getObject());
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean secondEntityManagerFactory() {

        HibernateJpaVendorAdapter jpaVendorAdapter = new HibernateJpaVendorAdapter();

        LocalContainerEntityManagerFactoryBean factoryBean = new LocalContainerEntityManagerFactoryBean();
        factoryBean.setDataSource(secondDataSource());
        factoryBean.setJpaVendorAdapter(jpaVendorAdapter);
        factoryBean.setJpaProperties(hibernateProperties());

        factoryBean.setPackagesToScan("com.karonda.springboot2datasourcesjpa.entity.second");

        return factoryBean;
    }

    @Bean
    @ConfigurationProperties("app.datasource.second")
    public DataSourceProperties secondDataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean
    @ConfigurationProperties("app.datasource.second.configuration")
    public DataSource secondDataSource() {
        return secondDataSourceProperties()
                .initializeDataSourceBuilder()
                .type(HikariDataSource.class)
                .build();
    }

    private Properties hibernateProperties() {
        final Properties hibernateProperties = new Properties();
        hibernateProperties.setProperty("hibernate.dialect", "org.hibernate.dialect.MySQL5Dialect");
//        hibernateProperties.setProperty("hibernate.hbm2ddl.auto", "create");
        return hibernateProperties;
    }

}
```

# dao

```
public interface ProductDao extends JpaRepository<Product, Integer> {

    @Query(value = "select p from Product p where p.id = id")
    Product findById(@Param("id") int id);
}
```

```
public interface StockDao extends JpaRepository<Stock, Integer> {

    @Query(value = "select s from Stock s where s.productId = productId")
    Stock findByProductId(@Param("productId") int productId);
}
```

# 使用示例

```
@Component
public class Task {

    @Autowired
    private ProductDao productDao;
    @Autowired
    private StockDao stockDao;

    @Scheduled(cron = "0/5 * * * * ? ")
    public void job(){

        final int productId = 1;
        Product product = productDao.findById(productId);
        Stock stock = stockDao.findByProductId(productId);

        System.out.println("产品名称: " + product.getName() + ", 库存: " + stock.getStockCount());
    }

}
```

# 注意事项

1. 使用多数据源，其中一个配置类需要添加 @Primary 注解 (有且仅有一个配置类需要添加)
1. 在配置类中需要同时配置 dao 层和实体类所在的包

参考：

+ [Configure Two DataSources](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#howto-two-datasources)
+ [spring+Jpa多数据源配置的方法示例](https://www.jb51.net/article/145486.htm)
+ [How to Configure Multiple Datasources with Spring Boot](http://www.javaoptimum.com/how-to-configure-multiple-datasources-with-spring-boot/)

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-2-datasources-jpa)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

