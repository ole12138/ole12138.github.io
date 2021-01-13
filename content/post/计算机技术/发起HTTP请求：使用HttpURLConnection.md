---
title: "发起HTTP请求：使用HttpURLConnection"
date: 2021-01-13T13:25:34+08:00
draft: false
categories: 
- Java
- HTTP
- TODO
series:
- Java发起HTTP请求
tags:
- Java
- HTTP
- HttpURLConnection
- HttpsURLConnection
---

# 发起HTTP请求：使用HttpURLConnection

使用JDK原生的HttpURLConnection发起HTTP请求
 * 注意：到Java8为止，HttpURLConnection不支持HTTP2，只支持到HTTP1.1。
 * 要使用支持HTTP2的HttpURLConnection，需要升级到Java11

```java
import lombok.Cleanup;

import java.io.*;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.nio.charset.StandardCharsets;

/**
 * 使用JDK原生的HttpURLConnection发起HTTP请求
 */
public class HttpURLConnectionUtil {
    public static String doGet(String httpurl) {
        // 返回结果字符串
        String result = null;
        try {
            // 创建远程url连接对象
            URL url = new URL(httpurl);
            // 通过远程url连接对象打开一个连接，强转成httpURLConnection类
            @Cleanup("disconnect") HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            // 设置连接方式：get
            connection.setRequestMethod("GET");
            // 设置连接主机服务器的超时时间：15000毫秒
            connection.setConnectTimeout(15000);
            // 设置读取远程返回的数据时间：60000毫秒
            connection.setReadTimeout(60000);
            // 发送请求
            connection.connect();
            // 通过connection连接，获取输入流
            if (connection.getResponseCode() == 200) {
                @Cleanup InputStream is = connection.getInputStream();
                // 封装输入流is，并指定字符集
                @Cleanup BufferedReader br = new BufferedReader(new InputStreamReader(is, StandardCharsets.UTF_8));
                // 存放数据
                StringBuffer sbf = new StringBuffer();
                String temp = null;
                while ((temp = br.readLine()) != null) {
                    sbf.append(temp);
                    sbf.append("\r\n");
                }
                result = sbf.toString();
            }
        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }

    public static String doPost(String httpUrl, String param) {
        String result = null;
        try {
            URL url = new URL(httpUrl);
            // 通过远程url连接对象打开连接
            @Cleanup("disconnect") HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            // 设置连接请求方式
            connection.setRequestMethod("POST");
            // 设置连接主机服务器超时时间：15000毫秒
            connection.setConnectTimeout(15000);
            // 设置读取主机服务器返回数据超时时间：60000毫秒
            connection.setReadTimeout(60000);
            // 默认值为：false，当向远程服务器传送数据/写数据时，需要设置为true
            connection.setDoOutput(true);
            // 默认值为：true，当前向远程服务读取数据时，设置为true，该参数可有可无
            connection.setDoInput(true);
            // 设置传入参数的格式:请求参数应该是 name1=value1&name2=value2 的形式。
            connection.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
            // 设置鉴权信息：Authorization: Bearer da3efcbf-0845-4fe3-8aba-ee040be542c0
            connection.setRequestProperty("Authorization", "Bearer da3efcbf-0845-4fe3-8aba-ee040be542c0");
            // 设置Cookies
            connection.setRequestProperty("Cookie", "id=12235;token=sdffwe233;");
            // 通过连接对象获取一个输出流
            @Cleanup OutputStream os = connection.getOutputStream();
            // 通过输出流对象将参数写出去/传输出去,它是通过字节数组写出的
            os.write(param.getBytes());
            // 通过连接对象获取一个输入流，向远程读取
            if (connection.getResponseCode() == 200) {
                @Cleanup InputStream is = connection.getInputStream();
                // 对输入流对象进行包装:charset根据工作项目组的要求来设置
                @Cleanup BufferedReader br = new BufferedReader(new InputStreamReader(is, StandardCharsets.UTF_8));
                StringBuffer sbf = new StringBuffer();
                String temp = null;
                // 循环遍历一行一行读取数据
                while ((temp = br.readLine()) != null) {
                    sbf.append(temp);
                    sbf.append("\r\n");
                }
                result = sbf.toString();
            }
        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }
}
```

