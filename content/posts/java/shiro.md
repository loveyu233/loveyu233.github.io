---
title: "Shiro"
date: 2019-08-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- java
tags: 
- java
- shiro
---

# 基本框架搭建：

- ## 创建UserRealm.java

  - ### 继承AuthorizingRealm

    - #### 实现AuthorizationInfo()：授权方法。

    - #### 实现AuthenticationInfo()：认证方法。

- ## 创建ShiroConfig.java配置类

  - ### @bean加载UserRealm()：加载UserRealm.java到Spring。

  - ### @bean加载DefaultWebSecurityManager()：shiro管理器。

  - ### @bean加载ShiroFilterFactoryBean()：shiro过滤器。

  - ### 根据需要可加载的方法：

  - ### ShiroDialect()：用来整合shiro thymeleaf。

  - ### HashedCredentialsMatcher()：设置加密。

  - ### EhCacheManager()：缓存。

  - ### DefaultWebSessionManager()：设置session管理器。

# UserRealm.java具体方法实现

## 授权方法：

>```java
>@Override
>protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
>SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
>Subject subject = SecurityUtils.getSubject();
>Users users = (Users)subject.getPrincipal();
>info.addStringPermission(users.getPerms());
>return info;
>}
>```
>
>
>
>- ### SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();授权方法。
>
>- ### Subject subject = SecurityUtils.getSubject();获取subject。
>
>- ### Users users = (Users)subject.getPrincipal();通过subject获取到当前用户。
>
>- #### users是由AuthenticationInfo()认证方法的SimpleAuthenticationInfo通过参数传入。
>
>- ### info.addStringPermission(users.getPerms());根据用户权限添加权限。
>
>- ### return info即可

## 认证方法：

>```java
>@Autowired
>UserServiceImpl userService;
>@Override
>protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
>UsernamePasswordToken upToken = (UsernamePasswordToken) token;
>Users users = userService.queryUserByName(upToken.getUsername());
>if (users == null) {
>return null;
>}
>Subject subject = SecurityUtils.getSubject();
>Session session = subject.getSession();
>session.setAttribute("loginuser",users);
>//ByteSource.Util.bytes(users.getSalt()):告诉shiro加盐的盐值
>if (users.getSalt() != null){//必须判断是否有盐值不然会报错：null
>return new SimpleAuthenticationInfo(users,users.getPassword(),ByteSource.Util.bytes(users.getSalt()),"");
>}
>return new SimpleAuthenticationInfo(users,users.getPassword(),"");
>}
>```
>
>
>
>- ### UsernamePasswordToken upToken = (UsernamePasswordToken) token;获取账号密码令牌
>
>- ### @autoWired属性注入UserServiceImpl
>
>- ### Users users = userService.queryUserByName(upToken.getUsername())；
>
>- #### upToken.getUsername()：获取当前用户名称。
>
>- #### userService.queryUserByName：查询该用户，并返回。
>
>- ### 判断users是否为空，如果为空返回null。shiro会自动报错：没有用户错误。
>
>- ### Subject subject = SecurityUtils.getSubject();获取subject。
>
>- ### Session session = subject.getSession();获取session。
>
>- ### session.setAttribute("loginuser",users);把用户信息传入session方便后边使用。
>
>- ### users.getSalt() != null;判断该用户是否有加盐。
>
>- ### 不等于null代表加盐，需要返回盐值。
>
>- #### return new SimpleAuthenticationInfo(users,users.getPassword(),ByteSource.Util.bytes(users.getSalt()),"");
>
>  - #### 第一个参数为传递该用户的信息
>
>  - #### 第二个参数：users.getPassword()：传入密码由shiro来判断密码是否正确。
>
>  - #### 第三个参数：ByteSource.Util.bytes(users.getSalt())：传入盐值
>
>  - #### 第四个参数：当前的realm名称
>
>- ### 等于null代表没有加盐，不需要返回盐值。
>
>- #### return new SimpleAuthenticationInfo(users,users.getPassword(),"");
>
># Controller.java
>
>```java
>@RequestMapping("/login")
>   public ModelAndView log(String username,String password,boolean rememberme){
>       ModelAndView modelAndView = new ModelAndView();
>       //获取当前用户
>       Subject subject = SecurityUtils.getSubject();
>       //封装用户的登录数据
>       UsernamePasswordToken token = new UsernamePasswordToken(username, password);
>       try {
>           subject.login(token);//执行登录方法，如果没有异常就ok
>           token.setRememberMe(rememberme);
>           modelAndView.setViewName("/index");
>       } catch (UnknownAccountException e) {//用户名不存在异常
>           modelAndView.addObject("msgUP","用户名不存在或者错误");
>           modelAndView.setViewName("/login");
>           System.out.println(e);
>       } catch (IncorrectCredentialsException e) {//密码不存在
>           modelAndView.addObject("msgUP","密码不存在");
>           modelAndView.setViewName("/login");
>           System.out.println(e);
>       }
>       return modelAndView;
>   }
>```
>
>>#### 登录过程
>
>>- #### 登录页面填写账号密码.
>>
>>- #### 传入到对应的映射方法.
>>
>>- #### 方法调用subject.login(token)进行登录.
>>
>>- #### 跳转到认证方法方法进行账号密码判断.
>>
>>- #### 判断之后返回到映射方法.
>>
>>- #### 映射方法根据返回结果进行对应逻辑.

