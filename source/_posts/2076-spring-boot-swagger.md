---
title: Spring Boot 2 集成 Swagger
date: 2019-06-26 10:30:00
updated: 2020-06-10 10:30:00
categories: [IT]
tags: [Spring Boot, Swagger]
---

本文测试代码使用 Spring Boot 2.1.6.RELEASE + Swagger 2.9.2

# 添加依赖

```
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.9.2</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.9.2</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
    </dependency>
```

# 添加配置

```
swagger:
  enable: true
```

# 添加配置类

```
@Configuration
@EnableSwagger2 // 启用 Swagger
@ConditionalOnExpression("${swagger.enable:true}") // 根据配置决定是否最终启用 Swagger
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.karonda.springbootswagger.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("SWAGGER 测试平台")
                .description("测试 SWAGGER 用")
                .termsOfServiceUrl("https://github.com/VictorBu")
                .version("1.0.0")
                .build();
    }
}
```

# 添加测试接口

```
@RestController
@Api(tags = "测试接口")
public class HiController {

    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    @ApiOperation(value="打招呼")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "firstName", value = "名字", paramType = "query", required = true),
            @ApiImplicitParam(name = "lastName", value = "姓氏", paramType = "query", required = true)
    })
    public String sayHi(@RequestParam("firstName") String firstName, @RequestParam("lastName") String lastName){
        return "Hi, " + firstName + " " + lastName;
    }
}
```

# 测试

启动程序输入 http://localhost:8080/swagger-ui.html 即可看到效果

# 注意事项

如果在项目中使用了 Spring Security 则需要[添加如下配置](https://stackoverflow.com/questions/37671125/how-to-configure-spring-security-to-allow-swagger-url-to-be-accessed-without-aut)：

```
http.authorizeRequests()
        .antMatchers(
                // -- swagger ui
                "/v2/api-docs",
                "/swagger-resources",
                "/swagger-resources/**",
                "/configuration/ui",
                "/configuration/security",
                "/swagger-ui.html",
                "/webjars/**").permitAll()
        .anyRequest().authenticated();
```



完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-swagger)

*--- 2020-06-10 更新开始 ---*

最近使用中发现会报错：

```
No mapping for GET /swagger-ui.html
```

应该是 Spring Boot 版本问题，修复：

```
@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {

    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
        super.addResourceHandlers(registry);
    }
}
```

参考：[swagger报错No handler found for GET /swagger-ui.html](https://www.cnblogs.com/guanghe/p/10238990.html)

*--- 2020-06-10 更新结束 ---*

