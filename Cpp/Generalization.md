#C++泛化
##一、C++11可变参数模板的使用：
###1.简介：
1)普通模板只可以采取固定数量的模板参数。然而，有时候我们希望模板可以接收任意数量的模板参数，这个时候可以采用可变参数模板。

2)可变参数模板对参数进行了高度的泛化，是一个接受可变数目参数的**模板函数或模板类**，在模板参数列表中，<font color=#2456>`typename…`指出接下来的参数表示0个或多个类型的列表(也就是接下来的参数其实是一个参数包，一个类型名后面跟省略号表示0个或多个给定类型的**非类型参数**的列表)</font>

3)对于可变参数模板，其将包含至少一个**模板参数包**，模板参数包是可以接收0个或者多个参数的模板参数。相应地，存在**函数参数包**，意味着这个函数参数可以接收任意数量的参数。<font color=red>在c++11中对于参数包用`unpack`和**类似函数重载的模板特化**来抽取参数。注意：在模板参数后面带省略号`...`就表示对其进行解包(unpack),会把这个参数包所表示的参数列表解开后去匹配新的模板,或是进行模板展开。</font>
###2.使用规则：
####2.1可变模板类参数定义如下：
```
// Types就表示一个模板参数包
template<typename ... Types>class Tuple {
  ......
};
```
可以用任意数量的类型来实例化Tuple：
```
Tuple<> t0;  // 这里用0个模板参数实例化了可变参数模板
Tuple<int> t1;
Tuple<int, string> t2;
// Tuple<0> error;  0 is not a type
```
如果想避免出现用0个模板参数来实例化可变参数模板，可以这样定义模板：
```
template<typename T, typename... Types>class Tuple {
  ......
};
```
此时在实例化时需要至少传入一个模板参数，否则无法通过编译。

####2.2可变参数函数模板定义：
```
template<typename... Types>void f(Types... args) {
  ......
}

// 一些合法的调用
f();
f(1);
f(3.4, "hello");
```
####2.3注意事项：
对于**类模板**来说，可变模板参数包必须是模板参数列表中的最后一个参数。但是对于**函数模板**来说，则没有这个限制，考虑下面的情况:
```
template<typename ... Ts, typename U>class Invalid {
  ......
};   // 这是非法的定义，因为永远无法推断出U的类型

template<typename ... Ts, typename U>void valid(U u, Ts ... args) {  // 这是合法的，因为可以推断出U的类型(虽然在模板参数列表中U类型在后面，但是在声明函数模板的时候将u类型放在了前面，而上面的类模板就无法如此)
  ......
}
// void invalid(Ts ... args, U u); // 注意，这种使用非法的，永远无法推断出U的类型

valid(1.0, 1, 2, 3); // 此时，U的类型是double，Ts是{int, int, int}
```

####2.4示例：
```
#include <iostream>
#include <string>
// 用于终止的基函数
void tprintf(const char* format) {
  std::cout << format << std::endl;
}
template<typename T, typename ...Ts> void tprintf(const char* format, T&& value, Ts&& ...args) {
  for(; *format != '\0'; ++format) {
    if((*format) == '%'){
      std::cout << value;
      tprintf(format+1, std::forward<Ts>(args) ...);  // 解包并递归，默认传入的args包的第一个参数为value
      return;
    }
    std::cout << *format; // 依次输出char字符
  }
  return;
}
int main() {
  tprintf("% world% %\n", "Hello", '!', 2019);
  std::cin.ignore(10);
  return 0;
}
```
###3.可变参数模板函数展开参数包：
####3.1以递归方式展开：
虽然我们无法直接遍历传给可变参数模板的不同参数，<font color=#4589>但是可以借助递归的方式来使用可变参数模板。</font>**可变参数模板允许创建类型安全的可变长度参数列表**。下面定义一个可变参数函数模板`processValues()`，它允许以类型安全的方式接受不同类型的可变数目的参数。函数`processValues()`会处理可变参数列表中的每个值，对每个参数执行对应版本的`handleValue()`
```
#include <iostream>
#include <string>
// 处理每个类型的实际函数
void handleValue(int value) {
  std::cout << "integer" << std::endl;
}
void handleValue(double value) {
  std::cout << "double" << std::endl;
}
void handleValue(std::string value) {
  std::cout << "string" << std::endl;
}
// 用于终止迭代的基函数
template<typename T>void processValues(T arg) {
  handleValue(arg);
}
// 用于递归的处理函数，Ts是一个用于递归处理的模板参数包
template<typename T, typename ...Ts> void processValues(T arg, Ts ...args) {
  handleValue(arg);
  processValues(args ...);  // 解包，然后递归(args其实就是参数包)
}
int main() {
  processValues(1, 2.5, "test");
  return 0;
}
```

