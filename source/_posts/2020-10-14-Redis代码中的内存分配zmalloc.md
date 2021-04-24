---
title: Redis代码中的内存分配zmalloc
date: 2020-10-14 23:19:15
categories:
- C++ 
- redis
---

## 一、`zmalloc(size_t size)`
<!-- more -->

### 1.1 函数源码
``` c
void *zmalloc(size_t size) {
    void *ptr = malloc(size+PREFIX_SIZE);

    if (!ptr) zmalloc_oom_handler(size);
#ifdef HAVE_MALLOC_SIZE
    update_zmalloc_stat_alloc(zmalloc_size(ptr));
    return ptr;
#else
    *((size_t*)ptr) = size;
    update_zmalloc_stat_alloc(size+PREFIX_SIZE);
    return (char*)ptr+PREFIX_SIZE;
#endif
}
```

### 1.2 `void *ptr = malloc(size+PREFIX_SIZE);`
* `void *ptr = malloc(size+PREFIX_SIZE);`中调用了malloc函数，并且该函数分配内存时，增加了`PREFIX_SIZE`的大小；
* 下面是`PREFIX_SIZE`的定义，定义了`HAVE_MALLOC_SIZE`时，`PREFIX_SIZE`为0，否则根据下面的条件有两种大小：`sizeof(long long)`和`sizeof(size_t)`


``` c
#ifdef HAVE_MALLOC_SIZE
#define PREFIX_SIZE (0)
#else
#if defined(__sun) || defined(__sparc) || defined(__sparc__)
#define PREFIX_SIZE (sizeof(long long))
#else
#define PREFIX_SIZE (sizeof(size_t))
#endif
#endif
```
* 下面是`HAVE_MALLOC_SIZE`的定义。redis内存管理中实现了对多种接口的封装，tcmalloc(google)，jemalloc(facebook)，其他（mac系统等）。
redis3.0中linux使用的是jemalloc。其中Makefile中读取到系统类型`uname_S`是`Linux`，设置`MALLOC=jemalloc`。后面判断MALLOC为jemalloc时，定义环境变量`USE_JEMALLOC`。
同时，根据jemalloc.h中关于`JEMALLOC_VERSION_MAJOR`等变量的定义，可知，这里`HAVE_MALLOC_SIZE`等于1。
``` c
#elif defined(USE_JEMALLOC)
#define ZMALLOC_LIB ("jemalloc-" __xstr(JEMALLOC_VERSION_MAJOR) "." __xstr(JEMALLOC_VERSION_MINOR) "." __xstr(JEMALLOC_VERSION_BUGFIX))
#include <jemalloc/jemalloc.h>
#if (JEMALLOC_VERSION_MAJOR == 2 && JEMALLOC_VERSION_MINOR >= 1) || (JEMALLOC_VERSION_MAJOR > 2)
#define HAVE_MALLOC_SIZE 1
#define zmalloc_size(p) je_malloc_usable_size(p)
#else
#error "Newer version of jemalloc required"
#endif
```
* 因此，根据上面可知，根据不同的系统来决定，是否需要定义`PREFIX_SIZE`来表示内存的大小。

### 1.3 `update_zmalloc_stat_alloc(zmalloc_size(ptr));`
* 函数`update_zmalloc_stat_alloc`用于更新已分配的内存大小
* 从下面代码可知，没有定义`HAVE_MALLOC_SIZE`、有`PREFIX_SIZE`时，采用下面定义的函数`zmalloc_size`。这里ptr指向的是分配的内存开始位置，在存放内存大小的内存之后，因此`realptr`用于指向存放内存大小的头部。
``` c
#ifndef HAVE_MALLOC_SIZE
size_t zmalloc_size(void *ptr) {
    void *realptr = (char*)ptr-PREFIX_SIZE;
    size_t size = *((size_t*)realptr);
    /* Assume at least that all the allocations are padded at sizeof(long) by
     * the underlying allocator. */
    if (size&(sizeof(long)-1)) size += sizeof(long)-(size&(sizeof(long)-1));
    return size+PREFIX_SIZE;
}
#endif
```

* `update_zmalloc_stat_alloc`函数：
``` c
#define update_zmalloc_stat_alloc(__n) do { \
    size_t _n = (__n); \
    if (_n&(sizeof(long)-1)) _n += sizeof(long)-(_n&(sizeof(long)-1)); \  
    if (zmalloc_thread_safe) { \
        update_zmalloc_stat_add(_n); \
    } else { \
        used_memory += _n; \
    } \
} while(0)
```
* `update_zmalloc_stat_alloc`函数的第三行等价于`if(_n&7) _n += 8 - (_n&7);`，用于判断`_n`是否为8的倍数，如果不是8的倍数，加上相应的偏移量，使之为8的倍数。这里不用`_%8`是因为位操作效率更高。
* 之前malloc分配内存时，需要补齐为8的倍数的。因此，这里通过这样补齐`_n`才能正确计算占用的内存字节数。
* 这里如果是线程安全的话，需要采用系统的原子增加n接口`__sync_add_and_fetch`，或者线程锁;如果不需要线程安全的话，直接给`used_memory`加n就行。
* 在`zmalloc`函数中，如果未定义`HAVE_MALLOC_SIZE`，需要通过分配的额外`PREFIX_SIZE`空间来自行存储分配的内存大小。

