> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_42745404/article/details/117376963)

### 文章目录

*   [引 spring-boot-starter-cache 漫画](#springbootstartercache_2)
*   [spring-boot-starter-cache 项目整合 demo](#springbootstartercachedemo_6)
*   *   [项目结构](#_7)
    *   [pom.xml](#pomxml_9)
    *   [RedisConfig.java 配置好对应缓存对应的配置](#RedisConfigjava__47)
    *   [HelloRespDTO.java](#HelloRespDTOjava_86)
    *   [HelloService.java](#HelloServicejava_125)
    *   [HelloController.java](#HelloControllerjava_156)
    *   [DemoApplication.java 启动类](#DemoApplicationjava__187)
    *   [效果展示](#_202)
    *   [demo 地址](#demo_212)
*   [引 J2Cache 漫画](#J2Cache_219)
*   [J2Cache 整合 springboot-demo](#J2Cachespringbootdemo_223)
*   *   [项目结构](#_224)
    *   [j2cache 配置文件](#j2cache_227)
    *   [HelloService.java](#HelloServicejava_265)
    *   [HelloRespDTO.java](#HelloRespDTOjava_299)
    *   [HelloController.java](#HelloControllerjava_340)
    *   [DemoApplication.java](#DemoApplicationjava_373)
    *   [效果展示](#_388)
    *   [demo 地址](#demo_400)
    *   [原理图](#_406)
    *   [二级缓存框架的连接](#_411)
*   [参考连接](#_416)

引 spring-boot-starter-cache 漫画
==============================

> 针对热数据，一般我们会把他放到缓冲里，减轻数据库的压力，针对集群下的缓冲，一般我们会把缓冲放到 redis 里

![](https://picture.lingzero.cn/img/202209142156945.png)![](https://picture.lingzero.cn/img/202209142200542.png)

spring-boot-starter-cache 项目整合 demo
===================================

项目结构
----

![](https://img-blog.csdnimg.cn/20210530114726387.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNzQ1NDA0,size_16,color_FFFFFF,t_70)

pom.xml
-------

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
    </dependencies>
</project>
```

RedisConfig.java 配置好对应缓存对应的配置
-----------------------------

```java
package com.example.demo.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.cache.RedisCacheWriter;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@EnableCaching
@Configuration
public class RedisConfig {
    @Autowired
    RedisConnectionFactory redisConnectionFactory;
    @Bean
    public CacheManager cacheManager() {
        // 设置缓存有效期1小时
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofHours(1))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer((new StringRedisSerializer())))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer((new GenericJackson2JsonRedisSerializer())))
                .computePrefixWith((name)->name+":");
        return RedisCacheManager
                .builder(RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory))
                .cacheDefaults(redisCacheConfiguration).build();
    }
}
```

HelloRespDTO.java
-----------------

```java
package com.example.demo.dto;

import java.io.Serializable;

public class HelloRespDTO implements Serializable {

    private String name;

    private String address;

    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

HelloService.java
-----------------

```java
package com.example.demo.service;

import com.example.demo.dto.HelloRespDTO;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class HelloService {

    @Cacheable(cacheNames = "employee",key = "'detail'+#id")
    public HelloRespDTO helloCache(String id){
        HelloRespDTO hello = new HelloRespDTO();
        hello.setName("ben");
        hello.setAddress("广州市白云区");
        hello.setAge(18);
        return hello;
    }

    @CacheEvict(value = "employee",key = "'detail'+#id")
    public HelloRespDTO helloClear(String id){
        HelloRespDTO hello = new HelloRespDTO();
        hello.setName("ben");
        hello.setAddress("广州市白云区");
        hello.setAge(18);
        return hello;
    }
}
```

HelloController.java
--------------------

```java
package com.example.demo.controller;

import com.example.demo.dto.HelloRespDTO;
import com.example.demo.service.HelloService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    HelloService helloService;

    @GetMapping("/hello")
    public HelloRespDTO hello(){
        String id="1";
        HelloRespDTO helloRespDTO = helloService.helloCache(id);
        return helloRespDTO;
    }

    @GetMapping("/hello2")
    public HelloRespDTO helloClear(){
        String id="1";
        HelloRespDTO helloRespDTO = helloService.helloClear(id);
        return helloRespDTO;
    }
}
```

DemoApplication.java 启动类
------------------------

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

效果展示
----

访问网址：

```
http://localhost:8889/hello/hello
```

![](https://img-blog.csdnimg.cn/20210530115814819.png)  
redis 里面已经有数据了  
![](https://img-blog.csdnimg.cn/20210530115935874.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNzQ1NDA0,size_16,color_FFFFFF,t_70)  
下次走 helloCache 接口时，从 redis 里拿取

demo 地址
-------

```
https://gitee.com/null_751_0808/spring-boot-demo
分支：origin/spring-boot2-cache-manager
```

![](https://picture.lingzero.cn/img/202209142156482.png)

引 J2Cache 漫画
============

![](https://picture.lingzero.cn/img/202209142156556.png)  
![](https://img-blog.csdnimg.cn/20210530145920298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNzQ1NDA0,size_16,color_FFFFFF,t_70)  
![](https://picture.lingzero.cn/img/202209142200174.png)

J2Cache 整合 springboot-demo
==========================

项目结构
----

![](https://picture.lingzero.cn/img/202209142156261.png)

j2cache 配置文件
------------

j2cache.properties  
这里不放出来了，主要是配置一级缓存是什么，[二级缓存](https://so.csdn.net/so/search?q=%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98&spm=1001.2101.3001.7020)是什么，redis 广播通知节点的方式，想看的可以去官网看，或者下载此 demo 看  
[我的 demo 配置](https://gitee.com/null_751_0808/spring-boot-j2-cache/blob/master/src/main/resources/j2cache.properties)  
![](https://picture.lingzero.cn/img/202209142156321.png)  
一级缓存用了 caffeine, 二级缓存用了 lettuce(连接 redis 的)

application.yml 配置对应 j2cache

```yaml
server:
  port: 8889
  servlet:
    context-path: /hello
spring:
  application:
    name: hello
  redis:
    host: 127.0.0.1
    port: 6379
    password:
    database: 2
  cache:
    type: none
j2cache:
  config-location: /j2cache.properties
  redis-client: lettuce
  open-spring-cache: true
```

caffeine.properties

```properties
#########################################
# Caffeine configuration
# [name] = size, xxxx[s|m|h|d]
#########################################

default = 1000, 30m
```

HelloService.java
-----------------

```java
package com.example.demo.service;

import com.example.demo.dto.HelloRespDTO;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class HelloService {

    //加缓存
    @Cacheable(value="employee",key = "'detail'+#id")
    public HelloRespDTO helloCache(String id){
        HelloRespDTO hello = new HelloRespDTO();
        hello.setName("ben");
        hello.setAddress("广州市白云区");
        hello.setAge(18);
        return hello;
    }

    //清缓存
    @CacheEvict(value="employee",key = "'detail'+#id")
    public HelloRespDTO helloClear(String id){
        HelloRespDTO hello = new HelloRespDTO();
        hello.setName("ben");
        hello.setAddress("广州市白云区");
        hello.setAge(18);
        return hello;
    }
}
```

HelloRespDTO.java
-----------------

```java
package com.example.demo.dto;


import java.io.Serializable;

public class HelloRespDTO implements Serializable {

    private String name;

    private String address;

    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

HelloController.java
--------------------

```
package com.example.demo.controller;

import com.example.demo.dto.HelloRespDTO;
import com.example.demo.service.HelloService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    HelloService helloService;

    @GetMapping("/hello")
    public HelloRespDTO hello(){
        String id="1";
        HelloRespDTO helloRespDTO = helloService.helloCache(id);
        return helloRespDTO;
    }

    @GetMapping("/hello2")
    public HelloRespDTO helloClear(){
        String id="1";
        HelloRespDTO helloRespDTO = helloService.helloClear(id);
        return helloRespDTO;
    }

}
```

DemoApplication.java
--------------------

```
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

效果展示
----

访问

```
http://localhost:8889/hello/hello
```

![](https://img-blog.csdnimg.cn/20210530152947981.png)  
![](https://picture.lingzero.cn/img/202209142156797.png)  
从源码这里可以看到 j2cache 缓存  
AbstractCacheInvoker doGet 方法  
![](https://picture.lingzero.cn/img/202209142156352.png)

demo 地址
-------

```
https://gitee.com/null_751_0808/spring-boot-demo/tree/spring-boot2-j2cache/
分支：origin/spring-boot2-j2cache
```

![](https://picture.lingzero.cn/img/202209142156624.png)

原理图
---

redis 订阅与发布  
![](https://picture.lingzero.cn/img/202209142156092.png)  
![](https://picture.lingzero.cn/img/202209142156459.png)

二级缓存[框架](https://so.csdn.net/so/search?q=%E6%A1%86%E6%9E%B6&spm=1001.2101.3001.7020)的连接
---------------------------------------------------------------------------------------

[J2Cache](https://gitee.com/ld/J2Cache)  
[hotkey](https://gitee.com/jd-platform-opensource/hotkey?spm=a2c6h.12873639.0.0.6241765fJ6pES3)  
[redis6.0 客户端缓存](https://www.cnblogs.com/remcarpediem/p/12872053.html)

参考连接
====

[SpringBoot 中 Cache 缓存的使用](https://blog.csdn.net/weixin_36279318/article/details/82820880?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162229841316780274152971%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162229841316780274152971&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~sobaiduend~default-1-82820880.nonecase&utm_term=springboot+cache&spm=1018.2226.3001.4450)  
[@Cacheable 缓存解决双冒号:: 问题](https://blog.csdn.net/u012725623/article/details/107671343)