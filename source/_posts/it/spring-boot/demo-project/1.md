---
title: Spring Boot 构建电商基础秒杀项目 (一) 项目搭建
date: 2019-03-15 17:00:00
updated: 2019-03-15 17:00:00
categories: [IT]
tags: [Java, Spring Boot, Mybatis]
---

> [SpringBoot构建电商基础秒杀项目](https://www.imooc.com/video/18390) 学习笔记

Spring Boot 其实不是什么新的框架，它默认配置了很多框架的使用方式，就像 maven 整合了所有的 jar 包， Spring Boot 整合了所有的框架，并通过一行简单的 main 方法启动应用


使用 IDEA 新建 maven-archetype-quickstart 项目

# 添加 Spring Boot 依赖

```
<parent>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>2.0.5.RELEASE</version>
</parent>
  
  
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>  
```

修改 App.java

```
@EnableAutoConfiguration
@RestController
public class App 
{
    @RequestMapping("/")
    public String home(){
        return "Hello World!";
    }

    public static void main( String[] args )
    {
        System.out.println( "Hello World!" );

        SpringApplication.run(App.class, args);
    }
}
```

运行，在浏览器输入：localhost:8080

注：8080 是默认端口，如果要使用其他端口，可以在 application.properties 修改，如：server.port=8090

# 接入 Mybatis

## 添加依赖

```
<!--mysql jdbc 配置-->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.41</version>
</dependency>
<!--阿里巴巴德鲁伊连接池-->
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid</artifactId>
  <version>1.1.3</version>
</dependency>
<!--spring boot 对 mybatis 的支持-->
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>1.3.1</version>
</dependency>


<!--mybatis 自动生成工具用来生成对应的数据库文件的映射-->
<plugin>
  <groupId>org.mybatis.generator</groupId>
  <artifactId>mybatis-generator-maven-plugin</artifactId>
  <version>1.3.5</version>
  <dependencies>
	<dependency>
	  <groupId>org.mybatis.generator</groupId>
	  <artifactId>mybatis-generator-core</artifactId>
	  <version>1.3.5</version>
	</dependency>
	<dependency>
	  <groupId>mysql</groupId>
	  <artifactId>mysql-connector-java</artifactId>
	  <version>5.1.41</version>
	</dependency>
  </dependencies>
  <executions>
	<execution>
	  <id>mybaits generator</id>
	  <phase>package</phase>
	  <goals>
		<goal>generate</goal>
	  </goals>
	</execution>
  </executions>
  <configuration>
	<!--允许移动生成的文件-->
	<verbose>true</verbose>
	<!--允许文件自动覆盖，实际开发中千万不要设置为 true-->
	<overwrite>true</overwrite>
	<!--generator 配置文件路径-->
	<configurationFile>
	  src/main/resources/mybatis-generator.xml
	</configurationFile>
  </configuration>
</plugin>
```

## 在 application.properties 添加配置

```
mybatis.mapper-locations=classpath:mapping/*.xml
```

## 新建数据表

数据库名为 seckill

```
create table if not exists user_info(
    id int not null auto_increment,
	name varchar(64) not null default '',
	gender tinyint not null default 0 comment '1: 男, 2: 女',
	age int not null default 0,
	telphone varchar(16) not null default '',
	register_mode varchar(64) not null default '' comment 'byphone, bywechat, byalipay',
	third_party_id varchar(64) not null default '',
	primary key (id)
);

create table if not exists user_password (
	id int not null auto_increment,
	encrpt_password varchar(128) not null default '',
	user_id int not null default 0,
	primary key (id)
);
```

## 添加配置文件 mybatis-generator.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<!--http://www.mybatis.org/generator/configreference/xmlconfig.html-->

<generatorConfiguration>

    <context id="MySQLTables" targetRuntime="MyBatis3">
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/seckill"
                        userId="root"
                        password="root">
        </jdbcConnection>

        <!--生成 DataObject 类存放位置-->
        <javaModelGenerator targetPackage="com.karonda.dataobject" targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!--生成映射文件存放位置-->
        <sqlMapGenerator targetPackage="mapping"  targetProject="src/main/resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!--生成 Dao 类存放位置-->
        <!--客户端代码，生成易于使用的针对 Model 对象和 XML 配置文件的代码-->
            <!--type="ANNOTATEDMAPPER", 生成 Java Model 和基于注解的 Mapper 对象-->
            <!--type="MIXEDMAPPER", 生成基于注解的 Java Model 和相应的 Mapper 对象-->
            <!--type="XMLMAPPER", 生成 SQLMap XML 文件和独立的 Mapper 接口-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.karonda.dao"  targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!--生成对应表及类名-->
        <!--*Example 禁用自动生成的复杂查询-->
        <table tableName="user_info" domainObjectName="UserDO"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false"></table>
        <table tableName="user_password" domainObjectName="UserPasswordDO"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false"></table>

    </context>
</generatorConfiguration>
```

## 新建 Maven 命令

Run -- Edit Configurations  -- 新增 -- Maven：

+ Name: mybatis-generator
+ Command line: mybatis-generator:generate

Run 'mybatis-generator'

## 配置数据源

在 application.properties 添加配置：

```
spring.datasource.name=seckill
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/seckill
spring.datasource.username=root
spring.datasource.password=root

# 使用 druid 数据源
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

## 修改 App.java

```
@SpringBootApplication(scanBasePackages = {"com.karonda"})
@RestController
@MapperScan("com.karonda.dao")
public class App {

    @Autowired
    private UserDOMapper userDOMapper;

    @RequestMapping("/")
    public String home(){
        UserDO userDO = userDOMapper.selectByPrimaryKey(1);
        if(userDO == null){
            return "用户对象不存在";
        }else{
            return userDO.getName();
        }
    }

    public static void main( String[] args )
    {
        System.out.println( "Hello World!" );

        SpringApplication.run(App.class, args);
    }
}
```


源码：[spring-boot-seckill](https://github.com/VictorBu/spring-boot-seckill)