**可以看到这个例子用了三次`…`运算符，但是有两层不同的含义:**
1)用在参数模板列表以及函数参数列表，其表示的是参数包。前面说到，参数包可以接受任意数量的参数。
2)用在函数实际调用中的`…`运算符，它表示参数包扩展，此时会对args解包，展开各个参数，并用逗号分隔。模板总是至少需要一个参数，通过对`args…`解包可以递归调用`processValues()`，这样每次调用都会至少用到一个模板参数。对于递归来说，需要终止条件，当解包后的参数只有一个时，调用接收一个参数模板的`processValues()`函数，从而终止整个递归。

假如对processValues()进行如下调用:
```
processsValues(1, 2.5, "test");
```
其产生的递归调用如下：
```
processsValues(1, 2.5, "test");
    handleValue(1);
    processsValues(2.5, "test");
        handleValue(2.5);
        processsValues("test");
            handleValue("test");
```

但是前面的实现有一个致命的缺陷：那就是递归调用时参数是复制传值的，对于有些类型参数，其代价可能会很高。一个高效且合理的方式是按引用传值，但是对于字面量调用processValues()这样会存在问题，因为字面量仅允许传给const引用参数。比较幸运的是，我们可以考虑右值引用：

`std::forward()`函数可以实现这样的处理。当把右值引用传递给`processValues()`函数时，它就传递为右值引用，但是如果把左值引用传递给`processValues()`函数时，它就传递为左值引用。(其实就是引用的折叠)：
```
#include <iostream>
#include <string>
// 处理每个类型的实际函数
void handleValue(int value) {
  std::cout << "integer" << std::endl;
}
void handleValue(double value) {
  std::cout << "double" << std::endl;
}
void handleValue(std::string value) {
  std::cout << "string" << std::endl;
}
// 用于终止迭代的基函数
template<typename T>void processValues(T&& arg) {
  handleValue(std::forward<T>(arg));
}
// 用于递归的处理函数，Ts是一个用于递归处理的模板参数包
template<typename T, typename ...Ts> void processValues(T&& arg, Ts&& ...args) {
  handleValue(std::forward<T>(arg));
  processValues(std::forward<Ts>(args) ...);   // 先使用forward函数处理后，再解包，然后递归
}
int main() {
  processValues(1.0, 1, "test");
  return 0;
}
```
####3.2逗号表达式展开：
递归函数展开参数包是一种标准做法，也比较好理解，但也有一个缺点,就是**必须要有一个重载的递归终止函数**，即必须要有一个同名的终止函数来终止递归，这样可能会感觉稍有不便。有没有一种更简单的方式呢？其实还有一种方法可以不通过递归方式来展开参数包，这种方式需要借助逗号表达式和初始化列表。
```
#include <iostream>
#include <string>
template<typename T>void print(T t) {
  std::cout << t << std::endl;
}
template<typename ...Ts>void Expand(Ts... args) {
  std::cout << "参数的size： " << sizeof...(args) << std::endl;  // 可以输出参数包的size
  int arr1[] = {(args)...};  // 在数组的构造过程中展开参数包并对其初始化
  // int arr2[] = {(print(args),args)...};  // 在数组的构造过程中展开参数包args

  std::cout << "输出数组中被初始化的元素：" << std::endl;
  for (int i = 0; i < sizeof...(args); i++)
    std::cout << arr1[i] << " ";
  std::cout << std::endl;
}
// 支持lambda表达式
template<typename T, typename... Args>void expand(const T& func, Args&& ...args) {  // 传递给Expand的第一个参数是一个lambda表达式（这里使用了完美转发）
  std::initializer_list<int> list{(func(std::forward<Args>(args)),args)...};  // initializer_list构造的过程中展开参数包args，并将其每个元素初始化为展开的那个参数
  std::cout << "输出list中初始化的所有元素：" << std::endl;
  for (auto val : list)
    std::cout << val << " ";
  std::cout << std::endl;
}
int main() {
  Expand(1,2,3,4);
  /*用到数组的初始化列表，这个数组的目的纯粹是为了在数组构造的过程展开参数包。
    {(print(args), 0)...}将会展开成((print(arg1),0), (print(arg2),0), (print(arg3),0),  etc... )，
    最终会创建一个元素值都为0的数组int arr[sizeof...(Args)]。print便会处理参数包中每一个参数。
   */
  expand([](int i){std::cout << "lambda表达式传入的参数： " << i << std::endl;}, 5, 6, 7, 8);
  return 0;
}
```
输出结果：
```
参数的size： 4
输出数组中被初始化的元素：
1 2 3 4
lambda表达式传入的参数： 5
lambda表达式传入的参数： 6
lambda表达式传入的参数： 7
lambda表达式传入的参数： 8
输出list中初始化的所有元素：
5 6 7 8
```
###4.可变模版参数类展开参数包：
可变参数模板类的参数包展开需要通过模板特化和继承方式去展开。


