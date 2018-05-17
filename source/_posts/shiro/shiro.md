---
title: shiro密码加密
categories:
  - Java
tags:
  - shiro
abbrlink: 53575a2a
date: 2017-08-16 14:07:09
---

shiro密码加密
@Bean
public MyShiroRealm myShiroRealm(){
    MyShiroRealm myShiroRealm = new MyShiroRealm();
    myShiroRealm.setCredentialsMatcher(hashedCredentialsMatcher());
    return myShiroRealm;
}

@Bean
public HashedCredentialsMatcher hashedCredentialsMatcher(){
    HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
    hashedCredentialsMatcher.setHashAlgorithmName("md5");//散列算法:这里使用MD5算法;
    hashedCredentialsMatcher.setHashIterations(2);//散列的次数，比如散列两次，相当于 md5(md5(""));
    return hashedCredentialsMatcher;
}

public void encryptPassword(User user) {
    String newPassword = new SimpleHash(algorithmName, user.getPassword(),  ByteSource.Util.bytesuser.getUsername()), hashIterations).toHex();
    user.setPassword(newPassword);
}
