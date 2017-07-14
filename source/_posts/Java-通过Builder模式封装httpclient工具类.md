---
title: 通过Builder模式封装httpclient工具类
date: 2017-05-23 15:22:01
categories: [Java]
tags: [apache, httpclient, httpcore]
description:
permalink: httpclient-builder
---

依赖包
```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5</version>
</dependency>
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpcore</artifactId>
    <version>4.4.1</version>
</dependency>
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.9</version>
</dependency>
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.21</version>
</dependency>
```
<!-- more -->
具体代码实现
```java
import org.apache.http.HttpEntity;
import org.apache.http.HttpStatus;
import org.apache.http.NameValuePair;
import org.apache.http.ParseException;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.config.CookieSpecs;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.methods.HttpUriRequest;
import org.apache.http.config.ConnectionConfig;
import org.apache.http.config.MessageConstraints;
import org.apache.http.config.Registry;
import org.apache.http.config.RegistryBuilder;
import org.apache.http.config.SocketConfig;
import org.apache.http.conn.HttpConnectionFactory;
import org.apache.http.conn.ManagedHttpClientConnection;
import org.apache.http.conn.routing.HttpRoute;
import org.apache.http.conn.socket.ConnectionSocketFactory;
import org.apache.http.conn.socket.PlainConnectionSocketFactory;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.ManagedHttpClientConnectionFactory;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.ssl.SSLContexts;
import org.apache.http.util.Args;
import org.apache.http.util.EntityUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.net.ssl.SSLContext;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.nio.charset.Charset;
import java.nio.charset.CodingErrorAction;
import java.util.List;

/**
 * http client工具类
 * <p>推荐使用成员变量，或者spring bean的方式初始化对象，这样可以有效的使用到连接池管理器
 * <pre>
 * socketTimeout：socketTimeout超时时间。默认：8000（单位：ms）
 * connectTimeout：connectTimeout超时时间。默认：8000（单位：ms）
 * connectionRequestTimeout：connectionRequestTimeout超时时间。默认：10*1000（单位：ms）
 * ignoreCookies：是否忽略cookies。默认：true
 * tcpNoDelay：是否使用套接字配置。默认：true
 * validateAfterInactivity：在不活动时验证连接间隔。默认：30 * 1000（单位：ms）
 * maxHeaderCount：最大header信息条数。默认：200
 * setMaxLineLength：最大头部每行长度。默认：2000
 * maxTotal：配置连接池最大连接数。默认：150
 * defaultMaxPerRoute：每个路由最大租用连接数量。默认：10
 * charset：编码格式。默认：UTF-8
 * </pre>
 *
 * @author Created by YL on 2017/5/23.
 */
public class Client {
    private final int socketTimeout;
    private final int connectTimeout;
    private final int connectionRequestTimeout;
    private final boolean ignoreCookies;
    private final boolean tcpNoDelay;
    private final int validateAfterInactivity;
    private final int maxHeaderCount;
    private final int setMaxLineLength;
    private final int maxTotal;
    private final int defaultMaxPerRoute;
    private final String charset;

    private final Logger logger = LoggerFactory.getLogger(Client.class);
    private CloseableHttpClient client;

    private Client(final int socketTimeout, final int connectTimeout, final int connectionRequestTimeout, final
    boolean ignoreCookies, final boolean tcpNoDelay, final int validateAfterInactivity, final int maxHeaderCount,
                   final int setMaxLineLength, final int maxTotal, final int defaultMaxPerRoute, final String charset) {
        super();
        this.socketTimeout = socketTimeout;
        this.connectTimeout = connectTimeout;
        this.connectionRequestTimeout = connectionRequestTimeout;
        this.ignoreCookies = ignoreCookies;
        this.tcpNoDelay = tcpNoDelay;
        this.validateAfterInactivity = validateAfterInactivity;
        this.maxHeaderCount = maxHeaderCount;
        this.setMaxLineLength = setMaxLineLength;
        this.maxTotal = maxTotal;
        this.defaultMaxPerRoute = defaultMaxPerRoute;
        this.charset = charset;

        /*
         * RequestConfig
         */
        RequestConfig.Builder builder = RequestConfig.custom();
        builder.setSocketTimeout(this.getSocketTimeout());
        builder.setConnectTimeout(this.getConnectTimeout());
        builder.setConnectionRequestTimeout(this.getConnectionRequestTimeout());
        if (this.isIgnoreCookies()) {
            builder.setCookieSpec(CookieSpecs.IGNORE_COOKIES);
        }
        // builder.setExpectContinueEnabled(true);
        // builder.setTargetPreferredAuthSchemes(Arrays.asList(AuthSchemes.NTLM,
        // AuthSchemes.DIGEST));
        // builder.setProxyPreferredAuthSchemes(Arrays.asList(AuthSchemes.BASIC));
        // HttpHost proxy = new HttpHost("10.121.2.5", 3128);
        // builder.setProxy(proxy);
        RequestConfig rc = builder.build();

        /*
         * Use a custom connection factory to customize the process of
         * initialization of outgoing HTTP connections. Beside standard
         * connection configuration parameters HTTP connection factory can
         * define message parser / writer routines to be employed by individual
         * connections.
         */
        HttpConnectionFactory<HttpRoute, ManagedHttpClientConnection> cf = new ManagedHttpClientConnectionFactory();

        /*
         * SSL context for secure connections can be created either based on
         * system or application specific properties.
         */
        SSLContext sslc = SSLContexts.createSystemDefault();

        /*
         * Create a registry of custom connection socket factories for supported
         * protocol schemes.
         */
        Registry<ConnectionSocketFactory> sfr = RegistryBuilder.<ConnectionSocketFactory>create().register("http",
                PlainConnectionSocketFactory.INSTANCE).register("https", new SSLConnectionSocketFactory(sslc)).build();

        /*
         * Create socket configuration
         */
        SocketConfig sc = SocketConfig.custom().setTcpNoDelay(this.isTcpNoDelay()).build();

        /*
         * Create a connection manager with custom configuration.
         */
        PoolingHttpClientConnectionManager manager = new PoolingHttpClientConnectionManager(sfr, cf);
        // 将连接管理器配置为使用套接字配置（默认情况下或特定主机）。
        manager.setDefaultSocketConfig(sc);
        // manager.setSocketConfig(new HttpHost("somehost", 80),
        // socketConfig);
        // 在不活动30秒后验证连接
        manager.setValidateAfterInactivity(this.getValidateAfterInactivity());

        /*
         * Create message constraints
         */
        MessageConstraints mc = MessageConstraints.custom().setMaxHeaderCount(this.getMaxHeaderCount())
                .setMaxLineLength(this.getSetMaxLineLength()).build();

        /*
         * Create connection configuration
         */
        ConnectionConfig cc = ConnectionConfig.custom().setMalformedInputAction(CodingErrorAction.IGNORE)
                .setUnmappableInputAction(CodingErrorAction.IGNORE).setCharset(Charset.forName(this.getCharset()))
                .setMessageConstraints(mc).build();
        // 将连接管理器配置为使用默认情况下或特定主机的连接配置。
        manager.setDefaultConnectionConfig(cc);
        // manager.setConnectionConfig(new HttpHost("somehost", 80),
        // ConnectionConfig.DEFAULT);

        // 配置连接池最大连接数，和每个路由最大租用连接数量
        manager.setMaxTotal(this.getMaxTotal());
        manager.setDefaultMaxPerRoute(this.getDefaultMaxPerRoute());

        /*
         * 4.3以上版本推荐使用HttpClientBuilder来创建client
         */
        HttpClientBuilder clientBuilder = HttpClients.custom();
        clientBuilder.setConnectionManager(manager);
        // clientBuilder.setDefaultCookieStore(cookieStore);
        // clientBuilder.setDefaultCredentialsProvider(credentialsProvider);
        // clientBuilder.setProxy(new HttpHost("myproxy", 8080));
        clientBuilder.setDefaultRequestConfig(rc);
        client = clientBuilder.build();
    }

    public static Builder custom() {
        return new Builder();
    }

    public static Builder copy(final Client copy) {
        return new Builder().socketTimeout(copy.socketTimeout).connectTimeout(copy.connectTimeout)
                .connectionRequestTimeout(copy.connectionRequestTimeout).ignoreCookies(copy.ignoreCookies).tcpNoDelay
                        (copy.tcpNoDelay).validateAfterInactivity(copy.validateAfterInactivity).maxHeaderCount(copy
                        .maxHeaderCount).setMaxLineLength(copy.setMaxLineLength).maxTotal(copy.maxTotal)
                .defaultMaxPerRoute(copy.defaultMaxPerRoute).charset(copy.charset);
    }

    public int getSocketTimeout() {
        return socketTimeout;
    }

    public int getConnectTimeout() {
        return connectTimeout;
    }

    public int getConnectionRequestTimeout() {
        return connectionRequestTimeout;
    }

    public boolean isIgnoreCookies() {
        return ignoreCookies;
    }

    public boolean isTcpNoDelay() {
        return tcpNoDelay;
    }

    public int getValidateAfterInactivity() {
        return validateAfterInactivity;
    }

    public int getMaxHeaderCount() {
        return maxHeaderCount;
    }

    public int getSetMaxLineLength() {
        return setMaxLineLength;
    }

    public int getMaxTotal() {
        return maxTotal;
    }

    public int getDefaultMaxPerRoute() {
        return defaultMaxPerRoute;
    }

    public String getCharset() {
        return charset;
    }

    @Override
    public String toString() {
        StringBuilder builder = new StringBuilder();
        builder.append("TSHttpClient [socketTimeout=").append(socketTimeout).append(", connectTimeout=").append
                (connectTimeout).append(", connectionRequestTimeout=").append(connectionRequestTimeout).append(", " +
                "ignoreCookies=").append(ignoreCookies).append(", tcpNoDelay=").append(tcpNoDelay).append(", " +
                "validateAfterInactivity=").append(validateAfterInactivity).append(", maxHeaderCount=").append
                (maxHeaderCount).append(", setMaxLineLength=").append(setMaxLineLength).append(", maxTotal=").append
                (maxTotal).append(", defaultMaxPerRoute=").append(defaultMaxPerRoute).append(", charset=").append
                (charset).append("]");
        return builder.toString();
    }

    public static class Builder {
        private int socketTimeout = 8000;
        private int connectTimeout = 8000;
        private int connectionRequestTimeout = 10 * 1000;
        private boolean ignoreCookies = true;
        private boolean tcpNoDelay = true;
        private int validateAfterInactivity = 30 * 1000;
        private int maxHeaderCount = 200;
        private int setMaxLineLength = 2000;
        private int maxTotal = 150;
        private int defaultMaxPerRoute = 10;
        private String charset = "UTF-8";

        Builder() {
        }

        public Builder socketTimeout(int socketTimeout) {
            this.socketTimeout = socketTimeout;
            return this;
        }

        public Builder connectTimeout(int connectTimeout) {
            this.connectTimeout = connectTimeout;
            return this;
        }

        public Builder connectionRequestTimeout(int connectionRequestTimeout) {
            this.connectionRequestTimeout = connectionRequestTimeout;
            return this;
        }

        public Builder ignoreCookies(boolean ignoreCookies) {
            this.ignoreCookies = ignoreCookies;
            return this;
        }

        public Builder tcpNoDelay(boolean tcpNoDelay) {
            this.tcpNoDelay = tcpNoDelay;
            return this;
        }

        public Builder validateAfterInactivity(int validateAfterInactivity) {
            this.validateAfterInactivity = validateAfterInactivity;
            return this;
        }

        public Builder maxHeaderCount(int maxHeaderCount) {
            this.maxHeaderCount = maxHeaderCount;
            return this;
        }

        public Builder setMaxLineLength(int setMaxLineLength) {
            this.setMaxLineLength = setMaxLineLength;
            return this;
        }

        public Builder maxTotal(int maxTotal) {
            this.maxTotal = maxTotal;
            return this;
        }

        public Builder defaultMaxPerRoute(int defaultMaxPerRoute) {
            this.defaultMaxPerRoute = defaultMaxPerRoute;
            return this;
        }

        public Builder charset(String charset) {
            this.charset = charset;
            return this;
        }

        public Client build() {
            return new Client(socketTimeout, connectTimeout, connectionRequestTimeout, ignoreCookies, tcpNoDelay,
                    validateAfterInactivity, maxHeaderCount, setMaxLineLength, maxTotal, defaultMaxPerRoute, charset);
        }
    }

    /**
     * 发送get请求
     *
     * @author YangLi : 2015年12月16日
     */
    public Response get(String url) throws HttpException {
        HttpGet httpGet = new HttpGet(url);
        return get(httpGet);
    }

    /**
     * 发送get请求
     *
     * @author YangLi : 2015年12月16日
     */
    public Response get(HttpGet httpGet) throws HttpException {
        return execute(httpGet);
    }

    /**
     * 发送post请求：不带输入数据
     *
     * @param url 地址
     * @throws HttpException
     * @author YangLi : 2016年3月24日
     */
    public Response post(String url) throws HttpException {
        HttpPost httpPost = new HttpPost(url);
        return post(httpPost);
    }

    /**
     * 发送post请求：使用UrlEncodedFormEntity模拟表单提交，K-V形式参数
     *
     * @param url     地址
     * @param nvpList 传值示例：<br/>
     *                List&lt;NameValuePair> nvpList = new ArrayList
     *                &lt;NameValuePair>();<br/>
     *                nvpList.add(new BasicNameValuePair(key, value));
     * @throws HttpException
     * @author YangLi : 2016年3月24日
     */
    public Response post(String url, List<NameValuePair> nvpList) throws HttpException {
        try {
            return post(url, new UrlEncodedFormEntity(nvpList, this.getCharset()));
        } catch (UnsupportedEncodingException e) {
            throw new HttpException("不支持的编码格式：" + this.getCharset(), e);
        }
    }

    /**
     * 发送post请求：使用StringEntity提交application/json格式，JSON形式
     *
     * @author YangLi : 2016年2月24日
     */
    public Response post(String url, String json) throws HttpException {
        return post(url, new StringEntity(json, ContentType.APPLICATION_JSON));
    }

    /**
     * 发送post请求
     * <p>
     * 1、发送json格式字符串（默认UTF-8编码）：new StringEntity(text, ContentType.APPLICATION_JSON)
     * <p>
     * 2、post提交表单：new UrlEncodedFormEntity(nvpList, this.getCharset())
     * <p>
     * 3、上传文件：
     * <pre>
     * MultipartEntityBuilder meBuilder = MultipartEntityBuilder.create();
     * // 带其他参数
     * for (NameValuePair nameValuePair : nvpList) {
     *  meBuilder.addPart(nameValuePair.getName(), new StringBody(nameValuePair.getValue(), ContentType.TEXT_PLAIN));
     * }
     * // 多个文件使用for循环，单个文件不需要for循环
     * for (File file : files) {
     *  FileBody fileBody = new FileBody(file);
     *  meBuilder.addPart("files", fileBody);
     * }
     * HttpEntity httpEntity = meBuilder.build();
     * </pre>
     *
     * @author YangLi : 2016年2月24日
     */
    public Response post(String url, HttpEntity entity) throws HttpException {
        HttpPost httpPost = new HttpPost(url);
        httpPost.setEntity(entity);
        return post(httpPost);
    }

    /**
     * 发送post请求
     *
     * @author YangLi : 2015年12月16日
     */
    private Response post(HttpPost httpPost) throws HttpException {
        return execute(httpPost);
    }

    private Response execute(HttpUriRequest request) throws HttpException {
        Args.notNull(request, "HTTP request");
        logDebug("method: {}, uri: {}", request.getMethod(), request.getURI());
        CloseableHttpResponse httpResponse = null;
        try {
            // HttpClientContext context = HttpClientContext.create();
            // httpResponse = client.execute(request, context);
            httpResponse = client.execute(request);
            int statusCode = httpResponse.getStatusLine().getStatusCode();
            logDebug("method: {}, uri: {}, statusCode: {}", request.getMethod(), request.getURI(), statusCode);
            if (statusCode != HttpStatus.SC_OK) {
                throw new HttpException(getCause(statusCode), statusCode);
            }
            HttpEntity entity = httpResponse.getEntity();
            String result = EntityUtils.toString(entity, this.getCharset());
            logDebug("method: {}, uri: {}, response: {}", request.getMethod(), request.getURI(), result);
            Response response = new Response();
            response.setResponseAsString(result);
            return response;
        } catch (ParseException e) {
            throw new HttpException("解析错误", e);
        } catch (ClientProtocolException e) {
            throw new HttpException("HTTP协议错误", e);
        } catch (IOException e) {
            throw new HttpException("I/O操作异常", e);
        } finally {
            if (httpResponse != null) {
                try {
                    httpResponse.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 通过statusCode获取错误信息
     *
     * @author YangLi : 2015年12月16日
     */
    private String getCause(int statusCode) {
        String message = "";
        switch (statusCode) {
            case HttpStatus.SC_NOT_MODIFIED:
                break;
            case HttpStatus.SC_BAD_REQUEST:
                message = "请求出错";
                break;
            case HttpStatus.SC_UNAUTHORIZED:
                message = "未授权";
                break;
            case HttpStatus.SC_FORBIDDEN:
                message = "拒绝访问";
                break;
            case HttpStatus.SC_NOT_FOUND:
                message = "请求的URI是无效的或者所请求的资源不存在";
                break;
            case HttpStatus.SC_NOT_ACCEPTABLE:
                message = "不接受请求";
                break;
            case HttpStatus.SC_INTERNAL_SERVER_ERROR:
                message = "内部服务器错误";
                break;
            case HttpStatus.SC_BAD_GATEWAY:
                message = "错误的网关";
                break;
            case HttpStatus.SC_SERVICE_UNAVAILABLE:
                message = "暂停服务";
                break;
            default:
                message = "其他错误";
        }
        return "[ " + statusCode + " ] " + message;
    }

    public void logDebug(String format, Object... arguments) {
        if (logger.isDebugEnabled()) {
            logger.debug(format, arguments);
        }
    }
}

```
