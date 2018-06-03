---
title: Java、JS版RSA加密算法工具类
categories:
  - Java
tags:
  - Javascript
  - RSA
abbrlink: 5314af17
date: 2018-02-18 12:43:30
---
[TOC]
    本文主要是实现`RSA`算法的`Java`版本和`Javascript`版本，并提供测试例子。
    1、RSA算法可以用于数据加密和数字签名
    2、RSA算法相对于DES/AES等对称加密算法，他的速度要慢的多
    3、使用原则：公钥加密，私钥解密；私钥加密，公钥解密



<!-- more -->



## Java 版 RSA 工具类
要引入的 jar 包
```xml
<!-- jdk1.5 ~ jdk1.8 -->
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.46</version>
</dependency>
```
```java
package cn.tisson.wechat.common.util;

import org.bouncycastle.jce.provider.BouncyCastleProvider;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.ShortBufferException;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.math.BigInteger;
import java.net.URLDecoder;
import java.net.URLEncoder;
import java.security.InvalidKeyException;
import java.security.Key;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.NoSuchProviderException;
import java.security.Provider;
import java.security.SecureRandom;
import java.security.Security;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.RSAPrivateKeySpec;
import java.security.spec.RSAPublicKeySpec;

/**
 * RSA 加解密算法工具
 * <pre>
 *     依赖第三方jar包：bcprov-jdk16-1.46.jar 或者 bcprov-jdk15on-1.46.jar
 *     记录：
 *     1、Java Sun 的 security provider 默认：RSA/None/PKCS1Padding
 *     2、Android 的 security provider 是 Bouncycastle Security provider，默认：RSA/None/NoPadding
 *     3、Cipher.getInstance("RSA/ECB/NoPadding")，有一个缺点就是解密后的明文比加密之前多了很多空格
 *     推荐：
 *     Cipher.getInstance("RSA"，"BC")
 * </pre>
 * <pre>
 *     一般用法：
 *     1、私钥加密，公钥解密（服务端加密，客户端解密）
 *     2、公钥加密，私钥解密（客户端加密，服务端解密）
 *     注意：私钥不要泄漏出去，客户端一般使用公钥
 * </pre>
 *
 * @author Created by YL on 2018/2/9
 */
public class RSAUtils {
    // /**
    //  * RSA 最大加密明文大小
    //  */
    // private static final int MAX_ENCRYPT_BLOCK = 117;
    //
    // /**
    //  * RSA 最大解密密文大小
    //  */
    // private static final int MAX_DECRYPT_BLOCK = 128;
    /**
     * 编码
     */
    private static final String CHARSET = "UTF-8";

    /**
     * 加密算法 RSA
     */
    private static final String ALGORITHM = "RSA";

    /**
     * 这个值关系到块加密的大小，可以更改，但是不要太大，否则效率会低
     */
    private static final int KEY_SIZE = 1024;

    // private static byte[] hexStringToBytes(String hexString) {
    //     if (hexString == null || hexString.equals("")) {
    //         return null;
    //     }
    //     hexString = hexString.toUpperCase();
    //     int length = hexString.length() / 2;
    //     char[] hexChars = hexString.toCharArray();
    //     byte[] d = new byte[length];
    //     for (int i = 0; i < length; i++) {
    //         int pos = i * 2;
    //         d[i] = (byte) (charToByte(hexChars[pos]) << 4 | charToByte(hexChars[pos + 1]));
    //     }
    //     return d;
    // }
    //
    // /**
    //  * Convert char to byte * @param c char * @return byte
    //  */
    // private static byte charToByte(char c) {
    //     return (byte) "0123456789ABCDEF".indexOf(c);
    // }

    /**
     * 把 byte 数组变换为16进制的字符串
     *
     * @param bytes byte 数组
     *
     * @return 16进制的字符串
     */
    private static String byteToString(byte[] bytes) {
        StringBuilder sb = new StringBuilder();
        for (byte by : bytes) {
            int d = by;
            if (d < 0) {
                d += 256;
            }
            if (d < 16) {
                sb.append("0");
            }
            sb.append(Integer.toString(d, 16));
        }
        return sb.toString();
    }

    /**
     * 默认的安全服务提供者
     */
    private static final Provider BC_PROVIDER = new BouncyCastleProvider();
    private static final String BC = BouncyCastleProvider.PROVIDER_NAME;

    private static void addProvider() {
        String bc = Security.getProperty(BC);
        if (bc == null || "".equals(bc.trim())) {
            Security.addProvider(BC_PROVIDER);
        }
    }

    /**
     * 初始化 RSA，生成密钥对：KeyPair
     * <pre>
     *     生成密钥对是非常耗时的，一般要 2s 左右，所以最好缓存密钥对起来，定期去做更新，不用每次都初始化
     * </pre>
     */
    public static KeyPair generateKeyPair() {
        try {
            addProvider();
            KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance(ALGORITHM, BC);
            // keysize 这个值关系到块加密的大小，可以更改，但是不要太大，否则效率会低
            keyPairGen.initialize(KEY_SIZE, new SecureRandom());
            KeyPair keyPair = keyPairGen.generateKeyPair();
            System.out.println("init：");
            System.out.println(keyPair.getPrivate());
            System.out.println(keyPair.getPublic());
            return keyPair;
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("RSA 初始化失败。无此算法", e);
        } catch (NoSuchProviderException e) {
            throw new RuntimeException("RSA 初始化失败。无此安全服务提供者", e);
        }
    }

    /**
     * 使用模（n）、公钥指数（e），还原公钥
     *
     * @param modulus        模（n）
     * @param publicExponent 公钥指数（e）
     *
     * @return RSAPublicKey
     */
    public static RSAPublicKey generatePublicKey(String modulus, String publicExponent) {
        try {
            addProvider();
            KeyFactory keyFac = KeyFactory.getInstance(ALGORITHM, BC);
            BigInteger bi1 = new BigInteger(modulus, 16);
            BigInteger bi2 = new BigInteger(publicExponent, 16);
            RSAPublicKeySpec pubKeySpec = new RSAPublicKeySpec(bi1, bi2);
            return (RSAPublicKey) keyFac.generatePublic(pubKeySpec);
        } catch (InvalidKeySpecException e) {
            throw new RuntimeException("无效的密钥", e);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("无此算法", e);
        } catch (NoSuchProviderException e) {
            throw new RuntimeException("无此安全服务提供者", e);
        }
    }

    /**
     * * 使用模（n）、私钥指数（d），还原私钥
     *
     * @param modulus         模（n）
     * @param privateExponent 私钥指数（d）
     *
     * @return RSAPrivateKey
     */
    public static RSAPrivateKey generatePrivateKey(String modulus, String privateExponent) {
        try {
            addProvider();
            KeyFactory keyFac = KeyFactory.getInstance(ALGORITHM, BC);
            BigInteger bi1 = new BigInteger(modulus, 16);
            BigInteger bi2 = new BigInteger(privateExponent, 16);
            RSAPrivateKeySpec priKeySpec = new RSAPrivateKeySpec(bi1, bi2);
            return (RSAPrivateKey) keyFac.generatePrivate(priKeySpec);
        } catch (InvalidKeySpecException e) {
            throw new RuntimeException("无效的密钥", e);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("无此算法", e);
        } catch (NoSuchProviderException e) {
            throw new RuntimeException("无此安全服务提供者", e);
        }
    }

    /**
     * 返回模值n
     */
    public static String getModulus(RSAPublicKey publicKey) {
        return publicKey.getModulus().toString(16);
    }

    /**
     * 返回模值n
     */
    public static String getModulus(RSAPrivateKey privateKey) {
        return privateKey.getModulus().toString(16);
    }

    /**
     * 返回公钥指数e
     */
    public static String getPublicExponent(RSAPublicKey publicKey) {
        return publicKey.getPublicExponent().toString(16);
    }

    /**
     * 返回私钥指数d
     */
    public static String getPrivateExponent(RSAPrivateKey privateKey) {
        return privateKey.getPrivateExponent().toString(16);
    }

    /**
     * 加密
     *
     * @param key  密钥（公钥、私钥）
     * @param data 要被加密的明文
     *
     * @return 加密后的数据
     */
    private static byte[] encrypt(Key key, byte[] data) {
        try {
            addProvider();
            Cipher cipher = Cipher.getInstance(ALGORITHM, BC);
            cipher.init(Cipher.ENCRYPT_MODE, key);
            // 127
            int blockSize = cipher.getBlockSize();// 获得加密块大小，如：加密前数据为128个byte，而key_size=1024
            // 加密块大小为127
            // byte,加密后为128个byte;因此共有2个加密块，第一个127
            // byte第二个为1个byte
            int length = data.length;
            int outputSize = cipher.getOutputSize(length);// 获得加密块加密后块大小
            int leavedSize = length % blockSize;
            int blocksSize = leavedSize != 0 ? length / blockSize + 1 : length / blockSize;
            byte[] raw = new byte[outputSize * blocksSize];
            int i = 0;
            while (length - i * blockSize > 0) {
                /*
                 * 这里面doUpdate方法不可用，查看源代码后发现每次doUpdate后并没有什么实际动作除了把byte[]放到
                 * ByteArrayOutputStream中，而最后doFinal的时候才将所有的byte[]进行加密，可是到了此时加密块大小很可能已经超出了
                 * OutputSize所以只好用dofinal方法。
                 */
                if (length - i * blockSize > blockSize) {
                    cipher.doFinal(data, i * blockSize, blockSize, raw, i * outputSize);
                } else {
                    cipher.doFinal(data, i * blockSize, length - i * blockSize, raw, i * outputSize);
                }
                i++;
            }
            return raw;
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("无此算法", e);
        } catch (InvalidKeyException e) {
            throw new RuntimeException("无效的密钥", e);
        } catch (ShortBufferException e) {
            throw new RuntimeException("缓冲区异常", e);
        } catch (NoSuchPaddingException e) {
            throw new RuntimeException("无此填充方式", e);
        } catch (BadPaddingException e) {
            throw new RuntimeException("错误的填充", e);
        } catch (NoSuchProviderException e) {
            throw new RuntimeException("无此安全服务提供者", e);
        } catch (IllegalBlockSizeException e) {
            throw new RuntimeException("非法的块大小", e);
        }
    }

    /**
     * 解密
     *
     * @param key  密钥（公钥、私钥）
     * @param data 要被解密的密文
     *
     * @return 解密后的明文
     */
    private static byte[] decrypt(Key key, byte[] data) {
        ByteArrayOutputStream bout = null;
        try {
            addProvider();
            Cipher cipher = Cipher.getInstance(ALGORITHM, BC);
            cipher.init(Cipher.DECRYPT_MODE, key);
            // 128
            int blockSize = cipher.getBlockSize();
            bout = new ByteArrayOutputStream(64);
            int i = 0;
            int length = data.length;
            while (length - i * blockSize > 0) {
                bout.write(cipher.doFinal(data, i * blockSize, blockSize));
                i++;
            }
            return bout.toByteArray();
        } catch (BadPaddingException e) {
            throw new RuntimeException("错误的填充", e);
        } catch (IOException e) {
            throw new RuntimeException("IO操作异常", e);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("无此算法", e);
        } catch (InvalidKeyException e) {
            throw new RuntimeException("无效的密钥", e);
        } catch (NoSuchPaddingException e) {
            throw new RuntimeException("无此填充方式", e);
        } catch (NoSuchProviderException e) {
            throw new RuntimeException("无此安全服务提供者", e);
        } catch (IllegalBlockSizeException e) {
            throw new RuntimeException("非法的块大小", e);
        } finally {
            if (bout != null) {
                try {
                    bout.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private static String encrypt(Key pk, String data) {
        try {
            data = URLEncoder.encode(data, CHARSET);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        // 处理空格问题：URLEncoder.encode 后，会把空格变成“+”号
        if (data.contains("+")) {
            data = data.replace("+", "%20");
        }
        byte[] b1;
        try {
            b1 = data.getBytes(CHARSET);
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException(String.format("不支持的编码格式：%s", CHARSET), e);
        }
        byte[] b2 = encrypt(pk, b1);
        return byteToString(b2);
    }

    private static String decrypt(Key pk, String data) {
        // byte[] b1 = hexStringToBytes(data);
        byte[] b1 = org.bouncycastle.util.encoders.Hex.decode(data);
        // 使用 new BigInteger(data, 16).toByteArray() 解密 js 加密的密文有问题，但是解密 java 端的密文确正常
        // byte[] b1 = new BigInteger(data, 16).toByteArray();
        byte[] b2 = decrypt(pk, b1);
        String str = new String(b2);
        // str = new StringBuffer(str).reverse().toString();
        if (str.contains("%")) {
            str = str.replaceAll("%(?![0-9a-fA-F]{2})", "%25");
        }
        try {
            return URLDecoder.decode(str, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException(String.format("不支持的编码格式：%s", CHARSET), e);
        }
    }

    /**
     * 用公钥加密
     *
     * @param pk  公钥
     * @param str 要加密的明文
     */
    public static String encryptPublic(RSAPublicKey pk, String str) {
        return encrypt(pk, str);
    }

    /**
     * 用公钥加密
     *
     * @param modulus        模
     * @param publicExponent 公钥指数
     * @param str            要加密的明文
     */
    public static String encryptPublic(String modulus, String publicExponent, String str) {
        RSAPublicKey publicKey = generatePublicKey(modulus, publicExponent);
        return encrypt(publicKey, str);
    }

    /**
     * 用公钥解密
     *
     * @param pk  公钥
     * @param str 要解密的密文
     */
    public static String decryptPublic(RSAPublicKey pk, String str) {
        return decrypt(pk, str);
    }

    /**
     * 用公钥解密
     *
     * @param modulus        模
     * @param publicExponent 公钥指数
     * @param str            要解密的密文
     */
    public static String decryptPublic(String modulus, String publicExponent, String str) {
        RSAPublicKey publicKey = generatePublicKey(modulus, publicExponent);
        return decrypt(publicKey, str);
    }

    /**
     * 用私钥加密
     *
     * @param pk  私钥
     * @param str 要加密的明文
     */
    public static String encryptPrivate(RSAPrivateKey pk, String str) {
        return encrypt(pk, str);
    }

    /**
     * 用私钥加密
     *
     * @param modulus         模
     * @param privateExponent 私钥指数
     * @param str             要加密的明文
     */
    public static String encryptPrivate(String modulus, String privateExponent, String str) {
        RSAPrivateKey privateKey = generatePrivateKey(modulus, privateExponent);
        return encrypt(privateKey, str);
    }

    /**
     * 用私钥解密
     *
     * @param pk  私钥
     * @param str 要解密的密文
     */
    public static String decryptPrivate(RSAPrivateKey pk, String str) {
        return decrypt(pk, str);
    }

    /**
     * 用私钥解密
     *
     * @param modulus         模
     * @param privateExponent 私钥指数
     * @param str             要解密的密文
     */
    public static String decryptPrivate(String modulus, String privateExponent, String str) {
        RSAPrivateKey privateKey = generatePrivateKey(modulus, privateExponent);
        return decrypt(privateKey, str);
    }
}
```



