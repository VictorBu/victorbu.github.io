---
title: Spring Boot + MongoDB 使用示例
date: 2019-07-29 19:30:00
updated: 2019-07-29 19:30:00
categories: [IT]
tags: [Spring Boot, MongoDB]
---

> 本文分别使用 MongoRepository 和 MongoTemplate 实现 MongoDB 的简单的增删改查

本文使用 [docker 安装 MongoDB](https://www.cnblogs.com/xinsen/p/10588767.html)：



# 使用示例

application.yml:

```
spring:
  data:
    mongodb:
      uri: mongodb://test:123456@192.168.30.101:27017/test
```

entity:

```
public class User {
    @Id
    private String id;
    private String name;
    private Integer gender;

    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private LocalDate birthday;

    /*getter setter*/
}
```

service:

```
public interface UserService {

    User save(User user);

    void deleteById(String id);

    void deleteAll();

    Iterable<User> findAll();

    Iterable<User> findAll(Integer pageNum, Integer pageSize);

    List<User> findAllByName(String name);

    Page<User> findAllByName(Integer pageNum, Integer pageSize, String name);
}
```

## 1.1 使用 MongoRepository

### 1.1.1 新建 UserMongoRepository

```
public interface UserMongoRepository extends MongoRepository<User, String> {

    List<User> findByName(String name);
}
```

### 1.1.2 新建 UserServiceMongoRepository

```
@Profile("MongoRepository")
@Service
public class UserServiceMongoRepository implements UserService {

    @Autowired
    private UserMongoRepository userMongoRepository;

    @Override
    public User save(User user) {
        return userMongoRepository.save(user);
    }

    @Override
    public void deleteById(String id) {
        userMongoRepository.deleteById(id);
    }

    @Override
    public void deleteAll() {
        userMongoRepository.deleteAll();
    }

    @Override
    public Iterable<User> findAll() {
        return userMongoRepository.findAll();
    }

    @Override
    public Iterable<User> findAll(Integer pageNum, Integer pageSize) {

        Pageable pageable = PageRequest.of(pageNum, pageSize);
        return userMongoRepository.findAll(pageable);
    }

    @Override
    public List<User> findAllByName(String name) {
        return userMongoRepository.findByName(name);
    }

    @Override
    public Page<User> findAllByName(Integer pageNum, Integer pageSize, String name) {

        User user = new User();
        user.setName(name);

        ExampleMatcher matcher = ExampleMatcher.matching();
        Example<User> userExample = Example.of(user, matcher);

        Pageable pageable = PageRequest.of(pageNum, pageSize);

        return userMongoRepository.findAll(userExample, pageable);
    }
}
```

## 1.2 使用 MongoTemplate

## 1.2.1 新建 UserServiceMongoTemplate

```
@Profile("MongoTemplate")
@Service
public class UserServiceMongoTemplate implements UserService {

    @Autowired
    private MongoTemplate mongoTemplate;

    @Override
    public User save(User user) {
        return mongoTemplate.save(user);
    }

    @Override
    public void deleteById(String id) {
        Query query = new Query(Criteria.where("id").is(id));
        mongoTemplate.remove(query);
    }

    @Override
    public void deleteAll() {
        mongoTemplate.remove(User.class);
    }

    @Override
    public Iterable<User> findAll() {
        return mongoTemplate.findAll(User.class);
    }

    @Override
    public Iterable<User> findAll(Integer pageNum, Integer pageSize) {
        Query query = new Query();
        query.skip(pageNum * pageSize);
        query.limit(pageSize);

        return mongoTemplate.find(query, User.class);
    }

    @Override
    public List<User> findAllByName(String name) {
        Query query = new Query(Criteria.where("name").is(name));
        return mongoTemplate.find(query, User.class);
    }

    @Override
    public Page<User> findAllByName(Integer pageNum, Integer pageSize, String name) {

        Query query = new Query();
        query.skip(pageNum * pageSize);
        query.limit(pageSize);

        Criteria criteria = new Criteria();
        criteria.and("name").equals(name);

        query.addCriteria(criteria);

        List<User> userList = mongoTemplate.find(query, User.class);

        long total = mongoTemplate.count(query, User.class);

        Pageable pageable = PageRequest.of(pageNum, pageSize);

        Page<User> userPage = new PageImpl(userList, pageable, total);
        return userPage;
    }
}
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-2-mongodb)

# 存在问题

MongoRepository 中 deleteById 和 MongoTemplate 中 deleteById, deleteAll 未生效，暂不知原因


