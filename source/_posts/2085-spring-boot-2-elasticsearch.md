---
title: Spring Boot + Elasticsearch 使用示例
date: 2019-07-24 19:30:00
updated: 2019-07-24 19:30:00
categories: [IT]
tags: [Spring Boot, Elasticsearch]
---

> 本文分别使用 Elasticsearch Repository 和 ElasticsearchTemplate 实现 Elasticsearch 的简单的增删改查

# 一、Elastic Stack

Elastic Stack 是 ELK Stack 在 5.0 版本加入 Beats 套件后的新称呼

1. Elasticsearch: 一个基于 JSON 的分布式搜索和分析引擎
1. Logstash: 动态数据收集管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到您最喜欢的“存储库”
1. Kibana: 探索数据并管理堆栈，实现数据可视化
1. Beats: 一个面向轻量型采集器的平台，从成百上千或成千上万台机器和系统向 Logstash 或 Elasticsearch 发送数据，目前包含：
    1. Filebeat: 日志文件 (用于转发和汇总日志与文件)
    1. Metricbeat: 指标 (用于从系统和服务收集指标)
    1. Packetbeat: 网络数据 (用于深挖网线上传输的数据，了解应用程序动态)
    1. Winlogbeat: Windows 事件日志 (用于密切监控基于 Windows 的基础设施上发生的事件)
    1. Auditbeat: 审计数据 (收集 Linux 审计框架的数据，监控文件完整性)
    1. Heartbeat: 运行时间监控 (通过主动探测来监测服务的可用性)
    1. Functionbeat: 无需服务器的采集器 (收集、传送并监测来自云服务的相关数据)
1. APM Server: 从 APM agents 接收数据并将其转换为 Elasticsearch 文档
1. Elasticsearch Hadoop: 深度集成 Hadoop 和 ElasticSearch 的项目



# 二、安装 Elasticsearch

截至目前，spring-data-elasticsearch 支持的 ElasticSearch 的版本为 [6.2](https://github.com/spring-projects/spring-data-elasticsearch#maven-configuration)

本文使用 [docker 安装](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/docker.html)：

```
sudo docker pull docker.elastic.co/elasticsearch/elasticsearch:6.2.4
```

开发者模式运行：

```
sudo docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.2.4
```


# 三、使用示例

application.yml:

```
spring:
  data:
    elasticsearch:
      cluster-name: docker-cluster
      cluster-nodes: 192.168.30.101:9300
```

entity:

```
@Document(indexName = "user")
public class User {
    @Id
    private String Id;
    private String name;
    private Integer gender;
    
    /*getter setter*/
}
```

service:

```
public interface UserService {

    long count();

    User save(User user);

    void deleteById(String id);

    void deleteAll();

    Iterable<User> findAll();

    Iterable<User> findAll(Integer pageNum, Integer pageSize);

    List<User> findAllByName(String name);

    Page<User> findAllByName(Integer pageNum, Integer pageSize, String name);
}
```

## 3.1 使用 Elasticsearch Repository

### 3.1.1 新建 UserElasticsearchRepository

```
@Repository
public interface UserElasticsearchRepository extends ElasticsearchRepository<User, String> {
}
```

### 3.1.2 新建 UserServiceElasticsearchRepository

```
@Profile("ElasticsearchRepository")
@Service
public class UserServiceElasticsearchRepository implements UserService {

    @Autowired
    private UserElasticsearchRepository userElasticsearchRepository;

    @Override
    public long count() {
        return userElasticsearchRepository.count();
    }

    @Override
    public User save(User user) {
        return userElasticsearchRepository.save(user);
    }

    @Override
    public void deleteById(String id) {
        userElasticsearchRepository.deleteById(id);
    }

    @Override
    public void deleteAll() {
        userElasticsearchRepository.deleteAll();
    }

    @Override
    public Iterable<User> findAll() {
        return userElasticsearchRepository.findAll();
    }

    @Override
    public Iterable<User> findAll(Integer pageNum, Integer pageSize) {

        Pageable pageable = PageRequest.of(pageNum, pageSize);
        return userElasticsearchRepository.findAll(pageable);
    }

    @Override
    public List<User> findAllByName(String name) {

        MatchQueryBuilder matchQueryBuilder = new MatchQueryBuilder("name", name);

        Iterable<User> userIterable =  userElasticsearchRepository.search(matchQueryBuilder);

        List<User> userList = new ArrayList<>();
        userIterable.forEach(u -> userList.add(u));

        return userList;
    }

    @Override
    public Page<User> findAllByName(Integer pageNum, Integer pageSize, String name) {

        SearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.matchPhraseQuery("name", name))
                .withPageable(PageRequest.of(pageNum, pageSize))
                .build();
        return userElasticsearchRepository.search(searchQuery);
    }
}
```

## 3.2 使用 ElasticsearchTemplate

## 3.2.1 新建 UserServiceElasticsearchTemplate

```
@Profile("ElasticsearchTemplate")
@Service
public class UserServiceElasticsearchTemplate implements UserService {

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;

    @Override
    public long count() {

        SearchQuery searchQuery = new NativeSearchQueryBuilder()
                .build();
        return elasticsearchTemplate.count(searchQuery, User.class);
    }

    @Override
    public User save(User user) {

        IndexQuery indexQuery = new IndexQueryBuilder().withId(user.getId().toString()).withObject(user).build();
        elasticsearchTemplate.index(indexQuery);

        return user;
    }

    @Override
    public void deleteById(String id) {

        elasticsearchTemplate.delete(User.class, id);
    }

    @Override
    public void deleteAll() {

        elasticsearchTemplate.deleteIndex(User.class);
    }

    @Override
    public Iterable<User> findAll() {

        SearchQuery searchQuery = new NativeSearchQueryBuilder()
                .build();

        return elasticsearchTemplate.queryForList(searchQuery, User.class);
    }

    @Override
    public Iterable<User> findAll(Integer pageNum, Integer pageSize) {
        SearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withPageable(PageRequest.of(pageNum, pageSize))
                .build();

        return elasticsearchTemplate.queryForList(searchQuery, User.class);
    }

    @Override
    public List<User> findAllByName(String name) {

        SearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.matchPhraseQuery("name", name))
                .build();

        return elasticsearchTemplate.queryForList(searchQuery, User.class);
    }

    @Override
    public Page<User> findAllByName(Integer pageNum, Integer pageSize, String name) {

        SearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.matchPhraseQuery("name", name))
                .withPageable(PageRequest.of(pageNum, pageSize))
                .build();

        return elasticsearchTemplate.queryForPage(searchQuery, User.class);
    }
}
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-2-elasticsearch)

# 未解决问题

实体类中有 LocalDate 类型时报错：failed to map source


> 参考：

1. [SpringBoot+Elasticsearch](https://www.cnblogs.com/cjsblog/p/9756978.html)
1. [Spring Data Elasticsearch](https://docs.spring.io/spring-data/elasticsearch/docs/3.1.9.RELEASE/reference/html/#elasticsearch.repositories)

