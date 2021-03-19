---
title: "发起HTTP请求：Spring中的RestTempalate的基本使用（转载）"
date: 2021-01-07T17:28:40+08:00
draft: false
categories: 
- Java
- HTTP
series:
- Java发起HTTP请求
- RestTemplate
tags:
- Java
- HTTP
- Java实现HTTP请求
- RestTemplate
---

# RestTemplate的认识与基本使用

## RestTemplate介绍

在Spring的生态下，则可以利用`RestTemplate`来发起Http请求。

spring框架提供的RestTemplate类可用于在应用中调用rest服务，它简化了与http服务的通信方式，统一了RESTful的标准，封装了http链接， 我们只需要传入url及返回值类型即可。相较于之前常用的HttpClient，RestTemplate是一种更优雅的调用RESTful服务的方式。

RestTemplate默认依赖JDK提供http连接的能力（HttpURLConnection），如果有需要的话也可以通过setRequestFactory方法替换为例如 Apache HttpComponents、Netty或OkHttp等其它HTTP library。



RestTemplate构成与实现逻辑：

RestTemplate包含以下几个部分：

- HttpMessageConverter 对象转换器
- ClientHttpRequestFactory 默认是JDK的HttpURLConnection
- ResponseErrorHandler 异常处理
- ClientHttpRequestInterceptor 请求拦截器

用一张图可以很直观的理解：

