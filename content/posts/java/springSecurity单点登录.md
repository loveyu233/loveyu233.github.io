---
title: "SpringSecurity单点登录"
date: 2019-11-29T11:31:34+08:00
author: ["loveyu"]
draft: false
categories: 
- java
tags: 
- java
- springSecurity
- sso
---

# Spring Security

## pom.xml：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.4</version>
    <relativePath/>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.4.2</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
    </dependency>
    <dependency>
        <groupId>org.thymeleaf.extras</groupId>
        <artifactId>thymeleaf-extras-springsecurity5</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
</dependencies>
```

>thymeleaf-extras-springsecurity5：thymeleaf和Spring security的整合jar包

## application.yml：

```yml
spring:
  datasource:
    username: root
    password: Hzyy1214
    driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springSecurityTest?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
```

>配置数据库

## java

### 目录结构

```tex
<B>config

	<C>securityConfig.java

<B>controller

	<C>UserController.java

<B>>hander

	<C>MyAccessDeniedHandler.java

	<C>MyAuthenticationFailureHandle.java

	<C>MyAuthenticationSuccessHandle.java

<B>mapper

	<C>RoleMapper.java

	<C>UserMapper.java

<B>pojo

	<I>Role.java

	<I>Users.java

<B>service

	<I>MyAccess.java

	<C>MyAccessImpl.java

	<C>RoleService.java

	<C>UserService.java

	<C>UserDetailServiceImpl.java
```

><B>包；<I>接口<C>普通类

### pojo

> 对应数据库的实体类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Role {
    private String uid;
    private String roleName;
    private String roleDesc;
}
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Users {
    private Integer id;
    @TableId(type = IdType.ASSIGN_UUID)
    private String uid;
    private String username;
    private String password;
}
```

>@TableId(type = IdType.ASSIGN_UUID)：使用uid

### mapper

> 使用mybatisPlus操作数据库

```java
@Mapper
public interface RoleMapper extends BaseMapper<Role> {
}
```

```java
@Mapper
public interface UsersMapper extends BaseMapper<Users> {
}
```

### service

> 查询用户表的用户uid，username，paddword信息，再根据uid查询角色表对应用户角色权限

```java
@Service
public class RoleService {
    @Autowired
    private RoleMapper roleMapper;

    public List<Role> selectRole(String uid) {
        return roleMapper.selectList(new QueryWrapper<Role>().eq("uid",uid));
    }
}
```

```java
@Service
public class UsersService {
    @Autowired
    private UsersMapper usersMapper;

    public Users selectUsername(String username) {
        return usersMapper.selectOne(new QueryWrapper<Users>().eq("username", username));
    }
}
```

 自定义登录认证逻辑 

```java
@Service
@Slf4j
public class UserDetailServiceImpl implements UserDetailsService {
    @Autowired
    private UsersService usersService;
    @Autowired
    private RoleService roleService;

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        //根据用户名去数据库查询，不存在就抛异常UsernameNotFoundException
        Users user = usersService.selectUsername(s);
        if (user   null) {
            throw new UsernameNotFoundException("用户名不存在");
        }
        List<Role> roles = roleService.selectRole(user.getUid());
      	//创建list集合用来存储用户的权限角色
        ArrayList<SimpleGrantedAuthority> authorities = new ArrayList<>();
        if (roles.size() >= 1) {
            for (Role role : roles) {
                authorities.add(new SimpleGrantedAuthority(role.getRoleName()));
            }
        }
        log.info("执行自定义登录逻辑");
        log.info("角色权限:" + authorities);
      	//这个User是Security包下的User，第一个参数username，第二个参数passowrd(已加密过的)，第三个参数角色权限
        return new User(s, user.getPassword(), authorities);
    }
}
```

> 自定义access 

```java
public interface MyAccess {
    boolean hasPermission(HttpServletRequest request, Authentication authentication);
}
```

```java
@Service
public class MyAccessImpl implements MyAccess {
    @Override
    public boolean hasPermission(HttpServletRequest request, Authentication authentication) {
        Object obj = authentication.getPrincipal();
        if (obj instanceof UserDetails) {
            UserDetails userDetails = (UserDetails) obj;
            Collection<? extends GrantedAuthority> authorities = userDetails.getAuthorities();
            return authorities.contains(new SimpleGrantedAuthority(request.getRequestURI()));
        }
        return false;
    }
}
```

### config

