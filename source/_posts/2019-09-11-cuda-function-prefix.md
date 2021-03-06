---
title: CUDA 函数前缀
date: 2019-09-11 14:46:47
update: 2019-09-11 14:46:47
categories: C/C++
tags: [cuda, global, host, device]
---

CUDA 函数前缀作为 CUDA 编程中一种特殊的使用技巧，其具有一定的限制意义。

<!--more-->

CUDA 使用 cu 作为文件类型后缀，而在文件中又存在几种前缀，如果在修改编写 .cu 文件时不注意，会出现问题，比如：

```bash
error : calling a __host__ function from a __global__ function is not allowed.
```

在 CUDA 中有三种常见的前缀：`__device__`， `__global__`，`__host__`，其分别代表不同的意思，而这三个单词其实也是 CUDA 中常见的三种运行场景：

| 限定符 | 执行 | 调用 | 备注 |
| :---: | :---: | :---: | :---: |
| \_\_global\_\_ | 设备端执行 | 可以从主机调用也可以从计算能力3以上的设备调用 | 必须有一个void的返回类型 |
| \_\_device\_\_ | 设备端执行 | 设备端调用 | |
| \_\_host\_\_ | 主机端执行 | 主机调用 | 可以省略 |

因为函数前缀设定了函数的运行环境，因此对函数内部实现也做出了一定的限制，具体来说就是，device 函数因为只能在 GPU 上执行，因此不能调用常见的一些 C/C++ 函数（没有 GPU 实现），global 函数虽然能在 CPU 上运行，但是也能在 GPU 上面运行，因此同理。host 函数则没有这个限制，可以调用普通函数实现。

因此，在出现报错如：`error : calling a __host__ function from a __global__ function is not allowed.` 时候，即为将一个普通的函数错误地添加进入了 global 前缀定义函数，在 .cu 文件中是不允许的。

注意，有时候一个函数可以同时被多个前缀修饰的，比如 CUDA 10 浮点数的转换：

```c++
__host__ ​ __device__ ​ __half 	__float2half ( const float  a ) throw ( )
```

以上修饰这个函数可以在 host 端被调用，也可以在 device 端被调用。实际上这个函数在 CUDA 9.2 以后才允许在 host 端调用，其 CUDA 8.0 的版本：

```c++
__device__ ​ __half __float2half ( const float  a )
```

因此，我们通过一个函数的前缀就可以判断这个函数的运行环境。
