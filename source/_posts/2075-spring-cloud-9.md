---
title: Spring Cloud 学习 (九) Spring Security, OAuth2
date: 2019-06-22 10:30:00
updated: 2020-04-24 00:00:00
categories: [IT]
tags: [Microservices, Spring Cloud, Spring Security, OAuth2]
---

# Spring Security

Spring Security 是 Spring Resource 社区的一个安全组件。在安全方面，有两个主要的领域，一是“认证”，即你是谁；二是“授权”，即你拥有什么权限，Spring Security 的主要目标就是在这两个领域

# Spring OAuth2

OAuth2 是一个标准的授权协议，允许不同的客户端通过认证和授权的形式来访问被其保护起来的资源

OAuth2 协议在 Spring Resource 中的实现为 Spring OAuth2，Spring OAuth2 分为：OAuth2 Provider 和 OAuth2 Client

## OAuth2 Provider

OAuth2 Provider 负责公开被 OAuth2 保护起来的资源

OAuth2 Provider 需要配置代表用户的 OAuth2 客户端信息，被用户允许的客户端就可以访问被 OAuth2 保护的资源。OAuth2 Provider 通过管理和验证 OAuth2 令牌来控制客户端是否有权限访问被其保护的资源

另外，OAuth2 Provider 还必须为用户提供认证 API 接口。根据认证 API 接口，用户提供账号和密码等信息，来确认客户端是否可以被 OAuth2 Provider 授权。这样做的好处就是第三方客户端不需要获取用户的账号和密码，通过授权的方式就可以访问被 OAuth2 保护起来的资源

OAuth2 Provider 的角色被分为 Authorization Service (授权服务) 和 Resource Service (资源服务)，通常它们不在同一个服务中，可能一个 Authorization Service 对应多个 Resource Service

Spring OAuth2 需配合 Spring Security 一起使用，所有的请求由 Spring MVC 控制器处理,并经过一系列的 Spring Security 过滤器

在 Spring Security 过滤器链中有以下两个节点，这两个节点是向 Authorization Service 获取验证和授权的：

1. 授权节点：默认为 /oauth/authorize
1. 获取 Token 节点：默认为 /oauth/token

## OAuth2 Client

OAuth2 Client (客户端) 用于访问被 OAuth2 保护起来的资源


# 新建 spring-security-oauth2-server

## pom

```
<parent>
    <artifactId>spring-cloud-parent</artifactId>
    <groupId>com.karonda</groupId>
    <version>1.0.0</version>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>spring-security-oauth2-server</artifactId>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-oauth2</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
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
  port: 8081

  servlet:
    context-path: /uaa # User Account and Authentication

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8001/eureka/

spring:
  application:
    name: oauth2-server

  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/spring-security-auth2?useSSL=false
    username: root
    password: root

    hikari:
      maximum-pool-size: 20
      minimum-idle: 5

  jpa:
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    show-sql: true
    hibernate:
      ddl-auto: update

  main:
    allow-bean-definition-overriding: true
```

## 数据表

创建用户和角色及中间表：

```
DROP TABLE IF EXISTS role; 
CREATE TABLE role
(
    id bigint(20) NOT NULL AUTO_INCREMENT,
    name varchar(255) NOT NULL,
    PRIMARY KEY (id)
);


DROP TABLE IF EXISTS user; 
CREATE TABLE user 
(
    id bigint(20) NOT NULL AUTO_INCREMENT,
    password varchar(255) DEFAULT NULL, 
    username varchar(255) NOT NULL ,
    PRIMARY KEY (id ), 
    UNIQUE KEY (username)
);


DROP TABLE IF EXISTS user_role; 
CREATE TABLE user_role (
    user_id bigint(20) NOT NULL,
    role_id bigint(20) NOT NULL, 
    KEY (user_id), 
    KEY (role_id), 
    FOREIGN KEY (user_id) REFERENCES user (id),
    FOREIGN KEY (role_id) REFERENCES role (id)
);
````

OAuth2 Client 信息可以存储在数据库中，Spring OAuth2 已经设计好了数据表，且不可变，创建数据表的脚本：[schema.sql](https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/test/resources/schema.sql) (如果使用 MySQL 需要将 LONGVARBINARY 替换为 BLOB)

初始化数据

```
INSERT INTO role (name) VALUES ('ROLE_USER');
INSERT INTO role (name) VALUES ('ROLE_ADMIN');

INSERT INTO user (username, password) VALUES ('test', '123');

INSERT INTO user_role (user_id, role_id) VALUES (1, 1);
```

## Entity & Dao & Service

```
@Entity
public class Role implements GrantedAuthority {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false)
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    @Override
    public String getAuthority() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString(){
        return name;
    }
}

