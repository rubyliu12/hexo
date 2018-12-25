---
title: Oracle DES 加解密
categories:
  - Oracle
tags:
  - DES
abbrlink: 2b37b6b3
date: 2018-12-04 10:30:05
---



# DES encrypt

```sql
CREATE OR REPLACE FUNCTION DES_ENCRYPT(P_TEXT VARCHAR2, -- 明文。长度要是 8 的倍数，若不足 8 的倍数，则用空白补足
                                       P_KEY  VARCHAR2) -- 密钥。长度最少要 8 位以上; 不同的 Key，加密结果将会不同
 RETURN VARCHAR2 IS
  V_TEXT    VARCHAR2(4000); -- 不足 8 的倍数，补足后的字符串
  V_ENCRYPT RAW(2048); -- 加密后的字符串
BEGIN
  -- 补足。CHR(0) 隐藏字符串
  V_TEXT    := RPAD(P_TEXT, (TRUNC(LENGTHB(P_TEXT) / 8) + 1) * 8, CHR(0));
  V_ENCRYPT := DBMS_OBFUSCATION_TOOLKIT.DESENCRYPT(INPUT => UTL_I18N.STRING_TO_RAW(V_TEXT,
                                                                                   'ZHS16GBK'), -- 转换成 16 进制
                                                   KEY   => UTL_I18N.STRING_TO_RAW(P_KEY,
                                                                                   'ZHS16GBK')); -- 转换成 16 进制
  RETURN RAWTOHEX(V_ENCRYPT);
END;

```



<!-- more -->



# DES decrypt

```sql
CREATE OR REPLACE FUNCTION DES_DECRYPT(P_TEXT VARCHAR2, -- 密文
                                       P_KEY  VARCHAR2) -- 密钥
 RETURN VARCHAR2 IS
  V_DECRYPT RAW(2048); -- 解密后的明文
BEGIN
  IF (P_TEXT IS NULL OR P_TEXT = '') THEN
    RETURN '';
  END IF;
  V_DECRYPT := DBMS_OBFUSCATION_TOOLKIT.DESDECRYPT(INPUT => HEXTORAW(P_TEXT),
                                                   KEY   => UTL_I18N.STRING_TO_RAW(P_KEY,
                                                                                   'ZHS16GBK')); -- 需转换成 16 进制
  -- RTRIM 去除字符串右侧空白：CHR(0)
  -- RAW_TO_CHAR 转换成字符串
  RETURN RTRIM(UTL_I18N.RAW_TO_CHAR(V_DECRYPT, 'ZHS16GBK'), CHR(0));
END;

```

