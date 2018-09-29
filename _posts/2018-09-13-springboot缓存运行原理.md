---
layout:     post
title:      springboot缓存运行原理
subtitle:   springboot缓存运行原理
date:       2018-09-13
author:     BY
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - springboot
    - 缓存
---
### 一、spring boot缓存原理:
#### 1、自动配置类:CacheAutoConfiguration，类中导入了缓存配置类：![](/img/CacheAutoConfiguration.png)

#### 2、当容器中配置了哪个缓存组件进去，那个缓存的配置类就会被注入，默认SimpleCacheConfiguation会被匹配上。

#### 3、SimpleCacheConfiguation向容器中注册了一个CacheManager:ConcurrentMapCacheManager
#### 当redis配置进来后，默认的RedisCacheManager会被注入进来，而默认的ConcurrentMapCacheManager会失效。  

#### 4、getCache（）方法可以获取和创建ConcurrentMapCache缓存组件  ConcurrentMapCache的作用是将数据保存到ConcurrentMap中，spring boot默认将数据保存到ConcurrentMap中，作为缓存。

### 二、@Cacheable注解的运行流程:
```
缓存的可以的生成策略:
key(默认为方法参数)默认使用SimpleKeyGenerator生成的
   SimpleKeyGenerator的生成key的默认策略:
     1>如果没有参数，key=new SimpleKey();
     2>如果有参数，key=参数的值;
     3>如果有多个参数，key=new SimpleKey(params);
```
### 流程:
#### 1、目标方法运行之前，先去查询Cache（缓存组件),按照cacheName指定的名字进行获取
（CacheManager先获取对应的缓存），第一次获取缓存时，如果没有Cache组件会自动创建。
#### 2、去Cache查找缓存（ConcurrentMapCache的lookup方法）的内容，使用key查询。
#### 3、如果没有查到缓存，就调用目标方法。
#### 4、将目标方法返回的结果放进缓存中。

```
总结:@Cacheable标注的方法，在方法执行之前先去检查缓存中是有此数据，默认按照参数的值作为key查询缓存，如果没有就执行方法并将结果放入到缓存里，再调用方法时，就可以使用缓存中的数据
```