```

Role 实现了 GrantedAuthority 接口

```
@Entity
public class User implements UserDetails, Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false, unique = true)
    private String username;
    @Column
    private String password;
    @ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    @JoinTable(name = "user_role", joinColumns = @JoinColumn(name = "user_id", referencedColumnName = "id"),
            inverseJoinColumns = @JoinColumn(name = "role_id", referencedColumnName = "id"))
    private List<Role> authorities;

    public User(){

    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    @Override
    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    @Override
    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    public void setAuthorities(List<Role> authorities) {
        this.authorities = authorities;
    }

    @Override
    public boolean isAccountNonExpired(){
        return true;
    }

    @Override
    public boolean isAccountNonLocked(){
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired(){
        return true;
    }

    @Override
    public boolean isEnabled(){
        return true;
    }

}
```

User 实现了 UserDetails 接口，该接口是 Spring Security 认证信息的核心接口

```
public interface UserDao extends JpaRepository<User, Long> {
    User findByUsername(String username);
}
```

```
@Service
public class UserServiceDetail implements UserDetailsService {

    @Autowired
    private UserDao userDao;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userDao.findByUsername(username);
    }
}
```

UserServiceDetail 实现了 UserDetailsService 接口

## 启动类

```
@EnableEurekaClient
@SpringBootApplication
public class Oauth2ServerApp {

    public static void main(String[] args){
        SpringApplication.run(Oauth2ServerApp.class, args);
    }

}
```

## Spring Security 配置

```
@Configuration
@EnableWebSecurity // 开启 Spring Security
@EnableGlobalMethodSecurity(prePostEnabled = true) // 开启方法级别上的保护
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserServiceDetail userServiceDetail;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated() // 所有请求都需要安全验证
                .and()
                .csrf().disable();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userServiceDetail).passwordEncoder(passwordEncoder());
    }

    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}
```

**Spring Security 5 使用 Spring Security 4 的配置会报 There is no PasswordEncoder mapped for the id "null" 异常，解决方法是使用 NoOpPasswordEncoder (临时解决方案，非最优方案)**

*--- 2020-04-24 更新开始 ---*

PasswordEncoder 可以使用自定义 Encoder：

```
@Bean
public PasswordEncoder passwordEncoder() {
	return new MyPasswordEncoder();
}
```

MyPasswordEncoder 代码：

```
public class MyPasswordEncoder implements PasswordEncoder {

    @Override
    public String encode(CharSequence charSequence) {
        return PasswordUtil.getencryptPassword((String)charSequence);
    }

    @Override
    public boolean matches(CharSequence charSequence, String s) {
        boolean result = PasswordUtil.matches(charSequence, s);
        return result;
    }
}
```

PasswordUtil 代码：

```
public class PasswordUtil {

    private static final BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();

    public static String getencryptPassword(String password){
        return encoder.encode(password);
    }

    public static boolean matches(CharSequence rawPassword, String encodedPassword){
        return encoder.matches(rawPassword, encodedPassword);
    }
}
```



*--- 2020-04-24 更新结束 ---*

## Authorization Server 配置

```
@Configuration
@EnableAuthorizationServer // 开启授权服务
@EnableResourceServer // 需要对外暴露获取和验证 Token 的接口，所以也是一个资源服务
public class OAuth2Config extends AuthorizationServerConfigurerAdapter{

    @Autowired
    private DataSource dataSource;

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private UserServiceDetail userServiceDetail;

    @Override
    // 配置客户端信息
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory() // 将客户端的信息存储在内存中
                .withClient("browser") // 客户端 id, 需唯一
                .authorizedGrantTypes("refresh_token", "password") // 认证类型为 refresh_token, password
                .scopes("ui") // 客户端域
                .and()
                .withClient("eureka-client") // 另一个客户端
                .secret("123456")  // 客户端密码
                .authorizedGrantTypes("client_credentials", "refresh_token", "password")
                .scopes("server");
    }

    @Override
    // 配置授权 token 的节点和 token 服务
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenStore(tokenStore()) // token 的存储方式
                .authenticationManager(authenticationManager) // 开启密码验证，来源于 WebSecurityConfigurerAdapter
                .userDetailsService(userServiceDetail); // 读取验证用户的信息
    }

    @Override
    // 配置 token 节点的安全策略
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.tokenKeyAccess("permitAll()") // 获取 token 的策略
                .checkTokenAccess("isAuthenticated()");
    }

    @Bean
    public TokenStore tokenStore() {

//        return new InMemoryTokenStore();

        return new JdbcTokenStore(dataSource);
    }
}
```

## RemoteTokenServices 接口

```
@RestController
@RequestMapping("/users")
public class UserController {

