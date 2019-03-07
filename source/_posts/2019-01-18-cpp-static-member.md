---
title: C++类的静态成员初始化
date: 2019-01-18 09:19:44
update: 2019-01-18 09:19:44
categories: C++
tags: [C++, 静态成员, static, 类, 初始化]
---

C++类的静态成员初始化不同于普通成员，有一定的限制和要求。

<!-- more -->

### C++类的静态成员初始化特性

1. 静态成员可以直接被静态或者非静态成员函数访问，不需要作用域运算符。

2. 静态成员函数不可以声明为const，同时不隐式传入this指针，也不能能使用this指针。

3. 一般在类内声明静态成员，在类外定义静态成员变量，类外定义时不需要`static`关键字。

4. 类的普通静态成员不能在类内初始化，只能在外部初始化，一般在类定义的`.cpp`文件中初始化。

    > 因为静态数据成员不属于类的任何一个对细节，所以它们并不是在创建类的对象时被定义的，这意味着它们不是由类的构造函数初始化的。而且一般来说，我们不能在类的内部初始化静态成员。相反的，必须在类的外部定义和初始化每个静态成员。和其他对象一样，一个静态数据成员只能定义一次。

    > 类似于全局变量，静态数据成员定义于任何函数之外。因此一旦它被定义就存在于整个生命周期中。

（以上来自c++ primer 5th）

5. **如果想在类内初始化，则应该将静态成员声明为constexpr，也就是常量表达式的形式，而且必须是整数类型（不一定是整型，char也行），同时，用于初始化它们的初始值也必须是常量表达式。**

### 总结

对于类的静态成员变量，只有常量表达式的整数类型才能在类的内部定义和初始化，否则只能在类内声明，类外定义（并且只能定义一次，但是全局有效）。