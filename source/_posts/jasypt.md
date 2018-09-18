---
title: jasypt优雅加密
date: 2018-09-18 08:41:27
tags:
---
![](http://www.wailian.work/images/2018/09/18/botanical-close-up-color-788485.jpg)

# 介绍

使用Spring Boot开发时，如果配置文件的敏感信息以明文方式展示，将加大系统的安全隐患，使用jasypt加密，十分简单的解决了这个问题

# 使用方法

1. 在项目的启动类中(Application.java)增加注解`@EnableEncryptableProperties`;
2. 增加配置文件`jasypt.encryptor.password = xxxx`,这是加密的密钥;
3. 将配置文件中所有的敏感信息替换为`ENC(加密字符串)`;
4. 引入maven依赖;

```
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>1.9</version>
</dependency>
```

## 生成密钥的方法

```
java -cp /Users/wangweiye/.m2/repository/org/jasypt/jasypt/1.9.2/jasypt-1.9.2.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI input="root" password=xxxx algorithm=PBEWithMD5AndDES
```

- input为需要加密的字符串;
- password为配置文件中设置的加密密钥;

## Java项目内使用

### 加密

```
public void encrypt() {
    // 创建加密器
    StandardPBEStringEncryptor encryptor = new StandardPBEStringEncryptor();
    // 配置
    EnvironmentStringPBEConfig config = new EnvironmentStringPBEConfig();
    config.setAlgorithm("PBEWithMD5AndDES");// 加密算法
    config.setPassword("xxxx");// 密码
    encryptor.setConfig(config);

    String plaintext = "root"; //明文
    String ciphertext = encryptor.encrypt(plaintext); // 加密
    System.out.println(plaintext + " : " + ciphertext);// 运行结果：root : root : zLdyNB+Dj3iw+J+TXZiv5g==
}
```

### 解密

```
public void decrypt() {
    StandardPBEStringEncryptor encryptor = new StandardPBEStringEncryptor();
    EnvironmentStringPBEConfig config = new EnvironmentStringPBEConfig();
    config.setAlgorithm("PBEWithMD5AndDES");
    config.setPassword("xxxx");
    encryptor.setConfig(config);
    String ciphertext = "zLdyNB+Dj3iw+J+TXZiv5g==";// 密文

    //解密
    String plaintext = encryptor.decrypt(ciphertext); // 解密
    assertThat(plaintext).isEqualTo("root");
}
```
