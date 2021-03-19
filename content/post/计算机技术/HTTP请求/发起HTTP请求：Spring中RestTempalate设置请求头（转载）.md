---
title: "发起HTTP请求：Spring中RestTemplate设置与携带请求头（转载）"
date: 2021-01-08T16:56:48+08:00
draft: false
categories: 
- Java
- HTTP
series:
- RestTemplate
- Java发起HTTP请求
tags:
- Java
- HTTP
- Java实现HTTP请求
- RestTemplate
---

# Rest设置请求头以及进一步配置

本章节“Rest设置与携带请求头”部分非原创，转载来源：https://juejin.cn/post/6844904202397827086

本节主要集中在如何携带自定义的请求头，如设置 User-Agent，携带 Cookie

- Get 携带请求头
- Post 携带请求头
- 拦截器方式设置统一请求头

## I. 项目搭建

### 1. 配置

借助 SpringBoot 搭建一个 SpringWEB 项目，提供一些用于测试的 REST 服务

- SpringBoot 版本: `2.2.1.RELEASE`
- 核心依赖: `spring-boot-stater-web`

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

为了后续输出的日志更直观，这里设置了一下日志输出格式，在配置文件`application.yml`中，添加

```yaml
logging:
  pattern:
    console: (%msg%n%n){blue}
```

### 2. Rest 服务

添加三个接口，分别提供 GET 请求，POST 表单，POST json 对象，然后返回请求头、请求参数、cookie，具体实现逻辑相对简单，也不属于本篇重点，因此不赘述说明

```java
@RestController
public class DemoRest {

    private String getHeaders(HttpServletRequest request) {
        Enumeration<String> headerNames = request.getHeaderNames();
        String name;

        JSONObject headers = new JSONObject();
        while (headerNames.hasMoreElements()) {
            name = headerNames.nextElement();
            headers.put(name, request.getHeader(name));
        }
        return headers.toJSONString();
    }

    private String getParams(HttpServletRequest request) {
        return JSONObject.toJSONString(request.getParameterMap());
    }

    private String getCookies(HttpServletRequest request) {
        Cookie[] cookies = request.getCookies();
        if (cookies == null || cookies.length == 0) {
            return "";
        }

        JSONObject ck = new JSONObject();
        for (Cookie cookie : cookies) {
            ck.put(cookie.getName(), cookie.getValue());
        }
        return ck.toJSONString();
    }

    private String buildResult(HttpServletRequest request) {
        return buildResult(request, null);
    }

    private String buildResult(HttpServletRequest request, Object obj) {
        String params = getParams(request);
        String headers = getHeaders(request);
        String cookies = getCookies(request);

        if (obj != null) {
            params += " | " + obj;
        }

        return "params: " + params + "\nheaders: " + headers + "\ncookies: " + cookies;
    }

    @GetMapping(path = "get")
    public String get(HttpServletRequest request) {
        return buildResult(request);
    }


    @PostMapping(path = "post")
    public String post(HttpServletRequest request) {
        return buildResult(request);
    }

    @Data
    @NoArgsConstructor
    public static class ReqBody implements Serializable {
        private static final long serialVersionUID = -4536744669004135021L;
        private String name;
        private Integer age;
    }

    @PostMapping(path = "body")
    public String postBody(@RequestBody ReqBody body) {
        HttpServletRequest request =
                ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        return buildResult(request, body);
    }
}
```

## II. 使用姿势

最常见的携带请求头的需求，无非是 referer 校验，user-agent 的防爬以及携带 cookie，使用 RestTemplate 可以借助`HttpHeaders`来处理请求头

### 1. Get 携带请求头

前一篇博文介绍了 GET 请求的三种方式，但是`getForObject`/`getForEntity`都不满足我们的场景，这里需要引入`exchange`方法

```java
public void header() {
        RestTemplate restTemplate = new RestTemplate();

        HttpHeaders headers = new HttpHeaders();
        headers.set("user-agent",
                "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36");
        headers.set("cookie", "my_user_id=haha123; UN=1231923;gr_user_id=welcome_yhh;");

        // 注意几个请求参数
        HttpEntity<String> res = restTemplate
                .exchange("http://127.0.0.1:8080/get?name=一灰灰&age=20", HttpMethod.GET, new HttpEntity<>(null, headers),
                        String.class);
        log.info("get with selfDefine header: {}", res);
}
```

