---
title: Redis代码中的内存分配zmalloc
date: 2020-10-14 23:19:15
categories:
- C++
---

## 一、malloc
* redis内存管理中实现了对多种接口的封装，tcmalloc(google)，jemalloc(facebook)，其他（mac系统等）;
* redis3.0中linux使用的是jemalloc。其中Makefile中读取到系统类型`uname_S`是`Linux`，设置`MALLOC=jemalloc`。后面判断MALLOC为jemalloc时，定义环境变量`USE_JEMALLOC`

