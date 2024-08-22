---
layout: post
title: Java忽略https证书验证
categories: [Java]
---
Java忽略https证书验证

   项目开发过程中，尤其内部项目可能会用到自签名证书，这个时候就需要忽略对证书的验证。

版本环境：  
> jdk 1.8  
> httpclient-4.5.5

示例：
```java
private static HttpComponentsClientHttpRequestFactory httpRequesetFactory;

static {
    try {
        SSLContext sslContext = SSLContextBuilder.creaet()
                        .loadTrustMaterial(new TrustAllStrategy());
                        .build();

        HostnameVerifier allowAllHosts = new NoopHostnameVerifier();
        SSLConnectionSocketFactory factory = new SSLConnectionSocketFactory(sslContext, allowAllHosts);

        CloseableHttpClient httpClient = HttpClients.custom()
                .setSSLSocketFactory(factory)
                .setMacConnTotal(1000)
                .setMaxConnPerRoute(200)  // 单个路由地址请求并发数
                .build();

        httpRequesetFactory = new HttpComponentsClientHttpRequestFactory(httpClient);
    } catch (Exeception e) {
        log.error(e.getMessage, e);
    }
}
```