---
title: "发起HTTP请求：使用Apache的HttpClient"
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

参考了别人的代码,简单使用：

```java
/**
 * 使用apache的HttpClient发起HTTP以及HTTPS请求
 * 注意：当前的apache的HttpClient4.5只支持到HTTP/1.1
 * 参考：https://zhuanlan.zhihu.com/p/69285935
 * 参考：https://blog.csdn.net/happylee6688/article/details/47148227
 */
public class HttpClientUtil {

    private static PoolingHttpClientConnectionManager connMgr;
    private static RequestConfig requestConfig;
    private static final int MAX_TIMEOUT = 7000;

    static {
        // 设置连接池
        connMgr = new PoolingHttpClientConnectionManager();
        // 设置连接池大小
        connMgr.setMaxTotal(100);
        connMgr.setDefaultMaxPerRoute(connMgr.getMaxTotal());

        RequestConfig.Builder configBuilder = RequestConfig.custom();
        // 设置连接超时
        configBuilder.setConnectTimeout(MAX_TIMEOUT);
        // 设置读取超时
        configBuilder.setSocketTimeout(MAX_TIMEOUT);
        // 设置从连接池获取连接实例的超时
        configBuilder.setConnectionRequestTimeout(MAX_TIMEOUT);
        // 在提交请求之前 测试连接是否可用
        configBuilder.setStaleConnectionCheckEnabled(true);
        requestConfig = configBuilder.build();
    }

    /**
     * 发送 GET 请求（HTTP），K-V形式
     *
     * @param url
     * @param params
     * @return
     */
    public static String doGet(String url, Map<String, Object> params) {
        String result = null;
        try {
            //url上拼接参数
            if (params != null) {
                StringBuffer param = new StringBuffer();
                int i = 0;
                for (String key : params.keySet()) {
                    param.append(i == 0 ? "?" : "&");
                    param.append(key).append("=").append(params.get(key));
                    i++;
                }
                url += param;
            }
            // 通过址默认配置创建一个httpClient实例
            @Cleanup CloseableHttpClient httpClient = HttpClients.createDefault();
            // 创建httpGet远程连接实例
            HttpGet httpGet = new HttpGet(url);
            // 设置请求头信息示例（不需要可注释掉）：鉴权
            httpGet.setHeader("Authorization", "Bearer da3efcbf-0845-4fe3-8aba-ee040be542c0");
            // 设置配置请求参数
            /*RequestConfig requestConfig = RequestConfig.custom()
                    // 连接主机服务超时时间
                    .setConnectTimeout(35000)
                    // 请求超时时间
                    .setConnectionRequestTimeout(35000)
                    // 数据读取超时时间
                    .setSocketTimeout(60000)
                    .build();*/
            // 为httpGet实例设置配置
            httpGet.setConfig(requestConfig);
            // 执行get请求得到返回对象
            HttpResponse response = httpClient.execute(httpGet);

            int statusCode = response.getStatusLine().getStatusCode();
            //System.out.println("执行状态码 : " + statusCode);

            // 通过返回对象获取返回数据
            HttpEntity entity = response.getEntity();
            // 通过EntityUtils中的toString方法将结果转换为字符串
            result = EntityUtils.toString(entity);
            if (response != null) {
                EntityUtils.consume(response.getEntity());
            }
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 发送 GET 请求（HTTP），不带输入数据
     *
     * @param url
     * @return
     */
    public static String doGet(String url) {
        return doGet(url, null);
    }

    /**
     * 发送 POST 请求（HTTP），K-V形式
     *
     * @param apiUrl API接口URL
     * @param params 参数map
     * @return
     */
    public static String doPost(String url, Map<String, Object> params) {
        String result = "";
        try {
            // 创建httpClient实例
            @Cleanup CloseableHttpClient httpClient = HttpClients.createDefault();
            // 创建httpPost远程连接实例
            HttpPost httpPost = new HttpPost(url);
            // 配置请求参数实例
            /*RequestConfig requestConfig = RequestConfig.custom()
                    // 设置连接主机服务超时时间
                    .setConnectTimeout(35000)
                    // 设置连接请求超时时间
                    .setConnectionRequestTimeout(35000)
                    // 设置读取数据连接超时时间
                    .setSocketTimeout(60000)
                    .build();*/
            // 为httpPost实例设置配置
            httpPost.setConfig(requestConfig);
            // 设置请求头示例（不需要可注释掉）
            httpPost.setHeader("Content-Type", "application/x-www-form-urlencoded");
            httpPost.setHeader("Cookie", "id=123;token=234");
            // 封装post请求参数
            //将传入post参数，转换为HttpClient可以接受的形式
            if (params != null && params.size() > 0) {
                List<NameValuePair> pairList = new ArrayList<>(params.size());
                for (Map.Entry<String, Object> entry : params.entrySet()) {
                    NameValuePair pair = new BasicNameValuePair(entry.getKey(), entry
                            .getValue().toString());
                    pairList.add(pair);
                }
                // 为httpPost设置封装好的请求参数
                httpPost.setEntity(new UrlEncodedFormEntity(pairList, StandardCharsets.UTF_8));
            }
            // httpClient对象执行post请求,并返回响应参数对象
            HttpResponse httpResponse = httpClient.execute(httpPost);
            // 从响应对象中获取响应内容
            HttpEntity entity = httpResponse.getEntity();
            if (entity != null) {
                result = EntityUtils.toString(entity);
            }
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }


    /**
     * 发送 POST 请求（HTTP），JSON形式
     *
     * @param apiUrl
     * @param json   json对象
     * @return
     */
    public static String doPost(String apiUrl, String jsonStr) {
        CloseableHttpClient httpClient = HttpClients.createDefault();
        String httpStr = null;
        HttpPost httpPost = new HttpPost(apiUrl);
        CloseableHttpResponse response = null;
        try {
            httpPost.setConfig(requestConfig);
            //解决中文乱码问题
            StringEntity stringEntity = new StringEntity(jsonStr, StandardCharsets.UTF_8);
            stringEntity.setContentEncoding("UTF-8");
            stringEntity.setContentType("application/json");
            httpPost.setEntity(stringEntity);
            response = httpClient.execute(httpPost);
            HttpEntity entity = response.getEntity();
            //System.out.println(response.getStatusLine().getStatusCode());
            httpStr = EntityUtils.toString(entity);
            if (response != null) {
                EntityUtils.consume(response.getEntity());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return httpStr;
    }

    /**
     * 发送 POST 请求（HTTP），不带输入数据
     *
     * @param apiUrl
     * @return
     */
    public static String doPost(String apiUrl) {
        return doPost(apiUrl, (Map<String, Object>) null);
    }

    /**
     * 发送 SSL POST 请求（HTTPS），K-V形式
     *
     * @param apiUrl API接口URL
     * @param params 参数map
     * @return
     */
    public static String doPostSSL(String apiUrl, Map<String, Object> params) {
        CloseableHttpClient httpClient =
                HttpClients.custom().setSSLSocketFactory(createSSLConnSocketFactory()).setConnectionManager(connMgr).setDefaultRequestConfig(requestConfig).build();
        HttpPost httpPost = new HttpPost(apiUrl);
        CloseableHttpResponse response = null;
        String httpStr = null;

        try {
            httpPost.setConfig(requestConfig);
            List<NameValuePair> pairList = new ArrayList<NameValuePair>(params.size());
            for (Map.Entry<String, Object> entry : params.entrySet()) {
                NameValuePair pair = new BasicNameValuePair(entry.getKey(), entry
                        .getValue().toString());
                pairList.add(pair);
            }
            httpPost.setEntity(new UrlEncodedFormEntity(pairList, StandardCharsets.UTF_8));
            response = httpClient.execute(httpPost);
            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode != HttpStatus.SC_OK) {
                return null;
            }
            HttpEntity entity = response.getEntity();
            if (entity == null) {
                return null;
            }
            httpStr = EntityUtils.toString(entity, "utf-8");
            if (response != null) {
                EntityUtils.consume(response.getEntity());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return httpStr;
    }

    /**
     * 发送 SSL POST 请求（HTTPS），JSON形式
     *
     * @param apiUrl API接口URL
     * @param json   JSON对象
     * @return
     */
    public static String doPostSSL(String apiUrl, Object json) {
        CloseableHttpClient httpClient =
                HttpClients.custom().setSSLSocketFactory(createSSLConnSocketFactory()).setConnectionManager(connMgr).setDefaultRequestConfig(requestConfig).build();
        HttpPost httpPost = new HttpPost(apiUrl);
        CloseableHttpResponse response = null;
        String httpStr = null;

        try {
            httpPost.setConfig(requestConfig);
            //解决中文乱码问题
            StringEntity stringEntity = new StringEntity(json.toString(), "UTF-8");
            stringEntity.setContentEncoding("UTF-8");
            stringEntity.setContentType("application/json");
            httpPost.setEntity(stringEntity);
            response = httpClient.execute(httpPost);
            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode != HttpStatus.SC_OK) {
                return null;
            }
            HttpEntity entity = response.getEntity();
            if (entity == null) {
                return null;
            }
            httpStr = EntityUtils.toString(entity, "utf-8");
            if (response != null) {
                EntityUtils.consume(response.getEntity());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return httpStr;
    }

    /**
     * 创建SSL安全连接
     *
     * @return
     */
    private static SSLConnectionSocketFactory createSSLConnSocketFactory() {
        SSLConnectionSocketFactory sslsf = null;
        try {
            SSLContext sslContext = new SSLContextBuilder().loadTrustMaterial(null, new TrustStrategy() {
                @Override
                public boolean isTrusted(X509Certificate[] chain, String authType) throws CertificateException {
                    return true;
                }
            }).build();
            sslsf = new SSLConnectionSocketFactory(sslContext, new HostnameVerifier() {
                @Override
                public boolean verify(String host, SSLSession session) {
                    return true;
                }
            });
        } catch (GeneralSecurityException e) {
            e.printStackTrace();
        }
        return sslsf;
    }
}
```