![img](https://picgo12138.oss-cn-hangzhou.aliyuncs.com/md/rest01.png)

## RestTempalate的基本使用

本节"RestTempalate的基本使用"非原创，转载来源：https://juejin.cn/post/6844903656165212174

### 0. 目标

在介绍如何使用RestTemplate之前，我们先抛出一些小目标，至少需要知道通过RestTemplate可以做些什么，以及我们要用它来干些什么

简单的给出了一下常见的问题如下

- 普通的Get请求获取返回数据，怎么玩？
- post提交表达的请求，如何处理
- post请求中RequestBody的请求方式与普通的请求方式区别
- https/http两种访问如何分别处理
- 如何在请求中带上指定的Header
- 有跨域的问题么？如果有怎么解决
- 有登录验证的请求，该怎么办，怎样携带身份信息
- 上传文件可以支持么
- 对于需要代理才能访问的http资源，加代理的姿势是怎样的

上面的问题比较多，目测不是一篇博文可以弄完的，因此对这个拆解一下，本节主要关注在RestTemplate的简单Get/Post请求的使用方式上：

- 普通的Get请求获取返回数据，怎么玩？
- post提交表达的请求，如何处理

### 1. 基本接口

捞出源码，看一下其给出的一些常用接口，基本上可以分为下面几种

```java
// get 请求
public <T> T getForObject();
public <T> ResponseEntity<T> getForEntity();

// head 请求
public HttpHeaders headForHeaders();

// post 请求
public URI postForLocation();
public <T> T postForObject();
public <T> ResponseEntity<T> postForEntity();

// put 请求
public void put();

// pathch 
public <T> T patchForObject

// delete
public void delete()

// options
public Set<HttpMethod> optionsForAllow

// exchange
public <T> ResponseEntity<T> exchange()
```

上面提供的几个接口，基本上就是Http提供的几种访问方式的对应，其中exchange却又不一样，后面细说



### 2. Get请求

从上面的接口命名上，可以看出可以使用的有两种方式 `getForObject` 和 `getForEntity`，那么这两种有什么区别？

- 从接口的签名上，可以看出一个是直接返回预期的对象，一个则是将对象包装到 `ResponseEntity` 封装类中
- 如果只关心返回结果，那么直接用 `GetForObject` 即可
- 如果除了返回的实体内容之外，还需要获取返回的header等信息，则可以使用 `getForEntit`

#### a. 创建Get接口

为了验证RestTemplate的使用姿势，当然得先提供一个后端的REST服务，这了直接用了我个人的一个古诗词的后端接口，来作为简单的Get测试使用

请求连接： `https://story.hhui.top/detail?id=666106231640`

返回结果:

```json
{
    "status": {
        "code": 200,
        "msg": "SUCCESS"
    },
    "result": {
        "id": 666106231640,
        "title": "西塞山二首（今谓之道士矶，即兴国军大冶县",
        "author": "王周",
        "content": "西塞名山立翠屏，浓岚横入半江青。\n千寻铁锁无由问，石壁空存道者形。\n匹妇顽然莫问因，匹夫何去望千春。\n翻思岵屺传诗什，举世曾无化石人。",
        "explain": "",
        "theme": "无",
        "dynasty": "唐诗"
    }
}
```

#### b. getForObject方式

首先看下完整的接口签名

```java
@Nullable
public <T> T getForObject(String url, Class<T> responseType, Object... uriVariables) throws RestClientException ;

@Nullable
public <T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException ;

@Nullable
public <T> T getForObject(URI url, Class<T> responseType) throws RestClientException;
```

有三个重载的方法，从接口上也比较容易看出如何使用，其中有点疑惑的则是第一钟，参数应该怎么传了，下面给出上面几种的使用姿势

```java
public class RestTestmplateTest {
    private RestTemplate restTemplate;

    @Before
    public void init() {
        restTemplate = new RestTemplate();
    }

    @lombok.Data
    static class InnerRes {
        private Status status;
        private Data result;
    }

    @lombok.Data
    static class Status {
        int code;
        String msg;
    }

    @lombok.Data
    static class Data {
        long id;
        String theme;
        String title;
        String dynasty;
        String explain;
        String content;
        String author;
    }

    @Test
    public void testGet() {
        // 使用方法一，不带参数
        String url = "https://story.hhui.top/detail?id=666106231640";
        InnerRes res = restTemplate.getForObject(url, InnerRes.class);
        System.out.println(res);


        // 使用方法一，传参替换
        url = "https://story.hhui.top/detail?id={?}";
        res = restTemplate.getForObject(url, InnerRes.class, "666106231640");
        System.out.println(res);

        // 使用方法二，map传参
        url = "https://story.hhui.top/detail?id={id}";
        Map<String, Object> params = new HashMap<>();
        params.put("id", 666106231640L);
        res = restTemplate.getForObject(url, InnerRes.class, params);
        System.out.println(res);

        // 使用方法三，URI访问
        URI uri = URI.create("https://story.hhui.top/detail?id=666106231640");
        res = restTemplate.getForObject(uri, InnerRes.class);
        System.out.println(res);
    }
}
```

看上面的testcase，后面两个方法的使用没什么好说的，主要看一下`org.springframework.web.client.RestTemplate#getForObject(java.lang.String, java.lang.Class<T>, java.lang.Object...)` 的使用姿势

- 根据实际传参替换url模板中的内容
- 使用方法一时，模板中使用 `{?}` 来代表坑位，根据实际的传参顺序来填充
- 使用方法二时，模板中使用 `{xx}`, 而这个xx，对应的就是map中的key

上面执行后的截图如下

![3AD423F4F3C673F2D366772612B4355A.jpg](https://picgo12138.oss-cn-hangzhou.aliyuncs.com/md/1653356c4d152ec7)

#### c. getForEntity方式

既然getForObject有三种使用方法，那么getForEntity理论上也应该有对应的三种方式

```java
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables) throws RestClientException ;
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException;
public <T> ResponseEntity<T> getForEntity(URI url, Class<T> responseType) throws RestClientException;

```

因为使用姿势和上面一致，因此只拿一个进行测试

```java
@Test
public void testGetForEntity() {
    String url = "https://story.hhui.top/detail?id=666106231640";
    ResponseEntity<InnerRes> res = restTemplate.getForEntity(url, InnerRes.class);
    System.out.println(res);
}
```

对这个，我们主要关注的就是ResponseEntity封装中，多了哪些东西，截图如下

![C2D317A154B33CF76F6DB438D1589754.jpg](https://picgo12138.oss-cn-hangzhou.aliyuncs.com/md/1653356c4d06c3ac)

从上面可以看出，多了两个东西

- 一个返回的http状态码，如200表示请求成功，500服务器错误，404not found等
- 一个 ResponseHeader

### 3. Post请求

从上面的接口说明上看，post请求除了有forObject 和 forEntity之外，还多了个forLocation；其次post与get一个明显的区别就是传参的姿势问题，get的参数一般会待在url上；post的则更常见的是通过表单的方式提交

因此接下来关注的重点在于forLocation是什么，以及如何传参

#### a. post接口mock

首先创建一个简单的提供POST请求的REST服务，基于Spring-boot简单搭建一个，如下

```java
@ResponseBody
@RequestMapping(path = "post", method = {RequestMethod.GET, RequestMethod.OPTIONS, RequestMethod.POST})
public String post(HttpServletRequest request,
        @RequestParam(value = "email", required = false) String email,
        @RequestParam(value = "nick", required = false) String nick) {
   Map<String, Object> map = new HashMap<>();
   map.put("code", "200");
   map.put("result", "add " + email + " # " + nick + " success!");
   return JSON.toJSONString(map);
}
```

#### b. postForObject方法

首先看一下接口签名

```java
public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables) throws RestClientException ;

public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException;

public <T> T postForObject(URI url, @Nullable Object request, Class<T> responseType) throws RestClientException ;
```

上面的三个方法，看起来和前面并没有太大的区别，只是多了一个request参数，那么具体的使用如何呢？

下面分别给出使用用例

```java
@Test
public void testPost() {
    String url = "http://localhost:8080/post";
    String email = "test@hhui.top";
    String nick = "一灰灰Blog";

    MultiValueMap<String, String> request = new LinkedMultiValueMap<>();
    request.add("email", email);
    request.add("nick", nick);

    // 使用方法三
    URI uri = URI.create(url);
    String ans = restTemplate.postForObject(uri, request, String.class);
    System.out.println(ans);

    // 使用方法一
    ans = restTemplate.postForObject(url, request, String.class);
    System.out.println(ans);

    // 使用方法一，但是结合表单参数和uri参数的方式，其中uri参数的填充和get请求一致
    request.clear();
    request.add("email", email);
    ans = restTemplate.postForObject(url + "?nick={?}", request, String.class, nick);
    System.out.println(ans);


    // 使用方法二
    Map<String, String> params = new HashMap<>();
    params.put("nick", nick);
    ans = restTemplate.postForObject(url + "?nick={nick}", request, String.class, params);
    System.out.println(ans);
}
```

上面分别给出了三种方法的调用方式，其中post传参区分为两种，一个是uri参数即拼接在url中的，还有一个就是表单参数

- uri参数，使用姿势和get请求中一样，填充uri中模板坑位
- 表单参数，由`MultiValueMap`封装，同样是kv结构

#### c. postForEntity

和前面的使用姿势一样，无非是多了一层包装而已，略过不讲

#### d. postForLocation

这个与前面有点区别，从接口定义上来说，主要是

> POST 数据到一个URL，返回新创建资源的URL

同样提供了三个接口，分别如下，需要注意的是返回结果，为URI对象,即网络资源

```java
public URI postForLocation(String url, @Nullable Object request, Object... uriVariables)
		throws RestClientException ;

public URI postForLocation(String url, @Nullable Object request, Map<String, ?> uriVariables)
		throws RestClientException ;

public URI postForLocation(URI url, @Nullable Object request) throws RestClientException ;

```

那么什么样的接口适合用这种访问姿势呢？

想一下我们一般登录or注册都是post请求，而这些操作完成之后呢？大部分都是跳转到别的页面去了，这种场景下，就可以使用 `postForLocation` 了，提交数据，并获取返回的URI，一个测试如下

首先mock一个后端接口

```java
@ResponseBody
@RequestMapping(path = "success")
public String loginSuccess(String email, String nick) {
    return "welcome " + nick;
}

@RequestMapping(path = "post", method = {RequestMethod.GET, RequestMethod.OPTIONS, RequestMethod.POST})
public String post(HttpServletRequest request, @RequestParam(value = "email", required = false) String email,
        @RequestParam(value = "nick", required = false) String nick) {
    return "redirect:/success?email=" + email + "&nick=" + nick + "&status=success";
}
```

> 转载者注：重定向可以试试另一种方法
>
> ```java
> @RequestMapping("/redirect")
> @ResponseBody
> public ResponseEntity redirect() {
>     return ResponseEntity.status(301).header("Location", "/greeting").body("");
> }
> ```



访问的测试用例，基本上和前面的一样，没有什么特别值得一说的

```java
@Test
public void testPostLocation() {
    String url = "http://localhost:8080/post";
    String email = "test@hhui.top";
    String nick = "一灰灰Blog";

    MultiValueMap<String, String> request = new LinkedMultiValueMap<>();
    request.add("email", email);
    request.add("nick", nick);

    // 使用方法三
    URI uri = restTemplate.postForLocation(url, request);
    System.out.println(uri);
}
```

执行结果如下



![C4272685267804B4A1C1837EAEB76762.jpg](https://picgo12138.oss-cn-hangzhou.aliyuncs.com/md/1653356c4d2331ec)



获取到的就是302跳转后端url，细心的朋友可能看到上面中文乱码的问题，如何解决呢？

一个简单的解决方案就是url编码一下

```java
@RequestMapping(path = "post", method = {RequestMethod.GET, RequestMethod.OPTIONS, RequestMethod.POST},
            produces = "charset/utf8")
public String post(HttpServletRequest request, @RequestParam(value = "email", required = false) String email,
        @RequestParam(value = "nick", required = false) String nick) throws UnsupportedEncodingException {
    return "redirect:/success?email=" + email + "&nick=" + URLEncoder.encode(nick, "UTF-8") + "&status=success";
}
```

。