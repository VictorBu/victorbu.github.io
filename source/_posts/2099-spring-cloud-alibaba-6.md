---
title: Spring Cloud Alibaba 初体验(六) Seata
date: 2020-04-20 15:00:00
updated: 2020-04-20 15:00:00
categories: [IT]
tags: [Spring Cloud, Alibaba, Seata]
---

# 一、下载与运行

本文使用 Seata 1.1.0：https://github.com/seata/seata/releases

Windows 环境下双击 bin/seata-server.bat 启动 Seata Server

# 二、使用

以 Service1 为例

## 2.1 添加引用

```
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

## 2.2 添加配置

### 2.2.1 registry.conf

从 [Spring Cloud 快速集成 Seata](https://github.com/seata/seata-samples/blob/master/doc/quick-integration-with-spring-cloud.md) 复制

放置在 resources 目录，内容详见代码，均采用了默认配置

### 2.2.2 file.conf

从 [Spring Cloud 快速集成 Seata](https://github.com/seata/seata-samples/blob/master/doc/quick-integration-with-spring-cloud.md) 复制

放置在 resources 目录，内容详见代码，均采用了默认配置


### 2.2.3 yml 配置

```
spring:
  cloud:
    alibaba:
      seata:
        tx-service-group: my_test_tx_group
```

**注意**：

file.conf 中 vgroup_mapping 的 key 值为 my_test_tx_group：
 
```
service {
  vgroup_mapping.my_test_tx_group = "default"
}
```
则 yml 中 tx-service-group 的值也必须为 my_test_tx_group；如果 vgroup_mapping 的 key 值改为 abcdefg，则 yml 中 tx-service-group 的值也必须为 abcdefg，即这两个值必须保持一致，否则会报错：

```
no available service 'null' found, please make sure registry config correct
```

### 2.2.4 添加数据源配置

```
@Configuration
public class DataSourceProxyConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource() {
        DruidDataSource druidDataSource = new DruidDataSource();
        return druidDataSource;
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath*:/mapper/*.xml"));
        return factoryBean.getObject();
    }
}
```

**在2.2.0.RELEASE及以后，数据源代理自动实现了，不需要再手动去配置一个代理类**，官方文档还需要配置 DataSourceProxy 应该是文档没有及时更新

### 2.2.5 添加 undo_log 表

在业务相关的数据库中添加 undo_log 表，用于保存需要回滚的数据

```
CREATE TABLE `undo_log`
(
    `id`            BIGINT(20)   NOT NULL AUTO_INCREMENT,
    `branch_id`     BIGINT(20)   NOT NULL,
    `xid`           VARCHAR(100) NOT NULL,
    `context`       VARCHAR(128) NOT NULL,
    `rollback_info` LONGBLOB     NOT NULL,
    `log_status`    INT(11)      NOT NULL,
    `log_created`   DATETIME     NOT NULL,
    `log_modified`  DATETIME     NOT NULL,
    `ext`           VARCHAR(100) DEFAULT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8
```

## 2.3 使用

添加 @GlobalTransactional 注解即可

```
@Service
public class BusinessServiceImpl implements BusinessService {

    @Reference
    private StorageService storageService;
    @Reference
    private OrderService orderService;

    @GlobalTransactional
    @Override
    public void purchase(Order order) throws Exception {
        storageService.deduct(order.getCommodityCode(), order.getCount());
        orderService.create(order);
    }
}
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-cloud-alibaba-parent)

> Seata 1.1.0 及之后版本支持直接在 application.yml 添加配置，可以替换掉 registry.conf 和 file.conf，本人暂未测试

> 参考

1. [Spring Cloud 快速集成 Seata](https://github.com/seata/seata-samples/blob/master/doc/quick-integration-with-spring-cloud.md)
1. [Seata初试采坑](https://blog.csdn.net/Yunwei_Zheng/article/details/104839881)
1. [二、springboot项目使用seata实现分布式事务](https://www.cnblogs.com/lay2017/p/12383066.html)

