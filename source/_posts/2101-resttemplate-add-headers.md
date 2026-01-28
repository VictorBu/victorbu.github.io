---
title: RestTemplate 统一添加 Header
date: 2020-04-15 21:00:00
updated: 2020-04-15 21:00:00
categories: [IT]
tags: [RestTemplate]
---


# 一、添加拦截器

```
public class HeaderRequestInterceptor implements ClientHttpRequestInterceptor {

    private final String headerName;

    private final String headerValue;

    public HeaderRequestInterceptor(String headerName, String headerValue) {
        this.headerName = headerName;
        this.headerValue = headerValue;
    }

    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
        request.getHeaders().set(headerName, headerValue);
        return execution.execute(request, body);
    }
}
```

# 二、RestTemplate Bean

```
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate() {
        List<ClientHttpRequestInterceptor> interceptors = new ArrayList<>();
        interceptors.add(new HeaderRequestInterceptor("token", "123"));

        RestTemplate restTemplate = new RestTemplate();
        restTemplate.setInterceptors(interceptors);

        return restTemplate;
    }
}
```

# 三、使用

```
    @Autowired
    private RestTemplate restTemplate;
```

> 参考

[spring - resttemplate添加header - resttemplate设置header](https://code-examples.net/zh-CN/q/1258f3b)
