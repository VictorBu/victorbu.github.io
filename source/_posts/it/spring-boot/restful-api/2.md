---
title: Spring Boot 2.x 编写 RESTful API (二) 校验
date: 2019-04-04 08:00:00
updated: 2019-04-04 08:00:00
categories: [IT]
tags: [Java, Spring Boot, RESTful API]
---

> [用Spring Boot编写RESTful API](https://study.163.com/course/courseMain.htm?courseId=1005213034) 学习笔记



![](https://oss.x8y.cc/blog-img/2060/2/BeanValidation.jpg)

约束规则对子类依旧有效

# groups 参数

+ 每个约束用注解都有一个 groups 参数
+ 可接收多个 class 类型 (必须是接口)
+ 不声明 groups 参数是默认组 javax.validation.groups.Default
+ 声明了 groups 参数的会从 Default 组移除，如需加入 Default 组需要显示声明，例如 @Null(groups={Default.class, Step1.class})

# @Valid vs. @Validated

+ @Valid 是 JSR 标准定义的注解，只验证 Default 组的约束
+ @Validated 是spring 定义的注解，可以通过参数来指定验证的组，例如：@Validated({Step1.class, Default.class}) 表示验证 Step1 和 Default 两个组的约束
+ @Valid 可用在成员变量上，进行级联验证；@Validated 只能用在参数上，表示这个参数需要验证

# BindingResult

```
public Object doSomething(@Validated @RequestBody OneDto oneDto) {
    // 通不过校验的参数，不会执行这个方法
}


public Object doSomething(@Validated @RequestBody OneDto oneDto, BindingResult result) {
    // 参数通不过校验也会进入方法执行，校验结果会通过result参数传递进来
    if (result.hasErrors()){
        // 没通过校验
    }
}

```

# 手动验证

```
// 装载验证器
@Autowired Validator validator;

// 验证某个类，下面是执行默认的验证组，如果需要指定验证组，多传一个 class 参数
Set<ConstraintViolation<?>> result = validator.validate(obj);
 
// 通不过校验 result 的集合会有值，可以通过 size() 判断
```


源码：[spring-boot-2-restful](https://github.com/VictorBu/spring-boot-2-restful)