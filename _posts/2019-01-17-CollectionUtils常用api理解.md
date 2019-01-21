---
layout:     post
title:      CollectionUtils常用api总结
subtitle:   教你如何使用CollectionUtils
date:       2019-01-17
author:     pengfeng
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - 常用工具类
    - CollectionUtils
---

> CollectionUtils

#### 1.依赖

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>2.0.6.RELEASE</version>
    </dependency>
    
#### 2.全限定名

    org.springframework.util.CollectionUtils;
    
#### 3.常用api
1.containsAny：判断两个集合是否有交集，有则返回true，否则返回false

    boolean any = CollectionUtils.containsAny(list1, list2);
2.
   
 
 



