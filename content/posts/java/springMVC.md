---
title: "SpringMVC"
date: 2019-05-14T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- java
tags: 
- java
- springMVC
---

# 概念：

- 前端控制器:DispatcherServlet：调用其他功能组件。

- 处理器映射器:HandlerMapping：地址解析返回对应的执行链。

- 处理器适配器:HandlerAdapter：被前端该控制器调用，执行调用处理器。

- 处理器:Handler：特有行为进行封装的组件。

- 视图解析器:ViewResolver：解析视图。

- 视图:View：视图信息。

处理器映射器:HandlerMapping：地址解析返回对应的执行链

处理器适配器:HandlerAdapter：被前端该控制器调用，执行调用处理器

视图解析器:ViewResolver：解析视图

称为mvc的三大组件

# 页面跳转

>### 返回字符串

```java
    @RequestMapping("/quick")
    public String save(){
        return "redirect:/jsp/success.jsp";
    }
```

>### 返回模型视图

```java
//可以返回可以是整个ModelAndView
    @RequestMapping("/quick")
    public ModelAndView save2(){
        ModelAndView modelAndView = new ModelAndView();
//        设置视图
        modelAndView.setViewName("success");
//        设置模型（相当于向域对象添加数据jsp页面可以获取的），object类型不局限于string
        modelAndView.addObject("username","zhangsan");
        return modelAndView;
    }

//可以直接在方法参数与中获取
    @RequestMapping("/quick")
    public ModelAndView save2(ModelAndView modelAndView){
//        设置视图
        modelAndView.setViewName("success");
//        设置模型（相当于向域对象添加数据jsp页面可以获取的），object类型不局限于string
        modelAndView.addObject("username","zhangsan");
        return modelAndView;
    }

//可以只获取model，但是返回值必须是字符串
    @RequestMapping("/quick")
    public String save2(Model model){
//        设置模型（相当于向域对象添加数据jsp页面可以获取的），object类型不局限于string
        model.addAttribute("username","zhangsan");
        return "success";
    }
/*
解释：
    参数中的ModelAndView或者Model在被springMVC执行时都会进行属性注入，所以可以直接使用。
    参数也可以是一些常用的对象例如
    request。response。。
*/
```

# 内部视图解析器

```xml
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/jsp/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
    <!--prefix:前缀-->
    <!--suffix:后缀-->
```

>#### 解析器使用注意：
>
>public static final java.lang.String REDIRECT_URL_PREFIX = "redirect:";
>public static final java.lang.String FORWARD_URL_PREFIX = "forward:";
>
>* redirect跳转
>* forward转发默认
>  重点:
>   在配置内置内部资源视图解析器后如果只进行转发则可以正常操作
>   但是：
>   如果进行跳转则必须写全路径否则就会404
>   例如
>       <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
>           <property name="prefix" value="/jsp/"></property>
>           <property name="suffix" value=".jsp"></property>
>       </bean>
>       只进行转发则可以：
>     =>> return "success";
>       但是如果要跳转则必须写成：
>     =>> return "redirect:/jsp/success.jsp";
>   也就是说如果进行跳转则试图解析器配置无效；

>## 回写字符串

```java
    @RequestMapping("/quick5")
    @ResponseBody
    public String save5(){
        return "hello";
    }

//重点：@ResponseBody
//该注解的作用就是告知springMVC改方法不进行页面跳转，只是进行页面的回写
```

>### 不能直接回写字符串外的其他类型会报错，解决办法：

```xml
<!--    mvc注解驱动-->
    <mvc:annotation-driven/>
<!--添加mvc注解驱动mvc会自动配置json转换的-->
```

# 获取视图参数

>### 简单参数

```java
    @RequestMapping("/quick8")
    @ResponseBody
    public void save8(String name,Integer id) {
        System.out.println(name);
        System.out.println(id);
    }
//注意点：
//@ResponseBody注解不能删除因为这不是页面跳转
//String name,Integer id形参必须和视图参数名称一致
```

>### pojo形式的

```java
    @RequestMapping("/quick9")
    @ResponseBody
    public void save9(User user) {
        System.out.println(user);
    }
//注意点：
//参数类型中的属性必须和视图中的参数一致
```

