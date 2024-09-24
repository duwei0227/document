---
layout: post
title: Feign调用全局header参数设置
categories: [Spring]
permalink: spring/feign_headers.html
---



### 一、实现 `RequestInterceptor`

```java
import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.apache.commons.lang3.StringUtils;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.util.Objects;

@Component
public class FeignClientInterceptor implements RequestInterceptor{

    @Override
    public void apply(RequestTemplate template) {
        ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        if (Objects.nonNull(requestAttributes)) {
            HttpServletRequest request = requestAttributes.getRequest();
            String userLang = request.getHeader("UserLang");
            if (StringUtils.isNotBlank(userLang)) {
                template.header("UserLang", userLang);
            }
        }
    }
}
```

