---
title: eureka的服务发现
tags: [spring]
---

对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息。

1）修改服务提供者的Controller，提供服务发现信息

修改microservicecloud-provider-dept-8001模块（微服务提供者）的DeptController类

```
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
...

@Autowired
private DiscoveryClient client;

@RequestMapping(value="/dept/discovery",method=RequestMethod.GET)
public Object discovery() {
    List<String> list=client.getServices();
    
    System.out.println("注册中心的服务列表"+list);
    
    //获取注册中心的服务实例
    List<ServiceInstance> serviceInstanceList=client.getInstances("MICROSERVICECLOUD-DEPT");
    
    for(ServiceInstance instance:serviceInstanceList) {
        System.out.println(instance.getServiceId()+"\t"+
                instance.getHost()+"\t"+
                instance.getPort()+"\t"+
                instance.getUri());
    }
    
    return this.client;
}
```

2）修改启动类

修改microservicecloud-provider-dept-8001模块（微服务提供者）启动类，在启动类上添加注解@EnableDiscoveryClient

```
package org.sjq.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient //服务启动后自动注册到eureka服务中心
@EnableDiscoveryClient //启动服务发现功能
public class DeptProvider8001_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class, args);
    }
}
```

3）直接访问

```
http://localhost:8001/dept/discovery
```

4）在consumer端添加获取服务代码

修改microservicecloud-consumer-dept-80（服务消费者）模块的DeptController_Consumer类

```
/**
 * 访问服务提供者提供的服务信息
 * @return
 */
@RequestMapping("/consumer/dept/discovery")
public Object discovery() {
    return restTemplate.getForObject(REST_URL_PREFIX+"dept/discovery/", Object.class);
}
```

5）访问测试

```
http://localhost/consumer/dept/discovery
```