>### 数组类型

```java
    @RequestMapping("/quick10")
    @ResponseBody
    public void save10(String[] str) {
        for (int i = 0; i < str.length; i++) {
            System.out.println(str[i]);
        }
    }
//注意点：
//String[] str参数类型中的属性必须和视图中的参数一致
```

>### 集合类型

```java
    @RequestMapping("/quick11")
    @ResponseBody
    public void save11(VO vo) {
        System.out.println(vo);
    }
/*
结果:
VO{userList=[User{name='zhangsan', age=18}, User{name='lisi', age=20}]}
表单：
    <form action="${pageContext.request.contextPath}/quick11" method="post">
        <input name="userList[0].name" type="text">
        <input name="userList[0].age" type="text">
        <input name="userList[1].name" type="text">
        <input name="userList[1].age" type="text">
        <input type="submit" value="ok">
    </form>
注意点:
1：action
2：表单name属性值：
*/
```

# 静态资源获取

>两种方式写一种即可
>
>```xml
><!--   开放资源的访问 --> 
><mvc:resources mapping="/jsp/**" location="/jsp/"/>
><!--mapping：要开放的资源
>location：要开放的资源的具体路径-->
>```
>
>
>
>```xml
><mvc:default-servlet-handler/>
><!--表示当mvc找不到资源的时候就交给原始容器（tomcat）去找，tomcat是可以找到静态资源的-->
>```

## 乱码解决

