---
title: Feign 自定义 ErrorDecoder (捕获 Feign 服务端异常)
date: 2020-04-27 17:00:00
updated: 2020-04-27 17:00:00
categories: [IT]
tags: [Feign]
---

> 问题描述

Feign 客户端捕获不到服务端抛出的异常

> 问题解决

重新 ErrorDecoder 即可，比如下面例子中在登录鉴权时想使用认证服务器抛出 OAuth2Exception 的异常，代码如下：

```
import com.fasterxml.jackson.databind.ObjectMapper;
import feign.Response;
import feign.Util;
import feign.codec.ErrorDecoder;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.common.exceptions.OAuth2Exception;

import java.io.IOException;

@Configuration
public class FeignErrorDecoder implements ErrorDecoder {

    @Override
    public Exception decode(String methodKey, Response response) {
        try {
            String message = Util.toString(response.body().asReader());

            ObjectMapper objectMapper = new ObjectMapper();
            OAuth2Exception oAuth2Exception = objectMapper.readValue(message, OAuth2Exception.class);
            return oAuth2Exception;
        } catch (IOException ignored) {
        }

        return decode(methodKey, response);
    }
}
```