> 设置访问权限 

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private MyAccessDeniedHandler myAccessDeniedHandler;
    @Autowired
    private DataSource dataSource;
    @Autowired
    private PersistentTokenRepository persistentTokenRepository;
    @Autowired
    private UserDetailServiceImpl userDetailService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                //登录页面
                .loginPage("/toLogin")
                //登录表单提交的求情
                .loginProcessingUrl("/login")
                //登录成功跳转页面，这个是一个请求必须是post
//                .successForwardUrl("/index")
                //自定义登录成功的处理器
                .successHandler(new MyAuthenticationSuccessHandler("/index.html"))
                //登录错误跳转页面，这个是一个请求必须是post
//                .failureForwardUrl("/toError")
                //自定义登录失败的处理器
                .failureHandler(new MyAuthenticationFailureHandler("/error.html"))
                //自定义表单的name，默认就是username和password
//                .usernameParameter("username")
//                .passwordParameter("password")
                .and()
                .logout()
                .logoutUrl("/logout")
                .logoutSuccessUrl("/toLogin")
                .and()
                //认证
                .authorizeRequests()
                //登录页面和错误页面放行不拦截
                .antMatchers("/loginPage.html","/error.html","/toLogin")
                .permitAll()
                //权限访问,权限是随便的比如lv1,lv2等,可以设置多个权限或角色比如既是lv1又是lv2
                .antMatchers("/lv1.html").hasAnyAuthority("lv1")
                .antMatchers("/lv2.html").hasAnyAuthority("lv2")
                .antMatchers("/lv3.html").hasAnyAuthority("lv3")
                //角色访问必须是ROLE_xxx形式
                .antMatchers("/lv4.html").hasRole("hzyy")
                //ip地址
                .antMatchers("/ip.html").hasIpAddress("127.0.0.1")
                //自定义access,使用自定义的access就不能使用anyxxx的方法
//                .anyRequest().access("@myAccessImpl.hasPermission(request,authentication)")
                //其余所有请求
                .anyRequest()
                //需要认证
                .authenticated()
                .and()
                .exceptionHandling()
                .accessDeniedHandler(myAccessDeniedHandler);
        http.rememberMe()
                .tokenRepository(persistentTokenRepository)
                //默认就是remember-me
                //.rememberMeParameter("remember-me")
                .tokenValiditySeconds(60)
                .userDetailsService(userDetailService);
    }


    /**
     * 密码加密和匹配
     * @return 加密
     */
    @Bean
    public PasswordEncoder getPW() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public PersistentTokenRepository persistentTokenRepository() {
        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        jdbcTokenRepository.setDataSource(dataSource);
        //第一次需要打开会自动创建一张表
        //jdbcTokenRepository.setCreateTableOnStartup(true);
        return jdbcTokenRepository;
    }
}
```

### handler

>没有权限跳转的页面也就是403页面

```java
@Component
public class MyAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AccessDeniedException e) throws IOException, ServletException {
        httpServletResponse.setStatus(HttpServletResponse.SC_ACCEPTED);
        httpServletResponse.setHeader("Content-Type","application/json;charset=utf-8");
        PrintWriter writer = httpServletResponse.getWriter();
        writer.write("{\"status\":\"error403\",\"msg:权限不足\"}");
        writer.flush();
        writer.close();
    }
}
```

>自定义登录失败的逻辑，也可以使用自带的

```java
public class MyAuthenticationFailureHandler implements AuthenticationFailureHandler {
    private String url;

    public MyAuthenticationFailureHandler(String url) {
        this.url = url;
    }

    @Override
    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
        httpServletResponse.sendRedirect(url);
    }
}
```

>自定义登录成功的逻辑，也可以使用自带的

```java
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    private String url;

    public MyAuthenticationSuccessHandler(String url) {
        this.url = url;
    }

    @Override
    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
        User user = (User) authentication.getPrincipal();
        /**
         * org.springframework.security.core.userdetails.User
         * [Username=hzyy, Password=[PROTECTED], Enabled=true,
         * AccountNonExpired=true, credentialsNonExpired=true,
         * AccountNonLocked=true, Granted Authorities=[lv2, lv3]]
         */
        log.info("security的user:" + user);
        log.info("ip:" + httpServletRequest.getRemoteAddr());
        httpServletResponse.sendRedirect(url);
    }
}
```

### Other

```java
@EnableGlobalMethodSecurity(securedEnabled = true,prePostEnabled = true,proxyTargetClass = true,jsr250Enabled = true)
@SpringBootApplication
public class SpringSecuritySeniorApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringSecuritySeniorApplication.class, args);
    }

}
```

> 启动类使用注解@EnableGlobalMethodSecurity开启security的注解使用 ，参数有四个分别对应一个注解，最好用的一个：
>
>prePostEnabled：对应注解为@PreAuthorize，参数为表达式，可以设置权限角色
>
>```java
>@PreAuthorize("hasAnyAuthority('lv1')")
>@RequestMapping("/toAnn")
>public String annotation() {
>return "redirect:annotation.html";
>}
>```

> 密码加密，使用的security默认的加密方式 
>
>PasswordEncoder是个接口，但在config里面已经进行了bean注入
>
>```java
>@Bean
>public PasswordEncoder passwordEncoder() {
>return new BCryptPasswordEncoder();
>}
>```

```java
@Autowired
private PasswordEncoder passwordEncoder;

