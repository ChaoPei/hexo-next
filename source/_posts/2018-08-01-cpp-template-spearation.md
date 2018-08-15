---
title: C++模板声明和定义分离
date: 2018-08-01 09:19:44
update: 2018-08-01 09:19:44
categories: C++
tags: [C++, 模板, 定义, 声明]
---

C++有一个重要特性就是不支持分离编译，因此对于公共的模板函数或者模板类的声明和实现，一般都要放到头文件中。

<!--more-->

## 1.C++的编译和链接

首先我们来看看C++中的代码如何处理成为可执行文件。在C++中，一个编译单元是指一个`.cpp`文件以及它所`#include`的所有头文件，因为头文件的代码将会被扩展到包含它的`.cpp`文件中，因此C++中常见的做法就是在头文件中进行函数的声明，在同名的`.cpp`文件中进行函数的定义（实现），然后编译该`.cpp`文件为一个`.obj`文件（win32），后者拥有PE（Portable Executable）文件格式，并且本身包含的就已经是二进制码。但是这个文件不一定能够执行，因为不能够保证其中一定有main函数的入口。当编译器将一个工程里的所有`.cpp`文件以分离的方式编译完成之后，再由链接器（linker）链接成为可执行文件（exe）。

示例1：

```cpp
//---------------test.h-------------------//

void f();//这里声明一个函数f


//---------------test.cpp--------------//

#include”test.h”

void f() {
    //do something
    //这里实现出test.h中声明的f函数
}


//---------------main.cpp--------------//

#include”test.h”

int main(){
    f(); //调用f，f具有外部链接类型
}
```

在这个例子中，`test. cpp`和`main.cpp`各自被编译成不同的`.obj`文件（姑且命名为`test.obj`和`main.obj`），在`main.cpp`中，调用了`void f()` 函数，然而当编译器编译`main.cpp`时，它所仅仅知道的只是`main.cpp`中所包含的`test.h`文件中的一个关于`void f();`的声明，所以，编译器将这里的`void f()`看作外部链接类型，即认为它的函数实现代码在另一个`.obj`文件中，本例也就是`test.obj`，也就是说，**`main.obj`中实际没有关于f 函数的哪怕一行二进制代码，而这些代码实际存在于`test.cpp`所编译成的`test.obj`中**。这个一定要牢记，这是分离编译模式固有的。

在`main.obj`中对`void f()`的调用只会生成一行call指令，像这样：

```
call f [C++中这个名字当然是经过mangling[处理]过的]
```

在编译时，这个call指令显然是错误的，因为`main.obj`中并无一行`void f()`的实现代码。那怎么办呢？编译器完成了，那么链接器就要登场了，**链接器负责在其它的`.obj`中（本例为`test.obj`）寻找`void f()`的实现代码，找到以后将`call f`这个指令的调用地址换成实际的`void f()`的函数进入点地址**。需要注意的是：链接器实际上将工程里的多个`.obj`链接成了一个`.exe`文件，而它最关键的任务就是上面说的，**寻找一个外部链接符号在另一个`.obj中`的地址，然后替换原来的“虚假”地址**。

这个过程如果说的更深入就是：

`call f`这行指令其实并不是这样的，它实际上是所谓的`stub`，也就是一个`jmp 0xABCDEF`。这个地址可能是任意的，然而关键是这个地址上有一行指令来进行真正的`call f`动作。也就是说，这个`.obj`文件里面所有对`void f()`的调用都jmp向同一个地址，在后者那儿才真正`call f`。这样做的好处就是链接器修改地址时只要对后者的`call XXX`地址作改动就行了。但是，链接器是如何找到`void f()`的实际地址的呢（在本例中这处于`test.obj`中），因为`.obj`与`.exe`的格式是一样的，在这样的文件中有一个**符号导入表和符号导出表（import table和export table）**，其中将所有符号和它们的地址关联起来。这样链接器只要在`test.obj`的符号导出表中寻找符号f（当然C++对f作了mangling）的地址就行了，然后作一些偏移量处理后（因为是将两个`.obj`文件合并，当然地址会有一定的偏移，这个链接器会自己处理）写入`main.obj`中的符号导入表中f所占有的那一项即可。