# ShiroConfig.java配置类

## UserRealm()：UserRealm方法

>```java
>@Bean
>public UserRealm userRealm(@Qualifier("hashedCredentialsMatcher")HashedCredentialsMatcher hashedCredentialsMatcher) {
>UserRealm userRealm = new UserRealm();
>userRealm.setCredentialsMatcher(hashedCredentialsMatcher);//添加密码设置。
>return userRealm;
>}
>```
>
>## 该方法的主要作用就是把UserRealm.java类加载到Spring容器

## DefaultWebSecurityManager()：shiro安全管理器方法

>```java
>@Bean()
>public DefaultWebSecurityManager getDefaultWebSecurityManager(
>@Qualifier("userRealm") UserRealm userRealm,
>@Qualifier("ehCacheManager")EhCacheManager ehCacheManager,
>@Qualifier("defaultWebSessionManager")DefaultWebSessionManager defaultWebSessionManager,
>@Qualifier("cookieRememberMeManager")CookieRememberMeManager meManager) {
>DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
>//关联userRealm
>securityManager.setRealm(userRealm);
>//添加第三方缓存
>securityManager.setCacheManager(ehCacheManager);
>//添加设置的sessionManager
>securityManager.setSessionManager(defaultWebSessionManager);
>//添加cookie管理器
>securityManager.setRememberMeManager(meManager);
>return securityManager;
>}
>```
>
>## 该方法作用为：
>
>- ### 添加关联userRealm
>
>- ### 添加第三方缓存管理器
>
>- ### 添加自定义设置的session管理器

## ShiroFilterFactoryBean()：shiro权限过滤器方法

>```java
>@Bean
>public ShiroFilterFactoryBean shiroFilterFactoryBean(@Qualifier("getDefaultWebSecurityManager") DefaultWebSecurityManager defa) {
>
>ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
>//设置安全管理器
>bean.setSecurityManager(defa);
>//添加shiro的内置过滤器
>/* shiro已经定义的关键字
>           anon:无需认证就可以访问
>           authc：必须认证才能访问
>           user：必须拥有记住我功能才能访问
>           perms：拥有对某个资源的权限才能访问
>           logout:退出当前登录
>    */
>//设置访问拦截
>LinkedHashMap<String, String> hashMap = new LinkedHashMap<>();
>//授权
>hashMap.put("/user/add","perms[user:add]");
>hashMap.put("/user/update","perms[user:update]");
>//授权要在认证前边
>hashMap.put("/user/*", "authc");
>//退出登录,/logout不需要具体的映射方法，只要跳转到这个路径就退出。
>hashMap.put("/logout","logout");
>bean.setFilterChainDefinitionMap(hashMap);
>//设置登录页面
>bean.setLoginUrl("/toLogin");
>//设置未授权请求
>bean.setUnauthorizedUrl("/unauthorized");
>bean.setSuccessUrl("/toLogin");
>return bean;
>}
>```
>
>### 安全管理器需要添加到该bean： bean.setSecurityManager(defa);
>
>### 授权map集合的key为资源，value为访问这些资源要携带的权限字符串。
>
>### 退出登录的路径只需要和前端的退出按钮上的路径一样即可，不需要具体的映射实现方法。
>
>### 未授权请求就是当该用户没有该权限时访问该资源就会跳转到这个路径的前端页面上，这个需要映射方法的实现来进行跳转
>
>### 授权和认证的区别：
>
>### 认证就是必须要登录账号才可以访问
>
>### 授权时必须要用户登录并携带该权限才可以访问

## ShiroDialect()：用来整合shiro thymeleaf方法

