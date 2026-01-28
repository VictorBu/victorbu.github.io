---
title: Spring Cloud 学习 (五) Zuul
date: 2019-06-13 12:30:00
updated: 2019-06-13 12:30:00
categories: [IT]
tags: [Microservices, Spring Cloud, Zuul]
---

Zuul 作为路由网关组件，在微服务架构中有着非常重要的作用，主要体现在以下 6 个方面：

1. Zuul, Ribbon 以及 Eureka 相结合，可以实现智能路由和负载均衡的功能，Zuul 能够将请求流量按某种策略分发到集群状态的多个服务实例
1. 网关将所有服务的 API 接口统一聚合，并统一对外暴露。外界系统调用 API 接口时，都是由网关对外暴露的 API 接口，外界系统不需要知道微服务系统中各服务相互调用的复杂性。微服务系统也保护了其内部微服务单元的 API 接口 ， 防止其被外界直 接调用，导致服务的敏感信息对外暴露
1. 网关服务可以做用户身份认证和权限认证，防止非法请求操作 API 接口，对服务器起到保护作用
1. 网关可以实现监控功能，实时日志输出，对请求进行记录
1. 网关可以用来实现流量监控，在高流量的情况下，对服务进行降级
1. API 接口从内部服务分离出来，方便做测试

Zuul 的核心是一系列过滤器，可以在 Http 请求的发起和响应返回期间执行一系列的过滤器：

1. PRE 过滤器：在请求路由到具体的服务之前执行，这种类型的过滤器可以做安全验证，例如身份验证、 参数验证等
1. ROUTING 过滤器：用于将请求路由到具体的微服务实例。在默认情况下，它使用 Http Client 进行网络请求
1. POST 过滤器：在请求己被路由到微服务后执行。 一般情况下，用作收集统计 信息、指标，以及将响应传输到客户端
1. ERROR 过滤器：在其他过滤器发生错误时执行

Zuul 采取了动态读取、编译和运行这些过滤器。过滤器之间不能直接相互通信，而是通过 RequestContext 对象来共享数据，每个请求都会创建一个 RequestContext 对象。Zuul 过滤器具有以下关键特性：

1. Type (类型): Zuul 过滤器的类型，这个类型决定了过滤器在请求的哪个阶段起作用，例如 Pre、Post 阶段等
1. Execution Order (执行顺序): 规定了过滤器的执行顺序，Order 的值越小越先执行
1. Criteria (标准): Filter 执行所需的条件 
1. Action (行动): 如果符合执行条件，则执行 Action (即逻辑代码)

# 使用 Zuul

新建 spring-cloud-eureka-zuul-client

## pom

```
<parent>
    <artifactId>spring-cloud-parent</artifactId>
    <groupId>com.karonda</groupId>
    <version>1.0.0</version>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>spring-cloud-eureka-zuul-client</artifactId>


<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

## application.yml

```
server:
  port: 8051


eureka:
  client:
    service-url:
      defaultZone: http://localhost:8001/eureka/

spring:
  application:
    name: zuul-client

zuul:
  routes:
    hiapi:
      path: /hiapi/**
      serviceId: eureka-client
    ribbonapi:
      path: /ribbonapi/**
      serviceId: ribbon-client
    feignapi:
      path: /feignapi/**
      serviceId: feign-client
```

## 启动类

```
@EnableZuulProxy // 开启 Zuul
@SpringBootApplication
public class EurekaZuulClientApp {
    public static void main(String[] args){
        SpringApplication.run(EurekaZuulClientApp.class, args);
    }
}
```

## 测试

1. 启动 eureka-server
1. 启动 eureka-client (两个实例：一个 8011 端口，一个 8012 端口)
1. 启动 eureka-ribbon-client
1. 启动 eureka-feign-client
1. 启动 eureka-zuul-client


多次访问 http://localhost:8031/hiapi/hi?name=victor 可以看到 8011 和 8012 端口交替出现 (Zuul 默认 与 Ribbon 结合实现了负载均衡)

多次访问 http://localhost:8031/ribbonapi/hi?name=victor 可以看到 8011 和 8012 端口交替出现

多次访问 http://localhost:8031/feignapi/hi?name=victor 可以看到 8011 和 8012 端口交替出现

# 在 Zuul 上配置熔断器

## 实现 FallbackProvider 接口

```
@Component
public class MyFallbackProvider implements FallbackProvider {

    @Override
    public String getRoute() {
        return "eureka-client"; // 如果所有的路由服务都加熔断功能，返回 "*"
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return 200;
            }

            @Override
            public String getStatusText() throws IOException {
                return "OK";
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("error, fallback".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }
}
```

## 测试

1. 关闭所有的 eureka-client
1. 重启 eureka-zuul-client

# 在 Zuul 中使用过滤器

## 继承 ZuulFilter

```
@Component
public class MyFilter extends ZuulFilter {

    private static Logger logger = LoggerFactory.getLogger(MyFilter.class);

    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {

        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        Object accessToken = request.getParameter("token");

        if(accessToken == null){
            logger.warn("token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);

            try {
                ctx.getResponse().getWriter().write("token is empty");
            } catch (IOException e) {
                return null;
            }
        }

        logger.info("ok");

        return null;
    }
}
```

## 测试

1. 重启 eureka-zuul-client

访问 http://localhost:8051/hiapi/hi?name=victor

访问 http://localhost:8051/hiapi/hi?name=victor&token=xx



完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-cloud-parent)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