```xml
<!--web.xml中配置-->
 <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

# 参数绑定

>当视图传递的参数与方法参数不一致时可以使用
>@RequestParam("name")
>该注解有三个参数
>value:绑定的名称，也就是视图传递参数的名称
>required:默认为true，当视图不传递该参数就报错，当设置为false为，视图不传递该参数也不会报错，会是null
>defaultValue:当视图不传递该参数时，默认值是什么

# 自定义类型转换器

>1.写一个类实现Converter泛型第一个是字符串，第二个是要转换的类型
>2.重写convert方法
>
>```java
>package com.xiaoyu.converter;
>
>import org.springframework.core.convert.converter.Converter;
>
>import java.text.ParseException;
>import java.text.SimpleDateFormat;
>import java.util.Date;
>
>public class DateConverter implements Converter<String, Date> {
>@Override
>public Date convert(String s) {
>   SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
>   Date date = null;
>   try {
>       date = format.parse(s);
>   } catch (ParseException e) {
>       e.printStackTrace();
>   }
>   return date;
>}
>}
>
>```
>
>3.在springMVC.xml声明转换器
>
>```xml
><bean id="conversionServiceFactoryBean" class="org.springframework.context.support.ConversionServiceFactoryBean">
>   <property name="converters">
>       <list>
>           <bean class="com.xiaoyu.converter.DateConverter"/>
>       </list>
>   </property>
></bean>
>
>```
>
>4.在mvc注解驱动中添加
>
>
>```xml
><mvc:annotation-driven conversion-service="conversionServiceFactoryBean"/>
>```

# 获取请求数据

>获取请求头
>@RequestHeader
>两个参数：
>value:请求头的名称
>required:是否必须携带此请求头
>
>@CookieValue
>两个参数：
>value:指定cookie的名称
>required:是否必须携带此cookie

# 文件上传

>文件上传客户端三要素：
>1.表单项type="file"
>2.表单的提交方式是post
>3.表单的enctype属性是多部分表单的形式，及enctype=“multiparty/from-data”
>单文件上传步骤：
>1.导入fileupload和io坐标
>2.配置文件上传解析器
>
>```xml
>   <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
><!--        编码格式-->
>       <property name="defaultEncoding" value="utf-8"/>
><!--        最大上传大小-->
>       <property name="maxUploadSize" value="50000"/>
>       <!--...更多配置-->
>   </bean>
>```
>
>3.编写文件上传代码
>
>```java
>@RequestMapping("/quick12")
>   @ResponseBody
>   public void save12(@RequestParam("username") String name, MultipartFile uploadFile) {
>       System.out.println(name);
>       String filename = uploadFile.getOriginalFilename();
>       try {
>           uploadFile.transferTo(new File("Y:\\"+filename));
>       } catch (IOException e) {
>           e.printStackTrace();
>       }
>   }
>```
>
>​        多文件上传：
>
>```java
>@RequestMapping("/quick14")
>@ResponseBody
>public void save14(@RequestParam("username") String name, MultipartFile[] uploadFile) {
>   for (MultipartFile multipartFile : uploadFile) {
>       String originalFilename = multipartFile.getOriginalFilename();
>       try {
>           multipartFile.transferTo(new File("Y:\\"+originalFilename));
>       } catch (IOException e) {
>           e.printStackTrace();
>       }
>   }
>}
>```

# 拦截器

>#### 作用：
>
>```
>用于对处理器进行预处理和后处理。
>将拦截器按一定的顺序连接成一条链，这条链称为拦截器链(Interceptor Chain)，
>在访问被拦截的方法或字段时，拦截器链中的拦截器就会按其之前
>定义的顺序被调用，拦截器也是aop思想的具体实现。
>```
>
>#### 拦截器和过滤器的区别
>
>区别            过滤器                           拦截器
>使用范围        任何JavaWeb工程都可以使用           只有使用springMVC框架的工程才能用
>拦截范围        可以对所有要访问的资源进行拦截        只会拦截访问的控制器方法
>         （js，jsp，html）                  （只有方法可以拦截，资源是不可以的）
>
>#### 自定义拦截器
>
>#### 1.创建拦截器类实现HandlerInterceptor接口
>
>```java
>public class MyInterceptor implements HandlerInterceptor {
>//    在目标方法执行之前执行
>@Override
>public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
>   System.out.println("在目标方法执行之前执行preHandle");
>//        return false;后边则都不能执行
>   return true;
>}
>//    在目标方法执行之后视图返回之前执行
>@Override
>public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
>   System.out.println("目标方法执行之后视图返回之前执行postHandle");
>}
>//    在整个流程都执行完毕后执行
>@Override
>public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
>   System.out.println("在整个流程都执行完毕后执行afterCompletion");
>}
>}
>```
>
>#### 2.配置拦截器
>
>```xml
><!--配置拦截器-->
><mvc:interceptors>
>   <mvc:interceptor>
><!--            对那些资源执行拦截操作-->
>       <mvc:mapping path="/**"/>
>       对那些资源不执行拦截操作
>       <mvc:exclude-mapping path=""/>
>       <bean class="com.xiaoyu.interceptor.MyInterceptor"/>
>   </mvc:interceptor>
></mvc:interceptors>
>```
>
>#### 3.测试拦截器的拦截效果
>
>效果：
>
>```
>>在目标方法执行之前执行preHandle
>>目标资源执行...
>>目标方法执行之后视图返回之前执行postHandle
>>在整个流程都执行完毕后执行afterCompletion
>
>>设置多个拦截器效果（配置循序为1，2）
>>在目标方法执行之前执行preHandle
>>在目标方法执行之前执行preHandle2222
>>目标资源执行...
>>目标方法执行之后视图返回之前执行postHandle2222
>>目标方法执行之后视图返回之前执行postHandle
>>在整个流程都执行完毕后执行afterCompletion2222
>>在整个流程都执行完毕后执行afterCompletion
>```

# 异常处理

>1.MVC简单异常处理器
>
>```xml
><!--异常处理器-->
><bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
>   默认异常，跳转到error页面
>   <property name="defaultErrorView" value="error"/>
>   具体异常，配置具体异常跳转到具体的页面
>   <property name="exceptionMappings">
>       <map>
>           <entry key="java.lang.ClassCastException" value="error"/>
><!--                ...-->
>       </map>
>   </property>
></bean>
>```
>
>2.自定义异常处理器
>
>​	1.创建类实现HandlerExceptionResolver
>
>```java
>public class ExceptionError implements HandlerExceptionResolver {
>       @Override
>       public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
>           ModelAndView modelAndView = new ModelAndView();
>           if (e instanceof ClassCastException){
>               modelAndView.addObject("error","类转换异常");
>               modelAndView.setViewName("error");
>           }
>           return modelAndView;
>       }
>   }
>```
>
>​	2.在MVC.XML中配置
>
>```xml
><bean class="com.xiaoyu.exception.ExceptionError"/>
>```

# 资源文件：

## Web .xml

```xml
<web-app
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://xmlns.jcp.org/xml/ns/javaee"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
        id="WebApp_ID" version="3.1">