>```java
>@Bean
>public ShiroDialect shiroDialect(){
>return new ShiroDialect();
>}
>```
>
>```html
><h1>首页</h1>
><!--    th:if="${session.loginuser!=null}"判断是否有这个session值-->
><h2 th:if="${session.loginuser!=null}"><a href="/logout">退出</a></h2>
><h2 th:if="${session.loginuser==null}"><a href="/toLogin">登录</a></h2>
><p th:text="${msg}"></p>
><!--    shiro:hasPermission="user:add"检查是否有这个权限-->
><div shiro:hasPermission="user:add">
><a th:href="@{/user/add}">add</a>
></div>
><div shiro:hasPermission="user:update">
><a th:href="@{/user/update}">update</a>
></div>
>```
>
>```html
><!--    thymeleaf和shiro的名称空间-->
>xmlns:th="http://www.thymeleaf.org"
>xmlns:shiro="http://www.pollix.at/thymeleaf/shiro"
>```
>
>```xml
><!--   pom.xml坐标     -->
><dependency>
><groupId>com.github.theborakompanioni</groupId>
><artifactId>thymeleaf-extras-shiro</artifactId>
><version>2.0.0</version>
></dependency>
>```
>
>

## HashedCredentialsMatcher()：设置加密方法

>```java
>@Bean(name = "hashedCredentialsMatcher")
>public HashedCredentialsMatcher shiro(){
>HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher();
>//加密方式
>credentialsMatcher.setHashAlgorithmName("md5");
>//散列次数，次数整个项目要统一不然会报错
>credentialsMatcher.setHashIterations(1);
>return credentialsMatcher;
>}
>```
>
>```java
>@RequestMapping("/adduser")
>@ResponseBody
>public String md5(String username,String password){
>//在注册是对密码进行加密
>Md5Hash md5Hash = new Md5Hash(password);
>//加盐加密
>int num = new Random().nextInt(90000)+10000;
>Md5Hash md5Hash1 = new Md5Hash(password, num+"");
>//加盐加密加散列
>val md5Hash2 = new Md5Hash(password, num + "", 3);
>//第二种方法进行加密,和Md5Hash一样是重载形式的
>final SimpleHash md5 = new SimpleHash("md5", password, num + "", 3);
>return "只加密"+md5Hash.toString()+"<br>"
>+"加盐加密："+md5Hash1.toString()+"盐值："+num+"<br>"
>+"加盐加密加散列"+md5Hash2.toString()+"盐值："+num+"<br>"
>+"SimpleHash加密"+md5.toString()+"盐值："+num;
>}
>```
>
>### 这两个的散列次数要一致。
>
>### 散列不一定要随机数字，也可以是字符串，或者用户名称，或者自定义。
>
>### shiro默认的加盐是加在密码前边，这个可以自定义实现。
>
>### 加盐必须是字符串形式不然会报错。

## EhCacheManager()：缓存方法

>```java
>@Bean
>public EhCacheManager ehCacheManager(){
>EhCacheManager ehCacheManager = new EhCacheManager();
>ehCacheManager.setCacheManagerConfigFile("classpath:ehcache.xml");
>return ehCacheManager;
>}
>```
>
>```xml
><dependency>
><groupId>org.springframework.boot</groupId>
><artifactId>spring-boot-starter-cache</artifactId>
></dependency>
><dependency>
><groupId>net.sf.ehcache</groupId>
><artifactId>ehcache</artifactId>
></dependency>
><dependency>
><groupId>org.apache.shiro</groupId>
><artifactId>shiro-ehcache</artifactId>
><version>1.7.1</version>
></dependency>
>```
>
>```xml
><?xml version="1.0" encoding="utf-8" ?>
><ehcache updateCheck="false" dynamicConfig="false">
><diskStore path=""/>
><cache name="user" maxEntriesLocalHeap="1000"/>
><defaultCache
>       name="defaultCache"
>       maxElementsInMemory="1000"
>       maxElementsOnDisk="10000000"
>       eternal="false"
>       overflowToDisk="false"
>       diskPersistent="false"
>       timeToIdleSeconds="120"
>       timeToLiveSeconds="120"
>       diskExpiryThreadIntervalSeconds="120"
>       memoryStoreEvictionPolicy="LRU"
>/>
></ehcache>
>```
>
>第一步导入坐标。
>
>第二步在applcation.yml同级目录创建ehcache.xml
>
>第三步@bean

## DefaultWebSessionManager()：设置session管理器方法

>```java
>@Bean
>public DefaultWebSessionManager defaultWebSessionManager(){
>final DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
>//设置session过期时间为10秒，单位是毫秒
>sessionManager.setGlobalSessionTimeout(30*1000);
>return sessionManager;
>}
>```
>
>shiro默认的session是半个小时过期，这里可以自定义过期时间。