@Test
void contextLoads() {
    System.out.println(passwordEncoder.encode("hahha"));
}
```

> 自定义错误页面的展示 

```java
public class ErrorPageConfig implements ErrorPageRegistrar {
    @Override
    public void registerErrorPages(ErrorPageRegistry registry) {
        ErrorPage page404 = new ErrorPage(HttpStatus.NOT_FOUND, "/404");
        ErrorPage page403 = new ErrorPage(HttpStatus.FORBIDDEN, "/403");
        ErrorPage page500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/500");
        registry.addErrorPages(page403,page404,page500);
    }
}
```

```java
@Configuration
public class ErrorBean {
    @Bean
    public ErrorPageConfig errorPageConfig (){
        return new ErrorPageConfig();
    }
}
```



> JsonWebToken

# JWT

## pom.xml

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

## java

```java
package com.xiaoyu;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.impl.Base64Codec;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import sun.misc.BASE64Decoder;

import java.util.Date;

@SpringBootTest
class SpringSecurityJwtApplicationTests {

    @Test
    void creatToken() {
        JwtBuilder jwtBuilder = Jwts.builder()
                //唯一id
                .setId("666")
                //接受的用户
                .setSubject("Rose")
                //签发时间
                .setIssuedAt(new Date())

                //签名算法+密钥
                .signWith(SignatureAlgorithm.HS256, "hzyyyyyyyy");
        String token = jwtBuilder.compact();
        System.out.println(token);

        System.out.println("     ");

        String[] split = token.split("\\.");
        System.out.println(Base64Codec.BASE64.decodeToString(split[0]));
        System.out.println(Base64Codec.BASE64.decodeToString(split[1]));
    }

    @Test
    public void AnalysisToken() {
        String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI2NjYiLCJzdWIiOiJSb3NlIiwiaWF0IjoxNjMxMjgxNTUxfQ.cWEbwr3FjzM8oyQgEEEG3-M1Im6JJ1bpSxol00WHrpY";
        //Claims JWT的荷载对象
        Claims claims = (Claims) Jwts.parser()
                .setSigningKey("hzyyyyyyy")
                .parse(token)
                .getBody();
        System.out.println("id" + claims.getId());
        System.out.println("rose" + claims.getSubject());
        System.out.println("date" + claims.getIssuedAt());
    }

//设置失效时间
    @Test
    void creatTokenTimeout() {
        long data = System.currentTimeMillis();
        long exp = data+60*1000;
        JwtBuilder jwtBuilder = Jwts.builder()
                //唯一id
                .setId("666")
                //接受的用户
                .setSubject("Rose")
                //签发时间
                .setIssuedAt(new Date())
                //设置失效时间
                .setExpiration(new Date(exp))
                //签名算法+密钥
                .signWith(SignatureAlgorithm.HS256, "hzyyyyyyyy");
        String token = jwtBuilder.compact();
        System.out.println(token);

        System.out.println("     ");

        String[] split = token.split("\\.");
        System.out.println(Base64Codec.BASE64.decodeToString(split[0]));
        System.out.println(Base64Codec.BASE64.decodeToString(split[1]));
    }

