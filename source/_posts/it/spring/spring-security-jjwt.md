---
title: Spring Security + JJWT 实现 JWT 认证和授权
date: 2020-06-21 11:00:00
updated: 2020-06-21 11:00:00
categories: [IT]
tags: [Spring Boot, Spring Security, JJWT, JWT]
---

关于 JJWT 的使用，可以参考之前的文章：[JJWT 使用示例](https://www.cnblogs.com/victorbu/p/12758037.html)

# 一、鉴权过滤器

```
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,
                                    FilterChain filterChain) throws ServletException, IOException {
        String token = httpServletRequest.getHeader(HttpHeaders.AUTHORIZATION);
        if(!StringUtils.isEmpty(token)) {
            UserInfoModel userInfo = JwtUtil.verifyToken(token, JwtConstant.AIM_ACCESS);
            if(userInfo != null) {
                UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userInfo, null, userInfo.getAuthorities());
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(httpServletRequest));
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        }

        filterChain.doFilter(httpServletRequest, httpServletResponse);
    }
}
```

# 二、Spring Security 配置

```
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;
    @Autowired
    private EntryPointUnauthorizedHandler entryPointUnauthorizedHandler;
    @Autowired
    private RestAccessDeniedHandler restAccessDeniedHandler;

    @Bean
    GrantedAuthorityDefaults grantedAuthorityDefaults() {
        // 去除 ROLE_ 前缀
        return new GrantedAuthorityDefaults("");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/login").permitAll()
                .anyRequest().authenticated()
                .and().cors()
                .and().csrf().disable();
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
        http.exceptionHandling()
                .authenticationEntryPoint(entryPointUnauthorizedHandler)
                .accessDeniedHandler(restAccessDeniedHandler);
        // 不创建会话
        http.sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }
}
```

其中 EntryPointUnauthorizedHandler 和 RestAccessDeniedHandler 是未认证或未授权异常处理，详细代码可以看源码

# 三、获取当前登录用户

```
public class CurrentUserUtil {

    public static UserInfoModel getUserInfo() {

        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if(authentication != null) {
            if(authentication instanceof UsernamePasswordAuthenticationToken) {
                return (UserInfoModel)authentication.getPrincipal();
            }
        }

        return null;
    }
}
```

测试代码见 TestController，测试时在请求 Header 中添加 Authorization 即可


完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-security-jjwt)

参考：

1. [Spring Security和JWT实现登录授权认证](https://cloud.tencent.com/developer/article/1447720)
1. [How do I remove the ROLE_ prefix from Spring Security with JavaConfig?](https://stackoverflow.com/questions/38134121/how-do-i-remove-the-role-prefix-from-spring-security-with-javaconfig)