###5.参数模板的特例化：
模板的特例化包括函数模板的特例化和类模板的特例化，其中又分为<font color=#456987>全特化与偏特化</font>。主要的用途都是**对于特定的类型，指定特定的处理方式**。编译阶段确定如果是某个特化类型，就用特化的模板;如果都不是，就用最一般的模板。
#####5.1函数模板的特例化：
函数模板只能全特化，不能偏特化,如果要偏特化的话只能重载。
**函数模板全特化：**
```
template<typename T>T add(T a, T b) {
    return a + b;
}
// 注意，在定义全特化的函数模板之前必须要先定义一个普通的函数模板
template<>double add(double a, double b) {
  return a + b;
}

int main() {
  int a = 10;
  int b = 15;
  double x = 1.1, y = 2.2;
  std::cout << "调用普通的函数模板" << add(a, b) << std::endl;
  std::cout << "调用全特化的函数模板" << add(x, y) << std::endl;
  return 0;
}
```
**函数模板偏重载(不存在偏特化)：**
```
template<typename T>T add(T *a, T *b) {
  return *a + *b;
}
int main() {
  int a = 10;
  int b = 15;
  double x = 1.1, y = 2.2;
  int c = 11, d = 12;
  std::cout << "调用普通的函数模板" << add(a, b) << std::endl;
  std::cout << "调用全特化的函数模板" << add(x, y) << std::endl;
  std::cout << "使用重载的函数模板" << add(&c, &d) << std::endl;
  return 0;
}
```
如上，函数模板不存在偏特话，所以对于一个针对指针的特殊版本就可以进行偏重载。
#####5.2类模板的特例化：
与函数模板所不同，类模板既有全特化也有偏特化。
**类模板全特化：**
和函数模板一样，类模板的全特化是一个实例，当编译器匹配时会优先匹配参数一致的实例。
```
template<typename T>class A {
 private:
  T c;
};
// 和函数模板的全特化相似，这里也需要先声明一个与全特化模板类名字相同的普通模板类
template<>class A<char *>{  // 这里是一个全特化的模板类A，当用char*类型来实例化类模板A时，将会优先调用这个全特化实例
 private:
  char *t;
 public:
  explicit A(char *val) : t(val) {};  // 用val的地址初始化t的地址
  char* add(char* a, char* b) {
    return std::strcat(a, b);
  }
};
```
**类模板偏特化：**
类模板的偏特化会稍微复杂一点点，它有多种形式。类模板偏特化本质上是指定部分类型，让偏特化版本成为普通版本的子集，<font color=#5698>若实例化时参数类型为指定的类型，则优先调用特例化版本。</font>
1)第一种形式：
```
template<typename T1, typename T2>class B {  // 普通类模板，有两个类模板参数
 public:
  B(T1, T2) {
    std::cout << "实例化的时候调用普通的模板函数" << std::endl;
  }
};
// 偏特化版本，指定其中一个参数，即指定了部分类型
template<typename T2>class B<int, T2> {  // 当实例化的时候第一个参数为int则优先调用这个版本
 public:
  B(int a, T2 b) {
    std::cout << "实例化的时候调用偏特化的模板参数" << std::endl;
  }
};
int main() {
  B<int, std::string> b(4, "hello!");
  return 0;
}
```
2)第二种形式，比较重要：
```
template<typename T>class C {  // 普通的模板类
 public:
  C(T a) {
    std::cout << "实例化的时候调用普通的模板函数" << std::endl;
  }
};
template<typename T>class C<T*> {
 public:
  C(T* a) {
    std::cout << "实例化的时候调用偏特化的模板参数,类型为T*" << std::endl;
  }
};
template<typename T>class C<T&> {
 public:
  C(T& a) {
    std::cout << "实例化的时候调用偏特化的模板参数,类型为T&" << std::endl;
  }
};
int main() {
  int a = 4;
  int &b = a;
  C<int *> c1(&a);
  C<int &> c2(b);
  return 0;
}
```
3)第三种形式：
```
template<typename T>class D {
 public:
  D(T a) {
    std::cout << "实例化的时候调用普通的模板函数" << std::endl;
  }
};
template<typename T>class D<std::vector<T> > {  // 这种只接受用T实例化的vector作为模板实参．也是一种偏特化
 public:
  D(std::vector<T> vec) {
    std::cout << "实例化的时候调用偏特化的模板参数,类型为std::vector<T>" << std::endl;
  }
};
int main() {
  std::vector<int> vec;
  D<std::vector<int> > d1(vec);
  return 0;
}
```
**补充：**
1.特例化本质上是我们顶替了编译器的工作，我们帮编译器做了类型推导
2.全特化本质上是一个实例，而偏特化本质上还是一个模板，只是原来模板的一个子集
3.所以全特化的函数模板，本质上是实例，从而不会与函数模板产生二义性
4.若想让用户能使用特例化版本，特例化版本必须与模板定义在同一个.h头文件中
5.STL中的迭代器的高效实现与模板偏特化息息相关．
####2.6
