---
title: Spring Boot 上传文件
date: 2019-11-29 11:30:00
updated: 2019-11-29 11:30:00
categories: [IT]
tags: [Java, Spring Boot]
---

# 一、实体类

```
public class UploadModel {
    private Long userId;
    private MultipartFile img1;
    private MultipartFile img2;

    /* getter setter */
}
```

# 二、接收

```
@RestController
public class UploadController {

    @PostMapping("/upload")
    public void upload(UploadModel uploadModel) {
	
        /* 上传逻辑 */
    }

}
```

> 注意事项

1. 接收不需要 @RequestBody 或 @RequestParam 等注解
2. 测试时 Header 需要设置 Content-Type 为 multipart/form-data