测试：

```java
class HttpClientUtilTest {

    @Test
    void doGet() {
        System.out.println("Test doGet(): ");
        String url = "https://httpbin.org/anything";
        String res = HttpClientUtil.doGet(url);
        System.out.println(res);

        res = HttpClientUtil.doGet(url + "?key1=value1&key2=value2");
        System.out.println(res);

        Map<String, Object> params = new HashMap<>(8);
        params.put("key1", "value1");
        params.put("key2", "value2");
        res = HttpClientUtil.doGet(url, params);
        System.out.println(res);

    }

    @Test
    void doPost() {
        System.out.println("Test doPost(): ");
        String url = "https://httpbin.org/anything";
        String res = HttpClientUtil.doPost(url);
        System.out.println(res);


        Map<String, Object> params = new HashMap<>(8);
        params.put("key1", "value1");
        params.put("key2", "value2");
        res = HttpClientUtil.doPost(url, params);
        System.out.println(res);

        JsonMapper mapper = new JsonMapper();
        ObjectNode node = mapper.createObjectNode();
        node.put("key1", "value1");
        node.put("key2", "value2");
        String jsonStr = node.toString();
        res = HttpClientUtil.doPost(url, jsonStr);
        System.out.println(res);
    }

    @Test
    void doPostSSL() {
        System.out.println("Test doPostSSL(): ");
        String url = "https://httpbin.org/anything";

        Map<String, Object> params = new HashMap<>(8);
        params.put("key1", "value1");
        params.put("key2", "value2");
        String res = HttpClientUtil.doPostSSL(url, params);
        System.out.println(res);

        JsonMapper mapper = new JsonMapper();
        ObjectNode node = mapper.createObjectNode();
        node.put("key1", "value1");
        node.put("key2", "value2");
        String jsonStr = node.toString();
        res = HttpClientUtil.doPostSSL(url, jsonStr);
        System.out.println(res);
    }
}
```

[代码存放仓库](https://github.com/ole12138/myutils)

# 进一步阅读

[Apache HttpComponents官网](https://hc.apache.org/httpcomponents-client-ga/)

[HttpClient教程](https://www.yiibai.com/apache_httpclient/apache_httpclient_http_get_request.html)