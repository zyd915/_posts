---
title: 讲一讲Hive的自定义函数UDF, 顺便聊聊Hive中AES加解密的代码实现
permalink: hive-udf-function
date: 2021-07-29 18:47:13
updated: 2021-07-29 18:47:17
tags:
- 大数据
- hive
categories: [大数据,hive]
toc: true
thumbnail: https://static.studytime.xin//studytime/image/articles/fpGDvP.jpg
keywords: hive自定义函数,hive自定义函数UDF,hive自定义函数aes加解密,java aes加解密工具类,
excerpt: Hive的内置函数虽然提供了很对常用的方法，但面对多样的需求，还是有无法满足的情况，针对这种情况，Hive提供了自定义函数的功能。正好这段时间，有AES加解密的需求，用到了Hive的UDF自定义函数，所以也回顾了下之前的自定义函数，这里做下整理和讲解，希望帮助到大家。
---

Hive的内置函数虽然提供了很对常用的方法，但面对多样的需求，还是有无法满足的情况，针对这种情况，Hive提供了自定义函数的功能。

正好这段时间，有AES加解密的需求，用到了Hive的UDF自定义函数，所以也回顾了下之前的自定义函数，这里做下整理和讲解，希望帮助到大家。


## Hive常见自定义函数有哪些
- UDF：用户自定义函数，user defined function。一对一的输入输出，（最常用的）。
- UDTF：用户自定义表生成函数。user defined table-generate function.一对多的输入输出。类似 lateral view explode 函数
- UDAF：用户自定义聚合函数。user defined aggregate function，多对一的输入输出，类似 count sum max 等统计函数

## Hive UDF自定义函数怎么实现呢？
```
1. 继承org.apache.hadoop.hive.ql.exec.UDF
2. 重写evaluate()，

特殊说明：evaluate()方法不是由接口定义的,因为它可接受的参数个数,数据类型都是不确定的。Hive会检查UDF,看能否找到和函数调用相匹配的evaluate()方法
```

## Hive UDF自定义函数简单代码案例
```
public class FirstUDF extends UDF {
    public String evaluate(String str){
        String upper = null;
        //1、检查输入参数
        if (StringUtils.isEmpty(str)){
​
        } else {
            upper = str.toUpperCase();
        }
​
        return upper;
    }
​
    //调试自定义函数
    public static void main(String[] args){
        System.out.println(new firstUDF().evaluate("studytime"));
    }
}
```

## Hive 加解密自定义函数AES实现

### pom.xml代码
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>xin.studytime.udf</groupId>
    <artifactId>udf</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF8</project.build.sourceEncoding>
        <hadoop.version>2.7.6</hadoop.version>
        <hive.version>2.1.0</hive.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>${hive.version}</version>
        </dependency>
    </dependencies>
</project>
```

### AES 对称加密和解密工具类封装代码
```
package xin.studytime.hive.udf;

import javax.crypto.*;
import javax.crypto.spec.SecretKeySpec;

import org.apache.commons.lang3.StringUtils;

import java.security.SecureRandom;
import java.util.Scanner;


/**
 * AES加密解密工具类
 *
 * @author Super白
 */
public class AESUtil {
    private static final String defaultCharset = "UTF-8";
    private static final String KEY_AES = "AES";
    private static final String SECRET = "wanda";

    /**
     * AES 加密函数方法
     *
     * @param content 加密内容
     * @param secret  秘钥
     * @return
     */
    public static String encrypt(String content, String secret) {
        return doAES(content, secret, Cipher.ENCRYPT_MODE);
    }

    /**
     * AES 解密函数方法
     *
     * @param content 解密内容
     * @param secret  秘钥
     * @return
     */
    public static String decrypt(String content, String secret) {
        return doAES(content, secret, Cipher.DECRYPT_MODE);
    }

