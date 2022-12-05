---
title: "Java Email"
date: 2019-04-24T11:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- java
tags: 
- java
- email
---

# 一：使用

## 1.引入依赖

```xml
<!-- https://mvnrepository.com/artifact/javax.mail/mail -->
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>mail</artifactId>
    <version>1.4.7</version>
</dependency>

<!-- https://mvnrepository.com/artifact/javax.activation/activation -->
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
</dependency>
```

## 2.简单邮件

```java
package com.xiaoyu;

import com.sun.mail.util.MailSSLSocketFactory;

import javax.mail.*;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
import java.security.GeneralSecurityException;
import java.util.Properties;

public class ProSimpl {
    public static void main(String[] args) throws GeneralSecurityException, MessagingException {
        final Properties properties = new Properties();
        properties.setProperty("mail.host","smtp.qq.com");
        properties.setProperty("mail.transport.protocol","smtp");
        properties.setProperty("mail.smtp.auth","true");
//        qq邮箱还需要设置ssl加密，qq独有的其他邮箱没有
        final MailSSLSocketFactory mailSSLSocketFactory = new MailSSLSocketFactory();
        mailSSLSocketFactory.setTrustAllHosts(true);
        properties.put("mail.smtp.ssl.enable","true");
        properties.put("mail.smtp.ssl.socketFactory",mailSSLSocketFactory);

//        1.创建定义整个应用程序所需的环境信息的session对象
//        qq独有的其他邮箱没有
        final Session session = Session.getDefaultInstance(properties, new Authenticator() {
            @Override
            protected PasswordAuthentication getPasswordAuthentication() {
//                发件人邮箱用户名，授权码/
                return new PasswordAuthentication("xxxx@qq.com", "xxxxx");
            }
        });
//        1.开启debug模式
        session.setDebug(true);
//        2.通过session得到transport对象
        final Transport transport = session.getTransport();
//        3.使用邮箱的用户名和授权码连接邮箱服务器
        transport.connect("smtp.qq.com","xxxx@qq.com","xxxx");
//        4.创建邮件
        final MimeMessage message = new MimeMessage(session);
//        指明发件人
        message.setFrom(new InternetAddress("xxxx@qq.com"));
//        指明收件人
        message.setRecipients(Message.RecipientType.TO, String.valueOf(new InternetAddress("xxxx@qq.com")));
//        邮件标题
        message.setSubject("hello");
//        邮件内容
        message.setContent("<h1 style='color:red'>hello world</h1>","text/html;charset=utf-8");
//        5.发送邮件
        transport.sendMessage(message,message.getAllRecipients());

//        6.关闭
        transport.close();
    }
}
```

## 3.图片邮件

```java
package com.xiaoyu;

import com.sun.mail.util.MailSSLSocketFactory;

import javax.activation.DataHandler;
import javax.activation.FileDataSource;
import javax.mail.*;
import javax.mail.internet.InternetAddress;

import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeMessage;
import javax.mail.internet.MimeMultipart;
import java.security.GeneralSecurityException;
import java.util.Properties;

public class ProImage {
    public static void main(String[] args) throws GeneralSecurityException, MessagingException {
        final Properties properties = new Properties();
        properties.setProperty("mail.host","smtp.qq.com");
        properties.setProperty("mail.transport.protocol","smtp");
        properties.setProperty("mail.smtp.auth","true");
//        qq邮箱还需要设置ssl加密，qq独有的其他邮箱没有
        final MailSSLSocketFactory mailSSLSocketFactory = new MailSSLSocketFactory();
        mailSSLSocketFactory.setTrustAllHosts(true);
        properties.put("mail.smtp.ssl.enable","true");
        properties.put("mail.smtp.ssl.socketFactory",mailSSLSocketFactory);

//        1.创建定义整个应用程序所需的环境信息的session对象
//        qq独有的其他邮箱没有
        final Session session = Session.getDefaultInstance(properties, new Authenticator() {
            @Override
            protected PasswordAuthentication getPasswordAuthentication() {
//                发件人邮箱用户名，授权码/
                return new PasswordAuthentication("xxx@qq.com", "xxxx");
            }
        });
//        1.开启debug模式
        session.setDebug(true);
//        2.通过session得到transport对象
        final Transport transport = session.getTransport();
//        3.使用邮箱的用户名和授权码连接邮箱服务器
        transport.connect("smtp.qq.com","xxxx@qq.com","xxxx");
//        4.创建邮件
        final MimeMessage message = new MimeMessage(session);
//        指明发件人
        message.setFrom(new InternetAddress("xxxx@qq.com"));
//        指明收件人
        message.setRecipients(Message.RecipientType.TO, String.valueOf(new InternetAddress("xxxx@qq.com")));
//        邮件标题
        message.setSubject("hello");
//        邮件内容
//        图片
        MimeBodyPart image = new MimeBodyPart();
        DataHandler dataHandler = new DataHandler(new FileDataSource("src/main/resources/zhizhuxia.jpg"));
        image.setDataHandler(dataHandler);
        image.setContentID("zhizhuxia.jpg");
//        文字
        MimeBodyPart text = new MimeBodyPart();
        text.setContent("图片<img src='cid:zhizhuxia.jpg'>","text/html;charset=utf-8");
//        描述数据关系
        MimeMultipart mimeMultipart = new MimeMultipart();
        mimeMultipart.addBodyPart(image);
        mimeMultipart.addBodyPart(text);
        mimeMultipart.setSubType("related");
//        设置到消息中，保存修改
        message.setContent(mimeMultipart);
        message.saveChanges();
//        5.发送邮件
        transport.sendMessage(message,message.getAllRecipients());

//        6.关闭
        transport.close();
    }
}
```

