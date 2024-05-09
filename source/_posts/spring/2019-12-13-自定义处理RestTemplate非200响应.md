---
layout: post
title: 自定义处理RestTemplate非200响应
categories: [Java]
description: 自定义处理RestTemplate非200响应
permalink: spring/resttemplate_statuscode.html
tags: RestTemplate
---

`RestTemplate`对非200错误的响应默认做了处理，会丢失很多信息。尤其接口方在抛出错误的时候，将错误信息放在响应体的中；
这个时候如果不对响应做自定义处理是无法拿到响应结果的。  

自定义处理方式：  
```java
public class RestTemplateResponseErrorHandler implements ResponseErrorHandler {
    @Override
    public boolean hasError(ClientHttpResponse response) throws IOException {
        // 自定义响应码的正确性
        return true;   // 认为所有结果都是正确的，转到上层处理
    }

    @Override
    public void handleError(ClientHttpResponse response) throws IOException {
        //对错误的处理，可以不做任何处理
    }
}
```