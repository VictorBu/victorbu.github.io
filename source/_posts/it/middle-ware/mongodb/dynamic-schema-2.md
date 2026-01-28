---
title: 基于 MongoDB 动态字段设计的探索 (二) 聚合操作
date: 2019-08-03 08:30:00
updated: 2019-08-03 08:30:00
categories: [IT]
tags: [Spring Boot, MongoDB]
---

> 业务需求及设计见前文：[基于 MongoDB 动态字段设计的探索](/2019/08/2088-dynamic-schema-use-mongodb/)

# 根据专业计算各科平均分 (总分、最高分、最低分)

```
public Object avg(String major){

    Aggregation aggregation = Aggregation.newAggregation(
            Aggregation.unwind("courseList"),
            Aggregation.match(Criteria.where("major").is(major)),
            Aggregation.group("courseList.name").avg("courseList.score").as("avg")
    ); // avg 可以替换成 sum, max, min 分别求各科总分、最高分、最低分

    AggregationResults<BasicDBObject> aggregationResults = mongoTemplate.aggregate(aggregation, Student.class, BasicDBObject.class);

    List<BasicDBObject> result = new ArrayList<>();
    for(Iterator<BasicDBObject> iterator = aggregationResults.iterator(); iterator.hasNext();){
        result.add(iterator.next());
    }

    return result;
}
```

# 计算个人总分数

```
public Object sum(String name){

    Aggregation aggregation = Aggregation.newAggregation(
            Aggregation.unwind("courseList"),
            Aggregation.match(Criteria.where("name").is(name)),
            Aggregation.group("name").sum("courseList.score").as("sum")
    );

    AggregationResults<BasicDBObject> aggregationResults = mongoTemplate.aggregate(aggregation, Student.class, BasicDBObject.class);

    List<BasicDBObject> result = new ArrayList<>();
    for(Iterator<BasicDBObject> iterator = aggregationResults.iterator(); iterator.hasNext();){
        result.add(iterator.next());
    }

    return result;
}
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-2-mongodb)

参考：[MongoTemplate 聚合操作](https://blog.csdn.net/ruoguan_jishou/article/details/79289369)
