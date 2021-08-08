#C++静态成员访问非静态成员的方法（小结）
####**补充：**
1）**静态函数成员与非静态函数成员都为类所有**，<font color=red>对象中并不存在函数的拷贝</font> <font color=#67854>（每个对象所占用的存储空间只是该对象的数据成员所占用的存储空间，但是在逻辑上函数和数据是一起被封装进对象的）</font>。**静态成员函数和非静态成员函数的根本区别在于有无this指针**。非静态函数由`对象名.`或者`对象指针->`调用，调用时编译器会向函数传递this指针；静态成员函数则由`类名::`或者`对象名.`调用，没有this指针，不识别对象个体，经常用来操作类的静态数据成员。访问类的非静态成员要通过对象来实现。

2）对于类中的静态成员函数和静态成员变量为类本身所有，**在类加载的时候就会分配内存**，类似于普通的全局函数以及全局变量；与类中的非静态函数以及非静态变量在创建类的实例后才分配内存不同。

3）类的静态数据成员属于类，在类的所有对象间共享(共享区)；非静态数据成员在类的每个实例中都有一份拷贝(动态区)。<font color = #pinkred>在类的声明中，静态成员变量仅完成了声明过程，并没有定义以及赋初值；由于静态成员变量在编译时存储在静态存储区，即定义过程应该在编译时完成</font>，<font color=red>**一定要在类外定义，前面不加static以免与一般静态变量或者对象混淆**</font>，但是可以不进行初始化。

4）静态成员函数访问非静态成员报错：
由于类的静态成员在类加载的时候就已经分配内存，而此时类的非静态成员尚未分配内存，访问内存中不存在的东西自然会出错。