    @Test
    public void AnalysisTokenTimeout() {
        /*
        解析失效会报错
        io.jsonwebtoken.ExpiredJwtException
         */
        String token = "eyJhbGciOiJIUzI1NiJ9." +
                "eyJqdGkiOiI2NjYiLCJzdWIiOiJSb3NlIiwiaWF0IjoxNjMxMjgyMDQ2LCJleHAiOjE2MzEyODIxMDZ9." +
                "qSG3daBcCRQ92-p564daNgN7UPbquVz85c14zcj8rdQ";
        //Claims JWT的荷载对象
        Claims claims = (Claims) Jwts.parser()
                .setSigningKey("hzyyyyyyyy")
                .parse(token)
                .getBody();
        System.out.println("id" + claims.getId());
        System.out.println("rose" + claims.getSubject());
        System.out.println("date" + claims.getIssuedAt());
    }

//自定义声明
    @Test
    void creatTokenZDY() {
        JwtBuilder jwtBuilder = Jwts.builder()
                //唯一id
                .setId("666")
                //接受的用户
                .setSubject("Rose")
                //签发时间
                .setIssuedAt(new Date())
                //添加自定义
                .claim("name","zhangsan")
                .claim("logo","xxx.jpg")
                //签名算法+密钥
                .signWith(SignatureAlgorithm.HS256, "hzyyyyyyyy");
        String token = jwtBuilder.compact();
        System.out.println(token);

        System.out.println("     ");

        String[] split = token.split("\\.");
        System.out.println(Base64Codec.BASE64.decodeToString(split[0]));
        System.out.println(Base64Codec.BASE64.decodeToString(split[1]));
    }

    @Test
    public void AnalysisTokenZDY() {
        String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI2NjYiLCJzdWIiOiJSb3NlIiwiaWF0IjoxNjMxMjgyMzc3LCJuYW1lIjoiemhhbmdzYW4iLCJsb2dvIjoieHh4LmpwZyJ9.QuI6RAqoZ88nHrih3xA5dcT3SaWNFYrkobhGpNR5YEw";
        //Claims JWT的荷载对象
        Claims claims = (Claims) Jwts.parser()
                .setSigningKey("hzyyyyyyyy")
                .parse(token)
                .getBody();
        System.out.println("id:" + claims.getId());
        System.out.println("rose:" + claims.getSubject());
        System.out.println("date:" + claims.getIssuedAt());
        System.out.println("name:" + claims.get("name"));
        System.out.println("logo:" + claims.get("logo"));
    }
}
```

# OAuth2

>OAuth是一个协议有很多对应语言的实现，这使用的spring的实现
>
> Oauth2有四种模式 
>
> 1.授权码模式 ：例如登录页面的其他账号登录中使用微信登录
>
> 2.简化版的授权码模式 
>
> 3.密码模式 ：使用密码登录获取token
>
> 4.客户端模式 

<img src="/Users/ /Other/学习笔记/文本笔记/spring security Img/示例.png" alt="示例" style="zoom:50%;" />

## pom.xml

>注意Spring boot和Spring cloud的版本对应

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Hoxton.SR9</spring-cloud.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-oauth2</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <version>0.9.1</version>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## java

创建认证类继承UserDetailsService，重写loadUserByUsername()方法

```java
@Service
public class UserService implements UserDetailsService {
    @Autowired
    private PasswordEncoder passwordEncoder;
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        String password = passwordEncoder.encode("123");
        return new User(s,password, AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
    }
}
```

创建权限类继承WebSecurityConfigurerAdapter,重写configure(HttpSecurity http)方法

bean注入PasswordEncoder，返回为BCryptPasswordEncoder()：密码加密

AuthenticationManager，返回为super.authenticationManager()：jwt自添加内容时要用到

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http.authorizeRequests()
                .antMatchers("/oauth/**","/login/**").permitAll().anyRequest()
                .authenticated().and().formLogin().permitAll().and()
                .csrf().disable();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    @Bean
    protected AuthenticationManager authenticationManager() throws Exception {
        return super.authenticationManager();
    }
}
```

创建jwt存储类

TokenStore：声明存储token为jwt

JwtAccessTokenConverter：添加jwt密钥

JwtTokenEnhancer：后面jwt添加自定义内容要用到

```java
@Configuration
public class JwtTokenStoreConfig {
    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        jwtAccessTokenConverter.setSigningKey("miyaonamelenght");
        return jwtAccessTokenConverter;
    }

    @Bean
    public JwtTokenEnhancer jwtTokenEnhancer() {
        return new JwtTokenEnhancer();
    }
}
```

jwt添加自定义内容类，继承TokenEnhancer重写enhance(OAuth2AccessToken oAuth2AccessToken, OAuth2Authentication oAuth2Authentication)方法

map类型可以添加多个