    /**
     * 加解密
     *
     * @param content 待处理数据
     * @param secret  密钥
     * @param mode    加解密mode
     * @return
     */
    private static String doAES(String content, String secret, int mode) {
        try {
            if (StringUtils.isBlank(content) || StringUtils.isBlank(secret)) {
                return null;
            }
            //判断是加密还是解密
            boolean encrypt = mode == Cipher.ENCRYPT_MODE;
            byte[] data;

            //1.构造密钥生成器，指定为AES算法,不区分大小写
            KeyGenerator kgen = KeyGenerator.getInstance(KEY_AES);
            SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
            //2.根据ecnodeRules规则初始化密钥生成器
            //生成一个128位的随机源,根据传入的字节数组
            secureRandom.setSeed(secret.getBytes());
            //生成一个128位的随机源,根据传入的字节数组
            kgen.init(128, secureRandom);
            //3.产生原始对称密钥
            SecretKey secretKey = kgen.generateKey();
            //4.获得原始对称密钥的字节数组
            byte[] enCodeFormat = secretKey.getEncoded();
            //5.根据字节数组生成AES密钥
            SecretKeySpec keySpec = new SecretKeySpec(enCodeFormat, KEY_AES);
            //6.根据指定算法AES自成密码器
            Cipher cipher = Cipher.getInstance(KEY_AES);// 创建密码器
            //7.初始化密码器，第一个参数为加密(Encrypt_mode)或者解密解密(Decrypt_mode)操作，第二个参数为使用的KEY
            cipher.init(mode, keySpec);// 初始化

            if (encrypt) {
                data = content.getBytes(defaultCharset);
            } else {
                data = parseHexStr2Byte(content);
            }
            byte[] result = cipher.doFinal(data);
            if (encrypt) {
                //将二进制转换成16进制
                return parseByte2HexStr(result);
            } else {
                return new String(result, defaultCharset);
            }
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
        return null;
    }

    /**
     * 将二进制转换成16进制
     *
     * @param buf
     * @return
     */
    public static String parseByte2HexStr(byte buf[]) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < buf.length; i++) {
            String hex = Integer.toHexString(buf[i] & 0xFF);
            if (hex.length() == 1) {
                hex = '0' + hex;
            }
            sb.append(hex.toUpperCase());
        }
        return sb.toString();
    }

    /**
     * 将16进制转换为二进制
     *
     * @param hexStr
     * @return
     */
    public static byte[] parseHexStr2Byte(String hexStr) {
        if (hexStr.length() < 1) {
            return null;
        }
        byte[] result = new byte[hexStr.length() / 2];
        for (int i = 0; i < hexStr.length() / 2; i++) {
            int high = Integer.parseInt(hexStr.substring(i * 2, i * 2 + 1), 16);
            int low = Integer.parseInt(hexStr.substring(i * 2 + 1, i * 2 + 2), 16);
            result[i] = (byte) (high * 16 + low);
        }
        return result;
    }


    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        /*
         * 加密
         */
        System.out.println("使用AES对称加密，请输入秘钥");
        String Secret = scanner.next();

        System.out.println("请输入要加密的内容:");
        String content = scanner.next();

        System.out.println("根据输入的秘钥" + Secret + "加密后的密文是:" + encrypt(content, Secret));

        /*
         * 解密
         */
        System.out.println("使用AES对称解密，请输入加密的规则：(须与加密相同)");
        Secret = scanner.next();
        System.out.println("请输入要解密的内容（密文）:");
        content = scanner.next();
        System.out.println("根据输入的秘钥" + Secret + "解密后的明文是:" + decrypt(content, Secret));
    }
}
```

### Hive UDF AES加密函数代码
```
package xin.studytime.hive.udf;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.hive.ql.exec.UDF;

public class AESEncrypt extends UDF {

    public String evaluate(String content, String secret) throws Exception {
        if (StringUtils.isBlank(content)) {
            throw new Exception("content not is null");
        }
        if (StringUtils.isBlank(secret)) {
            secret = "wanda";
        }
        return AESUtil.encrypt(content, secret);
    }


    public static void main(String[] args) {
        try {
            System.out.println(new AESEncrypt().evaluate("test", "wanda"));
//            11A3557F7F95FA58016650C51C28A3A9
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Hive UDF AES解密函数代码
```
package xin.studytime.hive.udf;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.hive.ql.exec.UDF;

public class AESDecrypt extends UDF {

    public String evaluate(String content, String secret) throws Exception {
        if (StringUtils.isBlank(content)) {
            throw new Exception("content not is null");
        }
        if (StringUtils.isBlank(secret)) {
            secret = "wanda";
        }
        return AESUtil.decrypt(content, secret);
    }

    public static void main(String[] args) {
        try {
            System.out.println(new AESDecrypt().evaluate("11A3557F7F95FA58016650C51C28A3A9", "wanda"));
//            test
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

### 将Hive加解密UDF函数方法代码打jar包且上传到hdfs

通过maven打包代码，生成 udf-1.0-SNAPSHOT.jar包

![maven打包](https://static.studytime.xin//studytime/image/articles/pP8eTV.png)

![udf-1.0-SNAPSHOT.jar](https://static.studytime.xin//studytime/image/articles/K8fVkF.png)

```
-- 将udf-1.0-SNAPSHOT.jar包上传到hdfs上
hdfs dfs -mkdir /hive_UDF
hdfs dfs /root/data/udf-1.0-SNAPSHOT.jar /hive_UDF
```

### 进入hive客户端，创建UDF函数，加载及使用自定义Hive加解密函数
```
create function wd_encryp as 'xin.studytime.hive.udf.AESEncrypt' using jar 'hdfs://HDPCluster/tmp/data/user/resource/v_baihe/udf-1.0-SNAPSHOT.jar';
select wd_encryp(1, 'wanda') FROM `table`;

create function wd_decrypt as 'xin.studytime.hive.udf.AESDecrypt' using jar 'hdfs://HDPCluster/tmp/data/user/resource/v_baihe/udf-1.0-SNAPSHOT.jar';
select wd_decrypt('FD900F0727A2BC5A2F24DBD640A29217', 'wanda') FROM `table`;
```

## 问题整理

最近遇到`org.pentaho:pentaho-aggdesigner-algorithm:pom:5.1.5-jhyde`一直无法加载和下载的问题，一直报红，打包也无法成功，在网上找了很久发现是spring的maven包，外部加载被限制了，一番查找好，重要解决，下面整理下解决思路。

```
--  部分报错信息
Could not find artifact org.pentaho:pentaho-aggdesigner-algorithm:pom:5.1.5-jhyde in nexus-aliyun (http://maven.aliyun.com/nexus/content/groups/public)

-- 修改maven根目录下的conf文件夹，然后修改setting.xml文件，添加新的镜像

<mirror>
        <id>aliyun-spring</id>
        <mirrorOf>*</mirrorOf>
        <name>aliyun spring</name>
        <url>https://maven.aliyun.com/repository/spring</url>
</mirror>
```

## github 源代码地址
特说说明：也可以直接fork作者代码，直接用，hive 自定义aes加解密函数代码 github仓库地址。https://github.com/mystudytime/hive-udf-function
