---
title: IP 操作工具类
categories:
  - Java
tags:
  - Java
abbrlink: 27e8862d
date: 2018-04-14 21:47:17
---
# IpUtils.java
```java
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.LineNumberReader;
import java.net.InetAddress;
import java.net.UnknownHostException;

/**
 * IP 操作工具类
 *
 * @author Created by YL on 2018/4/4
 */
public class IpUtils {

    /**
     * 获取 IP
     * <pre>
     *     1、如果使用ngnix反向代理，导致不能使用HttpServletRequest.getRemoteAddr()获取来访者真实ip的问题，
     *     2、对于通过多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割
     * </pre>
     */
    public static String getIpAddr(HttpServletRequest request) {
        String ipAddr = request.getHeader("x-forwarded-for");
        if (ipAddr == null || ipAddr.length() == 0 || "unknown".equalsIgnoreCase(ipAddr)) {
            ipAddr = request.getHeader("Proxy-Client-IP");
        }
        if (ipAddr == null || ipAddr.length() == 0 || "unknown".equalsIgnoreCase(ipAddr)) {
            ipAddr = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ipAddr == null || ipAddr.length() == 0 || "unknown".equalsIgnoreCase(ipAddr)) {
            ipAddr = request.getRemoteAddr();
            if ("127.0.0.1".equals(ipAddr) || "0:0:0:0:0:0:0:1".equals(ipAddr)) {
                //根据网卡取本机配置的IP
                InetAddress inet = null;
                try {
                    inet = InetAddress.getLocalHost();
                    ipAddr = inet.getHostAddress();
                } catch (UnknownHostException e) {
                    e.printStackTrace();
                }
            }
        }
        // 对于通过多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割
        // 没有考虑ipv6
        // if (ipAddr != null && ipAddr.length() > 15) { //"***.***.***.***".length() = 15
        //     if (ipAddr.indexOf(",") > 0) {
        //         ipAddr = ipAddr.substring(0, ipAddr.indexOf(","));
        //     }
        // }
        if (ipAddr != null && ipAddr.contains(",")) {
            ipAddr = ipAddr.substring(0, ipAddr.indexOf(","));
        }
        return ipAddr;
    }

    /**
     * 根据 IP 获得 MAC 地址
     *
     * @param ip IP 地址
     */
    public static String getMacAddr(String ip) {
        String macAddr = "";
        InputStreamReader isr = null;
        LineNumberReader reader = null;
        try {
            Process p = Runtime.getRuntime().exec("nbtstat -A " + ip);
            isr = new InputStreamReader(p.getInputStream());
            reader = new LineNumberReader(isr);
            for (int i = 1; i < 100; i++) {
                String str = reader.readLine();
                if (str != null) {
                    if (str.indexOf("MAC Address") > 1) {
                        macAddr = str.substring(str.indexOf("MAC Address") + 14, str.length());
                        break;
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (isr != null) {
                try {
                    isr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return macAddr;
    }
}
```
