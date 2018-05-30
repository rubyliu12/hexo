---
title: Java、JS版AES加密算法工具类
categories:
  - Java
tags:
  - Java
  - Javascript
  - AES
abbrlink: 1f7e0ef5
date: 2018-02-18 12:43:30
---
[TOC]
    本文主要是实现`AES`算法的`Java`版本和`Javascript`版本，并提供测试例子。
    AES是一个对称分组密码算法，旨在取代DES成为广泛使用的标准。根据使用的密码长度，AES最常见的有3种方案，用以适应不同的场景要求，分别是AES-128、AES-192和AES-256。
    Java中的PKCS5Padding和Javascript中的PKCS7Padding的结果是一样。



<!-- more -->



## Java 版 AES 工具类
```java
package cn.tisson.wechat.common.util;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.spec.SecretKeySpec;
import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

/**
 * AES 加密、解密
 *
 * @author Created by YL on 2018/2/9
 */
public enum AESUtils {
    GBK {
        public String getCharset() {
            return "GBK";
        }
    },

    UTF8 {
        public String getCharset() {
            return "UTF-8";
        }
    };
    public static final String AES_KEY = "MHYKD5tes6fhkdv9";

    /**
     * 获取编码
     */
    public String getCharset() {
        throw new AbstractMethodError();
    }

    /**
     * 加密
     *
     * @param str 需要加密的内容
     * @param key 加密密码
     *
     * @return 加密后的字符串
     */
    private byte[] enc(String str, String key) {
        try {
            SecretKeySpec skeySpec = new SecretKeySpec(key.getBytes(getCharset()), "AES");
            Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
            cipher.init(Cipher.ENCRYPT_MODE, skeySpec);
            return cipher.doFinal(str.getBytes(getCharset()));
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException("不支持的编码格式", e);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("不支持的加密算法", e);
        } catch (NoSuchPaddingException e) {
            throw new RuntimeException("不支持的填充机制", e);
        } catch (InvalidKeyException e) {
            throw new RuntimeException("无效的Key、错误的长度、未初始化等", e);
        } catch (IllegalBlockSizeException e) {
            throw new RuntimeException("密码的块大小不匹配", e);
        } catch (BadPaddingException e) {
            throw new RuntimeException("输入的数据错误，导致填充机制未能正常填充", e);
        }
    }

    private byte[] dec(byte[] str, String key) {
        try {
            SecretKeySpec skeySpec = new SecretKeySpec(key.getBytes(getCharset()), "AES");
            Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
            cipher.init(Cipher.DECRYPT_MODE, skeySpec);
            return cipher.doFinal(str);
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException("不支持的编码格式", e);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("不支持的加密算法", e);
        } catch (NoSuchPaddingException e) {
            throw new RuntimeException("不支持的填充机制", e);
        } catch (InvalidKeyException e) {
            throw new RuntimeException("无效的Key、错误的长度、未初始化等", e);
        } catch (IllegalBlockSizeException e) {
            throw new RuntimeException("密码的块大小不匹配", e);
        } catch (BadPaddingException e) {
            throw new RuntimeException("输入的数据错误，导致填充机制未能正常填充", e);
        }
    }

    /**
     * AES 加密，使用默认 key 加密
     *
     * @param str 要加密的明文
     *
     * @return 加密后的密文
     */
    public String encrypt(String str) {
        return encrypt(str, AESUtils.AES_KEY);
    }

    /**
     * AES加密，使用指定 key 加密
     *
     * @param str 要加密的明文
     * @param key 加密 key
     *
     * @return 解密后字符串
     */
    public String encrypt(String str, String key) {
        byte[] encryptBytes = enc(str, key);
        return Base64Utils.UTF8.encode(encryptBytes);
    }

    /**
     * AES 解密，使用默认 key 解密
     *
     * @param str 要解密的密文
     *
     * @return 解密后的明文
     */
    public String decrypt(String str) {
        return decrypt(str, AESUtils.AES_KEY);
    }

    /**
     * AES 解密，使用默认 key 解密
     *
     * @param str 要解密的密文
     * @param key 解密 key
     *
     * @return 解密后的明文
     */
    public String decrypt(String str, String key) {
        if (str == null || "".equals(str)) {
            return "";
        }
        byte[] bytes = Base64Utils.UTF8.decode2byte(str);
        try {
            return new String(dec(bytes, key), getCharset());
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return "";
    }
}

```


## Java 版 AES 工具类测试

```java
package cn.tisson.wechat.common.util;

import org.junit.Assert;
import org.junit.Test;

/**
 * @author Created by YL on 2018/2/13
 */
public class AESUtilsTest {
    private static final String KEY = "SYhU9Qf3nmrZAx7D";
    private static final String STR = "法拉利_24234slfkjsl《》？：“”｛｝+——";

    @Test
    public void aes() {
        String encrypt = AESUtils.UTF8.encrypt(STR, KEY);
        System.out.println("encrypt ---> " + encrypt);
        String decrypt = AESUtils.UTF8.decrypt(encrypt, KEY);
        System.out.println(decrypt);
        Assert.assertEquals(STR, decrypt);
    }
}
```


## JS 版 AES 工具类

要引入的 js 文件
地址：[https://github.com/brix/crypto-js](https://github.com/brix/crypto-js)
```javascript
<script th:src="@{/crypto-js/core.js}"></script>
<script th:src="@{/crypto-js/enc-base64.js}"></script>
<script th:src="@{/crypto-js/cipher-core.js}"></script>
<script th:src="@{/crypto-js/aes.js}"></script>
<script th:src="@{/crypto-js/mode-ecb.js}"></script>
```


## JS 版 AES 工具类扩展

```javascript
function getRandom(len) {
    var a = len || 16,
        s = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789',
        n = s.length;
    var r = '';
    for (var i = 0; i < a; i++) {
        r += s.charAt(Math.floor(Math.random() * n));
    }
    return r;
}

function encryptAES(str, key) {
    var s = CryptoJS.enc.Utf8.parse(str),
        k = CryptoJS.enc.Utf8.parse(key),
        c = CryptoJS.AES.encrypt(s, k, {
            mode: CryptoJS.mode.ECB,
            padding: CryptoJS.pad.Pkcs7
        }),
        // 转换为字符串
        // return c.toString();
        // Hex 转为十六进制
        d = CryptoJS.enc.Hex.parse(c.ciphertext.toString());
    // Base64 编码
    return CryptoJS.enc.Base64.stringify(d);
}

function decryptAES(str, key) {
    var k = CryptoJS.enc.Utf8.parse(key),
        c = CryptoJS.AES.decrypt(str, k, {
            mode: CryptoJS.mode.ECB,
            padding: CryptoJS.pad.Pkcs7
        });
    // 转换为 UTF8 字符串
    return CryptoJS.enc.Utf8.stringify(c);
    // return c.toString(CryptoJS.enc.Utf8);
}
```


## JS 版 AES 工具类测试

```javascript
var key = getRandom();
console.log('key: %o', key);
var str = '法拉利_24234slfkjsl《》？：“”｛｝+——';
console.log('str: %o', str);
var e = encryptAES(str, key);
console.log('e: %o', e);
var d = decryptAES(e, key);
console.log('d: %o', d);
```
