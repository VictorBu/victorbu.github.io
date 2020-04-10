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
public class CommonReturn {
    private Integer code;
    private String msg;
    private Object data;

    private CommonReturn(CommonResult commonResult) {
        this.code = commonResult.getCode();
        this.msg = commonResult.getMsg();
    }

    public static CommonReturn success(Object obj) {
        CommonReturn commonReturn = new CommonReturn(ResultEnum.SUCCESS);
        commonReturn.data = obj;
        return commonReturn;
    }

    public static CommonReturn error(CommonResult commonResult) {
        CommonReturn commonReturn = new CommonReturn(commonResult);
        return commonReturn;
    }

    public static CommonReturn error(CommonResult commonResult, String msg) {
        CommonReturn commonReturn = new CommonReturn(commonResult);
        commonReturn.msg = msg;
        return commonReturn;
    }


    public Integer getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }

    public Object getData() {
        return data;
    }
}
```

## 1.2 返回接口

```
public interface CommonResult {

    Integer getCode();

    String getMsg();

    void setMsg(String msg);

}
```

## 1.3 返回枚举


```
public enum ResultEnum implements CommonResult {

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

    @Override
    public Integer getCode() {
        return code;
    }

    @Override
    public String getMsg() {
        return msg;
    }

    @Override
    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

## 1.4 自定义异常

```
public class BusinessException extends Exception implements CommonResult {

    private CommonResult commonResult;

    public BusinessException(CommonResult commonResult) {
        super();
        this.commonResult = commonResult;
    }

    public BusinessException(CommonResult commonResult, String msg) {
        super();
        this.commonResult = commonResult;
        this.setMsg(msg);
    }


    @Override
    public Integer getCode() {
        return this.commonResult.getCode();
    }

    @Override
    public String getMsg() {
        return this.commonResult.getMsg();
    }

    @Override
    public void setMsg(String msg) {
        this.commonResult.setMsg(msg);
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

        return CommonReturn.error(ResultEnum.PARAM_ERROR, errMsgBuilder.toString());
    }

    /**
     * 自定义异常
     */
    @ExceptionHandler(BusinessException.class)
    public Object handleBusinessException(BusinessException ex){
        return CommonReturn.error(ex);
    }

    @ExceptionHandler(Exception.class)
    public Object handleException(Exception ex){
        return CommonReturn.error(ResultEnum.UNKNOWN_ERROR, ex.getMessage());
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
            return CommonReturn.success(null);
        } else if(body instanceof CommonReturn) {
            return body;
        }

        return CommonReturn.success(body);
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
    public void exception() throws BusinessException {
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

> 同时可以使用 RequestBodyAdvice 统一处理入参，但是不支持 GET 方法