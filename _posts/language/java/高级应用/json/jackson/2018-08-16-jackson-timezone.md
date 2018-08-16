---
title: jackson的时区问题
tags: [jackson]
---

1）问题描述

controller层方法返回对象日期类型数据时，使用Jackson实现自动转换成json字符串，使用@JsonFormat注解，结果客户端获取的日期与实际的日期不符。

```
@JsonFormat(pattern="yyyy-MM-dd")
private Date feeDate;//缴费日期
```

原因是JsonFormat的timezone取TimeZone.getDefault返回值（用的是格林尼治时间），而中国使用的是东八区，可通过指定timezone参数的方式实现

```
@JsonFormat(pattern="yyyy-MM-dd",timezone="GMT+8")
```

2）解决方案

在spring配置中进行converter的全局配置。

```
package org.ipph.spider;

import java.util.TimeZone;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;

import com.fasterxml.jackson.databind.ObjectMapper;

@Configuration
public class SpiderConfiguration {
    
    @Bean
    public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter(ObjectMapper objectMapper) {
        MappingJackson2HttpMessageConverter converter=new MappingJackson2HttpMessageConverter();
        converter.setObjectMapper(objectMapper);
        return converter;
    }
    
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper=new ObjectMapper();
        mapper.setTimeZone(TimeZone.getTimeZone(System.getProperty("user.timezone")));
        return mapper;
    }
}
```