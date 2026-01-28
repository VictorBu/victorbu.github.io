---
title: Spring Cloud 学习 (十) Spring Security, OAuth2, JWT
date: 2019-06-23 10:30:00
updated: 2019-06-23 10:30:00
categories: [IT]
tags: [Microservices, Spring Cloud, Spring Security, OAuth2, JWT]
---

通过 Spring Security + OAuth2 认证和鉴权，每次请求都需要经过 OAuth Server 验证当前 token 的合法性，并且需要查询该 token 对应的用户权限，在高并发场景下会存在性能瓶颈。使用 JWT 的方式，OAuth Server 只验证一次，用户所有信息 (包括权限) 包含在返回的 JWT 中

# 准备工作

生成公钥、私钥

## 私钥

在控制台输入命令：

```
keytool -genkeypair -alias spring-jwt -validity 3650 -keyalg RSA -dname "CN=Victor,OU=Karonda,O=Karonda,L=Shenzhen,S=Guangdong,C=CN" -keypass abc123 -storepass abc123 -keystore spring-jwt.jks
```

各个参数的含义，可以通过命令查看：

```
keytool -genkeypair -help
```

其中 DName 各个参数代表的意义见： [X.500 Distinguished Names](https://docs.oracle.com/javase/7/docs/technotes/tools/solaris/keytool.html#DName)

## 公钥

在控制台输入命令：

```
keytool -list -rfc --keystore spring-jwt.jks | openssl x509 -inform pem -pubkey
```

会提示输入密码，密码为生成私钥命令里设置的密码

本文生成的公钥：

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA2HvMsVqrx60ESp30Ymx7
3Ce2h24QvG9rciDPl8+SxXRz79akmdRCB4HFhBb655aVAnQMj4SGzKcMyofOUt3o
X9tOPz3Y/B/D5viI3cNPYinyFVMawganROsM1meTFR1SPpL/kZUZqLm9pc8lpgat
LtU73ryioVe7FFndce6ZwTe24L4rK0jzseQ24FxoEQ+g0B1DCXZ4Gi9PwBpxWL6W
AG+/NEFFtOGtIJSIwCYzhGqDfyNaOt7JXYwGiWgh0npO3JVvgQVXBW9AdpT5JVSb
ScYktkqY3o0htsSueyne+FbS+OwBVaBewcswPVbEwa6dxtb0vBsp3pNiSdg7rDea
1QIDAQAB
-----END PUBLIC KEY-----
```

新建 public.cert 文件保存上面生成的公钥 (要包含公钥的完整信息，即 BEGIN PUBLIC KEY 和 END PUBLIC KEY 部分也要包含在文件中)

> Windows 系统需要先安装 OpenSSL: [下载链接](http://slproweb.com/products/Win32OpenSSL.html)


将私钥和公钥分别拷贝到 oauth2-server 和 eureka-client 的 resources 目录下

并在 oauth2-server 和 eureka-client 的 pom 添加配置：

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <configuration>
        <nonFilteredFileExtensions>
            <nonFilteredFileExtension>cert</nonFilteredFileExtension>
            <nonFilteredFileExtension>jks</nonFilteredFileExtension>
        </nonFilteredFileExtensions>
    </configuration>
</plugin>
```

因为密钥文件不需要编译

# oauth2-server

## 修改 Authorization Server 配置

```
@Configuration
@EnableAuthorizationServer // 开启授权服务
@EnableResourceServer // 需要对外暴露获取和验证 Token 的接口，所以也是一个资源服务
public class OAuth2Config extends AuthorizationServerConfigurerAdapter{

    @Autowired
    private AuthenticationManager authenticationManager;

    @Override
    // 配置客户端信息
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory() // 将客户端的信息存储在内存中
                .withClient("eureka-client") // 客户端
                .secret("123456")  // 客户端密码
                .authorizedGrantTypes("client_credentials", "refresh_token", "password")
                .accessTokenValiditySeconds(3600) // 设置 token 过期时间
                .scopes("server");
    }

    @Override
    // 配置授权 token 的节点和 token 服务
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenStore(tokenStore()) // token 的存储方式
                .authenticationManager(authenticationManager) // 开启密码验证，来源于 WebSecurityConfigurerAdapter
//                .userDetailsService(userServiceDetail); // 读取验证用户的信息
                .tokenEnhancer(jwtTokenEnhancer());
    }

    @Bean
    public TokenStore tokenStore() {

//        return new InMemoryTokenStore();

//        return new JdbcTokenStore(dataSource);

        return new JwtTokenStore(jwtTokenEnhancer());
    }

    @Bean
    protected JwtAccessTokenConverter jwtTokenEnhancer(){
        KeyStoreKeyFactory keyStoreKeyFactory =
                new KeyStoreKeyFactory(new ClassPathResource("spring-jwt.jks")
                        , "abc123".toCharArray()); // abc123 为 password
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setKeyPair(keyStoreKeyFactory.getKeyPair("spring-jwt")); // spring-jwt 为 alias
        return converter;
    }
}

```

# eureka-client

上一篇文章中的 OAuth2 Client 配置本文用不到，要移除

## Resource Server 配置

### 配置 JWT 转换器

```
@Configuration
public class JwtConfig {

    @Autowired
    JwtAccessTokenConverter jwtAccessTokenConverter;

    @Bean
    public TokenStore tokenStore(){
        return new JwtTokenStore(jwtAccessTokenConverter);
    }

    @Bean
    protected JwtAccessTokenConverter jwtTokenEnhancer(){
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        Resource resource = new ClassPathResource("public.cert"); // 公钥

        String publicKey;
        try{
            publicKey = new String(FileCopyUtils.copyToByteArray(resource.getInputStream()));
        }catch (IOException e){
            throw new RuntimeException(e);
        }

        converter.setVerifierKey(publicKey);

        return converter;
    }
}
```

### 修改 Resource Server 配置

```
@Configuration
@EnableResourceServer // 开启资源服务
@EnableGlobalMethodSecurity(prePostEnabled = true) // 开启方法级别上的保护
public class ResourceServerConfigurer extends ResourceServerConfigurerAdapter {

    @Autowired
    TokenStore tokenStore;

    @Override
    public void configure(HttpSecurity http) throws Exception {

        http.authorizeRequests()
                .antMatchers("/user/login", "/user/register").permitAll()
                .anyRequest().authenticated();
    }

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.tokenStore(tokenStore);
    }
}
```

## JWT 类

```
public class JWT {
    private String access_token;
    private String token_type;
    private String refresh_token;
    private int expires_in;
    private String scope;
    private String jti;

    public String getAccess_token() {
        return access_token;
    }

    public void setAccess_token(String access_token) {
        this.access_token = access_token;
    }

    public String getToken_type() {
        return token_type;
    }

    public void setToken_type(String token_type) {
        this.token_type = token_type;
    }

    public String getRefresh_token() {
        return refresh_token;
    }

    public void setRefresh_token(String refresh_token) {
        this.refresh_token = refresh_token;
    }

    public int getExpires_in() {
        return expires_in;
    }

    public void setExpires_in(int expires_in) {
        this.expires_in = expires_in;
    }

    public String getScope() {
        return scope;
    }

    public void setScope(String scope) {
        this.scope = scope;
    }

    public String getJti() {
        return jti;
    }

    public void setJti(String jti) {
        this.jti = jti;
    }

    @Override
    public String toString() {
        return "JWT{" +
                "access_token='" + access_token + '\'' +
                ", token_type='" + token_type + '\'' +
                ", refresh_token='" + refresh_token + '\'' +
                ", expires_in=" + expires_in +
                ", scope='" + scope + '\'' +
                ", jti='" + jti + '\'' +
                '}';
    }
}
```

## Feign 客户端

```
@FeignClient("oauth2-server")
public interface AuthServiceClient {

    @PostMapping("/uaa/oauth/token")
    JWT getToken(@RequestHeader(value = "Authorization") String authorization, @RequestParam("grant_type") String type,
                 @RequestParam("username") String username, @RequestParam("password") String password);
}
```

同时需要在启动类添加注解：

```
@EnableFeignClients
```

## DTO 及异常处理

```
public class UserLoginDTO {

    private JWT jwt;
    private User user;

    public JWT getJwt() {
        return jwt;
    }

    public void setJwt(JWT jwt) {
        this.jwt = jwt;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}
```

```
public class UserLoginException extends RuntimeException {

    public UserLoginException(String message){
        super(message);
    }
}
```

```
@ControllerAdvice // 表明该类是异常统一处理类
@ResponseBody
public class ExceptionHandle {

    @ExceptionHandler(UserLoginException.class)
    public ResponseEntity<String> handleException(Exception e){
        return new ResponseEntity(e.getMessage(), HttpStatus.OK);
    }
}
```

## service & controller

添加登录方法

```
    @Override
    public UserLoginDTO login(String username, String password) {
        User user = userDao.findByUsername(username);
        if(null == user){
            throw new UserLoginException("error username");
        }
        if(!password.equals(user.getPassword())){
            throw new UserLoginException("erro password");
        }

        JWT jwt = authServiceClient.getToken("Basic ZXVyZWthLWNsaWVudDoxMjM0NTY="
                , "password", username, password); // ZXVyZWthLWNsaWVudDoxMjM0NTY= 为 eureka-client:123456 Base64 加密后的值
        if(null == jwt){
            throw new UserLoginException("error internal");
        }

        UserLoginDTO userLoginDTO = new UserLoginDTO();
        userLoginDTO.setJwt(jwt);
        userLoginDTO.setUser(user);

        return userLoginDTO;
    }
```

```
    @RequestMapping(value = "/login", method = RequestMethod.POST)
    public UserLoginDTO login(@RequestParam("username") String username
            , @RequestParam("password") String password){
        return userService.login(username, password);
    }
```

# 测试

1. 启动 eureka-server
1. 启动 oauth2-server
1. 启动 config-server
1. 启动 eureka-client

先取消授权：

 ```
DELETE FROM user_role WHERE user_id = 2;
 ```

 使用 Postman 测试：

## 用户登录

|  |  |  |
| ------ | ------ | ------ |
| - | POST | localhost:8011/user/login |
| Body | | |
| - | username | admin |
| - | password | 123 |

```
{
    "jwt": {
        "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NjEyNjUyMzgsInVzZXJfbmFtZSI6ImFkbWluIiwianRpIjoiNTQxMDk3MDItNTU1Yy00NjA4LThkYzYtNzU3NWZjNGIwNGIyIiwiY2xpZW50X2lkIjoiZXVyZWthLWNsaWVudCIsInNjb3BlIjpbInNlcnZlciJdfQ.P6dtT76bFyQ6aF7-v6Vphi3ivLR0x4w739gwmBRGujaRpfDMjwQHCn5REyxEOAKdoxrVT__v73qcb78_8Ovb97L13ztnzdlPmLYzcAkQdMFz78yAjZIp2VtzxZ87Ecmk9f6-bIRlBxS9A24t0y4Tp1gkPITB1vxod0FewAHCsUJQ9WqLNeW9bxzZvy5DtlJlCCY7lOIjfDxlQdXygpwznZ4rIHv-O-eOr2aqcKMLZhdtW7hHsy2JccIUm1ZdpVQfUMD7XzWFAQoZYFLc0oXyVL0nFasOr-Ne1UR1iZYI4cS-ONVLMe78erVb-zRoyTAhEb7Pkyepkwm_Xv23U2CoeA",
        "token_type": "bearer",
        "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NjM4NTM2MzgsInVzZXJfbmFtZSI6ImFkbWluIiwianRpIjoiODhjYTQ0NjktMTIyNi00ZTFkLTlhMDktZDZlMTdhOTMyYzAzIiwiY2xpZW50X2lkIjoiZXVyZWthLWNsaWVudCIsInNjb3BlIjpbInNlcnZlciJdLCJhdGkiOiI1NDEwOTcwMi01NTVjLTQ2MDgtOGRjNi03NTc1ZmM0YjA0YjIifQ.RdBDYKhZJz5DntxK_4np1B4phnalT37srjycUUmCHVZ0BB4lEWAIT5YLlY7ZaaVM2AAhbeb1WO1dhlmvtmlkd8W6lowbtMeyMYqrKcbn1tYavLwZDHKWSHGiUW1bXivngwhixCqLwK0AA8Oe-9-ohC-c6G7cRN4r6bWkc4WiadlErg6MS7N6VGdQj26SgPVmTqvVhpm5mnzGJyM66d-kxneHyRjPVli1DFyxuUl8oRCTTFuamybXmD_niWCA-isDgF7loJFV6hMjoow6-3uK9rLthMADIM4YqAp8T8eGsup_7hIICwT7qUhOdzBjwsuX8ond3iu09322LsPEoTTXlg",
        "expires_in": 3599,
        "scope": "server",
        "jti": "54109702-555c-4608-8dc6-7575fc4b04b2"
    },
    "user": {
        "id": 2,
        "username": "admin",
        "password": "123",
        "authorities": [],
        "enabled": true,
        "credentialsNonExpired": true,
        "accountNonExpired": true,
        "accountNonLocked": true
    }
}
```


## 访问不需要权限的接口

|  |  |  |
| ------ | ------ | ------ |
| - | GET | localhost:8011/hi?name=Victor |
| Headers | | |
| - | Authorization | Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NjEyNjUyMzgsInVzZXJfbmFtZSI6ImFkbWluIiwianRpIjoiNTQxMDk3MDItNTU1Yy00NjA4LThkYzYtNzU3NWZjNGIwNGIyIiwiY2xpZW50X2lkIjoiZXVyZWthLWNsaWVudCIsInNjb3BlIjpbInNlcnZlciJdfQ.P6dtT76bFyQ6aF7-v6Vphi3ivLR0x4w739gwmBRGujaRpfDMjwQHCn5REyxEOAKdoxrVT__v73qcb78_8Ovb97L13ztnzdlPmLYzcAkQdMFz78yAjZIp2VtzxZ87Ecmk9f6-bIRlBxS9A24t0y4Tp1gkPITB1vxod0FewAHCsUJQ9WqLNeW9bxzZvy5DtlJlCCY7lOIjfDxlQdXygpwznZ4rIHv-O-eOr2aqcKMLZhdtW7hHsy2JccIUm1ZdpVQfUMD7XzWFAQoZYFLc0oXyVL0nFasOr-Ne1UR1iZYI4cS-ONVLMe78erVb-zRoyTAhEb7Pkyepkwm_Xv23U2CoeA |

```
Hello Victor, from port: 8011, version: 1.0.2
```

## 访问需要权限的接口

|  |  |  |
| ------ | ------ | ------ | 
| - | GET | localhost:8011/hello |
| Headers | | |
| - | Authorization | Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NjEyNjUyMzgsInVzZXJfbmFtZSI6ImFkbWluIiwianRpIjoiNTQxMDk3MDItNTU1Yy00NjA4LThkYzYtNzU3NWZjNGIwNGIyIiwiY2xpZW50X2lkIjoiZXVyZWthLWNsaWVudCIsInNjb3BlIjpbInNlcnZlciJdfQ.P6dtT76bFyQ6aF7-v6Vphi3ivLR0x4w739gwmBRGujaRpfDMjwQHCn5REyxEOAKdoxrVT__v73qcb78_8Ovb97L13ztnzdlPmLYzcAkQdMFz78yAjZIp2VtzxZ87Ecmk9f6-bIRlBxS9A24t0y4Tp1gkPITB1vxod0FewAHCsUJQ9WqLNeW9bxzZvy5DtlJlCCY7lOIjfDxlQdXygpwznZ4rIHv-O-eOr2aqcKMLZhdtW7hHsy2JccIUm1ZdpVQfUMD7XzWFAQoZYFLc0oXyVL0nFasOr-Ne1UR1iZYI4cS-ONVLMe78erVb-zRoyTAhEb7Pkyepkwm_Xv23U2CoeA |

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

重新登录后再次访问接口

 ```
hello!
 ```





完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-cloud-parent)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

