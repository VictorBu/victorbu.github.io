---
title: Spring Boot 统一返回结果及异常处理
date: 2020-04-10 17:00:00
updated: 2020-04-10 17:00:00
categories: [IT]
tags: [Spring Boot]
---

> 在 [Spring Boot 构建电商基础秒杀项目 (三) 通用的返回对象 & 异常处理](https://www.cnblogs.com/victorbu/p/10545405.html) 基础上优化、调整


# 一、通用类

## 1.1 通用的返回对象

```
public class CommonResult<T> {
    private Integer code;
    private String msg;
    private T data;

    private CommonResult(ResultEnum resultEnum) {
        this.code = resultEnum.getCode();
        this.msg = resultEnum.getMsg();
    }

    public static CommonResult success(Object obj) {
        CommonResult commonResult = new CommonResult(ResultEnum.SUCCESS);
        commonResult.data = obj;
        return commonResult;
    }

    public static CommonResult error(ResultEnum resultEnum) {
        CommonResult commonResult = new CommonResult(resultEnum);
        return commonResult;
    }

    public static CommonResult error(ResultEnum resultEnum, String msg) {
        CommonResult commonResult = new CommonResult(resultEnum);
        commonResult.msg = msg;
        return commonResult;
    }

    public Integer getCode() {
        return this.code;
    }

    public String getMsg() {
        return this.msg;
    }

    public T getData() {
        return this.data;
    }
}
```

## 1.2 异常枚举


```
public enum ResultEnum {

    SUCCESS(0, "成功。"),

    PARAM_ERROR(1001,"参数验证失败。"),

    UNKNOWN_ERROR(9999,"未知错误。"),
    ;

    private Integer code;
    private String msg;

    ResultEnum(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public Integer getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }
}
```

## 1.3 自定义异常

```
public class BusinessException extends RuntimeException {

    private CommonResult commonResult;

    public BusinessException(ResultEnum resultEnum) {
        super();
        this.commonResult = CommonResult.error(resultEnum);
    }

    public BusinessException(ResultEnum resultEnum, String msg) {
        super();
        this.commonResult = CommonResult.error(resultEnum, msg);
    }

    public CommonResult getCommonReturn() {
        return commonResult;
    }
}
```

# 二、统一异常处理

```
@RestControllerAdvice
public class BusinessExceptionHandler {
    /**
     * 参数校验异常
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Object handleBindException(MethodArgumentNotValidException ex) {

        StringBuilder errMsgBuilder = new StringBuilder();

        ex.getBindingResult().getFieldErrors().forEach((fieldError) -> {
            errMsgBuilder.append(fieldError.getDefaultMessage());
        });

        return CommonResult.error(ResultEnum.PARAM_ERROR, errMsgBuilder.toString());
    }

    /**
     * 自定义异常
     */
    @ExceptionHandler(BusinessException.class)
    public Object handleBusinessException(BusinessException ex){
        return ex.getCommonReturn();
    }

    @ExceptionHandler(Exception.class)
    public Object handleException(Exception ex){
        return CommonResult.error(ResultEnum.UNKNOWN_ERROR, ex.getMessage());
    }
}
```

# 三、统一返回结果

```
@RestControllerAdvice
public class ResponseAdvice implements ResponseBodyAdvice<Object> {
    @Override
    public boolean supports(MethodParameter returnType, Class converterType) {
        return true;
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType,
                                  Class<? extends HttpMessageConverter<?>> selectedConverterType,
                                  ServerHttpRequest request, ServerHttpResponse response) {

        // swagger 不处理
        String uri = request.getURI().toString();
        if(uri.contains("/v2/api-docs") || uri.contains("/swagger-resources") || uri.contains("/swagger-ui.html")) {
            return body;
        }

        if(body == null) {
            return CommonResult.success(null);
        } else if(body instanceof CommonResult) {
            return body;
        }

        return CommonResult.success(body);
    }
}
```

# 测试 Controller

```
@RestController
public class TestController {

    @RequestMapping("/empty")
    public void empty() {

    }

    @RequestMapping("/exception")
    public void exception() {
        throw new BusinessException(ResultEnum.PARAM_ERROR);
    }

    @RequestMapping("/object")
    public TestResult object() {
        TestResult testResult = new TestResult();
        testResult.setId(1L);
        testResult.setName("name");
        return testResult;
    }

    class TestResult {
        @JsonSerialize(using= ToStringSerializer.class)
        private Long id;
        private String name;

        public Long getId() {
            return id;
        }

        public void setId(Long id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
}
```

# 测试结果

```
// empty
{
  "code": 0,
  "msg": "成功。",
  "data": null
}

// exception
{
  "code": 1001,
  "msg": "参数验证失败。",
  "data": null
}

// object
{
  "code": 0,
  "msg": "成功。",
  "data": {
    "id": "1",
    "name": "name"
  }
}
```

注：如果返回类型为 String 需特殊处理，见：[SpringBoot 统一返回处理遇到的问题 cannot be cast to java.lang.String](https://www.jianshu.com/p/0f16c9f1760e)

> 同时可以使用 RequestBodyAdvice 统一处理入参，但是不支持 GET 方法