# 发起HTTPS请求：使用HttpsURLConnection

 * HTTPS的请求只需要将HttpURLConnection换为HttpsURLConnection即可
 * 如果要信任非CA可验证的证书，比如信任自定义证书或所有证书，可参考如下链接:
    * https://stackoverflow.com/questions/18918662/httpurlconnection-https-vs-http
    * https://blog.csdn.net/zz153417230/article/details/80271155
    * https://www.codota.com/code/java/methods/javax.net.ssl.SSLSocketFactory/%3Cinit%3E  TODO
    * https://www.codota.com/code/java/methods/javax.net.ssl.HttpsURLConnection/setSSLSocketFactory   TODO
    * x509协议的学习以及自定义证书验证  TODO

```java
import lombok.Cleanup;

import javax.net.ssl.HttpsURLConnection;
import java.io.*;
import java.net.MalformedURLException;
import java.net.URL;
import java.nio.charset.StandardCharsets;

/**
 * @author : wangjm
 * @date : 2021/1/12 16:37
 */
public class HttpsURLConnectionUtil {
    public static String doGet(String httpurl) {
        // 返回结果字符串
        String result = null;
        try {
            // 创建远程url连接对象
            URL url = new URL(httpurl);
            // 通过远程url连接对象打开一个连接，强转成httpURLConnection类
            @Cleanup("disconnect") HttpsURLConnection connection = (HttpsURLConnection) url.openConnection();
            // 设置连接方式：get
            connection.setRequestMethod("GET");
            // 设置连接主机服务器的超时时间：15000毫秒
            connection.setConnectTimeout(15000);
            // 设置读取远程返回的数据时间：60000毫秒
            connection.setReadTimeout(60000);
            // 发送请求
            connection.connect();
            // 通过connection连接，获取输入流
            if (connection.getResponseCode() == 200) {
                @Cleanup InputStream is = connection.getInputStream();
                // 封装输入流is，并指定字符集
                @Cleanup BufferedReader br = new BufferedReader(new InputStreamReader(is, StandardCharsets.UTF_8));
                // 存放数据
                StringBuffer sbf = new StringBuffer();
                String temp = null;
                while ((temp = br.readLine()) != null) {
                    sbf.append(temp);
                    sbf.append("\r\n");
                }
                result = sbf.toString();
            }
        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }
    public static String doPost(String urlPath, String params) {
        String result = null;
        try {
            URL url = new URL(urlPath);
            HttpsURLConnection conn = (HttpsURLConnection) url.openConnection();
            //conn.setSSLSocketFactory(getSSLContext(context).getSocketFactory());
            // 设置连接请求方式
            conn.setRequestMethod("POST");
            // 设置连接主机服务器超时时间：15000毫秒
            conn.setConnectTimeout(15000);
            // 设置读取主机服务器返回数据超时时间：60000毫秒
            conn.setReadTimeout(60000);
            // 默认值为：false，当向远程服务器传送数据/写数据时，需要设置为true
            conn.setDoOutput(true);
            // 默认值为：true，当前向远程服务读取数据时，设置为true，该参数可有可无
            conn.setDoInput(true);
            // 设置传入参数的格式:请求参数应该是 name1=value1&name2=value2 的形式。
            conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
            // 设置鉴权信息：Authorization: Bearer da3efcbf-0845-4fe3-8aba-ee040be542c0
            conn.setRequestProperty("Authorization", "Bearer da3efcbf-0845-4fe3-8aba-ee040be542c0");
            // 设置Cookies
            conn.setRequestProperty("Cookie", "id=12235;token=sdffwe233;");
            conn.setRequestProperty("User-Agent", "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.2; Trident/4.0; " +
                    ".NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3" +
                    ".5.30729)");
            conn.setRequestProperty("Charset", "UTF-8");
            @Cleanup OutputStream os = conn.getOutputStream();
            os.write(params.getBytes());

            /* 服务器返回的响应码 */
            int code = conn.getResponseCode();
            if (code == 200) {
                @Cleanup InputStream is = conn.getInputStream();
                @Cleanup BufferedReader br = new BufferedReader(new InputStreamReader(is, StandardCharsets.UTF_8));
                StringBuffer sbf = new StringBuffer();
                String temp = null;
                // 循环遍历一行一行读取数据
                while ((temp = br.readLine()) != null) {
                    sbf.append(temp);
                    sbf.append("\r\n");
                }
                result = sbf.toString();
            }
        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
}
```