<!--  配置全局过滤的(解决乱码问题)-->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <!--  配置springmvc前端控制器-->
  <servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
  <!--  全局初始化参数-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <!--  配置监听器-->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <servlet>
    <servlet-name>UserServlet</servlet-name>
    <servlet-class>com.xiaoyu.web.UserServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>UserServlet</servlet-name>
    <url-pattern>/userServlet</url-pattern>
  </servlet-mapping>

</web-app>

```

## Spring - mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">
<!--Controller组件扫描-->
    <context:component-scan base-package="com.xiaoyu.controller"></context:component-scan>
<!--    配置内部资源视图解析器-->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/jsp/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
<!--    配置处理器映射器-->
<!--    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">-->
<!--        <property name="messageConverters">-->
<!--            <list>-->
<!--                <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>-->
<!--            </list>-->
<!--        </property>-->
<!--    </bean>-->
<!--    mvc注解驱动-->
    <mvc:annotation-driven conversion-service="conversionServiceFactoryBean"/>
<!--   开放资源的访问 -->
    <mvc:resources mapping="/jsp/**" location="/jsp/"/>

    <mvc:default-servlet-handler/>
<!--声明转换器-->
    <bean id="conversionServiceFactoryBean" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <bean class="com.xiaoyu.converter.DateConverter"/>
            </list>
        </property>
    </bean>
<!--配置文件上传解析器-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
<!--        编码格式-->
        <property name="defaultEncoding" value="utf-8"/>
<!--        最大上传大小-->
        <property name="maxUploadSize" value="50000000000"/>
    </bean>

<!--配置拦截器-->
    <mvc:interceptors>
        <mvc:interceptor>
<!--            对那些资源执行拦截操作-->
            <mvc:mapping path="/**"/>

            <bean class="com.xiaoyu.interceptor.MyInterceptor"/>
        </mvc:interceptor>
        <mvc:interceptor>
            <!--            对那些资源执行拦截操作-->
            <mvc:mapping path="/**"/>
            <bean class="com.xiaoyu.interceptor.MyInterceptor2"/>
        </mvc:interceptor>
    </mvc:interceptors>

<!--异常处理器-->
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="defaultErrorView" value="error"/>
        <property name="exceptionMappings">
            <map>
                <entry key="java.lang.ClassCastException" value="error"/>
<!--                ...-->
            </map>
        </property>
    </bean>

<!--自定义异常处理器-->
    <bean class="com.xiaoyu.exception.ExceptionError"/>

</beans>
```

## Controller java代码

