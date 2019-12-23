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

# 第二章

程序性能分析， 使用程序性能（program performance）来指一个程序对内存和时间的需求，要求对数据结构和算法设计方法给予应有的评价。

## 什么是程序性能

程序性能是指运行这个程序所需要的内存和时间的多少，一般有两种方法来确定一个程序的性能，分析方法 和 实验方法，在`性能分析时` ，采用分析方法，而在`性能的测量`时使用实验方法。

所谓的一个程序的空间复杂度是指该程序的运行所需内存的大小，对程序的空间复杂度感兴趣的主要有原因如下：

1. 如果一个程序需要在多用户计算系统中运行，那么需要指明该成谦虚所需的内存大小
2. 在任何计算机上运行，都需要知道是否有足够的内存来运行该程序
3. 一个问题可能有若干解决方案，它们对内存的需求各不相同
4. 利用空间复杂度，可以估计一个程序所能解决的问题最大可以是什么规模。

所谓`时间复杂度` 是指程序所需要的时间，对程序的时间复杂度感兴趣的主要原因如下：

1. 有些计算机需要用户提供运行时间的上限，一旦达到这个上限，程序就会被强制结束掉。
2. 正在开发的程序可能需要一个令人满意的实时响应。
3. 如果一个问题有多重解决方案，具体采用哪种方案，蛀牙根据这些方案的性能差异，对于各种方案的时间和空间性能，采取加权测量方式进行评价

## 空间复杂度

### 空间复杂的的组成

程序所需要的空间主要由以下部分组成：

1. 指令空间

   指令空间是指编译后程序指令所需要的储存空间

2. 数据空间

   数据空间是指所有常量和变量所需要的储存空间，它由两个部分构成：常量和简单变量所需要的储存空间，动态数据和动态类实例对象所需要的空间

3. 环境栈空间

   环境栈用来保存暂停的函数和方法在恢复运行时所需要的信息。例如函数 foo 调用了函数 goo，那么至少要保存在函数 goo 结束时函数 foo 继续执行的指令地址

1) 指令空间

   指令空间的数量取决于如下因素：

   把程序转换成机器代码的编译器

   在程序编译时的编译器选项

   目标计算机

   所以在决定最终代码需要多少空间时，编译器是一个重要的因素，每个编译器产生的结果都不一样。再例如 Debug 模式和 release 所产生的可执行程序代码的体积是不一样的。

2) 数据空间

   对于各种数据类型 c++ 并没有指定特闷的空间大小，大多数的 c++ 编译器有响应的空间分配。所系需要关心结构变量和成员所需要的空间大小之和

3) 环境栈空间

   在开始分析性能之时，人们通常会忽略环境栈所需要的空间，因为人们捕猎机函数（特别是递归函数）是如何被调用的尔以及在函数调用结束后会发生什么。每当一个函数被调用时，下面的数据将被保存在环境栈：

   返回地址

   在程序调用的函数的所有局部变量的值以及形式参数的值（仅对递归函数而言）

可以吧一个程序所需要的空间分为两个部分：

1. 固定部分 它独立于实例特性，这一部分通常包括指令空间（即代码空间）， 简单变量空间和常量空间等
2. 可变部分 它由动态分配空间构成和递归栈空间构成，前者在某种程度上依赖实例特征，而后者主要依赖实例特征。

对于任意程序 P 所需要的空间可以表示为：

> c + Sp（实例特征）

c 是一个常量，标识空间需求的固定部分， Sp 表示空间需求的可变部分，要精确地分析空间性能，还要考虑在编译器所产生的临时变量所需的空间，与编译器有关，除了递归函数之外，它不依赖于实例特征，之后将会忽略编译器昌盛的变量空间。

对于分析一个程序的空间复杂度时，将集中计算 Sp，对于给定的任意问题，都先要明确哪些实例特征可以用来估算空间需求。

## 时间复杂度

### 时间复杂度的组成

影响空间复杂度的因素也会影响时间复杂度。例如一些编译器比另一些编译器生成的代码速度快。较小的问题实例通常比较大的问题实例用时要少。

用分析法分析确定一个程序的运行时间是很复杂的，因此只能估算时间，而且有两个比较容易控制的方法：

1. 找出一个或多个关键操作，确定他们的执行时间
2. 确定程序总的步数

### 操作计数

估计一个程序或函数的时间复杂度，一种方法是选择一种或者多种关键操作，例如 加 乘 比较，然后确定每一种操作的执行次数，使用这种方法成功与否取决于能否找到耗时最大的操作。

# 第三章

## 大 O 记法

> 令 p(n) 和 q(n) 是连个非负数，称 p(n)渐进大于 q(n) （p(n)渐进地优于 q(n)，当且仅当称 q(n)渐进地小于 p(n)，当且仅当 p(n)渐进地大于 q(n)，称 p(n)渐进地等于 q(n), 当且仅当任何一个都不是渐进地大于另一个。

下图是经常在步数分析中出现的项:

| 项    | 名称     |
| ----- | -------- |
| 1     | 常量     |
| logn  | 对数     |
| n     | 线性     |
| nlogn | n 倍对数 |
| n²    | 平方     |
| n³    | 立方     |
| 2^n   | 指数     |
| n!    | 阶乘     |

渐进记法，描述的是大实例特征的时间或空间复杂度，将用他来分数步数，时间复杂度和步数是同义词，如果实力只含有一个变量，例如 n，渐进计数法就用步数中渐进最大的一项来描述复杂度。
