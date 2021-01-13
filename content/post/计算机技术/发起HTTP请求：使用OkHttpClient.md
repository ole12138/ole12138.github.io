---
title: "发起HTTP请求：使用OkHttpClient"
date: 2021-01-13T15:54:12+08:00
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

# 发起HTTP请求：使用OkHttpClient

现在java比较好用的http客户端是开源的OkHttp。支持HTTP/2, 简单易用且灵活。

官网：https://square.github.io/okhttp/

简单使用测试（http、https类型的get，post）：

```java
class OkHttpClientUtilTest {
    @Test
    public void doGet() throws IOException {
        String url = "http://httpbin.org/anything";
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
                .url(url)
                .build();
        try (Response response = client.newCall(request).execute()) {
            String res = response.body().string();
            System.out.println(res);
        }

        request = new Request.Builder()
                .get()
                .url(url + "?key1=value1&key2=value2").header("Cookie", "cook1=sd;cook2=sdd;").build();
        try (Response response = client.newCall(request).execute()) {
            String res = response.body().string();
            System.out.println(res);
        }

        url = "https://httpbin.org/anything";
        request = new Request.Builder()
                .url(url)
                .build();
        try (Response response = client.newCall(request).execute()) {
            String res = response.body().string();
            System.out.println(res);
        }
    }

    @Test
    public void doPost() throws IOException {
        OkHttpClient client = new OkHttpClient();

        String url = "http://httpbin.org/anything";

        JsonMapper mapper = new JsonMapper();
        ObjectNode node = mapper.createObjectNode();
        node.put("key1", "value1");
        node.put("key2", "value2");
        String jsonStr = node.toString();

        RequestBody body = RequestBody.create(jsonStr, MediaType.get("application/json; charset=utf-8"));
        Request request = new Request.Builder()
                .url(url)
                .post(body)
                .build();
        try (Response response = client.newCall(request).execute()) {
            String res = response.body().string();
            System.out.println(res);
        }

        url = "https://httpbin.org/anything";
        request = new Request.Builder()
                .url(url)
                .post(body)
                .build();
        try (Response response = client.newCall(request).execute()) {
            String res = response.body().string();
            System.out.println(res);
        }
    }
}
```

更详细的用法还是要参考官网的文档。