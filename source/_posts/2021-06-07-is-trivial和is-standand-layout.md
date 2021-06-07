---
title: is_trivial和is_standand_layout
date: 2021-06-07 22:03:18
categories:
- C++
---

## 一、简介
* POD，全称plain old data，plain代表它是一个普通类型，old代表它可以与C兼容，可以使用比如memcpy()这类C中最原始的函数。C++11把POD分为了两个基本概念的集合。即：trivial(平凡的)和standard layout(标准布局的)。
* 下面主要针对std中的两个接口进行说明，具体和C++标准中的概念是否一一对应，有待进一步看看。

<!--more-->

## 二、`std::is_trivial`
* `template <class T> struct is_trivial;`
* 一个trivial类型满足：1. 存储连续(trivially copyable) 2. 只支持静态默认的初始化(trivially default constructiable)。且不管它是否为cv-qualified(const-qualified\volatile-qualified\const-volatile-qualified)。
* 包括scalar types(arithmetic \ pointer \ member pointer \ std::nullptr_t), trivial classes 或者上述类型的数组类型；
* trivial classes 是通过class、struct、union定义的，并且同时满足trivially default constructable和trivially copyable，即：
	1. 使用显式定义的、默认的copy and move constructors, copy and move assignments, and destructor。我们不能自定义上述函数，函数体为空也不行，但是可以通过`=default`指出;
	2. 没有virtual成员
	3. 没有采用brace- or equal- initializers方式初始化的非静态成员，其中brace or equal initializers的例子如下：
	``` cpp
	class X {
		int i = 4;
		int j {5};
	};
	```
	4. 如果判断目标是一个派生类，那它的基类必须是trivial type，如果目标作为一个类，它的非静态成员也必须是trivial type（递归定义）：
* 与`is_trivial`相对应，还有一系列函数`is_trivially_constructible`、`is_trivially_copy_assignable`等，限制的是某一方面的特点

## 三、`std::is_standard_layout`
* `template <class T> struct is_standard_layout;`
* 一个standard-layout类型指的是简单线性的结构，并且很容易和其他语言进行交互，比如C语言;
* 包括scalar types，standard-layout classes 和任意上述类型的数组类型;
* standard-layout class指的是通过class、struct、union定义的，满足以下条件的结构:
	1. 不包含虚函数，没有虚基类;
	2. 对于结构内的所有非静态成员，都有相同的访问权限，即private、public、protected等;
	3. 要么，在most derived class中没有非静态成员，并且最多只有一个基类，这个基类不能包含非静态成员;
	要么没有包含非静态成员的基类；
	其中most derived class，表示初始化对象时用的类型:
	``` cpp
	class base {};
	class derived : base {};
	class base2 {};
	class mostderived : derived, base2 {};

	mostderived md;
	```
	这里md的most derived class是实例化对象用的类型mostderived
	4. 它的基类也必须是standard-layout class, 并且第一个非静态成员不是基类的类型;

## 四、总结
* 描述如此之复杂，虽然看懂了，但感觉也没找到这些定义的规律，所以使用的时候，还是要继续翻下这些接口的标准定义。不过记录一下后，可以按照这些记录大致找回记忆，再去看接口的标准定义进行细化理解。
* 其实平时使用也不需要这么细致
* 使用memcopy memset等函数时，可以看下这些接口的注意事项，比如potentially-overlapping subobject 和not TriviallyCopyable的对象是不能使用这些函数的。其中，union中两个字段如果部分重叠，就会产生overlapping。

