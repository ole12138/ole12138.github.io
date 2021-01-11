---
title: "Java发起HTTP请求：使用Apache的HttpClient"
date: 2021-01-11T14:34:53+08:00
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

# Java发起HTTP请求：使用Apache的HttpClient

许多小伙伴都喜欢用apache的HttpComponts里的HttpClient来向远程发起HTTP请求

从[HttpComponents官网](#https://hc.apache.org/httpcomponents-client-ga/)的features说明来看，当前版本(HttpClient4.5)**仅支持到HTTP/1.1**。
但是HttpCore.5.1BETA将支持HTTP/2，可惜还在BETA阶段。

使用方式：

项目中引入对应的maven依赖：

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.13</version>
</dependency>
```

简单使用：
[参考](https://zhuanlan.zhihu.com/p/69285935)：

```java
public class HttpClientUtil {
    public static String doGet(String url) {
        String result = "";
        try {
            // 通过址默认配置创建一个httpClient实例
            @Cleanup CloseableHttpClient httpClient = HttpClients.createDefault();
            // 创建httpGet远程连接实例
            HttpGet httpGet = new HttpGet(url);
            // 设置请求头信息，鉴权
            httpGet.setHeader("Authorization", "Bearer da3efcbf-0845-4fe3-8aba-ee040be542c0");
            // 设置配置请求参数
            RequestConfig requestConfig = RequestConfig.custom()
                    // 连接主机服务超时时间
                    .setConnectTimeout(35000)
                    // 请求超时时间
                    .setConnectionRequestTimeout(35000)
                    // 数据读取超时时间
                    .setSocketTimeout(60000)
                    .build();
            // 为httpGet实例设置配置
            httpGet.setConfig(requestConfig);
            // 执行get请求得到返回对象
            @Cleanup CloseableHttpResponse response = httpClient.execute(httpGet);
            // 通过返回对象获取返回数据
            HttpEntity entity = response.getEntity();
            // 通过EntityUtils中的toString方法将结果转换为字符串
            result = EntityUtils.toString(entity);
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }

    public static String doPost(String url, Map<String, Object> paramMap) {
        String result = "";
        try {
            // 创建httpClient实例
            @Cleanup CloseableHttpClient httpClient = HttpClients.createDefault();
            // 创建httpPost远程连接实例
            HttpPost httpPost = new HttpPost(url);
            // 配置请求参数实例
            RequestConfig requestConfig = RequestConfig.custom()
                    // 设置连接主机服务超时时间
                    .setConnectTimeout(35000)
                    // 设置连接请求超时时间
                    .setConnectionRequestTimeout(35000)
                    // 设置读取数据连接超时时间
                    .setSocketTimeout(60000)
                    .build();
            // 为httpPost实例设置配置
            httpPost.setConfig(requestConfig);
            // 设置请求头
            httpPost.setHeader("Content-Type", "application/x-www-form-urlencoded");
            httpPost.setHeader("Cookie", "id=123;token=234");
            // 封装post请求参数
            if (null != paramMap && paramMap.size() > 0) {
                //将传入post参数，转换为HttpClient可以接受的形式
                List<NameValuePair> nvps = new ArrayList<NameValuePair>();
                // 通过map集成entrySet方法获取entity
                Set<Map.Entry<String, Object>> entrySet = paramMap.entrySet();
                // 循环遍历，获取迭代器
                Iterator<Map.Entry<String, Object>> iterator = entrySet.iterator();
                while (iterator.hasNext()) {
                    Map.Entry<String, Object> mapEntry = iterator.next();
                    nvps.add(new BasicNameValuePair(mapEntry.getKey(), mapEntry.getValue().toString()));
                }
                try {
                    // 为httpPost设置封装好的请求参数
                    httpPost.setEntity(new UrlEncodedFormEntity(nvps, "UTF-8"));
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            }

            // httpClient对象执行post请求,并返回响应参数对象
            @Cleanup CloseableHttpResponse httpResponse = httpClient.execute(httpPost);
            // 从响应对象中获取响应内容
            HttpEntity entity = httpResponse.getEntity();
            result = EntityUtils.toString(entity);
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }
}
```

测试：

```java
class HttpClientUtilTest {

    @Test
    void doGetTest() {
        String url = "http://localhost:8081/get?name=NAME1";
        String res = HttpClientUtil.doGet(url);
        System.out.println(res);
    }

    @Test
    void doPostTest() {
        String url = "http://localhost:8081/post";
        Map<String, Object> params = new HashMap<>(8);
        params.put("name", "Name2");
        params.put("age", "32");
        String res = HttpClientUtil.doPost(url, params);
        System.out.println(res);
    }
}
```

[代码存放仓库](https://github.com/ole12138/myutils)

# 进一步阅读

[Apache HttpComponents官网](https://hc.apache.org/httpcomponents-client-ga/)

[HttpClient教程](https://www.yiibai.com/apache_httpclient/apache_httpclient_http_get_request.html)