exchange 的使用姿势和我们前面介绍的`postForEntity`差不多，只是多了一个指定 HttpMethod 的参数而已

**重点在于将请求头塞入 HttpEntity**

输出结果

```
(get with selfDefine header: <200,params: {"name":["一灰灰"],"age":["20"]}
headers: {"cookie":"my_user_id=haha123; UN=1231923;gr_user_id=welcome_yhh;","host":"127.0.0.1:8080","connection":"keep-alive","accept":"text/plain, application/json, application/*+json, */*","user-agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36"}
cookies: {"my_user_id":"haha123","UN":"1231923","gr_user_id":"welcome_yhh"},[Content-Type:"text/plain;charset=UTF-8", Content-Length:"447", Date:"Mon, 29 Jun 2020 07:48:49 GMT"]>
```

### 2. Post 携带请求头

post 携带请求头，也可以利用上面的方式实现；当然我们一般直接借助`postForObject/postForEntity`就可以满足需求了

```java
// httpHeaders 和上面的一致，这里省略相关代码
// post 带请求头
MultiValueMap<String, Object> params = new LinkedMultiValueMap<>();
params.add("name", "一灰灰Blog");
params.add("age", 20);

String response = restTemplate
        .postForObject("http://127.0.0.1:8080/post", new HttpEntity<>(params, headers), String.class);
log.info("post with selfDefine header: {}", response);
```

输出结果

```
(post with selfDefine header: params: {"name":["一灰灰Blog"],"age":["20"]}
headers: {"content-length":"338","cookie":"my_user_id=haha123; UN=1231923;gr_user_id=welcome_yhh;","host":"127.0.0.1:8080","content-type":"multipart/form-data;charset=UTF-8;boundary=2VJHo9r6lYgR_WoSBy1FQC40jvBvGtLk7QUaymGg","connection":"keep-alive","accept":"text/plain, application/json, application/*+json, */*","user-agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36"}
cookies: {"my_user_id":"haha123","UN":"1231923","gr_user_id":"welcome_yhh"}
```

### 3. 拦截器方式

如果我们可以确定每次发起请求时，都要设置一个自定义的 `User-Agent`，每次都使用上面的两种姿势就有点繁琐了，因此我们是可以通过拦截器的方式来添加通用的请求头，这样使用这个 RestTemplate 时，都会携带上请求头

```java
// 借助拦截器的方式来实现塞统一的请求头
ClientHttpRequestInterceptor interceptor = (httpRequest, bytes, execution) -> {
    httpRequest.getHeaders().set("user-agent",
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36");
    httpRequest.getHeaders().set("cookie", "my_user_id=haha123; UN=1231923;gr_user_id=interceptor;");
    return execution.execute(httpRequest, bytes);
};

restTemplate.getInterceptors().add(interceptor);
response = restTemplate.getForObject("http://127.0.0.1:8080/get?name=一灰灰&age=20", String.class);
log.info("get with selfDefine header by Interceptor: {}", response);
```

上面这个使用姿势比较适用于通用的场景，测试输出

```
(get with selfDefine header by Interceptor: params: {"name":["一灰灰"],"age":["20"]}
headers: {"cookie":"my_user_id=haha123; UN=1231923;gr_user_id=interceptor;","host":"127.0.0.1:8080","connection":"keep-alive","accept":"text/plain, application/json, application/*+json, */*","user-agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36"}
cookies: {"my_user_id":"haha123","UN":"1231923","gr_user_id":"interceptor"}
```

注意：在我们使用自定义请求头时，有一个需要特殊重视的地方，HttpHeaders 使用不当，可能导致请求头爆炸。比如，若希望复用 HttpHeaders，但是不小心使用了`headers.add` 方式添加请求头,而不是前面的 `set`方式，则在每一次请求过后，请求头膨胀了一次。

# RestTemplate的进一步阅读

[RestTemplate使用教程](https://www.cnblogs.com/f-anything/p/10084215.html)

[The Guide to RestTemplate](https://www.baeldung.com/rest-template)

比较全面的一篇博文：[SpringBoot系列 - 使用RestTemplate](https://www.xncoding.com/2017/07/06/spring/sb-restclient.html)

[RestTemplate的API文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/index.html?org/springframework/web/client/RestTemplate.html)