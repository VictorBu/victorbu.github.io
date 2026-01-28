---
title: 使用 Zuul 聚合多个微服务的 Swagger 文档
date: 2019-07-03 15:30:00
updated: 2019-07-03 15:30:00
categories: [IT]
tags: [Microservices, Spring Cloud, Zuul, Swagger]
---


在 Spring Boot 中集成 Swagger 可参考之前的文章：[Spring Boot 2 集成 Swagger](/2019/06/2076-spring-boot-swagger/), 在各个微服务中的配置与之相同；本文仅介绍在 Zuul 中的配置

# 在 Zuul 项目中添加配置

```
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    
    @Autowired
    ZuulProperties properties;

    @Primary
    @Bean
    public SwaggerResourcesProvider swaggerResourcesProvider() {
        return () -> {
            List<SwaggerResource> resources = new ArrayList<>();
            properties.getRoutes().values().stream()
                    .forEach(route -> resources
                            .add(createResource(route.getServiceId(), route.getServiceId(), "2.0")));
            return resources;
        };
    }

    private SwaggerResource createResource(String name, String location, String version) {
        SwaggerResource swaggerResource = new SwaggerResource();
        swaggerResource.setName(name);
        swaggerResource.setLocation("/" + location + "/v2/api-docs");
        swaggerResource.setSwaggerVersion(version);
        return swaggerResource;
    }
}
```

其中 /v2/api-docs 为 Swagger 的 api

# 测试

访问 http://localhost:8762/swagger-ui.html 即可看到效果 (8762 为 Zuul 项目的端口)

# 注意事项

1. 各个微服务可以不引用 swagger-ui 依赖包，仅在 Zuul 项目引用即可
1. 如果微服务中使用了Spring Security 需要放行 /v2/api-docs


参考：[sample-zuul-swagger2](https://github.com/SpringCloud/sample-zuul-swagger2)

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/zuul-swagger)






