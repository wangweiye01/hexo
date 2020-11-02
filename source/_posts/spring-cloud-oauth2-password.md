---
title: Spring Cloud OAuth2 实现密码模式认证
date: 2020-10-15 09:19:00
tags:
---

OAuth2有四种授权模式 授权码模式(authorization code) 简化模式(implicit) 密码模式(resouce owner password credentials) 客户端模式，具体理解OAuth2可以参考[阮一峰文章](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)

今天我们来实现OAuth2的密码模式

# 使用场景

我们在日常生活中经常会出现微信，微博等第三方登录的场景。我们在使用这些第三方登录时，不需要注册，直接授权登录即可，非常快捷方便。对于开发者来说，也不需要存储用户的用户名密码，只需要存储第三方平台的唯一标识即可

如果我们自己来实现这种第三方的服务作为认证中心，其他服务就可共用该认证中心，实现登录的功能。并且只要一次登录，便可在多个服务中自由通行

# 密码模式流程

密码模式的实现流程图如下：

1. 用户向客户端提供用户名和密码
2. 客户端将用户名和密码发送给认证服务器，向后者请求令牌
3. 认证服务器确认无误后，向客户端提供访问令牌

![bg2014051206-password.png](https://p.130014.xyz/2020/10/15/bg2014051206-password.png)

在微服务流行的今天，一个电商平台的背后可能是由多个服务构成，比如订单服务、用户服务等。要想用户登录之后可以访问任意微服务，一定需要携带一个凭证，来标识自己的身份。此处的凭证就是OAuth2中的access_token

# 实现

## 一 认证服务端

### 1.引入maven

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

### 2.配置application.yml

```
spring:
  application:
    name: oauth2server
  datasource:
    url: jdbc:mysql://localhost:3306/oauth2server?characterEncoding=UTF-8&useSSL=false
    username: root
    password: abc123
    hikari:
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      maximum-pool-size: 9

server:
  port: 8001

jwt:
  signKey: wangweiye
```

### 3.spring security基础配置

```
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    // 允许匿名访问 oatuh 接口
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin().and().authorizeRequests().antMatchers(HttpMethod.POST, "/oauth/token").permitAll();
    }
}
```

这个类的重点是声明`PasswordEncoder`和`AuthenticationManager`两个Bean. `PasswordEncoder`是一个密码加密工具，可以实现不可逆的加密，`AuthenticationManager`是为了实现OAuth2的password模式必须指定的授权管理Bean

### 4.实现UserDetailsService

```
@Component
public class CustomUserDetailsService implements UserDetailsService {
    private static Logger log = LoggerFactory.getLogger(CustomUserDetailsService.class);

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        log.info(String.format("username is: %s", username));

        // 查询数据库操作
        if (!username.equals("admin")) {
            throw new UsernameNotFoundException("the user is not found");
        } else {
            // 用户角色也应在数据库中获取
            String role = "ROLE_ADMIN";
            List<SimpleGrantedAuthority> authorities = new ArrayList<>();
            authorities.add(new SimpleGrantedAuthority(role));

            // 线上环境应该通过用户名查询数据库获取加密后的密码
            String password = passwordEncoder.encode("123456");

            // 返回自定义的CustomUserDetailService
            User user = new User(username, password, authorities);
            return user;
        }
    }
}
```

核心是`loadUserByUsername`方法，接收一个字符串用户名，然后返回一个`UserDetails`对象。在生产环境中，此处的用户名和密码都需要在数据库中查出，此处为了便于举例，写死为用户名`admin`，密码`123456`

### 5.OAuth2配置文件

```
@Configuration
@EnableAuthorizationServer
public class OAuth2Config extends AuthorizationServerConfigurerAdapter {
    @Autowired
    private TokenEnhancer jwtTokenEnhancer;

    @Autowired
    private JwtAccessTokenConverter jwtAccessTokenConverter;

    @Autowired
    private TokenStore jwtTokenStore;

    @Autowired
    private UserDetailsService customUserDetailsService;

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private DataSource dataSource;

    @Override
    public void configure(final AuthorizationServerEndpointsConfigurer endpoints) {
        // jwt增强模式
        TokenEnhancerChain enhancerChain = new TokenEnhancerChain();
        List<TokenEnhancer> enhancerList = new ArrayList<>();
        enhancerList.add(jwtTokenEnhancer);
        enhancerList.add(jwtAccessTokenConverter);
        enhancerChain.setTokenEnhancers(enhancerList);
        endpoints.tokenStore(jwtTokenStore)
                .userDetailsService(customUserDetailsService)
                .authenticationManager(authenticationManager)
                .tokenEnhancer(enhancerChain)
                .accessTokenConverter(jwtAccessTokenConverter);
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.jdbc(dataSource);
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.allowFormAuthenticationForClients();
        security.checkTokenAccess("isAuthenticated()");
        security.tokenKeyAccess("isAuthenticated()");
    }
}
```

此处重写了三个`configure`方法

`AuthorizationServerEndpointsConfigurer`参数的重写：

`authenticationManage()` 调用此方法才能支持 password 模式

`userDetailsService()` 设置用户验证服务

`tokenStore()` 指定token的存储方式

`ClientDetailsServiceConfigurer`参数的重写：

`clients.jdbc(dataSource)` 是指客户端的管理有dataSource数据库来管理。数据库脚本已放入文末地址中。其中`authorized_grant_types`列可以填写的内容有`authorization_code`: 授权码模式, `implicit`: 隐式授权类型, `password`: 资源所有者密码类型, `client_credentials`: 客户端凭据(客户端ID以及Key)类型, `refresh_token`: 通过以上授权获得的刷新令牌来获取新的令牌

### 6.增强JWT

如果需求需要在jwt中添加额外字段，可以使用`TokenEnhancer`增强器

```
public class JWTokenEnhancer implements TokenEnhancer {
    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken oAuth2AccessToken, OAuth2Authentication oAuth2Authentication) {
        Map<String, Object> info = new HashMap<>();
        info.put("jwt-ext", "JWT 扩展信息");
        ((DefaultOAuth2AccessToken) oAuth2AccessToken).setAdditionalInformation(info);
        return oAuth2AccessToken;
    }
}
```

## 二 客户端

