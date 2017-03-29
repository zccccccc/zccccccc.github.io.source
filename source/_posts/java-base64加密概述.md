---
title: java base64加密概述
date: 2017-03-22 11:02:19
tags: java
layout: clean-blog
slug: java-base64

---
 
### 要点
* 安全和密码
* 常用安全体系介绍
* 密码分类及java的安全组成
* JDK相关包及第三扩展
* Base64算法介绍

>掌握用法
深入理解
不断实践
反复总结
再次深入理解实践



### 密码常用术语（一）
* 明文： 待加密信息
* 密文: 经过加密后的明文
* 加密：明文转为密文的过程
* 加密算法：明文转为密文的转换算法
* 加密密钥：通过加密算法进行加密操作用的密钥
* 解密：将密文转化为明文的过层
* 解密算法：密文转明文算法
* 解密蜜钥：通过解密算法进行接密操作的密钥
 
### 密码常用术语（二）
* 密码分析：截获密文者试图通过分析截获得密文从而推断出原来的明文或密钥过程
* 主动攻击：攻击者非法入侵密码系统，采用伪造，修改，删除等手段向系统注入假消息进进行欺骗(对密文具有破坏作用)
* 被动攻击：对一个保密系统采取截获密文并对其进行分析和攻击（对密文没有破坏作用）
* 密码体制：由明文空间，密文空间，密钥空间，加密算法和解密算法五部分构成
 
### 密码常用术语（三）
* 密码协议：也是安全协议,指以密码学为基础的消息交换的通信协议，目前是网络环境中提供安全的服务
* 密码系统: 指用于加密，解密系统
* 柯克霍夫原则：数据的安全基于密钥而不是算法的保密。既系统的安全取决于密钥，对密钥 保密，对算法公开 －－－ 现代密码学设计的基础原则


### 密码分类
 * 古典密码：以字符为基本加密单位
 * 现代密码：以信息块为基本加密单元
 
 * 保密内容算法
 
|名称 | 说明 | 应用领域 |类别|
|----|------|--------|--- |
|受限制算法 |  算法保密性基于保持算法的秘密，(算法保密)  | 军事    |  古典密码  |
|基于密钥算法 | 算法的保密性基于对密钥的保密(算法公开)  |  |现代密码|
 
 * 密码体制
 
|名称 | 别名 | 详细说明|
|----|------|--------|---|
|对称密码   |单钥密码或私钥密码 | 指加密密钥于解密密钥相同            |
|非对称密码 |双钥密码或公钥密码 | 指加密密钥与解密密钥不同，密钥分公钥和私钥|
|对称密码算法 | 单钥密码算法或私钥密码算法| 指应用于对称密码的加密，解密算法|
|非对称密码算法| 双钥密码算法或公钥密码算法| 指对应非对称密码加密和解密算法|

* 明文处理方法
  * 分组加密: 指加密时将名分成固定长度的组，用同一密钥和算法对没一块加密，输出也是固定长度的密文。多用于网络加密
  * 流密码：也称序列密码。指加密时每次加密一位或者一个字节明文(产生一串序列来作为加密的密钥流，从密钥流中按规则依次选取密钥，这些不同的密钥分别来加密每个字符)
* 散列函数 
  * 散列函数验证数据完整性: 长度不限制 哈希值容易计算 散列运算过程不可逆
     * 消息摘要算法MD5等
     * SHA --- 安全散列算法
     * MAC --- 消息认证算法
* 数字签名 主要针对以数字形式存储的消息进行的处理    
     
     
###  安全体系 
* OSI (Open System Interconnection) 开发通信 网络7层模型
* TCP/IP安全体系 
 
  

### java安全组成部分 
 
 * JCA  
 * JCE 
 * JSSE 
 * JAAS 
 
 可以修改JDK添加第三方加解密提供者 $JAVA_HOME/jre/lib/security/java.security 

### 相关java包，类

* java.security
  * 消息摘要 
* javax.crypot
  * 安全消息摘要，消息认证（鉴别）码
* java.net.ssl
   * 安全套接字，网络传输
### 第三方java扩展
* Bouncy Castle
  * 两种支持方案: 1) 配置  2) 调用
* Commons CodeC
  * Apache 
  * Base64 二进制 十六进制,字符集编码
  * Url 编码/解码
  

### 实现Base64算法
  
  * Base64算法 提供者 1.Jdk 2.Commons Codec 3.Bouncy Castle

创建maven项目，pom.xml

````
<dependencies>
  <dependency>
        <groupId>org.bouncycastle</groupId>
        <artifactId>bcprov-jdk16</artifactId>
        <version>1.46</version>
    </dependency>
    <dependency>
        <groupId>commons-codec</groupId>
        <artifactId>commons-codec</artifactId>
        <version>1.10</version>
    </dependency>
    </dependencies>
    
````
  


Base64Test.java


````
import org.apache.commons.codec.binary.Base64;
import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;
 
public class Base64Test {

    private static String src="yundongshequ security base64";

   public static void main(String [] args) throws  Exception{

       System.out.println("-------jdkBase64---------");
       jdkBase64();

       System.out.println("-------commonsCodeBase64---------");
       commonsCodeBase64();

       System.out.println("-------bouncyCastleBase64---------");
       bouncyCastleBase64();
   }

    //jdk实现
   public static void  jdkBase64()throws  Exception{

       BASE64Encoder encoder = new BASE64Encoder();
       String encode = encoder.encode(src.getBytes());
       System.out.println("encode: "+encode);

       BASE64Decoder decoder = new BASE64Decoder();
       byte[] decodeBuffer = decoder.decodeBuffer(encode);

       System.out.println("decode: "+new String(decodeBuffer));

   }

   //Commons Codec
    public static void commonsCodeBase64(){

       byte[] encodeBase64 = Base64.encodeBase64(src.getBytes());
        System.out.println("encode: "+new String(encodeBase64));

        byte[] decodeBase64 =  Base64.decodeBase64(encodeBase64);

        System.out.println("decode: "+new String(decodeBase64));

    }

    //Bouncy Castle
    public static void bouncyCastleBase64(){

        byte[] encodeBase64=  org.bouncycastle.util.encoders.Base64.encode(src.getBytes());
        System.out.println("encode: "+new String(encodeBase64));
        byte[] decodeBase64 =  org.bouncycastle.util.encoders.Base64.decode(encodeBase64);

        System.out.println("decode: "+new String(decodeBase64));

    }



}


````

### Base64算法应用场景

 * e-mail,密钥,证书文件
 * 产生： 邮件的历史问题
 * 定义: 基于64个字符的编码算法
 * 关于RFC2045
 * 衍生: Base16,32 Url Base64
 * Base64算法与加密算法都是公开的

> 我们知道在计算机中任何数据都是按ascii码存储的，而ascii码的128～255之间的值是不可见字符。而在网络上交换数据时，比如说从A地传到B地，往往要经过多个路由设备，由于不同的设备对字符的处理方式有一些不同，这样那些不可见字符就有可能被处理错误，这是不利于传输的。所以就先把数据先做一个Base64编码，统统变成可见字符，这样出错的可能性就大降低了

 