## 4.附件邮件

```java
package com.xiaoyu;

import com.sun.mail.util.MailSSLSocketFactory;

import javax.activation.DataHandler;
import javax.activation.FileDataSource;
import javax.mail.*;
import javax.mail.internet.*;
import java.security.GeneralSecurityException;
import java.util.Properties;

public class ProImageAndFile {
    public static void main(String[] args) throws GeneralSecurityException, MessagingException {
        final Properties properties = new Properties();
        properties.setProperty("mail.host","smtp.qq.com");
        properties.setProperty("mail.transport.protocol","smtp");
        properties.setProperty("mail.smtp.auth","true");
//        qq邮箱还需要设置ssl加密，qq独有的其他邮箱没有
        final MailSSLSocketFactory mailSSLSocketFactory = new MailSSLSocketFactory();
        mailSSLSocketFactory.setTrustAllHosts(true);
        properties.put("mail.smtp.ssl.enable","true");
        properties.put("mail.smtp.ssl.socketFactory",mailSSLSocketFactory);

//        1.创建定义整个应用程序所需的环境信息的session对象
//        qq独有的其他邮箱没有
        final Session session = Session.getDefaultInstance(properties, new Authenticator() {
            @Override
            protected PasswordAuthentication getPasswordAuthentication() {
//                发件人邮箱用户名，授权码/
                return new PasswordAuthentication("xxxxx@qq.com", "xxxx");
            }
        });
//        1.开启debug模式
        session.setDebug(true);
//        2.通过session得到transport对象
        final Transport transport = session.getTransport();
//        3.使用邮箱的用户名和授权码连接邮箱服务器
        transport.connect("smtp.qq.com","xxxx@qq.com","xxxxx");
//        4.邮件内容
        final MimeMessage message = message(session);
//        5.发送邮件
        transport.sendMessage(message,message.getAllRecipients());

//        6.关闭
        transport.close();
    }
    public static MimeMessage message(Session session) throws MessagingException {
        final MimeMessage message = new MimeMessage(session);
        message.setFrom(new InternetAddress("xxxx@qq.com"));
        message.setRecipients(Message.RecipientType.TO,"xxxx@qq.com");
        message.setSubject("主题");
        //        图片
        MimeBodyPart image = new MimeBodyPart();
        DataHandler dataHandler = new DataHandler(new FileDataSource("src/main/resources/zhizhuxia.jpg"));
        image.setDataHandler(dataHandler);
        image.setContentID("zhizhuxia.jpg");
//        文字
        MimeBodyPart text = new MimeBodyPart();
        text.setContent("图片<img src='cid:zhizhuxia.jpg'>","text/html;charset=utf-8");
//        附件
        MimeBodyPart file1 = new MimeBodyPart();
        file1.setDataHandler(new DataHandler(new FileDataSource("src/main/resources/a.text")));
        file1.setFileName("a.text");
        MimeBodyPart file2 = new MimeBodyPart();
        file2.setDataHandler(new DataHandler(new FileDataSource("src/main/resources/b.text")));
        file2.setFileName("b.text");
//        描述数据关系
        MimeMultipart mimeMultipart = new MimeMultipart();
        mimeMultipart.addBodyPart(image);
        mimeMultipart.addBodyPart(text);
        mimeMultipart.setSubType("related");
//        拼装正文为主体
        final MimeBodyPart mimeBodyPart = new MimeBodyPart();
        mimeBodyPart.setContent(mimeMultipart);
//        拼接附件
        final MimeMultipart allfile = new MimeMultipart();
        allfile.addBodyPart(file1);
        allfile.addBodyPart(file2);
        allfile.addBodyPart(mimeBodyPart);
        allfile.setSubType("mixed");
//        放到消息中
        message.setContent(allfile);
        message.saveChanges();
        return message;
    }
}
```



# 二：boot整合

1.引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

2.yml配置

```yml
spring:
  mail:
    username: xxxx@qq.com
    password: xxxx
    host: smtp.qq.com
    properties.mail.smtp.ssl.enable: true
```

3.使用

```java
@Autowired
private JavaMailSenderImpl mailSender;
@Test
void contextLoads() {
    final SimpleMailMessage message = new SimpleMailMessage();
    message.setSubject("hello");
    message.setText("hello world");
    message.setFrom("xxx@qq.com");//从哪来
    message.setTo("xxx@qq.com");//去哪
    mailSender.send(message);
}

@Test
void t2() throws Exception {
    final MimeMessage message = mailSender.createMimeMessage();
    final MimeMessageHelper helper = new MimeMessageHelper(message,true);
    helper.setSubject("hello");
    helper.setText("hello world");
    helper.addAttachment("1.jpg",new File("classpath:1.jpg"));
    helper.addAttachment("a.text",new File("classpath:a.text"));
    helper.setFrom("xxx@qq.com");
    helper.setTo("xxxx@qq.com");
    mailSender.send(message);
}
```