```java
public class JwtTokenEnhancer implements TokenEnhancer {
    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken oAuth2AccessToken, OAuth2Authentication oAuth2Authentication) {
        Map<String,Object> map = new HashMap<>();
        map.put("uid","hzyy");
        ((DefaultOAuth2AccessToken)oAuth2AccessToken).setAdditionalInformation(map);
        return oAuth2AccessToken;
    }
}
```

授权服务器类，继承AuthorizationServerConfigurerAdapter

必须加注解@EnableAuthorizationServer开启授权服务

passwordEncoder：密码加密

authenticationManager：认证管理

userService：认证服务

jwtTokenStore：jwtToken存储

JwtTokenEnhancer：jwtToken增加自定义内容用

jwtAccessTokenConverter：jwtToken转换器

configure(ClientDetailsServiceConfigurer clients)：配置授权服务器

configure(AuthorizationServerEndpointsConfigurer endpoints)：配置jwt自定义内容

configure(AuthorizationServerSecurityConfigurer security)：获取密钥要认证，单点登录需要

```java
@Configuration
//开启授权服务器
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private PasswordEncoder passwordEncoder;
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private UserService userService;

    @Autowired
    @Qualifier("jwtTokenStore")
    private TokenStore jwtTokenStore;

    @Autowired
    private JwtTokenEnhancer jwtTokenEnhancer;

    @Autowired
    private JwtAccessTokenConverter jwtAccessTokenConverter;

    //http://localhost:8080/oauth/authorize?response_type=code&client_id=client&redirct_url=http://localhost:8080/getCode&scope=all
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                //客户端id
                .withClient("client")
                //密钥
                .secret(passwordEncoder.encode("112233"))
                //重定向地址
                .redirectUris("http://localhost:8080/getCode")
                //授权范围
                .scopes("all")
                //token失效时间
                .accessTokenValiditySeconds(600)
                //授权类型  authorization_code：授权吗模式;password 密码模式 refresh_token刷新令牌
                .authorizedGrantTypes("authorization_code","password","refresh_token")
                //刷新令牌的失效时间
                .refreshTokenValiditySeconds(6000)
                //自动授权不需要点击确定
                .autoApprove(true);
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        //设置增强内容
        TokenEnhancerChain chain = new TokenEnhancerChain();
        List<TokenEnhancer> delegates = new ArrayList<>();
        delegates.add(jwtTokenEnhancer);
        delegates.add(jwtAccessTokenConverter);
        chain.setTokenEnhancers(delegates);
        endpoints.authenticationManager(authenticationManager)
                .userDetailsService(userService)
                .tokenStore(jwtTokenStore)
                .accessTokenConverter(jwtAccessTokenConverter)
                .tokenEnhancer(chain);
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        //获取密钥必须要身份认证，单点登录必须要配置
        security.tokenKeyAccess("isAuthenticated()");
    }
}
```

资源服务器类，继承ResourceServerConfigurerAdapter，重写configure(HttpSecurity http)

需要注意的是：

1. 加注解@EnableResourceServer开启资源服务
2. 一定要先全部拦截然后认证进行放行

必须先.authorizeRequests().anyRequest().authenticated()再.requestMatchers().antMatchers("/user/**");不然会报错

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .requestMatchers()
                .antMatchers("/user/**");
    }
}
```

如果资源服务器和授权服务器分离的话需要多两个步骤

1. jwtToken存储配置

```java
@Configuration
public class JwtTokenStoreConfig {

    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        //设置jwt密钥
        jwtAccessTokenConverter.setSigningKey("miyaonamelenght");
        return jwtAccessTokenConverter;
    }
}
```

2. 添加jwt验证

   和授权服务器资源服务器在一起的区别就是要添加jwtToken认证

       @Autowired
       @Qualifier("jwtTokenStore")
       private TokenStore jwtToken;
       @Override
       public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
           resources.tokenStore(jwtToken);
       }

   ```java
   @Configuration
   @EnableResourceServer
   public class ResourceConfig extends ResourceServerConfigurerAdapter {
       @Autowired
       @Qualifier("jwtTokenStore")
       private TokenStore jwtToken;
       @Override
       public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
           resources.tokenStore(jwtToken);
       }
   
       @Override
       public void configure(HttpSecurity http) throws Exception {
           http.csrf().disable()
                   .authorizeRequests()
                   .anyRequest().authenticated()
                   .and()
                   .requestMatchers()
                   .antMatchers("/user/**");
       }
   }
   ```

   
