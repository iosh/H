title: c++ algorithms
date: 2019-12-15 19:02:58
tags: algorithms

---

learn c++ algorithms

<!-- more -->

# 第一章

主要是 c++ 基础回顾

## 函数与参数

函数 `形参` `实参` 区别和 `按值传参` 等基础知识。

### 模板函数

```cpp
template<class T> T test(T a, T b, T c){return a + b + c};
```

### 引用参数

`引用参数` 基础，节省内存减少开销加快运行速度。

### 常量引用参数

`c++ 常量引用` 不可修改引用。

```cpp
template<class T> T test(const T&a, const T&b, const T&c) {return a + b + c};
```

### 返回值

基础

### 重载函数

定义多个同名不同参数的函数称为`函数重载`，编译器通过匹配函数参数列表来匹配实际调用的函数。
