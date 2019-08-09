---
title: MongoTemplate 移除 _class 字段
date: 2019-08-09 10:30:00
updated: 2019-08-09 10:30:00
categories: [IT]
tags: [Spring Boot, MongoDB]
---

```
@Configuration
public class ApplicationReadyListener implements ApplicationListener<ContextRefreshedEvent> {
 
  @Autowired
  MongoTemplate mongoTemplate;
 
  @Override
  public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
    MongoConverter converter = mongoTemplate.getConverter();
    if (converter.getTypeMapper().isTypeKey("_class")) {
      ((MappingMongoConverter) converter).setTypeMapper(new DefaultMongoTypeMapper(null));
    }
  }
}
```

> 参考：

[Spring Boot 学习笔记：MongoTemplate 移除 _class 字段](https://blog.csdn.net/cuixianlong/article/details/81227412)

