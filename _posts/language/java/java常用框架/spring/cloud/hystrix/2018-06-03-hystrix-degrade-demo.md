---
title: spring cloud hystrix实例--服务降级
tags: [spring]
---

服务熔断是配置在服务提供者端，而服务降级是配置在服务消费端的，与服务端没有关系。

1）服务降级

服务降级要理解为服务暂停或部分资源停掉了（即对外不再提供服务或降低提供服务的能力）。整体资源快不够了，忍痛间爱你个某些服务先关掉，待渡过难关，再开启回来。

服务降级是快速处理服务不可访问的策略（服务处理能力下降了，超时发生频率变高），客户端本身就带着fallback。

注意区别熔断，熔断是解决网络或服务异常时的响应策略，服务器端带有fallback处理。

2）修改公共API模块（提供对微服务的降级操作）

修改mirocservicecloud-api工程，根据已经有的DeptClientService接口新建一个实例FallbackFactory接口类（DeptClientServiceFallbackFactory）。

工作原理：当调用微服务不通时，即DeptClientService访问接口不通时，会交由DeptClientServiceFallbackFactory来处理，该类会实现DeptClientService中定义的所有方法。客户端调用DeptClientService接口的哪个方法没有成功，都会交由DeptClientServiceFallbackFactory类的相应方法来处理。

注：DeptClientService是我们使用Feign实现接口方式的负载均衡时的处理类

a.查看DeptClientService接口的定义

```
package org.sjq.test.service;

import java.util.List;

import org.sjq.test.entities.Dept;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

//填写微服务名称
//提供接口，供访问微服务的程序使用
@FeignClient(value="MICROSERVICECLOUD-DEPT")
public interface DeptClientService {
    @RequestMapping(value="/dept/get/{id}",method=RequestMethod.GET)
    public Dept get(@PathVariable("id")Long id);
    
    @RequestMapping(value="/dept/list",method=RequestMethod.GET)
    public List<Dept> list();
    
    @RequestMapping(value="/dept/add",method=RequestMethod.POST)
    public boolean add(Dept dept);
}
```

注：该方法提供了get、list和add方法。

b.创建DeptClientServiceFallbackFactory处理类

该类是Feign包中提供的，因此API模块不需要引入hystrix相关的依赖。这里也要注意与在服务端提供熔断有所区别。

```
package org.sjq.test.service;

import java.util.List;

import org.sjq.test.entities.Dept;
import org.springframework.stereotype.Component;

import feign.hystrix.FallbackFactory;

@Component//不要忘记添加注解
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService>{

    @Override
    public DeptClientService create(Throwable arg0) {
        return new DeptClientService() {
            
            //这里只处理get方法，处理由其他客户端调用已经被关掉的服务方法的处理方法
            @Override
            public Dept get(Long id) {
                return new Dept().setDeptNo(id).setDName("该ID："+id+"没有对应的信息，consumer客户端提供了服务降级信息，此刻服务不可用")
                        .setDb_source("data not in mysql");
            }
            
            @Override
            public List<Dept> list() {
                return null;
            }
            
            @Override
            public boolean add(Dept dept) {
                return false;
            }
        };
    }
}
```

c.在DeptClientService接口上修改注解FeignClient

在注解@FeignClient中指定fallbackFactory的实现类，即服务出现问题会到该接口中查找相应的处理方法。该接口中提供对DeptClientService接口所有方法的熔断处理方法。

```
@FeignClient(value="MICROSERVICECLOUD-DEPT",
    fallbackFactory=DeptClientServiceFallbackFactory.class)
public interface DeptClientService {
    ...
}
```

d.打包jar文件

```
mvn clean install
```

注：该jar会引入到consumer模块中，因此，consumer模块要开启熔断（在配置文件中完成）

3）修改microservicecloud-consumer-dept-feign-80模块的配置文件

修改配置文件application.yml，添加熔断机制

```
feign:
 hystrix:
  enabled: true
```

4）测试启动

```
nohup java -jar microservicecloudEureka7001.jar >/dev/null &
nohup java -jar microservicecloudEureka7002.jar >/dev/null &
nohup java -jar microservicecloudEureka7003.jar >/dev/null &
# 启动服务提供者
nohup java -jar microservicecloudProviderDept8001.jar >/dev/null &
# 启动服务消费者
nohup java -jar microservicecloudConsumerDeptFeign80.jar >/dev/null &
```

注意，这里启动的服务提供者是普通的服务提供者，并不是带熔断机制的服务提供者。带熔断机制的服务提供者为microservicecloudProviderDeptHytrix8001.jar

测试访问时正常，故意关闭microservicecloud-provider-dept-8001服务，然后再调用查看。