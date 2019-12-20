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

## 异常

### 抛出异常

```cpp
throw "error"
```

cpp 的异常是从基类 `exception` 派生而来的。

### 处理异常

```cpp
catch(char * e){};

catch(bad_alloc e){};

try {

} catch(){

}
```

## 动态储存空间分配

-

### 操作符 new

cpp 操作符 new 用来进行动态储存分配或进行储存分配，它的值是一个指针，指向所分配空间。

```cpp
int *y = new int;
y = 1000;

int *y;
y = new int(1000);

int *z = new int(10000);
```

### 一维数组

通过 new 来实现运行时的数组分配，它们随着函数调用的变化而变化，采用如下方法进行动态分配。

```cpp
float * x = new float[1000];
```

### 异常处理

```cpp
float *x =float[n];
```

对于 n 个浮点数，计算机可能没有足够的内存可以分配，在这种情况下 new 也不会分配内存，而是抛出一个 bad_alloc 的异常，可以通过 try-catch 进行捕捉错误。

```cpp
float *x;
try{
  x = new float[n];
} catch (bad_alloc e) {
  cerr << "Out of Memory" << endl;
  exit(1);
}
```

### 操作符 delete

动态分配的储存空间不需要时应该把它释放掉，通过调用`delete` 操作符来释放由 `new` 所分配的空间。

```cpp
int*x = new int(1000);
delete x;
```

### 二维数组

```cpp
char c[7][5]
```

```cpp
char(*c)[5];
try {
  c = new char[n][6];
}catch (bad_alloc e) {
  cerr << "Out of Memory" << endl;
  exit(1);
}
```

动态创建一个二维数组

```cpp
template<class T>
bool make2dArray(T **&x, int numberOfRows, int numberOfColumns){
  try {
    x = new T * [numberOfRows];

    for(int i = 0; i < numberOfRows; i++) {
      x[i] = new int[numberOfColumns];
    }
		return true;
  }catch(bad_alloc){
    return false
  }
}
```

## 自有数据类型

### 类 currency

自定义数据类型使用 cpp 的 class 类

```cpp
class currency {
  public:
  	currency(signType theSIgn = plus, unsigned long theDollars = 0, unsigned int theCents = 0);
  ~currency(){};

  void setValue(signType, unsigned long, unsigned int); // 构造函数
  void setValue(double);
  signType getSign() const (return sign); //常量函数
  unsigned long getDollars() const {return dollars};
  unsigned int getCents()const (return cents);
  currency add(const currency&) const;
  currency& increment(const currency&);
  void output()const;

  private:
  signType: sign;
  unsigned long dollars;
  unsigned int cents;
}
```

### #ifndef #define #endif

包含在这组语句之内的代码只编译一次。

```cpp
#ifndef Currency_
#define Currency_



#edfif
```

## 异常类

```cpp
class illegalParameterValue {
  public:
  	illegalParamterValue():
  		message("Illegalparameter value"){}
  	illegalParamterValue(char *theMessage){
      message = theMessage;
    }
  	void outputMessage() {
      cout << message << endl;
    }
  private:
  	string message;
}
```

```cpp
throw illegalParameterValue('test error')
```

```cpp
try {

} catch(illegalParameterValue e) {

}
```

## 递归函数

递归函数或者方法自己调用自己，在直接递归函数中，递归函数 f 的代码包含了调用 f 的语句，而在间接递归中，递归函数 f 懂了 g，而 g 又调用了函数 h，最后调用了 f。

```cpp
int factorial(int n) {
  if (n <= 1) return 1;
  else return n * factorial(n - 1);
}
```

## 标准模板库

c++ 标准模板库(STL) 是一个容器 适配器 迭代器 函数对象和算法的集合。

### accumulate

`accumulate` 是对顺序表元素累积求和

```cpp
accumulate(start, end, initialValue)
```

```cpp
template<class T>
T sum(T a[], int n){
  T theSum = 0;
  return accumulate(am a+n, theSum);
}
```

accumulate 利用++操作符，从 start 开始， 到 end 结束，相继访问要累积求和的顺序表元素。

accumulate 更通用的方法是如下方法

```cpp
accumulate(start, end, initValue, operator);
```

Operator 是一个函数，它规定了累积过程中的操作

### copy and next_permutation

算法 copy 吧一个顺序列表的元素从一个位置复制到另一个位置 `copy(start, end, to)`

next_permutation(start, end) 对于范围 start 到 end，内的元素，按字典顺序，产生下一个更大的排列。当且仅当这个排列存在是，返回为 true，
