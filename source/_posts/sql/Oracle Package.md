---
title: Oracle Package 包说明和包体
categories:
  - Oracle
tags:
  - Package
abbrlink: 6b176628
date: 2018-12-04 16:30:05
---



> 在包中定义的函数，包体中实现的时候，函数名、参数名、类型都要一致，否则报错



# 定义包说明

```sql
CREATE OR REPLACE PACKAGE WX_CRYPTO IS
  -- 定义 DES encrypt 函数
  FUNCTION DES_ENCRYPT(INPUT_STRING IN VARCHAR2) RETURN VARCHAR2;
  FUNCTION DES_ENCRYPT(INPUT_STRING IN VARCHAR2, KEY_STRING IN VARCHAR2)
    RETURN VARCHAR2;

  -- 定义 DES decrypt 函数
  FUNCTION DES_DECRYPT(INPUT_STRING IN VARCHAR2) RETURN VARCHAR2 DETERMINISTIC;
  FUNCTION DES_DECRYPT(INPUT_STRING IN VARCHAR2, KEY_STRING IN VARCHAR2)
    RETURN VARCHAR2 DETERMINISTIC;
END WX_CRYPTO;
```

> 在创建基于自定义函数时, 指定 `deterministic` 参数，在创建函数索引，就没有问题了



<!-- more -->



比如：

```sql
/*
 * 如果函数不加 deterministic，以下创建索引 SQL 语句会报错：
 * ORA-30553: The function is not deterministic
 */
create index i_user_decrypt_user_no on USER (wx_crypto.des_decrypt(user_no));


-- 以下查询是会走索引的
select * from user where wx_crypto.des_decrypt(user_no) = '123';
```



# 定义包体

```sql
CREATE OR REPLACE PACKAGE BODY WX_CRYPTO IS
  -- 实现 DES encrypt 函数（一个参数）
  FUNCTION DES_ENCRYPT(INPUT_STRING IN VARCHAR2 -- 明文
                       ) RETURN VARCHAR2 AS
  BEGIN
    RETURN WX_CRYPTO.DES_ENCRYPT(INPUT_STRING => INPUT_STRING,
                                 KEY_STRING   => 'eda93763');
  END;
  -- 实现 DES encrypt 函数（两个参数）
  FUNCTION DES_ENCRYPT(INPUT_STRING IN VARCHAR2, -- 明文
                       KEY_STRING   IN VARCHAR2 -- 密钥
                       ) RETURN VARCHAR2 AS
    V_TEXT        VARCHAR2(4000); -- 明文。长度要是 8 的倍数，若不足 8 的倍数，则用隐藏字符串 chr(0) 补足
    V_TEXT_RAW    RAW(2048); -- 明文
    V_KEY_RAW     RAW(128); -- 密钥
    V_ENCRYPT_RAW RAW(2048); -- 加密后的字符串
  BEGIN
    IF INPUT_STRING IS NULL THEN
      RETURN NULL;
    ELSE
      -- 向右补足。CHR(0) 隐藏字符串
      V_TEXT := RPAD(INPUT_STRING,
                     (TRUNC(LENGTHB(INPUT_STRING) / 8) + 1) * 8,
                     CHR(0));
      -- 转换成 16 进制
      V_TEXT_RAW    := UTL_I18N.STRING_TO_RAW(V_TEXT, 'ZHS16GBK');
      V_KEY_RAW     := UTL_I18N.STRING_TO_RAW(KEY_STRING, 'ZHS16GBK');
      V_ENCRYPT_RAW := DBMS_OBFUSCATION_TOOLKIT.DESENCRYPT(INPUT => V_TEXT_RAW,
                                                           KEY   => V_KEY_RAW);
      RETURN '{DES}' || RAWTOHEX(V_ENCRYPT_RAW);
    END IF;
  END;

  -- 实现 DES decrypt 函数（一个参数）
  FUNCTION DES_DECRYPT(INPUT_STRING IN VARCHAR2 -- 明文
                       ) RETURN VARCHAR2 DETERMINISTIC AS
  BEGIN
    RETURN WX_CRYPTO.DES_DECRYPT(INPUT_STRING => INPUT_STRING,
                                 KEY_STRING   => 'eda93763');
  END;
  -- 实现 DES decrypt 函数（两个参数）
  FUNCTION DES_DECRYPT(INPUT_STRING IN VARCHAR2, -- 密文
                       KEY_STRING   IN VARCHAR2 -- 密钥
                       ) RETURN VARCHAR2 DETERMINISTIC AS
    V_TEXT_RAW       RAW(2048); -- 密文
    V_KEY_RAW        RAW(128); -- 密钥
    V_DECRYPT_RAW    RAW(2048); -- 解密后的明文
    V_DECRYPT_STRING VARCHAR2(4000); -- 解密后的明文
  BEGIN
    IF INPUT_STRING IS NULL THEN
      RETURN NULL;
    ELSE
      IF INSTR(INPUT_STRING, '{DES}') > 0 THEN
        V_TEXT_RAW := HEXTORAW(SUBSTR(INPUT_STRING, 6));
        -- 转换成 16 进制
        V_KEY_RAW := UTL_I18N.STRING_TO_RAW(KEY_STRING, 'ZHS16GBK');
        -- 解密
        V_DECRYPT_RAW := DBMS_OBFUSCATION_TOOLKIT.DESDECRYPT(INPUT => V_TEXT_RAW,
                                                             KEY   => V_KEY_RAW);
        -- RAW_TO_CHAR 转换成字符串
        V_DECRYPT_STRING := UTL_I18N.RAW_TO_CHAR(V_DECRYPT_RAW, 'ZHS16GBK');
        -- RTRIM 去除字符串右侧的隐藏字符串 CHR(0)
        RETURN RTRIM(V_DECRYPT_STRING, CHR(0));
      ELSE
        RETURN INPUT_STRING;
      END IF;
    END IF;
  END;
END WX_CRYPTO;
```



