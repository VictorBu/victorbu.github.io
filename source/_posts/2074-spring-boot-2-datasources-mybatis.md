---
title: Spring Boot 2.x 多数据源配置之 MyBatis 篇
date: 2019-06-06 11:20:00
updated: 2019-06-06 11:20:00
categories: [IT]
tags: [Spring Boot, MyBatis]
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
@MapperScan(
        basePackages  = {"com.karonda.springboot2datasourcesmybatis.dao.first"},// 1. dao 层所在的包
        sqlSessionTemplateRef = "firstSqlSessionTemplate")
public class FirstConfig {

    @Bean
    @Primary
    public SqlSessionTemplate firstSqlSessionTemplate() throws Exception {
        return new SqlSessionTemplate(firstSqlSessionFactory());
    }

    @Bean
    @Primary
    public DataSourceTransactionManager firstTransactionManager(){
        return new DataSourceTransactionManager(firstDataSource());
    }

    @Bean
    @Primary
    public SqlSessionFactory firstSqlSessionFactory() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(firstDataSource());
        factoryBean.setMapperLocations(
                new PathMatchingResourcePatternResolver()
                        .getResources("classpath:mapper/first/*.xml")); // 2. xml 所在路径
        return factoryBean.getObject();
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

}
```

## SecondConfig

```
@Configuration
@MapperScan(
        basePackages = {"com.karonda.springboot2datasourcesmybatis.dao.second"},
        sqlSessionTemplateRef = "secondSqlSessionTemplate")
public class SecondConfig {

    @Bean
    public SqlSessionTemplate secondSqlSessionTemplate() throws Exception {
        return new SqlSessionTemplate(secondSqlSessionFactory());
    }

    @Bean
    public DataSourceTransactionManager secondTransactionManager(){
        return new DataSourceTransactionManager(secondDataSource());
    }

    @Bean
    public SqlSessionFactory secondSqlSessionFactory() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(secondDataSource());
        factoryBean.setMapperLocations(
                new PathMatchingResourcePatternResolver()
                        .getResources("classpath:mapper/second/*.xml"));
        return factoryBean.getObject();
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

}
```

# dao

```
public interface ProductMapper {

    Product getOneById(int id);
}
```

```
public interface StockMapper {

    Stock getOneByProductId(int productId);
}
```

# xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.karonda.springboot2datasourcesmybatis.dao.first.ProductMapper">
  <resultMap id="BaseResultMap" type="com.karonda.springboot2datasourcesmybatis.entity.Product">
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="name" jdbcType="VARCHAR" property="name" />
  </resultMap>
  <sql id="Base_Column_List">
    id, name
  </sql>
  <select id="getOneById" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from product
    where id = #{id,jdbcType=INTEGER}
  </select>
</mapper>
```

```
<mapper namespace="com.karonda.springboot2datasourcesmybatis.dao.second.StockMapper">
  <resultMap id="BaseResultMap" type="com.karonda.springboot2datasourcesmybatis.entity.Stock">
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="product_id" jdbcType="INTEGER" property="productId" />
    <result column="stock_count" jdbcType="INTEGER" property="stockCount" />
  </resultMap>
  <sql id="Base_Column_List">
    id, product_id, stock_count
  </sql>
  <select id="getOneByProductId" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from stock
    where product_id = #{productId,jdbcType=INTEGER}
  </select>
</mapper>
```

# 使用示例

```
@Component
public class Task {

    @Autowired
    private ProductMapper productMapper;
    @Autowired
    private StockMapper stockMapper;

    @Scheduled(cron = "0/5 * * * * ? ")
    public void job(){

        final int productId = 1;
        Product product = productMapper.getOneById(productId);
        Stock stock = stockMapper.getOneByProductId(productId);

        System.out.println("产品名称: " + product.getName() + ", 库存: " + stock.getStockCount());
    }

}
```

# 注意事项

1. 使用多数据源，其中一个配置类需要添加 @Primary 注解 (有且仅有一个配置类需要添加)
1. 在配置类中需要同时配置 dao 层所在的包和 xml 所在的路径

# 总结

与 JPA 使用多数据源配置基本相同，具体可对比上一篇文章 [Spring Boot 2.x 多数据源配置之 JPA 篇](/2019/05/2073-spring-boot-2-datasources-jpa/)

参考：

[Spring Boot(七)：Mybatis 多数据源最简解决方案](https://www.cnblogs.com/ityouknow/p/6102399.html)

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-2-datasources-mybatis)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

