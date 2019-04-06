---
title: Spring Boot 2.x 编写 RESTful API (四) 使用 Mybatis
date: 2019-04-06 20:00:00
updated: 2019-04-06 20:00:00
categories: [IT]
tags: [Java, Spring Boot, RESTful API]
---

> [用Spring Boot编写RESTful API](https://study.163.com/course/courseMain.htm?courseId=1005213034) 学习笔记


# 添加依赖

```
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.2</version>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.41</version>
		</dependency>
```

# 添加配置

```
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/tvseries?useSSL=false
    username: root
    password: root
    
    
mybatis:
  configuration:
    map-underscore-to-camel-case: true # 下划线转驼峰命名
```

# 启动类添加

```
@MapperScan("com.karonda.restapi.dao")
```

# 新建 dao

```
public interface TvSeriesDao {

    @Select("select * from tv_series")
    public List<TvSeries> getAll();
}

```

# 新建 service

```
@Service
public class TvSeriesService {
    @Autowired
    TvSeriesDao tvSeriesDao;

    public List<TvSeries> getAllSeries(){
        return tvSeriesDao.getAll();
    }
}
```

修改 Controller

```
    @Autowired
    TvSeriesService tvSeriesService;
    
    @GetMapping
    public List<TvSeries> getAll() {
        if(log.isTraceEnabled()) {
            log.trace("getAll() ");
        }
        List<TvSeries> list = tvSeriesService.getAllSeries();
        
        return list;
    }
```


源码：[spring-boot-2-restful](https://github.com/VictorBu/spring-boot-2-restful)