## CookieRememberMeManager()：设置记住我功能

>添加cookie
>
>```java
>@Bean
>public CookieRememberMeManager cookieRememberMeManager(){
>final CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
>final SimpleCookie simpleCookie = new SimpleCookie("rememberme");
>//cookie存活时间，单位秒
>simpleCookie.setMaxAge(30*24*60*60);
>return cookieRememberMeManager;
>}
>```
>
>

# 其他

jdbc和mybatis配置

```yml
spring:
  datasource:
    username: root
    password: Hzyy1214
    type: com.alibaba.druid.pool.DruidDataSource
    driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
    dbcp2:
      initial-size: 5
      min-idle: 5
      max-idle: 20
      max-wait-millis: 3000
      time-between-eviction-runs-millis: 3000
      min-evictable-idle-time-millis: 2000
      validation-query: SELECT 1 FROM DUAL
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      pool-prepared-statements: true
mybatis:
  type-aliases-package: com.example.pojo
  mapper-locations: mapper/*.xml
```

# 多realm解决方案

## 链式

多个realm都会被执行，只要其中一个通过即可

```java
@Bean
public DefaultWebSecurityManager securityManager(){
  final DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();

  securityManager.setAuthenticator(authenticator());

  final ArrayList<Realm> realms = new ArrayList<>();
  realms.add(userRealm());
  realms.add(mangerRealm());
  securityManager.setRealms(realms);
  return securityManager;
}
```

多个realm是通过集合进行添加。其余操作不变。

## 分支

进行判断选择其中的一个或几个realm来进行执行

### 第一步：自定义token

> ## 继承UsernamePasswordToken,增强其方法，添加一个logintype参数。

```java
package com.example.shiroConfig;

import org.apache.shiro.authc.UsernamePasswordToken;

public class MyToken extends UsernamePasswordToken {
    private String loginType;

    public MyToken(String username,String password,String loginType) {
        super(username,password);
        this.loginType = loginType;
    }

    public String getLoginType() {
        return loginType;
    }

    public void setLoginType(String loginType) {
        this.loginType = loginType;
    }
}
```

### 第二步：自定义认证器

> ### 继承ModularRealmAuthenticator,重写AuthenticationInfo方法，在方法中进行判断选择要使用的realm。

```java
package com.example.shiroConfig;

import com.sun.org.slf4j.internal.Logger;
import com.sun.org.slf4j.internal.LoggerFactory;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.pam.ModularRealmAuthenticator;
import org.apache.shiro.realm.Realm;
 
import java.util.ArrayList;
import java.util.Collection;

public class MyAuthenticator extends ModularRealmAuthenticator {
    @Override
    protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
        Collection<Realm> realms = this.getRealms();
        final MyToken token = (MyToken) authenticationToken;
        final ArrayList<Realm> myRealm = new ArrayList<>();//自定义一个realm集合
        for (Realm realm:realms) {
            if (realm.getName().startsWith(token.getLoginType())) {//根据logintype来判断使用的realm，把要使用的realm添加到自定义的realm集合中。
                myRealm.add(realm);
            }
        }
        return myRealm.size() == 1 ?
                this.doSingleRealmAuthentication(myRealm.iterator().next(), authenticationToken)
                : this.doMultiRealmAuthentication(myRealm, authenticationToken);
      //这里返回判断要使用自定义的realm集合。
    }
}
```

### 第三步：添加bean

```java
@Bean
public MyAuthenticator authenticator(){
    return new MyAuthenticator();
}
```

### 第四步：加入安全管理器

```java
@Bean
public DefaultWebSecurityManager securityManager(){
    final DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
	
    securityManager.setAuthenticator(authenticator());//加入安全管理器

    final ArrayList<Realm> realms = new ArrayList<>();
    realms.add(userRealm());
    realms.add(mangerRealm());
    securityManager.setRealms(realms);
    return securityManager;
}
```

### 第五步：前端接收的参数传递到自定义的token

```java
@RequestMapping("/login")
public ModelAndView login(String username,String password,String logintype){
    System.out.println(username+password+logintype);
    ModelAndView modelAndView = new ModelAndView();
    //获取当前用户
    Subject subject = SecurityUtils.getSubject();
    //封装用户的登录数据
    final MyToken token = new MyToken(username, password, logintype);//自定义的token
    try {
        subject.login(token);
        modelAndView.setViewName("/index");
    } catch (Exception e) {
        modelAndView.setViewName("/login");
    }
    return modelAndView;
}
```

