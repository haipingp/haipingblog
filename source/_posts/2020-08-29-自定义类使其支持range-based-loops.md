---
title: 自定义类使其支持range-based-loops
date: 2020-08-29 18:18:53 
categories:
- C++
---

## 一、motivation
* 看项目代码的时候，同事用了，不明白为什么可以这么用，所以搜了一下

<!-- more -->
* 代码类似这样：
``` cpp
MyMap mymap;
mymap.load();
for (auto& [id, val] : mymap)
{
    printf("id: %u, val: %f\n", id, val);
}
```

## 二、类中通过begin和end声明范围
* 下面基本上是抄这里的:
`https://stackoverflow.com/questions/8164567/how-to-make-my-custom-type-to-work-with-range-based-for-loops`

* 可以在类中通过`begin()`和`end()`声明遍历的初始及结束条件，如下所示：
``` cpp
class MyMap
{
    public:
        MyMap(){}
        ~MyMap(){}
        typedef typename std::map<uint32_t, double>::const_iterator _mymap_iter;
        _mymap_iter begin() const
        {
            return _realmap.begin();
        }
        _mymap_iter end() const
        {
            return _realmap.end();
        }
        void load()
        {
            _realmap[1] = 0.1;
            _realmap[2] = 0.2;
            _realmap[3] = 0.3;
        }

    private:
        std::map<uint32_t, double> _realmap;
};
```

* 看到了吗，看到了吗，这里只要这样简单返回了初始与结束的iterator就可以了。那么还有个问题，这里`begin()`和`end()`返回的只能是迭代器吗。不是的。
* 这里`begin()`和`end()`只是定义了一个开始和结束的条件，一般的`for(:) loop`是这样的：
``` cpp
for( range_declaration : range_expression )
```
其等同于：
``` cpp
// (until C++17)
{
auto && __range = range_expression;
for (auto __begin = begin_expr,
          __end = end_expr;
          __begin != __end; ++__begin) {
          range_declaration = *__begin;
          loop_statement
          }
}
```

* 从上述扩展可以看到，这里`begin_expr`和`end_expr`不一定是迭代器，但是`begin()/end()`返回的值需要满足几个条件：
    1. 需要支持前置`++`算子
    2. `!=`在这里是有效的（我的理解是这里end()返回的值是能够达到的，我还没想到这里可能是浮点数的情况）
    3. `*`算子返回的值是有效的，比如这里需要一个`public`的构造函数，才能返回相应的值

* 但是`begin()/end()`返回的如果不是迭代器的话，后面也是很可能随着C++标准的变化，而不再支持。所以这里推荐返回的就是`iterator`类型的值。
* C++17开始，上述`for(:) loop`的定义变了，它变了:
``` cpp
// since c++17
{
  auto && __range = range_expression ;
    auto __begin = begin_expr;
    auto __end = end_expr;
    for (;__begin != __end; ++__begin) {
        range_declaration = *__begin;
        loop_statement
    }
}
```

* 上述变化是，begin和end的类型可以不同，不再耦合。

## 三、待续...
* 我感觉这一块还可以dfs一下，我现在有点累，被之前的自己搭的博客折磨的累了。有生之年再补充...我是真的菜