## Java 版 RSA 工具类测试

```java
package cn.tisson.wechat.common.util;

import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;

import java.security.KeyPair;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;

/**
 * @author Created by YL on 2018/2/11
 */
public class RSAUtilsTest {
    private RSAPublicKey publicKey;
    private RSAPrivateKey privateKey;

    private RSAPublicKey generatePublicKey;
    private RSAPrivateKey generatePrivateKey;

    private static final String str = "test_三地警方随即_1222——《》？“”：";

    @Before
    public void init() {
        KeyPair keyPair = RSAUtils.generateKeyPair();
        publicKey = (RSAPublicKey) keyPair.getPublic();
        privateKey = (RSAPrivateKey) keyPair.getPrivate();

        String modulus = RSAUtils.getModulus(publicKey);
        String publicExponent = RSAUtils.getPublicExponent(publicKey);
        String privateExponent = RSAUtils.getPrivateExponent(privateKey);
        System.out.println("n ---> " + modulus);
        System.out.println("e ---> " + publicExponent);
        System.out.println("d ---> " + privateExponent);

        // 恢复密钥
        long time = System.currentTimeMillis();
        generatePublicKey = RSAUtils.generatePublicKey(modulus, publicExponent);
        System.out.println("恢复公钥 ---> " + (System.currentTimeMillis() - time));
        time = System.currentTimeMillis();
        generatePrivateKey = RSAUtils.generatePrivateKey(modulus, privateExponent);
        System.out.println("恢复私钥 ---> " + (System.currentTimeMillis() - time));

        modulus = RSAUtils.getModulus(generatePublicKey);
        publicExponent = RSAUtils.getPublicExponent(generatePublicKey);
        privateExponent = RSAUtils.getPrivateExponent(generatePrivateKey);
        System.out.println("n ---> " + modulus);
        System.out.println("e ---> " + publicExponent);
        System.out.println("d ---> " + privateExponent);
    }

    @Test
    public void all() {
        String modulus = RSAUtils.getModulus(publicKey);
        String publicExponent = RSAUtils.getPublicExponent(publicKey);
        String privateExponent = RSAUtils.getPrivateExponent(privateKey);
        System.out.println("modulus ---> " + modulus);
        System.out.println("publicExponent ---> " + publicExponent);
        System.out.println("privateExponent ---> " + privateExponent);

        // 私钥加密，公钥解密
        System.out.println("-----------------------------------私钥加密，公钥解密-----------------------------------");
        String s = RSAUtils.encryptPrivate(privateKey, str);
        System.out.println("RSA私匙加密：" + s);
        String s1 = RSAUtils.decryptPublic(publicKey, s);
        System.out.println("RSA公匙解密：" + s1);
        Assert.assertEquals(str, s1);
        System.out.println("--------------------------------------------------------------------------------------");
        String ss = RSAUtils.encryptPrivate(modulus, privateExponent, str);
        System.out.println("RSA私匙加密：" + ss);
        String ss1 = RSAUtils.decryptPublic(modulus, publicExponent, ss);
        System.out.println("RSA公匙解密：" + ss1);
        Assert.assertEquals(str, ss1);

        // 公钥加密，私钥解密
        System.out.println("-----------------------------------私钥加密，私钥解密-----------------------------------");
        String s2 = RSAUtils.encryptPublic(publicKey, str);
        System.out.println("RSA公钥加密：" + s2);
        String s3 = RSAUtils.decryptPrivate(privateKey, s2);
        System.out.println("RSA私钥解密：" + s3);
        Assert.assertEquals(str, s3);
        System.out.println("--------------------------------------------------------------------------------------");
        String ss2 = RSAUtils.encryptPublic(modulus, publicExponent, str);
        System.out.println("RSA公钥加密：" + ss2);
        String ss3 = RSAUtils.decryptPrivate(modulus, privateExponent, ss2);
        System.out.println("RSA私钥解密：" + ss3);
        Assert.assertEquals(str, ss3);
    }

    @Test
    public void getModulus() {
        String modulus = RSAUtils.getModulus(publicKey);
        String modulus1 = RSAUtils.getModulus(generatePublicKey);
        Assert.assertEquals(modulus, modulus1);

        String modulus2 = RSAUtils.getModulus(privateKey);
        String modulus3 = RSAUtils.getModulus(generatePrivateKey);
        Assert.assertEquals(modulus2, modulus3);
    }

    @Test
    public void getPublicExponent() {
        String publicExponent = RSAUtils.getPublicExponent(publicKey);
        String publicExponent1 = RSAUtils.getPublicExponent(generatePublicKey);
        Assert.assertEquals(publicExponent, publicExponent1);
    }

    @Test
    public void getPrivateExponent() {
        String privateExponent = RSAUtils.getPrivateExponent(privateKey);
        String privateExponent1 = RSAUtils.getPrivateExponent(generatePrivateKey);
        Assert.assertEquals(privateExponent, privateExponent1);
    }

    @Test
    public void encryptPublic() {
        String s = RSAUtils.encryptPublic(publicKey, str);
        String s1 = RSAUtils.encryptPublic(generatePublicKey, str);
        Assert.assertEquals(s, s1);
    }

    @Test
    public void decryptPublic() {
        String s = RSAUtils.encryptPrivate(privateKey, str);

        String s1 = RSAUtils.decryptPublic(publicKey, s);
        String s2 = RSAUtils.decryptPublic(generatePublicKey, s);
        Assert.assertEquals(s1, s2);
    }

    @Test
    public void encryptPrivate() {
        String s = RSAUtils.encryptPrivate(privateKey, str);
        String s1 = RSAUtils.encryptPrivate(generatePrivateKey, str);
        Assert.assertEquals(s, s1);
    }

    @Test
    public void decryptPrivate() {
        String s = RSAUtils.encryptPublic(publicKey, str);

        String s1 = RSAUtils.decryptPrivate(privateKey, s);
        String s2 = RSAUtils.decryptPrivate(generatePrivateKey, s);
        Assert.assertEquals(s1, s2);
    }
}
```



