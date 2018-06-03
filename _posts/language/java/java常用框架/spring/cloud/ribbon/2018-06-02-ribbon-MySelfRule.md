---
title: spring cloud ribbon使用自定义负载均衡规则
tags: [spring]
---

负载均衡应用在服务消费端，因此需要改造microservicecloud-consumer-dept-80模块。

1）在主启动类上添加注解@RibbonClient

该注解允许我们指定特定的微服务使用特定的规则实现负载均衡。

```
package org.sjq.test;

import org.sjq.rule.MySelfRule;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.ribbon.RibbonClient;

@SpringBootApplication
@EnableEurekaClient
//name指定需要新负载均衡规则的服务名，configuration指定一个configuration类，配置类中返回自定义的规则
//要避免MySelfRule类所在的路径被主启动类的ComponentScan扫描到，这样所有的微服务都会使用该规则
@RibbonClient(name="MICROSERVICECLOUD-DEPT",configuration=MySelfRule.class)
public class DeptConsumer80_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_App.class, args);
    }
}
```

2）创建自定义负载均衡处理配置类

保证该配置类的定义不在主启动类所在包及其子包下，否则所有的微服务负载均衡都会使用该算法，而不能实现指定微服务使用该负载均衡算法（RibbonClient注解的name属性指定的微服务）。

```
package org.sjq.rule;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;

@Configuration
public class MySelfRule {

    @Bean
    public IRule myRule() {
        return new RandomRule();
    }
}
```

注：这里新建包org.sjq.rule，保证MySelfRule类不在主启动类DeptConsumer80_App所在包及其子包下。

3）测试

按照ribbon-muliti-provider.md文件中的模块启动并测试。

访问服务消费者，不断刷新，查看是否是对服务进行随机访问。