    @RequestMapping(value = "/current", method = RequestMethod.GET)
    public Principal getUser(Principal principal){
        return principal;
    }
}
```

本文采用 RemoteTokenServices 这种方式对 Token 进行验证，如果其他资源服务需要验证 Token 则需要远程调用授权服务暴露的验证 Token 的 API 接口

## 测试

1. 启动 eureka-server
1. 启动 oauth2-server

使用 Postman 测试：

|  |  |  |
| ------ | ------ | ------ |
| - | POST | http://localhost:8081/uaa/oauth/token |
| Headers | | |
| - | Authorization | Basic ZXVyZWthLWNsaWVudDoxMjM0NTY= |
| Body | | |
| - | username | test |
| - | password | 123 |
| - | grant_type | password |


其中 Authorization 的值为 Basic clientId:secret (本文中为 eureka-client:123456) Base64 加密后的值

返回结果：

```
{
    "access_token": "5dc978ab-8c7e-4286-92f5-5655b8d15c98",
    "token_type": "bearer",
    "refresh_token": "7ef02b1c-6e8a-485f-adc9-18a48c2ae410",
    "expires_in": 43199,
    "scope": "server"
}
```


# eureka-client

## 添加依赖

```
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-oauth2</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
```

## application.xml 添加

```
security:
  oauth2:
    resource:
      user-info-uri: http://localhost:8081/uaa/users/current
    client:
      client-id: eureka-client
      client-secret: 123456
      access-token-uri: http://localhost:8081/uaa/oauth/token
      grant-type: client_credentials, password
      scope: server
```

数据库配置同 oauth2-server 未列出

## 配置 Resource Server

```
@Configuration
@EnableResourceServer // 开启资源服务
@EnableGlobalMethodSecurity(prePostEnabled = true) // 开启方法级别上的保护
public class ResourceServerConfigurer extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {

        http.authorizeRequests()
                .antMatchers("/user/register").permitAll()
                .anyRequest().authenticated();
    }
}
```

## 配置 OAuth2 Client

```
@EnableOAuth2Client // 开启 OAuth2 Client
@EnableConfigurationProperties
@Configuration
public class OAuth2ClientConfig {

    @Bean
    @ConfigurationProperties(prefix = "security.oauth2.client")
    // 配置受保护的资源信息
    public ClientCredentialsResourceDetails clientCredentialsResourceDetails(){
        return new ClientCredentialsResourceDetails();
    }

    @Bean
    // 过滤器，存储当前请求和上下文
    public RequestInterceptor oAuth2FeignRequestInterceptor(){
        return new OAuth2FeignRequestInterceptor(new DefaultOAuth2ClientContext(), clientCredentialsResourceDetails());
    }

    @Bean
    public OAuth2RestTemplate clientCredentialsRestTemplate (){
        return new OAuth2RestTemplate(clientCredentialsResourceDetails());
    }
}
```

## Entity & Dao & Service

与 oauth2-server 类似，具体见代码

## Controller

```
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping(value = "/register", method = RequestMethod.POST)
    public User createUser(@RequestParam("username") String username
            , @RequestParam("password") String password){
        return userService.create(username, password);
    }
}
```

```
@RestController
public class HiController {

    @Value("${server.port}")
    int port;

    @Value("${version}")
    String version;

    @GetMapping("/hi")
    public String home(@RequestParam String name){
        return "Hello " + name + ", from port: " + port + ", version: " + version;
    }

    @PreAuthorize("hasAuthority('ROLE_ADMIN')") // 需要权限
    @RequestMapping("/hello")
    public String hello(){
        return "hello!";
    }
}
```


## 测试

1. 启动 eureka-server
1. 启动 oauth2-server
1. 启动 config-server
1. 启动 eureka-client

使用 Postman 测试：

### 注册用户

|  |  |  |
| ------ | ------ | ------ |
| - | POST | localhost:8011/user/register |
| Body | | |
| - | username | admin |
| - | password | 123 |

返回结果：

```
{
    "id": 2,
    "username": "admin",
    "password": "123",
    "authorities": null,
    "enabled": true,
    "accountNonExpired": true,
    "credentialsNonExpired": true,
    "accountNonLocked": true
}
```

### 请求 token

|  |  |  |
| ------ | ------ | ------ |
| - | POST | http://localhost:8081/uaa/oauth/token |
| Headers | | |
| - | Authorization | Basic ZXVyZWthLWNsaWVudDoxMjM0NTY= |
| Body | | |
| - | username | admin |
| - | password | 123 |
| - | grant_type | password |

```
{
    "access_token": "dc4959fb-9ced-430e-9a78-5e9609c3baac",
    "token_type": "bearer",
    "refresh_token": "06cdcf55-fe6a-4367-94e1-051c2da86e37",
    "expires_in": 43199,
    "scope": "server"
}
```

### 访问不需要权限的接口

|  |  |  |
| ------ | ------ | ------ |
| - | GET | localhost:8011/hi?name=Victor |
| Headers | | |
| - | Authorization | Bearer dc4959fb-9ced-430e-9a78-5e9609c3baac |

```
Hello Victor, from port: 8011, version: 1.0.2
```

### 访问需要权限的接口

|  |  |  |
| ------ | ------ | ------ | 
| - | GET | localhost:8011/hello |
| Headers | | |
| - | Authorization | Bearer dc4959fb-9ced-430e-9a78-5e9609c3baac |

 ```
{
    "error": "access_denied",
    "error_description": "不允许访问"
}
 ```

 手动授权：

 ```
INSERT INTO user_role (user_id, role_id) VALUES (2, 2);
 ```

重新获取 token 后再次访问接口

 ```
hello!
 ```





完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-cloud-parent)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

