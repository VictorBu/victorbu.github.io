---
title: 使用 Jasypt 加密 Spring Boot 配置文件
date: 2020-04-17 21:00:00
updated: 2020-04-17 21:00:00
categories: [IT]
tags: [Spring Boot, Jasypt]
---


# 一、添加依赖包

```
<dependency>
	<groupId>com.github.ulisesbocchio</groupId>
	<artifactId>jasypt-spring-boot-starter</artifactId>
	<version>3.0.2</version>
</dependency>
```

# 二、配置加密盐

```
jasypt:
  encryptor:
    password: abc123 # 加密盐
```

# 三、加密字符串

```
@Component
public class TestRunner implements CommandLineRunner {

    @Autowired
    StringEncryptor stringEncryptor;

    @Override
    public void run(String... args) throws Exception {
        System.out.println(stringEncryptor.encrypt("PWD123456"));
    }
}
```

加密后的字符串为：eYlNuObA8plNuHpvRfgUfYfhlxWYQiU/1kudw7tfiWls93/1QJkG0vR14uu0yFGM

# 四、将加密后的字符串写入配置文件

```
password: ENC(eYlNuObA8plNuHpvRfgUfYfhlxWYQiU/1kudw7tfiWls93/1QJkG0vR14uu0yFGM)
```

# 五、使用

```
@Component
public class TestRunner implements CommandLineRunner {

    @Value("${password}")
    String password;

    @Override
    public void run(String... args) throws Exception {
        System.out.println(password);
    }
}
```

> 注意

为了防止加密盐泄露反解出密码，可以在项目启动的时候通过参数传入加密盐的值；或者在服务器的环境变量里配置加密盐

*--- 2020-06-11 更新开始 ---*

也可以使用 jasypt-maven-plugin 加密解密

[【Jasypt】给你的配置加把锁](https://www.jianshu.com/p/efa8c063a915)

*--- 2020-06-11 更新结束 ---*