---
title: "发起HTTP请求：使用JDK11的HttpClient"
date: 2021-01-11T17:08:09+08:00
draft: false
categories: 
- Java
- HTTP
series:
- Java发起HTTP请求
tags:
- Java
- HTTP
- HttpClient
---

# 发起HTTP请求：使用JDK11的HttpClient

JDK11 API中原生的增加了HTTP客户端类[HttpClient](https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/HttpClient.html)，用于发起HTTP请求，支持HTTP/2.

以下内容非原创，参考/转载来源：https://www.dariawan.com/tutorials/java/java-11-standard-http-client-vs-apache-httpclient/

JDK11的HttpClient使用测试：

```java
package com.wangjm.http.connection;

import com.google.common.net.UrlEscapers;
import org.junit.jupiter.api.Test;

import java.net.ProxySelector;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;

/**
 * @author : wangjm
 * @date : 2021/1/11 20:47
 * JDK11 API 提供了HttpClient，支持HTTP/2
 * 简单使用的情况下，http和https链接的请求都是相同的
 */
class JDKHttpClientUtilTest {
    @Test
    public void doTest() throws Exception {
        final HttpClient HTTP_CLIENT = HttpClient.newBuilder()
                .version(HttpClient.Version.HTTP_2) // default
                .followRedirects(HttpClient.Redirect.NORMAL) // Always redirect, except from HTTPS URLs to HTTP URLs.
                .proxy(ProxySelector.getDefault())
                .build();
        final HttpResponse.BodyHandler<String> AS_STRING = HttpResponse.BodyHandlers.ofString();

        //一个获取utc时间的REST接口
        final String URL_OFC_GET_UTC_DATE = "https://www.onlinefreeconverter.com/api/v1/getutcdate";
        // 构建发起获取UTC Date的请求
        var reqGetUtcDate = HttpRequest.newBuilder()
                .uri(URI.create(URL_OFC_GET_UTC_DATE))
                .timeout(Duration.ofSeconds(10))
                .header("Content-Type", "application/json") //.GET()  //默认是get方式发起请求
                .build();
        //用HttpClient的实例发起请求
        var resGetUtcDate = HTTP_CLIENT.send(reqGetUtcDate, AS_STRING);
        var resGetUtcDateStatusCode = resGetUtcDate.statusCode();
        System.out.println("<<< Java 11 REST API Call: Get UTC Date >>>");
        if (resGetUtcDateStatusCode == 200 || resGetUtcDateStatusCode == 201) {
            System.out.println("Success!\n" + resGetUtcDate.body());
        } else {
            System.out.println("Failure! Status Code: " + resGetUtcDateStatusCode);
        }

        //一个base64加密字符串的接口（注意：这里是https)
        final String URL_BASE64CODE_ENCODE = "https://www.base64code.com/api/v1/base64/encode/";
        //需要加密的字符串
        final String STR_BASE64CODE_ENCODE_INPUT = "Dariawan | Java Blog and Tutorials";
        //URLENCODE
        var inputUrlEncoded = UrlEscapers.urlFragmentEscaper().escape(STR_BASE64CODE_ENCODE_INPUT);
        //构建请求
        var reqBase64Encode = HttpRequest.newBuilder()
                .uri(URI.create(//Set the appropriate endpoint
                        new StringBuilder(URL_BASE64CODE_ENCODE)
                                .append(inputUrlEncoded)
                                .toString()))
                .timeout(Duration.ofSeconds(10))
                .header("Content-Type", "application/json")
                .build();
        //使用HttpClient实例发起请求，注意：在HTTP/2版本下，同一个HttpClient是可以复用的（上面已经用过一次了）
        var resBase64Encode = HTTP_CLIENT.send(reqBase64Encode, AS_STRING);
        var resBase64EncodeStatusCode = resBase64Encode.statusCode();
        System.out.println("\n<<< Java 11 REST API Call: Base64 Encode >>>");
        if (resBase64EncodeStatusCode == 200 || resBase64EncodeStatusCode == 201) {
            System.out.println("Success!\n" + resBase64Encode.body());
        } else {
            System.out.println("Failure! Status Code: " + resBase64EncodeStatusCode);
        }

        //一个base64解密的REST接口
        final String URL_BASE64CODE_DECODE = "https://www.base64code.com/api/v1/base64/decode/";
        //需要解密的base64字串
        final String STR_BASE64CODE_DECODE_INPUT = "RGFyaWF3YW4gfCBKYXZhIEJsb2cgYW5kIFR1dG9yaWFscw==";
        //构建HTTP请求内容
        var reqBase64Decode = HttpRequest.newBuilder()
                .uri(URI.create(//Set the appropriate endpoint
                        new StringBuilder(URL_BASE64CODE_DECODE)
                                .append(STR_BASE64CODE_DECODE_INPUT)
                                .toString()))
                .timeout(Duration.ofSeconds(10))
                .header("Content-Type", "application/json")
                .build();
        //使用HttpClient实例发起请求，注意：在HTTP/2版本下，同一个HttpClient是可以复用的（上面已经用过一次了）
        var resBase64Decode = HTTP_CLIENT.send(reqBase64Decode, AS_STRING);
        var resBase64DecodeStatusCode = resBase64Decode.statusCode();
        System.out.println("\n<<< Java 11 REST API Call: Base64 Decode >>>");
        if (resBase64DecodeStatusCode == 200 || resBase64DecodeStatusCode == 201) {
            System.out.println("Success!\n" + resBase64Decode.body());
        } else {
            System.out.println("Failure! Status Code: " + resBase64DecodeStatusCode);
        }
    }
}
```