举例：
```
#include<iostream>
class A {
 public:
  // 静态成员函数
  static void test();
 private:
  // 静态成员变量（在类中仅仅声明）
  static int m_static_value1;
  static int m_static_value2;
  int m_a;
};
int A::m_static_value1 = 1;  // 在类外定义静态成员变量并初始化
int A::m_static_value2;  // 在类外定义静态成员变量且不初始化
void A::test() {
  // int static_m1 = m_a;  // 在静态函数中使用非静态成员变量错误
  int static_m2 = m_static_value1;  // 在静态函数中使用静态成员变量
  std::cout << "the value of static1: " << static_m2 << std::endl;
}
int main() {
  A::test();
  return 0;
}

```
输出结果：
```
the value of static1: 1
```
####1.成员函数在回调函数中使用
#####1.1问题：
由于c++类的成员函数默认包含this指针，所以当我们直接将一个非静态成员函数注册给函数指针时会报错(因为此时类的实例还没有创建，也就不存在this指针)：
```
class A {
 public:
  //静态成员函数
  static void test() {
    m_staticA += 1;
  }
  void Hello() {
    std::cout << "say hello to caller" << std::endl;
  }
 private:
  static int m_staticA;
  int m_a;
};
// 定义一个指向类成员函数指针的别名(返回类型为void类型的函数指针)
typedef void(A::*Funptr)();
Funptr p = A::Hello;
int main() {
  p();  // 使用错误，因为非静态成员函数需要由类的实例去使用
  return 0;
}
```
#####1.2解决：
如果想要使用p，必须有一个分配好空间的实例才能来调用：
```
#include <iostream>

class A {
 public:
  static void test() {
    m_staticA += 1;
  }
  void Hello() {
    std::cout << "say hello to caller" << std::endl;
  }
 private:
  static int m_staticA;
  int m_a;
};
// 定义一个指向类成员函数指针的别名
typedef void(A::*Funptr)();
// 此处引用很重要，是用于绑定类实例成员函数的地址
Funptr p = &A::Hello;

int main() {
  A a;
  A *PA = new A();

  (a.*p)();
  (PA->*p)();
  return 0;
}

```
输出结果：
```
say hello to caller
say hello to caller
```
#####1.3 C++的类静态函数进行回调函数注册
此时不需要考虑this指针，直接当做普通函数绑定
```
class A {
 public:
  static void test() {
    m_staticA += 1;
    std::cout << "静态成员函数调用：" << m_staticA << std::endl;
  }
  void Hello() {
    std::cout << "say hello to caller" << std::endl;
  }
 private:
  static int m_staticA;
  int m_a = 1;
};
// 在类外定义
int A::m_staticA = 0;

// 对于类的静态成员函数直接当做普通函数指针绑定
typedef void(*FuncPtr)();
FuncPtr p = &A::test;
int main() {
  p();
  return 0;
}
```
输出结果：
```
静态成员函数调用：1
```
但问题是，此时静态函数不能拥有类的普通成员变量。
####2.静态函数访问非静态成员变量：
#####2.1静态成员函数的形参表里加上实例的地址
```
class A {
 public:
  static void test(A* a) {
    a->m_a += 1;
    std::cout << "静态成员函数调用非静态成员变量：" << (a->m_a) << std::endl;
  }
  void Hello() {
    std::cout << "say hello to caller" << std::endl;
  }
 private:
  static int m_staticA;
  int m_a = 1;
};
int A::m_staticA = 0;  // 静态成员变量必须在类外定义(可以不初始化)

// 对于类的静态成员函数直接当做普通函数指针绑定
typedef void(*FuncPtr)(A*);
FuncPtr p = &A::test;
int main() {
  A a = A();
  p(&a);
  return 0;
}

```
输出结果：
```
静态成员函数调用非静态成员变量：2
```
在回调函数的时候，可以通过这种方式来让本身不能访问成员非静态变量的静态函数来访问非静态成员变量。
#####2.2使用一个全局变量
```
#include <iostream>

// 先声明一个全局变量，接下来才能在类的内部使用
class A;
extern A g_a;

class A {
 public:
  static void test() {
    g_a.m_a += 1;
    std::cout << "静态成员函数调用非静态成员变量：" << (g_a.m_a) << std::endl;
  }
  void Hello() {
    std::cout << "say hello to caller" << std::endl;
  }
 private:
  static int m_staticA;
  int m_a = 1;
};

// 定义那个全局变量
A g_a = A();

// 在类外定义静态成员函数
int A::m_staticA = 0;
typedef void(* FuncPtr)();
FuncPtr p = &A::test;
int main() {
  p();
  return 0;
}

```
对于全局变量的做法并不推荐
#####2.3使用单例模式：
静态成员函数不能访问非静态成员，但是可以访问静态成员。所以，当类是单例的时候，我们可以在创建类的实例时把this指针赋值给静态成员，之后我们就可以在静态成员中使用非静态成员。
```
#include <iostream>
using namespace std;
class A {
 public:
 // 单例模式的get方法获得类的静态指针
  static A* get_instance() {
    if (m_ptr == nullptr)
      m_ptr = new A;
    std::cout << m_ptr->m_static_A << std::endl;
    return m_ptr;
  }
  int m_a = 1;
 private:
  // 私有构造函数
  A() {
    m_ptr = this;
  }
  static int m_static_A;
  // 单例模式有一个私有的静态指针
  static A* m_ptr;
};
// 初始化静态变量
int A::m_static_A = 6;
A* A::m_ptr = nullptr;

int main() {
  A* mptr =  A::get_instance();
  // 利用单例获得类的静态成员指针，调用非静态成员变量
  cout << mptr->m_a;
  return 0;
}
```
输出结果：
```
6
1
```
#####2.4行参列表里传入实参地址（一个`void*`指针）
```
#include <iostream>
class A {
 public:
  static void test(void* pData) {
    A* a = static_cast<A*>(pData);
    a->m_a += 1;
    std::cout << a->m_a << std::endl;
  }
 private:
  static int m_static_A;
  int m_a = 0;
};
int A::m_static_A = 0;
int main() {
  A a;
  // A a = A();  // 等价方式
  A::test(&a);
  return 0;
}
```