## JS 版 RSA 工具类

```javascript
/*
 * RSA, a suite of routines for performing RSA public-key computations in JavaScript.
 * Copyright 1998-2005 David Shapiro.
 * Dave Shapiro
 * dave@ohdave.com
 * changed by Fuchun, 2010-05-06
 * fcrpg2005@gmail.com
 */
;(function($w) {

    if (typeof $w.RSAUtils === 'undefined')
        var RSAUtils = $w.RSAUtils = {};

    var biRadixBase = 2;
    var biRadixBits = 16;
    var bitsPerDigit = biRadixBits;
    var biRadix = 1 << 16; // = 2^16 = 65536
    var biHalfRadix = biRadix >>> 1;
    var biRadixSquared = biRadix * biRadix;
    var maxDigitVal = biRadix - 1;
    var maxInteger = 9999999999999998;

    //maxDigits:
    //Change this to accommodate your largest number size. Use setMaxDigits()
    //to change it!
    //
    //In general, if you're working with numbers of size N bits, you'll need 2*N
    //bits of storage. Each digit holds 16 bits. So, a 1024-bit key will need
    //
    //1024 * 2 / 16 = 128 digits of storage.
    //
    var maxDigits;
    var ZERO_ARRAY;
    var bigZero, bigOne;

    var BigInt = $w.BigInt = function(flag) {
        if (typeof flag == "boolean" && flag == true) {
            this.digits = null;
        } else {
            this.digits = ZERO_ARRAY.slice(0);
        }
        this.isNeg = false;
    };

    RSAUtils.setMaxDigits = function(value) {
        maxDigits = value;
        ZERO_ARRAY = new Array(maxDigits);
        for (var iza = 0; iza < ZERO_ARRAY.length; iza++) ZERO_ARRAY[iza] = 0;
        bigZero = new BigInt();
        bigOne = new BigInt();
        bigOne.digits[0] = 1;
    };
    RSAUtils.setMaxDigits(20);

    //The maximum number of digits in base 10 you can convert to an
    //integer without JavaScript throwing up on you.
    var dpl10 = 15;

    RSAUtils.biFromNumber = function(i) {
        var result = new BigInt();
        result.isNeg = i < 0;
        i = Math.abs(i);
        var j = 0;
        while (i > 0) {
            result.digits[j++] = i & maxDigitVal;
            i = Math.floor(i / biRadix);
        }
        return result;
    };

    //lr10 = 10 ^ dpl10
    var lr10 = RSAUtils.biFromNumber(1000000000000000);

    RSAUtils.biFromDecimal = function(s) {
        var isNeg = s.charAt(0) == '-';
        var i = isNeg ? 1 : 0;
        var result;
        // Skip leading zeros.
        while (i < s.length && s.charAt(i) == '0') ++i;
        if (i == s.length) {
            result = new BigInt();
        } else {
            var digitCount = s.length - i;
            var fgl = digitCount % dpl10;
            if (fgl == 0) fgl = dpl10;
            result = RSAUtils.biFromNumber(Number(s.substr(i, fgl)));
            i += fgl;
            while (i < s.length) {
                result = RSAUtils.biAdd(RSAUtils.biMultiply(result, lr10),
                    RSAUtils.biFromNumber(Number(s.substr(i, dpl10))));
                i += dpl10;
            }
            result.isNeg = isNeg;
        }
        return result;
    };

    RSAUtils.biCopy = function(bi) {
        var result = new BigInt(true);
        result.digits = bi.digits.slice(0);
        result.isNeg = bi.isNeg;
        return result;
    };

    RSAUtils.reverseStr = function(s) {
        var result = "";
        for (var i = s.length - 1; i > -1; --i) {
            result += s.charAt(i);
        }
        return result;
    };

    var hexatrigesimalToChar = [
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j',
        'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't',
        'u', 'v', 'w', 'x', 'y', 'z'
    ];

    RSAUtils.biToString = function(x, radix) { // 2 <= radix <= 36
        var b = new BigInt();
        b.digits[0] = radix;
        var qr = RSAUtils.biDivideModulo(x, b);
        var result = hexatrigesimalToChar[qr[1].digits[0]];
        while (RSAUtils.biCompare(qr[0], bigZero) == 1) {
            qr = RSAUtils.biDivideModulo(qr[0], b);
            digit = qr[1].digits[0];
            result += hexatrigesimalToChar[qr[1].digits[0]];
        }
        return (x.isNeg ? "-" : "") + RSAUtils.reverseStr(result);
    };

    RSAUtils.biToDecimal = function(x) {
        var b = new BigInt();
        b.digits[0] = 10;
        var qr = RSAUtils.biDivideModulo(x, b);
        var result = String(qr[1].digits[0]);
        while (RSAUtils.biCompare(qr[0], bigZero) == 1) {
            qr = RSAUtils.biDivideModulo(qr[0], b);
            result += String(qr[1].digits[0]);
        }
        return (x.isNeg ? "-" : "") + RSAUtils.reverseStr(result);
    };

    var hexToChar = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        'a', 'b', 'c', 'd', 'e', 'f'
    ];

    RSAUtils.digitToHex = function(n) {
        var mask = 0xf;
        var result = "";
        for (i = 0; i < 4; ++i) {
            result += hexToChar[n & mask];
            n >>>= 4;
        }
        return RSAUtils.reverseStr(result);
    };

    RSAUtils.biToHex = function(x) {
        var result = "";
        var n = RSAUtils.biHighIndex(x);
        for (var i = RSAUtils.biHighIndex(x); i > -1; --i) {
            result += RSAUtils.digitToHex(x.digits[i]);
        }
        return result;
    };

    RSAUtils.charToHex = function(c) {
        var ZERO = 48;
        var NINE = ZERO + 9;
        var littleA = 97;
        var littleZ = littleA + 25;
        var bigA = 65;
        var bigZ = 65 + 25;
        var result;

        if (c >= ZERO && c <= NINE) {
            result = c - ZERO;
        } else if (c >= bigA && c <= bigZ) {
            result = 10 + c - bigA;
        } else if (c >= littleA && c <= littleZ) {
            result = 10 + c - littleA;
        } else {
            result = 0;
        }
        return result;
    };

    RSAUtils.hexToDigit = function(s) {
        var result = 0;
        var sl = Math.min(s.length, 4);
        for (var i = 0; i < sl; ++i) {
            result <<= 4;
            result |= RSAUtils.charToHex(s.charCodeAt(i));
        }
        return result;
    };

    RSAUtils.biFromHex = function(s) {
        var result = new BigInt();
        var sl = s.length;
        for (var i = sl, j = 0; i > 0; i -= 4, ++j) {
            result.digits[j] = RSAUtils.hexToDigit(s.substr(Math.max(i - 4, 0), Math.min(i, 4)));
        }
        return result;
    };

    RSAUtils.biFromString = function(s, radix) {
        var isNeg = s.charAt(0) == '-';
        var istop = isNeg ? 1 : 0;
        var result = new BigInt();
        var place = new BigInt();
        place.digits[0] = 1; // radix^0
        for (var i = s.length - 1; i >= istop; i--) {
            var c = s.charCodeAt(i);
            var digit = RSAUtils.charToHex(c);
            var biDigit = RSAUtils.biMultiplyDigit(place, digit);
            result = RSAUtils.biAdd(result, biDigit);
            place = RSAUtils.biMultiplyDigit(place, radix);
        }
        result.isNeg = isNeg;
        return result;
    };

    RSAUtils.biDump = function(b) {
        return (b.isNeg ? "-" : "") + b.digits.join(" ");
    };

    RSAUtils.biAdd = function(x, y) {
        var result;

        if (x.isNeg != y.isNeg) {
            y.isNeg = !y.isNeg;
            result = RSAUtils.biSubtract(x, y);
            y.isNeg = !y.isNeg;
        } else {
            result = new BigInt();
            var c = 0;
            var n;
            for (var i = 0; i < x.digits.length; ++i) {
                n = x.digits[i] + y.digits[i] + c;
                result.digits[i] = n % biRadix;
                c = Number(n >= biRadix);
            }
            result.isNeg = x.isNeg;
        }
        return result;
    };

    RSAUtils.biSubtract = function(x, y) {
        var result;
        if (x.isNeg != y.isNeg) {
            y.isNeg = !y.isNeg;
            result = RSAUtils.biAdd(x, y);
            y.isNeg = !y.isNeg;
        } else {
            result = new BigInt();
            var n, c;
            c = 0;
            for (var i = 0; i < x.digits.length; ++i) {
                n = x.digits[i] - y.digits[i] + c;
                result.digits[i] = n % biRadix;
                // Stupid non-conforming modulus operation.
                if (result.digits[i] < 0) result.digits[i] += biRadix;
                c = 0 - Number(n < 0);
            }
            // Fix up the negative sign, if any.
            if (c == -1) {
                c = 0;
                for (var i = 0; i < x.digits.length; ++i) {
                    n = 0 - result.digits[i] + c;
                    result.digits[i] = n % biRadix;
                    // Stupid non-conforming modulus operation.
                    if (result.digits[i] < 0) result.digits[i] += biRadix;
                    c = 0 - Number(n < 0);
                }
                // Result is opposite sign of arguments.
                result.isNeg = !x.isNeg;
            } else {
                // Result is same sign.
                result.isNeg = x.isNeg;
            }
        }
        return result;
    };

    RSAUtils.biHighIndex = function(x) {
        var result = x.digits.length - 1;
        while (result > 0 && x.digits[result] == 0) --result;
        return result;
    };

    RSAUtils.biNumBits = function(x) {
        var n = RSAUtils.biHighIndex(x);
        var d = x.digits[n];
        var m = (n + 1) * bitsPerDigit;
        var result;
        for (result = m; result > m - bitsPerDigit; --result) {
            if ((d & 0x8000) != 0) break;
            d <<= 1;
        }
        return result;
    };

    RSAUtils.biMultiply = function(x, y) {
        var result = new BigInt();
        var c;
        var n = RSAUtils.biHighIndex(x);
        var t = RSAUtils.biHighIndex(y);
        var u, uv, k;

        for (var i = 0; i <= t; ++i) {
            c = 0;
            k = i;
            for (j = 0; j <= n; ++j, ++k) {
                uv = result.digits[k] + x.digits[j] * y.digits[i] + c;
                result.digits[k] = uv & maxDigitVal;
                c = uv >>> biRadixBits;
                //c = Math.floor(uv / biRadix);
            }
            result.digits[i + n + 1] = c;
        }
        // Someone give me a logical xor, please.
        result.isNeg = x.isNeg != y.isNeg;
        return result;
    };

    RSAUtils.biMultiplyDigit = function(x, y) {
        var n, c, uv;

        result = new BigInt();
        n = RSAUtils.biHighIndex(x);
        c = 0;
        for (var j = 0; j <= n; ++j) {
            uv = result.digits[j] + x.digits[j] * y + c;
            result.digits[j] = uv & maxDigitVal;
            c = uv >>> biRadixBits;
            //c = Math.floor(uv / biRadix);
        }
        result.digits[1 + n] = c;
        return result;
    };

    RSAUtils.arrayCopy = function(src, srcStart, dest, destStart, n) {
        var m = Math.min(srcStart + n, src.length);
        for (var i = srcStart, j = destStart; i < m; ++i, ++j) {
            dest[j] = src[i];
        }
    };

    var highBitMasks = [0x0000, 0x8000, 0xC000, 0xE000, 0xF000, 0xF800,
        0xFC00, 0xFE00, 0xFF00, 0xFF80, 0xFFC0, 0xFFE0,
        0xFFF0, 0xFFF8, 0xFFFC, 0xFFFE, 0xFFFF
    ];

    RSAUtils.biShiftLeft = function(x, n) {
        var digitCount = Math.floor(n / bitsPerDigit);
        var result = new BigInt();
        RSAUtils.arrayCopy(x.digits, 0, result.digits, digitCount,
            result.digits.length - digitCount);
        var bits = n % bitsPerDigit;
        var rightBits = bitsPerDigit - bits;
        for (var i = result.digits.length - 1, i1 = i - 1; i > 0; --i, --i1) {
            result.digits[i] = ((result.digits[i] << bits) & maxDigitVal) |
                ((result.digits[i1] & highBitMasks[bits]) >>>
                    (rightBits));
        }
        result.digits[0] = ((result.digits[i] << bits) & maxDigitVal);
        result.isNeg = x.isNeg;
        return result;
    };

    var lowBitMasks = [0x0000, 0x0001, 0x0003, 0x0007, 0x000F, 0x001F,
        0x003F, 0x007F, 0x00FF, 0x01FF, 0x03FF, 0x07FF,
        0x0FFF, 0x1FFF, 0x3FFF, 0x7FFF, 0xFFFF
    ];

    RSAUtils.biShiftRight = function(x, n) {
        var digitCount = Math.floor(n / bitsPerDigit);
        var result = new BigInt();
        RSAUtils.arrayCopy(x.digits, digitCount, result.digits, 0,
            x.digits.length - digitCount);
        var bits = n % bitsPerDigit;
        var leftBits = bitsPerDigit - bits;
        for (var i = 0, i1 = i + 1; i < result.digits.length - 1; ++i, ++i1) {
            result.digits[i] = (result.digits[i] >>> bits) |
                ((result.digits[i1] & lowBitMasks[bits]) << leftBits);
        }
        result.digits[result.digits.length - 1] >>>= bits;
        result.isNeg = x.isNeg;
        return result;
    };

    RSAUtils.biMultiplyByRadixPower = function(x, n) {
        var result = new BigInt();
        RSAUtils.arrayCopy(x.digits, 0, result.digits, n, result.digits.length - n);
        return result;
    };

    RSAUtils.biDivideByRadixPower = function(x, n) {
        var result = new BigInt();
        RSAUtils.arrayCopy(x.digits, n, result.digits, 0, result.digits.length - n);
        return result;
    };

    RSAUtils.biModuloByRadixPower = function(x, n) {
        var result = new BigInt();
        RSAUtils.arrayCopy(x.digits, 0, result.digits, 0, n);
        return result;
    };

    RSAUtils.biCompare = function(x, y) {
        if (x.isNeg != y.isNeg) {
            return 1 - 2 * Number(x.isNeg);
        }
        for (var i = x.digits.length - 1; i >= 0; --i) {
            if (x.digits[i] != y.digits[i]) {
                if (x.isNeg) {
                    return 1 - 2 * Number(x.digits[i] > y.digits[i]);
                } else {
                    return 1 - 2 * Number(x.digits[i] < y.digits[i]);
                }
            }
        }
        return 0;
    };

    RSAUtils.biDivideModulo = function(x, y) {
        var nb = RSAUtils.biNumBits(x);
        var tb = RSAUtils.biNumBits(y);
        var origYIsNeg = y.isNeg;
        var q, r;
        if (nb < tb) {
            // |x| < |y|
            if (x.isNeg) {
                q = RSAUtils.biCopy(bigOne);
                q.isNeg = !y.isNeg;
                x.isNeg = false;
                y.isNeg = false;
                r = biSubtract(y, x);
                // Restore signs, 'cause they're references.
                x.isNeg = true;
                y.isNeg = origYIsNeg;
            } else {
                q = new BigInt();
                r = RSAUtils.biCopy(x);
            }
            return [q, r];
        }

        q = new BigInt();
        r = x;

        // Normalize Y.
        var t = Math.ceil(tb / bitsPerDigit) - 1;
        var lambda = 0;
        while (y.digits[t] < biHalfRadix) {
            y = RSAUtils.biShiftLeft(y, 1);
            ++lambda;
            ++tb;
            t = Math.ceil(tb / bitsPerDigit) - 1;
        }
        // Shift r over to keep the quotient constant. We'll shift the
        // remainder back at the end.
        r = RSAUtils.biShiftLeft(r, lambda);
        nb += lambda; // Update the bit count for x.
        var n = Math.ceil(nb / bitsPerDigit) - 1;

        var b = RSAUtils.biMultiplyByRadixPower(y, n - t);
        while (RSAUtils.biCompare(r, b) != -1) {
            ++q.digits[n - t];
            r = RSAUtils.biSubtract(r, b);
        }
        for (var i = n; i > t; --i) {
            var ri = (i >= r.digits.length) ? 0 : r.digits[i];
            var ri1 = (i - 1 >= r.digits.length) ? 0 : r.digits[i - 1];
            var ri2 = (i - 2 >= r.digits.length) ? 0 : r.digits[i - 2];
            var yt = (t >= y.digits.length) ? 0 : y.digits[t];
            var yt1 = (t - 1 >= y.digits.length) ? 0 : y.digits[t - 1];
            if (ri == yt) {
                q.digits[i - t - 1] = maxDigitVal;
            } else {
                q.digits[i - t - 1] = Math.floor((ri * biRadix + ri1) / yt);
            }

            var c1 = q.digits[i - t - 1] * ((yt * biRadix) + yt1);
            var c2 = (ri * biRadixSquared) + ((ri1 * biRadix) + ri2);
            while (c1 > c2) {
                --q.digits[i - t - 1];
                c1 = q.digits[i - t - 1] * ((yt * biRadix) | yt1);
                c2 = (ri * biRadix * biRadix) + ((ri1 * biRadix) + ri2);
            }

            b = RSAUtils.biMultiplyByRadixPower(y, i - t - 1);
            r = RSAUtils.biSubtract(r, RSAUtils.biMultiplyDigit(b, q.digits[i - t - 1]));
            if (r.isNeg) {
                r = RSAUtils.biAdd(r, b);
                --q.digits[i - t - 1];
            }
        }
        r = RSAUtils.biShiftRight(r, lambda);
        // Fiddle with the signs and stuff to make sure that 0 <= r < y.
        q.isNeg = x.isNeg != origYIsNeg;
        if (x.isNeg) {
            if (origYIsNeg) {
                q = RSAUtils.biAdd(q, bigOne);
            } else {
                q = RSAUtils.biSubtract(q, bigOne);
            }
            y = RSAUtils.biShiftRight(y, lambda);
            r = RSAUtils.biSubtract(y, r);
        }
        // Check for the unbelievably stupid degenerate case of r == -0.
        if (r.digits[0] == 0 && RSAUtils.biHighIndex(r) == 0) r.isNeg = false;

        return [q, r];
    };

    RSAUtils.biDivide = function(x, y) {
        return RSAUtils.biDivideModulo(x, y)[0];
    };

    RSAUtils.biModulo = function(x, y) {
        return RSAUtils.biDivideModulo(x, y)[1];
    };

    RSAUtils.biMultiplyMod = function(x, y, m) {
        return RSAUtils.biModulo(RSAUtils.biMultiply(x, y), m);
    };

    RSAUtils.biPow = function(x, y) {
        var result = bigOne;
        var a = x;
        while (true) {
            if ((y & 1) != 0) result = RSAUtils.biMultiply(result, a);
            y >>= 1;
            if (y == 0) break;
            a = RSAUtils.biMultiply(a, a);
        }
        return result;
    };

    RSAUtils.biPowMod = function(x, y, m) {
        var result = bigOne;
        var a = x;
        var k = y;
        while (true) {
            if ((k.digits[0] & 1) != 0) result = RSAUtils.biMultiplyMod(result, a, m);
            k = RSAUtils.biShiftRight(k, 1);
            if (k.digits[0] == 0 && RSAUtils.biHighIndex(k) == 0) break;
            a = RSAUtils.biMultiplyMod(a, a, m);
        }
        return result;
    };


    $w.BarrettMu = function(m) {
        this.modulus = RSAUtils.biCopy(m);
        this.k = RSAUtils.biHighIndex(this.modulus) + 1;
        var b2k = new BigInt();
        b2k.digits[2 * this.k] = 1; // b2k = b^(2k)
        this.mu = RSAUtils.biDivide(b2k, this.modulus);
        this.bkplus1 = new BigInt();
        this.bkplus1.digits[this.k + 1] = 1; // bkplus1 = b^(k+1)
        this.modulo = BarrettMu_modulo;
        this.multiplyMod = BarrettMu_multiplyMod;
        this.powMod = BarrettMu_powMod;
    };

    function BarrettMu_modulo(x) {
        var $dmath = RSAUtils;
        var q1 = $dmath.biDivideByRadixPower(x, this.k - 1);
        var q2 = $dmath.biMultiply(q1, this.mu);
        var q3 = $dmath.biDivideByRadixPower(q2, this.k + 1);
        var r1 = $dmath.biModuloByRadixPower(x, this.k + 1);
        var r2term = $dmath.biMultiply(q3, this.modulus);
        var r2 = $dmath.biModuloByRadixPower(r2term, this.k + 1);
        var r = $dmath.biSubtract(r1, r2);
        if (r.isNeg) {
            r = $dmath.biAdd(r, this.bkplus1);
        }
        var rgtem = $dmath.biCompare(r, this.modulus) >= 0;
        while (rgtem) {
            r = $dmath.biSubtract(r, this.modulus);
            rgtem = $dmath.biCompare(r, this.modulus) >= 0;
        }
        return r;
    }

    function BarrettMu_multiplyMod(x, y) {
        /*
        x = this.modulo(x);
        y = this.modulo(y);
        */
        var xy = RSAUtils.biMultiply(x, y);
        return this.modulo(xy);
    }

    function BarrettMu_powMod(x, y) {
        var result = new BigInt();
        result.digits[0] = 1;
        var a = x;
        var k = y;
        while (true) {
            if ((k.digits[0] & 1) != 0) result = this.multiplyMod(result, a);
            k = RSAUtils.biShiftRight(k, 1);
            if (k.digits[0] == 0 && RSAUtils.biHighIndex(k) == 0) break;
            a = this.multiplyMod(a, a);
        }
        return result;
    }

    var RSAKeyPair = function(encryptionExponent, decryptionExponent, modulus) {
        var $dmath = RSAUtils;
        this.e = $dmath.biFromHex(encryptionExponent);
        this.d = $dmath.biFromHex(decryptionExponent);
        this.m = $dmath.biFromHex(modulus);
        // We can do two bytes per digit, so
        // chunkSize = 2 * (number of digits in modulus - 1).
        // Since biHighIndex returns the high index, not the number of digits, 1 has
        // already been subtracted.
        this.chunkSize = 2 * $dmath.biHighIndex(this.m);
        this.radix = 16;
        this.barrett = new $w.BarrettMu(this.m);
    };

    RSAUtils.getKeyPair = function(encryptionExponent, decryptionExponent, modulus) {
        return new RSAKeyPair(encryptionExponent, decryptionExponent, modulus);
    };

    if (typeof $w.twoDigit === 'undefined') {
        $w.twoDigit = function(n) {
            return (n < 10 ? "0" : "") + String(n);
        };
    }

    // Altered by Rob Saunders (rob@robsaunders.net). New routine pads the
    // string after it has been converted to an array. This fixes an
    // incompatibility with Flash MX's ActionScript.
    RSAUtils.encryptedString = function(key, s) {
        var a = [];
        var sl = s.length;
        var i = 0;
        while (i < sl) {
            a[i] = s.charCodeAt(i);
            i++;
        }

        while (a.length % key.chunkSize != 0) {
            a[i++] = 0;
        }

        var al = a.length;
        var result = "";
        var j, k, block;
        for (i = 0; i < al; i += key.chunkSize) {
            block = new BigInt();
            j = 0;
            for (k = i; k < i + key.chunkSize; ++j) {
                block.digits[j] = a[k++];
                block.digits[j] += a[k++] << 8;
            }
            var crypt = key.barrett.powMod(block, key.e);
            var text = key.radix == 16 ? RSAUtils.biToHex(crypt) : RSAUtils.biToString(crypt, key.radix);
            result += text + " ";
        }
        return result.substring(0, result.length - 1); // Remove last space.
    };

    RSAUtils.decryptedString = function(key, s) {
        var blocks = s.split(" ");
        var result = "";
        var i, j, block;
        for (i = 0; i < blocks.length; ++i) {
            var bi;
            if (key.radix == 16) {
                bi = RSAUtils.biFromHex(blocks[i]);
            } else {
                bi = RSAUtils.biFromString(blocks[i], key.radix);
            }
            block = key.barrett.powMod(bi, key.d);
            for (j = 0; j <= RSAUtils.biHighIndex(block); ++j) {
                result += String.fromCharCode(block.digits[j] & 255,
                    block.digits[j] >> 8);
            }
        }
        // Remove trailing null, if any.
        if (result.charCodeAt(result.length - 1) == 0) {
            result = result.substring(0, result.length - 1);
        }
        return result;
    };
    RSAUtils.setMaxDigits(130);
})(window);
```