#  使用

```sql
-- 使用默认密钥
SELECT WX_CRYPTO.DES_ENCRYPT('1234中文abc') ENC,
       WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_ENCRYPT('1234中文abc')) DEC,
       WX_CRYPTO.DES_ENCRYPT(WX_CRYPTO.DES_ENCRYPT('1234中文abc')) ENC2,
       WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_ENCRYPT('1234中文abc'))) DEC2,
       WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_ENCRYPT(WX_CRYPTO.DES_ENCRYPT('1234中文abc'))) DEC3_1,
       WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_ENCRYPT(WX_CRYPTO.DES_ENCRYPT('1234中文abc')))) DEC3_2
  FROM DUAL;

-- 使用指定密钥
SELECT WX_CRYPTO.DES_ENCRYPT('1234中文abc', '12345678') ENC,
       WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_ENCRYPT('1234中文abc',
                                                   '12345678'),
                             '12345678') DEC,
       WX_CRYPTO.DES_ENCRYPT(WX_CRYPTO.DES_ENCRYPT('1234中文abc',
                                                   '12345678'),
                             '12345678') ENC2,
       WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_ENCRYPT('1234中文abc',
                                                                         '12345678'),
                                                   '12345678'),
                             '12345678') DEC2,
       WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_ENCRYPT(WX_CRYPTO.DES_ENCRYPT('1234中文abc',
                                                                         '12345678'),
                                                   '12345678'),
                             '12345678') DEC3_1,
       WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_DECRYPT(WX_CRYPTO.DES_ENCRYPT(WX_CRYPTO.DES_ENCRYPT('1234中文abc',
                                                                                               '12345678'),
                                                                         '12345678'),
                                                   '12345678'),
                             '12345678') DEC3_2
  FROM DUAL;
```

