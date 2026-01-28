---
title: Spring Boot 2.x 编写 RESTful API (四) 使用 Mybatis
date: 2019-04-06 20:00:00
updated: 2019-04-06 20:00:00
categories: [IT]
tags: [Java, Spring Boot, RESTful API, Mybatis]
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
  mapper-locations: classpath:com.karonda.restapi.dao/*.xml # 配置 mapper xml 文件所在路径
```

# 启动类添加

```
@MapperScan("com.karonda.restapi.dao")
```

# 新建 dao

```
public interface TvSeriesDao {

    @Select("select * from tv_series where id=#{id}")
    TvSeries getOneById(int id);

    @Select("select * from tv_series")
    List<TvSeries> getAll();

    int update(TvSeries tvSeries);
    int insert(TvSeries tvSeries);

    @Delete("delete from tv_series where id=#{id}")
    int delete(int id);

    @Update("update tv_series set status=-1, reason=#{reason} where id=#{id}")
    int logicDelete(int id, String reason);
}

```

# 新增 mapper TvCharacterDao.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.karonda.restapi.dao.TvSeriesDao" >
  
  <insert id="insert" parameterType="com.karonda.restapi.pojo.TvSeries"
  			useGeneratedKeys="true" keyProperty="id">
    insert into tv_series (name, season_count, origin_release )
    		values (#{name}, #{seasonCount}, #{originRelease} )
  </insert>
  
  <update id="update" parameterType="com.karonda.restapi.pojo.TvSeries">
    update tv_series set name=#{name}, season_count=#{seasonCount}, origin_release=#{originRelease} where id=#{id}
  </update>
  
</mapper>
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

# 修改 Controller

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