这就是大概的过程，总结其中最关键的几点就是：

* 编译`main.cpp`时，编译器不知道`void f()`的实现，所以当碰到对它的调用时只是给出一个指示，指示链接器应该为它寻找`void f()`的实现体。这也就是说`main.obj`中没有关于f的任何一行二进制代码。

* 编译`test.cpp`时，编译器找到了`void f()`的实现。于是`void f()`的实现（二进制代码）出现在`test.obj`里。

* 链接时，链接器在`test.obj`中找到`void f()`的实现代码（二进制）的地址（通过符号导出表）。然后将`main.obj`中悬而未决的`call XXX`地址改成`void f()`实际的地址，完成。

## 2.模板的实例化

讲完了C++一般的编译和链接，再看看模板。模板有一个非常重要的特性就是“实例化”：模板函数的代码其实并不能直接编译成二进制代码，其中要有一个实例化的过程。

示例2：
```cpp
template <typename T>
void f(T t) {
    // 函数实现
}

int main() {

    f(10);  //  call f<int> 编译器在这里决定给f一个f<int>的实例

    // ...
}
```

也就是说，如果你在main.cpp文件中没有调用过f，f也就得不到实例化，从而main.obj中也就没有关于f的任意一行二进制代码！如果你这样调用了：

`f(10);` // f<int>()得以实例化出来

`f(10.0);` // f<double>()得以实例化出来

这样`main.obj`中也就有了`f<int>()`，`f<double>()`两个函数的二进制代码段。以此类推。然而实例化要求编译器知道模板的定义，也就是说**在编译器所编译的单元中，需要看到模板的定义**，看下面的例子（将模板的声明和实现分离）。

示例3：
```cpp

//-------------test.h----------------//
template <typename T>
class A {
public:
    void f();   // 这里只是个声明
};


//---------------test.cpp-------------//
#include "test.h"

template <typename T>
void A<T>::f();     // 模板的实现


//---------------main.cpp---------------//

#include "test.h"

int main() {
    A<int> a;
    a.f();      // #1 实际调用

    // ....
}
```

分析一下上述的程序的编译和链接过程：

1. 编译器在`#1`处调用了模板函数，需要实例化，实例化就必须知道模板的定义，然后在`#1`处并不知道`A<int>::f()`的定义，因为它不在`test.h`里面；
2. 于是编译器只好寄希望于连接器，希望它能够在其他`.obj`里面找到`A<int>::f()`的实例，在本例中就是`test.obj`；
3. 因为C++标准明确表示，**当一个模板不被用到的时侯它就不该被实例化出来**，`test.cpp`并没有用到`A<int>::f()`，所以实际上`test.cpp`编译出来的`test.obj`文件中也没有关于`A<int>::f`的二进制代码。
4. 于是连接器就傻眼了，只好给出一个连接错误。

但是，如果在`test.cpp`中写一个函数，其中调用`A<int>::f()`，则编译器会将其实例化出来，因为在这个点上（`test.cpp`中），编译器知道模板的定义，所以能够实例化，于是，`test.obj`的符号导出表中就有了`A<int>::f()`这个符号的地址，于是连接器就能够完成任务。

## 3.总结

综合上述分析，**总结如下**：

在分离式编译的环境下，编译器编译某一个.cpp文件时并不知道另一个.cpp文件的存在，也不会去查找（当遇到未决符号时它会寄希望于连接器）。这种模式在没有模板的情况下运行良好，但遇到模板时就傻眼了，因为模板仅在需要的时候才会实例化出来，所以，当编译器只看到模板的声明时，它不能实例化该模板，只能创建一个具有外部连接的符号并期待连接器能够将符号的地址决议出来。然而当实现该模板的`.cpp`文件中没有用到模板的实例时，编译器懒得去实例化，所以，整个工程的`.obj`中就找不到一行模板实例的二进制代码，于是连接器也黔驴技穷了。