## JS 版 RSA 工具类扩展

```javascript
/**
 * RSA 加密数据：返回加密后的字符串
 * <p>e 公匙
 * <p>n 模
 * <p>s 要加密的字符串
 */
function encryptRSA(e, n, s) {
    RSAUtils.setMaxDigits(130);
    var keyPair = RSAUtils.getKeyPair(e, '', n);
    // URI 编码
    s = window.encodeURIComponent(s);
    // 反转
    s = s.split("").reverse().join("");
    return RSAUtils.encryptedString(keyPair, s);
}

/**
 * RSA 解密数据：返回解密后的字符串
 * <p>e 公匙
 * <p>n 模
 * <p>s 要解密的字符串
 */
function decryptRSA(e, n, s) {
    RSAUtils.setMaxDigits(130);
    var keyPair = RSAUtils.getKeyPair('', e, n);
    s = RSAUtils.decryptedString(keyPair, s);
    // 反转
    s = s.split("").reverse().join("");
    // URI 解码
    return window.decodeURIComponent(s);
}
```



## JS 版 RSA 工具类测试

```javascript
var n = 'e1eb7ab440eb2b3413146dc64c66b4047c7d035712201f944dc092d6d65fb6496c27bb6984477e9d4d683cfe28f06e03efdbe28e92134071f5867adb5789d3b076b79bba167a710197ef7f47894f1d3737e4bf5dd33a7db4de67eebbd85f72f7fd681f17f03a30575d613df6ed682fd324e12ec14bd17cc2b667f7536d8db137';
var e = '10001';
// 一般客户端不要传私钥，这里是为了测试
var d = '9316ca9c034c59a39cec87103d7bfca6931a9d8b1a0cfa228780e2d9a757478a8435562abbea04808bfe5affab4de682ffaeacd1e03f528d1faaffe0411d4649fb2e691f0eea759f609b26e04e86419e4e2c628da8ea54168999d744fd611aa1f3c67ceba4a87a289e251dc1dbdd91448d996971dd6daf5804843fe4d0776091';
var encrypt = '651e45bc3f35e766cb2397eef719401a88b6177fc1e566f97d24bf22398aad87b00e82a55cd70520172abd7e8d70a452ef22ce0c5a2477ea61f9430fa94c0913753f18781868bdb730fb78b812461884db72a2211ec681780bd7143d7fd0dddff81be42b887cf6fd70c2cad7a845f93bd79563dc60bb94b85610230cbb327c55';

console.log('------------- 服务端私钥加密，客户端公钥解密 -------------');
decryptR = decryptRSA(e, n, encrypt);
console.log('decryptR: %o', decryptR);

console.log('------------- 公钥加密，私钥解密 -------------');
var str = '客户端 - 法拉利_123_abc_《》';
console.log('str: %o', str);
var encryptR = encryptRSA(e, n, str);
console.log('encryptR: %o', encryptR);
var decryptR = decryptRSA(d, n, encryptR);
console.log('decryptR: %o', decryptR);
console.log('------------- 私钥加密，公钥解密 -------------');
console.log('str: %o', str);
encryptR = encryptRSA(d, n, str);
console.log('encryptR: %o', encryptR);
decryptR = decryptRSA(e, n, encryptR);
console.log('decryptR: %o', decryptR);
```