```java
package com.xiaoyu.controller;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.xiaoyu.domain.User;
import com.xiaoyu.domain.VO;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CookieValue;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.IOException;

@Controller
//@RequestMapping("/userController")
//如果在类上边加这个注解访问路径则需要加上/userController/quick
//RequestMapping作用进行模块间的区分
public class UserController {
//    请求映射
    @RequestMapping("/quick")
//        @RequestMapping(value = "/xx",method = RequestMethod.GET,params = {"name"})
//    参数value:请求地址只写"/xx"默认为value，method:请求方式，枚举类型
//    params：用于只当限定请求的，加上参数则请求时必须加上该参数才能访问，否则报错，参数具体值随便
//    请求地址:8080/quick
    public String save(){
        System.out.println("Controller save running..");
//        如果在类上边加上RequestMapping注解则需要返回“/success.jsp”
//        如果不加/则会报错404找不到加/表示从当前web应用下边去找资源
        return "redirect:/jsp/success.jsp";
        /*
        *     public static final java.lang.String REDIRECT_URL_PREFIX = "redirect:";
    public static final java.lang.String FORWARD_URL_PREFIX = "forward:";
    * redirect跳转
    * forward转发默认
    * redirect:success
        * */
    }

    @RequestMapping("/quick2")
    public ModelAndView save2(){
        ModelAndView modelAndView = new ModelAndView();
//        设置视图
        modelAndView.setViewName("success");
//        设置模型（相当于向域对象添加数据jsp页面可以获取的），object类型不局限于string
        modelAndView.addObject("username","zhangsan");
        return modelAndView;
    }

    @RequestMapping("/quick3")
    public String save3(HttpServletRequest request){
        request.setAttribute("username","zhangsan");
        return "success";
    }

//    可以进行回写，但是不推荐，因为耦合
    @RequestMapping("/quick4")
    public String save4(HttpServletResponse response) throws IOException {
        response.getWriter().println("aaa");
        return "success";
    }

//    推荐方式
    @RequestMapping("/quick5")
    @ResponseBody
    public String save5(){
        return "hello";
    }

//    输入json字符串
    @RequestMapping("/quick6")
    @ResponseBody
    public String save6() throws JsonProcessingException {
        User user = new User("zhangsan",20);
        ObjectMapper objectMapper = new ObjectMapper();
        String s = objectMapper.writeValueAsString(user);
        return s;
    }

    //    输入json字符串 利用处理映射器
    @RequestMapping("/quick7")
    @ResponseBody
    public User save7() throws JsonProcessingException {
        User user = new User("zhangsan",20);
        return user;
    }

    //    获取视图参数
    @RequestMapping("/quick8")
    @ResponseBody
    public void save8(String name,Integer id) {
        System.out.println(name);
        System.out.println(id);
    }

    //    获取视图参数
    @RequestMapping("/quick9")
    @ResponseBody
    public void save9(User user) {
        System.out.println(user);
    }
    @RequestMapping("/quick10")
    @ResponseBody
    public void save10(String[] str) {
        for (int i = 0; i < str.length; i++) {
            System.out.println(str[i]);
        }
    }

    @RequestMapping("/quick11")
    @ResponseBody
    public void save11(VO vo) {
        System.out.println(vo);
    }

  	//单文件上传
    @RequestMapping("/quick12")
    @ResponseBody
    public void save12(@RequestParam("username") String name, MultipartFile uploadFile) {
        System.out.println(name);
        String filename = uploadFile.getOriginalFilename();
        try {
            uploadFile.transferTo(new File("Y:\\"+filename));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

  	//多文件上传，不推荐
    @RequestMapping("/quick13")
    @ResponseBody
    public void save13(@RequestParam("username") String name, MultipartFile uploadFile1,MultipartFile uploadFile2) {
        System.out.println(name);
        String filename1 = uploadFile1.getOriginalFilename();
        String filename2 = uploadFile2.getOriginalFilename();
        try {
            uploadFile1.transferTo(new File("Y:\\"+filename1));
            uploadFile2.transferTo(new File("Y:\\"+filename2));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

  	//多文件上传，推荐
    @RequestMapping("/quick14")
    @ResponseBody
    public void save14(@RequestParam("username") String name, MultipartFile[] uploadFile) {
        for (MultipartFile multipartFile : uploadFile) {
            String originalFilename = multipartFile.getOriginalFilename();
            try {
                multipartFile.transferTo(new File("Y:\\"+originalFilename));
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    @RequestMapping("/quick15")
    @ResponseBody
    public void save15(@CookieValue(value = "JSESSIONID") String name) {
        System.out.println(name);
    }
}

```

## 拦截器

```java
package com.xiaoyu.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyInterceptor implements HandlerInterceptor {
//    在目标方法执行之前执行
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("在目标方法执行之前执行preHandle");
        String name = request.getParameter("name");
        if ("qwe".equals(name)){
            return true;
        }else {
            request.getRequestDispatcher("/jsp/error.jsp").forward(request,response);
            return false;
        }
//        return false;后边则都不能执行
//        return true;
    }
//    在目标方法执行之后视图返回之前执行
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("目标方法执行之后视图返回之前执行postHandle");
        modelAndView.addObject("name","xiaoyu666");
    }
//    在整个流程都执行完毕后执行
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("在整个流程都执行完毕后执行afterCompletion");
    }
}

```

## 异常处理器

```java
package com.xiaoyu.exception;

import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ExceptionError implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        ModelAndView modelAndView = new ModelAndView();
        if (e instanceof ClassCastException){
            modelAndView.addObject("error","类转换异常");
            modelAndView.setViewName("error");
        }
        return modelAndView;
    }
}

```

