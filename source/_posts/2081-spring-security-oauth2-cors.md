---
title: Spring Boot 中使用 Spring Security, OAuth2 跨域问题 (自己挖的坑)
date: 2019-07-12 18:30:00
updated: 2019-07-12 18:30:00
categories: [IT]
tags: [Spring Boot, Spring Security, OAuth2, CORS]
---


使用 Spring Boot 开发 API 使用 Spring Security + OAuth2 + JWT 鉴权，已经在 Controller 配置允许跨域：

```
@RestController
@CrossOrigin(allowCredentials = "true", allowedHeaders = "*")
public class XXController {

}
```

但是前端反馈，登录接口可以正常访问，但是需要鉴权的接口报跨域错误：

```
Access to XMLHttpRequest at 'http://******' from origin 'null' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

开始以为是前端写法问题，后来排查很久发现是 Spring Security 的配置文件中漏掉了配置项: cors() 详细代码如下：

```
    @Override
    public void configure(HttpSecurity http) throws Exception {

        http.authorizeRequests()
                .antMatchers("/v2/api-docs").permitAll()
                .anyRequest().authenticated()
                .and().cors() // 需要添加此配置项
                .and().csrf().disable();
    }
```

因为自己的疏忽，浪费不少时间，特此记录！
