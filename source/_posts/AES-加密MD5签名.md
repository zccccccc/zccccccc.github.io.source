---
title: AES 加密MD5签名
date: 2017-03-22 19:23:14
tags: java
layout: clean-blog
slug: java-AES-MD5

---

 
### 模拟服务调用和客户端调用，通过AES加密MD5校验


```

   <dependency>
        <groupId>commons-codec</groupId>
        <artifactId>commons-codec</artifactId>
        <version>1.10</version>
    </dependency>

```



```


import org.apache.commons.codec.binary.Base64;
import org.apache.commons.codec.binary.Hex;

import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import java.security.Key;
import java.security.MessageDigest;

/**
 * Created by zhangchao on 3/22/17.
 */
public class AESTest {

    private static String src = "我是数据源{json}";

    //26字符
    private static String srckey = "1234567890abcDEF";
    static String md5Key = "1234567890abcDEF";

    //aes 目前没有被破解
    public static void main(String args[]) throws Exception {



        //jdkAES();



        //数据加密
       String encryptJson = encrypt(src);
        System.out.println("encrypt data :"+encryptJson);
        //签名
        String md5Sign = MD5sign(encryptJson,md5Key);
        System.out.println("md5Sign data :"+md5Sign);
        //发送

        //数据校验
      //encryptJson  服务接口提供数据包
       //md5Sign 对方md5签名
        String vsign = MD5sign(encryptJson,md5Key);
        System.out.println("vsign data :"+vsign);

        System.out.println("是否一致"+(vsign.equals(md5Sign)));


        //数据解密
        String decryptJson =decrypt(encryptJson);

        System.out.println("decypt :"+decryptJson);

    }


    public static String MD5sign(String source,String key ) throws Exception{

        source = source+"&key="+key;
        MessageDigest md = MessageDigest.getInstance("MD5");
        byte[] md5Bytes = md.digest(source.getBytes());
        //byte[] 转16进制
        return Hex.encodeHexString(md5Bytes);
    }





    public static String encrypt(String source) throws Exception{
        //key转换
        Key key = new SecretKeySpec(srckey.getBytes(), "AES");

        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE, key);

        byte[] result = cipher.doFinal(source.getBytes());

         String resultStr   = Base64.encodeBase64String(result);

         return resultStr;
    }

    public static String decrypt(String source) throws Exception{
        //key转换
        Key key = new SecretKeySpec(srckey.getBytes(), "AES");

        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
        cipher.init(Cipher.DECRYPT_MODE, key);


       // byte[] result = cipher.doFinal(Hex.decodeHex(source.toCharArray()));
        byte[] result = cipher.doFinal(Base64.decodeBase64(source));

        String resultStr =new String(result);
         return resultStr;
    }


    @Deprecated
    public static void jdkAES() throws Exception {



        //key转换
        Key key = new SecretKeySpec(srckey.getBytes(), "AES");

        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE, key);

        byte[] result = cipher.doFinal(src.getBytes());


        String resultJson =  Hex.encodeHexString(result);
        System.out.println("jdk aes encrypt==>" +resultJson);


        cipher.init(Cipher.DECRYPT_MODE, key);
        byte[] decrypt = cipher.doFinal(Hex.decodeHex(resultJson.toCharArray()));

        System.out.println("jdk aes decrypt==>" + new String(decrypt));

    }

}

 

``` 