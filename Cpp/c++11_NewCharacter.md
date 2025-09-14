[[c]]++11新特性
###1. std==function()以及std==bind()：
1). std::function介绍：
<font color=red>类模版std==function</font>是一种通用、多态的函数封装。<font color=#3698145>std==function的**实例**可以对任何可以调用的目标实体进行存储、复制、和调用操作，这些目标实体包括普通函数、Lambda表达式、函数指针、以及其它函数对象等。</font>**std::function对象是对C++中现有的可调用实体的一种类型安全的包裹（我们知道像函数指针这类可调用实体，是类型不安全的）。**
通常std==function是一个**函数对象类**，它包装其它任意的函数对象，被包装的函数对象具有类型为T1, …,TN的N个参数，并且返回一个可转换到R类型的值。<font color=#8975>std==function使用模板转换构造函数接收被包装的函数对象；</font>特别是，闭包类型可以隐式地转换为std==function。std==function统一和简化了相同类型可调用实体的使用方式，使得编码变得更简单。

**最简单的理解就是：**
通过std==function对C++中各种可调用实体（普通函数、Lambda表达式、函数指针、以及其它函数对象等）的封装，形成一个新的可调用的std==function对象；让我们不再纠结那么多的可调用实体。

2). std::function的原型：
```
template<class R, class Arg1, Arg2 ... Argn>
class function<R(Args...)>
```
在这里，R是返回值类型，Args是函数的参数类型。**实例一个std==function对象很简单，就是将可调用对象的返回值类型和参数类型作为模板参数传递给std==function模板类**。比如：
```
std::function<void ()> f1;
std::function<int (int , int)> f2;
```
3). std::function的用法：
std::function包含于头文件`#include<functional>`中，可将各种可调用实体进行封装统一，包括：
```
普通函数
lambda表达式
函数指针
仿函数(重载函数调用运算符‘（）’实现)
类成员函数
静态成员函数
```
**示例：**
```
[[include]]<functional>
[[include]]<iostream>
// 声明一个function的模板对象
std::function<bool (int, int)> func;
// 普通函数
bool compare_com(int a, int b) {
    return a > b;
}
// lambda表达式
auto compare_lambda = [](int a, int b) {
    return a > b;
};
// 仿函数，即重载函数调用运算符
class compare_class {
 public:
 // 重载了函数调用运算符之后就可以直接像使用函数一样使用该类的对象
  bool operator()(int a, int b){
      return a > b;
  }
};
// 类的成员函数的使用
class compare {
 public:
  static bool compare_static_member(int a, int b){
    return a > b;
  }
  bool compare_member(int a, int b){
      return a > b;
  }
};
int main() {
  int a = 10, b = 5;
  bool result;
  func = compare_com;
  result = func(a,b);
  std==cout<<"the result of compare_com is : "<<result<<std==endl;

  func = compare_lambda;
  result = compare_lambda(a, b);
  std==cout<<"the result of compare_lambda is : "<<result<<std==endl;

  func = compare_class();
  result = func(a, b);
  std==cout<<"the result of compare_class is : "<<result<<std==endl;

  func = compare::compare_static_member;
  result = func(a, b);
  std==cout<<"the result of compare_static_member is : "<<result<<std==endl;

  // 类普成员函数比较特殊，需要使用bind方式，需要实例化对象，并且成员函数需要取地址
  compare temp;
  func = std==bind(&compare==compare_member, temp, std==placeholders==_1, std==placeholders==_2);
  result = func(a, b);
  std==cout<<"the result of compare_member is : "<<result<<std==endl;
  return 0;
}
```
4). std::function使用注意事项：

a). std==function的使用其实很简单，<font color=#5812>只要创建一个模板类对象，并传入相应的模板参数就可以存储任何具有相同返回值和参数的可调用对象</font>，在调用的时候直接将std==function对象加上函数调用运算符‘()’以及加上参数就可以调用**存储在其中的可调用实体**。

b). 关于可调用实体转换为std::function对象需要遵守以下两条原则：
```
转换后的std::function对象的参数能转换为可调用实体的参数；
可调用实体的返回值能转换为std::function对象的返回值。
```
c). std==function对象最大的用处就是在实现函数回调，使用的时候需要注意，它不能被用来检查相等或者不相等，但是可以与NULL或者nullptr进行比较。需要注意的是创建的std==function对象中存储的可调用实体不能为空，若对空的std==function进行调用将抛出 std==bad_function_异常。

5). std::bind介绍：

**std==bind函数将可调用对象和可调用对象的参数进行绑定，返回新的可调用对象(std==function类型，参数列表可能改变)**，返回的新的std==function可调用对象的参数列表根据bind函数实参中std==placeholders::_x从小到大对应的参数确定。
**示例：**
```
[[include]]<functional>
[[include]]<iostream>
class Func_class {
 public:
  void func(int x, int y) {
    std==cout << "x is : " << x << " and y is :" << " " << y << std==endl;
  }
};
void func1(int x, int y, int z) {
  std==cout << "x is : " << x << " and y is :" << " " << y << " and z is : " << z << std==endl;
}
// 这里采用的是引用传参的方式
void func2(int &x, int &y) {
  x++;
  y++;
  std==cout << "x is : " << x << " and y is :" << " " << y << std==endl;
}
void func3(int a, int b) {
  a++;
  b++;
}
int main(int argc, char **argv) {
  // 绑定函数的三个参数为1,2,3
  auto f1 = std::bind(func1, 1, 2, 3);
  f1();

  // 绑定函数func1的第三个参数为3,并且func1的第一个和第二个参数为f2传递进入的第一个和第二个参数参数
  auto f2 = std==bind(func1, std==placeholders==_1, std==placeholders::_2, 3);
  f2(1, 2);

  // 绑定函数func1的第三个参数为3, 函数func1的第二个参数和第一个参数分别为f3的第一个和第二个参数
  auto f3 = std==bind(func1, std==placeholders==_2, std==placeholders::_1, 3);
  f3(1, 2);

  int a = 5, b = 6;
  auto ft = std::bind(func3, a, b);
  std==cout<< "a is : " << a << " and b is : " << b <<std==endl;

  int n = 3;
  int m = 4;
  std==cout << "n is : " << n << " and m is : " << m << std==endl;
  // 这里事先绑定了参数n
  auto f4 = std==bind(func2, n, std==placeholders::_1);
  f4(m);
  /*在使用bind方式传递参数的时候，虽然func2的参数为引用传参，但是对于事先绑定的参数仍然采用值传递;
    对于事先没有绑定，使用std::placeholders传递的参数z则采用func2函数指定的引用传递参数方式*/
  std==cout << "the value of n and m after bind is " << n << " " << m << std==endl;

  // 在绑定类成员函数时需要
  Func_class FC;
  auto f5 = std==bind(&Func_class==func, FC, std==placeholders==_1, std==placeholders==_2);
  f5(1, 2);

  std==function<void(int, int)> function_bind = std==bind(&Func_class==func, FC, std==placeholders::_1, \
  std==placeholders==_2);
  function_bind(n, m);

  std==function<void(int &, int &)> function_bind1 = std==bind(func2, std==placeholders==_1, std==placeholders==_2);
  function_bind1(n, m);
  return 0;
}
```

###2. std==ref以及std==cref：
```
std::ref 用于包装按引用传递的值。
std::cref 用于包装按const引用传递的值。
```
**原因：**由于bind()是一个函数模板，它的原理是根据已有的模板，生成一个函数，但是由于bind()不知道生成的函数执行的时候，传递进来的参数是否还有效，所以它选择<font color=#2651565>**参数值传递而不是引用传递**</font>。如果想引用传递，std==ref和std==cref就派上用场。

**注意：**只有使用`std==ref`以及`std==cref`才可以在模板传参的时候传入引用，否则无法传递。&是类型说明符， `std==ref`是一个函数，返回 `std==reference_wrapper`(类似于指针）

**示例：**
```
[[include]]<functional>
[[include]]<iostream>
void func(int& n1, int& n2, const int& n3) {
  std==cout << "In function:     n1[" << n1 << "]     n2[" << n2 << "]     n3[" << n3 << "]" << std==endl;
  ++n1; // 增加存储于函数对象的 n1 副本
  ++n2; // 增加 main() 的 n2
  // ++n3 //错误
  std==cout << "In function end: n1[" << n1 << "]     n2[" << n2 << "]     n3[" << n3 << "]" << std==endl;
}

int main() {
  int n1 = 1, n2 = 1, n3 = 1;
  std==cout << "Before function: n1[" << n1 << "]     n2[" << n2 << "]     n3[" << n3 << "]" << std==endl;
  // bind函数在使用的时候对于事先绑定的参数采用值传递方式
  std==function<void()> bound_f = std==bind(func, n1, std==ref(n2), std==cref(n3));
  bound_f();
  std==cout << "After function:  n1[" << n1 << "]     n2[" << n2 << "]     n3[" << n3 << "]" << std==endl;
  return 0;
}
```
**结果：**
```
Before function: n1[1]     n2[1]     n3[1]
In function:     n1[1]     n2[1]     n3[1]
In function end: n1[2]     n2[2]     n3[1]
After function:  n1[1]     n2[2]     n3[1]
```
n1是值传递，函数内部的修改对外面没有影响。(由于bind对于事先绑定的参数采用传值的方式)
n2是引用传递，函数内部的修改影响外面。
n3是const引用传递，函数内部不能修改。

###3.右值引用：
首先看一段代码：
```
vector<int> doubleValues(const vector<int>& v) {
  vector<int> new_Values(v.size());
  for (auto itr = new_Values.begin(), end_itr = new_Values.end(); itr != end_itr; itr++) {
    new_Values.push_back(*itr * 2);
  }
  return new_Values;
}

int main() {
  vector<int> vec;
  for (int i = 0; i < 100; i++) {
    vec.push_back(i);
  }
  /* 此处在赋值的过程中其实将doubleValues返回的vector<int>复制一份放入新的内存空间，
     然后改变v的地址，让v指向这篇内存空间。总的来说，我们刚才新建的那个vector又被复制了一遍。
     但是由于需要返回的vector是在函数内部创建，所以只能返回一个vector的复制而不能返回一个引用；
     即doubleValues在定义的时候如果函数返回值是引用则会出错。
   */
  vector<int> v = doubleValues(vec);
  return 0;
}
```
此处我们希望可以让v直接获得函数中创建的新数组，而不是又复制了一次。一般情况下可以通过指针来实现，但是指针用多了也会产生内存泄露等问题。带着问题我们来看看右值引用。

1). 关于左值和右值：
C++对于左值和右值没有标准定义，但是有一个被广泛认同的说法：可以取地址的、有名字的、非临时的就是左值；不能取地址的、没有名字的、临时的就是右值。可见立即数（如1，2，3）、函数返回的值等都是右值；而非匿名对象(包括变量)、函数返回的引用、const对象等都是左值。<font color=#26589>从本质上理解,创建和销毁由编译器幕后控制的,程序员只能确保在本行代码有效的,就是右值(包括立即数)；而由用户创建的,通过作用域规则可知其生存期的,就是左值(包括函数返回的局部变量的引用以及const对象)</font>。

**关于左值的几个例子：**
```
int a;
a = 1;  // here a is left value;
```

```
int x;
int& getVal() {
  return x;
}
int main() {
  getval() = 4;  // 函数返回的引用也可以作为左值
  cout << x;
  return 0;
}
```
其实左值就是一个拥有地址的表达式，也就是说左指其实指向的是一个稳定的地址空间（即可以是在堆上由用户管理的内存空间，也可以是在栈上，离开了一个block就被销毁的内存空间）。在上面代码中，`getVal()`函数调用返回的就是一个全局变量(建立在堆上)，所以可以当作左值使用。
**关于右值的几个例子：**
右值与左值刚好相反，右值指向的是一个临时空间。
```
int x;
int getVal() {
  return x;
}
int main() {
  getval() = 4;  // 此处使用会出错，因为此时函数调用返回的是一个右值
  cout << x;
  return 0;
}
```
**所以右值只能用来给其他的左值赋值**。但是可以用一个const左值引用绑定一个右值。由于左值引用并不是左值，没有指定的内存空间，所以如果是非const类型就可以对其进行修改，但是右值又不能进行赋值操作；在对左值引用加上const之后就不可以进行修改，所以可以绑定到右值。
```
int x = 9;
int getVal() {
  return x;
}
int main() {
  const int& b = get();
  cout << "左值绑定到右值： " << b  << endl;
  return 0;
}
```

2).右值引用：
右值引用的方法：`Datatype&& Variable = rightValue`。右值引用是C++11新增的特性。**右值引用是将一个变量绑定到右值，绑定到右值以后本来会被销毁的右值的生存期会延长至与绑定到它的那个对象的生存期**。右值引用的存在是为了充分利用右值(特别是临时对象)来减少对象建构和析构操作以达到提高效率的目的。


**补充：**
右值引用关联到右值时，右值被存储到特定位置，右值引用指向该特定位置。也就是说，<font color = red>右值虽然无法获取地址，但是右值引用是可以获取地址的，该地址表示临时对象的存储位置</font>。

**右值引用与左值引用绑定规则:**
右值引用`rvalue reference '&&'`跟传统意义上的引用`reference '&'`很相似，为了更好地区分它们俩，<font color=#8745>传统意义上的引用又被称为左值引用（lvalue reference）</font>。
```
非const左值引用只能绑定到非const左值;

const左值引用可以绑定到const和非const左值，const和非cosnt右值;因为const左值引用不可以修改绑定的值

非const右值引用只能绑定到非const右值;

const右值引用只能绑定到const右值和非const右值(const右值引用只是为了语义的完整而存在,const左值引用就可以实现它的作用).
```
虽然从绑定规则中可以看出cosnt左值引用也可以绑定到右值,但显然不可以改变右值的值；右值引用绑定到右值的时候就可以改变右值的值,从而实现转移语义。**因为右值引用通常要改变所绑定的右值,所以被绑定的右值不能为const。**
```
[[include]]<iostream>
class A {
 public:
  A() {}
};
A lvalue; // 非const左值对象
const A const_lvalue; // const左值对象
A rvalue() {
  A temp;
  return temp;
} // 返回一个非const右值对象
const A const_rvalue() {
  A temp;
  return temp;
} // 返回一个const右值对象

// 规则一：非const左值引用只能绑定到非const左值
A& lvalue_reference1 = lvalue; // ok
A& lvalue_reference2 = const_lvalue; // error
A& lvalue_reference3 = rvalue(); // error
A& lvalue_reference4 = const_rvalue() // error


// 规则二：const左值引用可绑定到const左值、非const左值、const右值、非const右值
const A& lvalue_reference1 = lvalue; // ok
const A& lvalue_reference2 = const_lvalue; // ok
const A& lvalue_reference3 = rvalue(); // ok
const A& lvalue_reference4 = const_rvalue(); // ok

// 规则三：非const右值引用只能绑定到非const右值
A&& rvalue_reference1 = lvalue; // error
A&& rvalue_reference2 = const_rvalue; //error
A&& rvalue_reference3 = rvalue(); // ok
A&& rvalue_reference4 = const_rvalue(); // error
int&& rvalue_reference5 = 123; // 右值引用绑定到常量上也可以

// 规则四：const右值引用可绑定到const右值和非const右值，不能绑定到左值
const A&& rvalue_reference1 = lvalue; // error
const A&& rvalue_reference2 = const_lvalue; // error
const A&& rvalue_reference3 = rvalue(); // ok
const A&& rvalue_reference4 = const_rvalue(); // ok

// 规则五：函数类型除外
void fun() {}
typedef decltype(fun) Fun; // typedef void Fun,Fun是void的别名
Fun& lvalue_reference1 = fun; // 左值引用绑定到函数上
const Fun& lvalue_reference2 = fun; // const左值引用绑定到函数
Fun&& rvalue_reference1 = fun; // 右值引用绑定到函数
const Fun&& rvalue_reference2 = fun; // const右值引用绑定到函数
```
**注意:右值引用的对象其实是一个左值!**

###4.转移语义(move semantics)：
**右值引用被引入的目的之一就是实现转移语义**,转移语义可以将资源 ( 堆,系统对象等 ) 的所有权从一个对象(通常是匿名的临时对象)转移到另一个对象，从而减少对象构建及销毁操作,提高程序效率。

虽然普通的函数和操作符也可以利用右值引用实现转移语义，但**转移语义通常是通过转移构造函数和转移赋值操作符实现的**。转移构造函数的原型为`Classname(Typename&&)`，而拷贝构造函数的原型为`Classname(const Typename&)`；转移构造函数不会被编译器自动生成，需要自己定义，<font color=#9856>只定义转移构造函数也不影响编译器生成拷贝构造函数；如果传递的参数是左值，就调用拷贝构造函数，反之如果传递的参数是右值，就调用转移构造函数</font>.

1）转移构造函数：
```
[[include]] <iostream>
class Demo {
 public:
  // 普通的构造函数
  Demo() : arr(new int [1000]), size(1000) {
    for (int i=0; i < 1000; i++)
      arr[i] = i+1;
  };
  // 转移构造函数，使临时变量的生存周期延长到了和绑定的对象一样
  // 移动构造函数中lre虽然为左值，但是它的生命周期只有这个构造函数，所以构造函数结束后lre会析构。
  Demo(Demo&& lre) : arr(lre.arr), size(lre.size) {
    lre.size = 0;
    lre.arr = nullptr; // 防止临时对象析够的时候转移的资源被系统收回
  }
  // 拷贝构造函数
  Demo(const Demo& lre) : arr(new int [1000]), size(lre.size) {
    for (int index = 0; index < 1000; ++index) {
      this->arr[index] = lre.arr[index];
    }
  }
  int GetSize() {
    return this->size;
  }
  int* GetArr() {
    return this->arr;
  }
  ~Demo() {
    delete []arr;
  }
 private:
  int size;
  int* arr;
};
Demo func_test() {
  Demo demo1;
  return demo1;
}  // 返回一个右值
int main() {
  Demo demo2(func_test());  // 构造的时候传入一个右值
  std==cout <<  demo2.GetSize() << std==endl;
  std==cout <<  demo2.GetArr()[0] << std==endl;
  return 0;
}

```
从以上代码可以看出,拷贝构造函数在堆中重新开辟了一个大小为1000的int型数组,然后每个元素分别拷贝；而转移构造函数则是直接接管参数的指针所指向的资源,效率高下立判！需要注意的是：**转移构造函数传入的实参必须是右值,一般是临时对象,如函数的返回值等**,对于此类临时对象一般在当行代码之后就被销毁,而采用转移构造函数可以延长其生命期,可谓是物尽其用,同时有避免了重新开辟数组。

**转移构造函数的注意事项：**
`Demo(Demo&& lre) : arr(lre.arr), size(lre.size)({lre.arr=nullptr;}`在这里，`lre`是一个右值引用,通过它间接访问实参(临时对象)的资源来完成资源转移,<font color=red>lre绑定的对象(必须)是右值,但lre本身是左值</font>;

从一个对象移动数据并不会销毁此对象，但有时在移动操作完成后，源对象会被销毁。因此，当我们编写一个移动操作时，必须确保moved-from对象进入一种可析构的状态。在这里因为lre是构造函数的局部对象,`lre.arr=nullprt`必不可少,否则转移狗仔函数结尾调用析构函数销毁`lre`时仍然会将资源释放,转移的资源还是被系统收回。

**补充：**
<font color=red>拷贝构造函数中，对于指针，我们一定要采用深层复制，而移动构造函数中，对于指针，我们采用浅层复制。但是由于指针的浅层复制是非常危险的（浅层复制之所以危险，是因为两个指针共同指向一片内存空间，若第一个指针将其释放，另一个指针的指向就不合法了），所以我们只要避免第一个指针释放空间就可以了。避免的方法就是将第一个指针（比如a->arr）置为NULL，这样在调用析构函数的时候，由于有判断是否为NULL的语句，所以析构a的时候并不会回收a->arr指向的空间（同时也是b->arr指向的空间）</font>

2）转移赋值操作符：
移动赋值操作符是移动操作符的一个重载，
```
Mystring& operator=(Mystring&& str) {  // 转移赋值运算符
  std==cout << "Move Assignment is called! source: " << str._data << std==endl;
  // 只有在左右不相等的时候才会进行移动赋值，防止自赋值的发生
  if(this != &str){
    _len = str._len;
    _data = str._data;
    str._len = 0;
    str._data = nullptr;
  }
  return *this;
}
```
增加了转移构造函数和转移复制操作符后，我们的程序运行结果为 :
```
Move Assignment is called! source: Hello
Move Constructor is called! source: World
```
由此看出，编译器区分了左值和右值，对右值调用了转移构造函数和转移赋值操作符。节省了资源，提高了程序运行的效率。有了右值引用和转移语义，我们在设计和实现类时，对于需要动态申请大量资源的类，应该设计转移构造函数和转移赋值函数，以提高应用程序的效率。

###5.move函数：
深拷贝和move的区别：
![1](/assets/1.jpg)
可以看到。move的代价很小，只是转移了资源的所有权。

1）转移语义并不是万能的，上面的例子中，`Demo(Demo&& lre)`的实参必须是右值,有时候一个左值即将到达生存期,但是仍然想要使用转移语义接管它的资源,这时就需要move函数.

2）`std::move`函数定义在标准库`<utility>`中,<font color=#25689>它的作用是将左值强行转化为右值使用</font>,从实现上讲,`std:move`等同于`static_cast<T&&>(lvalue)`,由此看出,被转化的左值本身的生存期和左值属性并没有被改变,这类似于`const_cast`函数.因此被move的实参应该是即将到达生存期的左值,否则的话可能起到反面效果。

3）通过`std::move`，可以避免不必要的拷贝操作，只是将对象的状态或者所有权从一个对象转移到另一个对象，只是转移，没有内存的搬迁或者内存拷贝。
**注意：**
由于变量表达式是一个左值(即使这个变量是一个右值引用，它本质上也是一个左值)，所以我们不能直接将一个右值引用绑定到一个变量上。但是，我们可以显式的将一个左值转换为对应的右值引用类型，即通过move函数来获得绑定到左值上的右值引用：
```
int &&rr1 = 42;                //正确，字面值常量是右值
int &&rr2 = rr1;               //错误，变量rr1是左值
int &&rr3 = std::move(rr1);    //正确，通过move函数获得rr1的右值引用
```
move告诉编译器：我们有一个左值，但我们希望像一个右值一样处理它。我们必须认识到，调用move就意味着：除了对 rr1 赋值或销毁外，我们不再使用它。在调用move之后，我们不能对moved-from对象（即rr1）做任何假设。
**举例：**
```
[[include]] <iostream>
[[include]] <string>
[[include]] <utility>
[[include]] <vector>
int main() {
  std::string str("Hello");
  std==vector<std==string> vec;

  /*uses the push_back(const T&) overload,
    which means we'll incur the cost of copying str
   */
  vec.push_back(str);
  std::cout << "After copy, str is \"" << str << "\"\n";

  /*uses the rvalue reference push_back(T&&) overload,
    which means no strings will be copied; instead, the contents
    of str will be moved into the vector.  This is less
    expensive, but also means str might now be empty.
   */
  vec.push_back(std::move(str));
  std::cout << "After move, str is \"" << str << "\"\n";
  std::cout << "The contents of the vector are \"" << vec[0] << "\", \"" << vec[1] << "\"\n";
  return 0;
}
```
输出结果：
```
After copy, str is "Hello"
After move, str is ""
The contents of the vector are "Hello", "Hello"
```
<font color=red>可以看到，在上面的例子中，string类型的对象在moved之后，它的值也许就变为空了。</font>

std::move的本质：
```
void func(int&& a)
{
    cout << a << endl;
}

int a = 6;
func(std::move(a));

int b = 10;
func(static_cast<int&&>(b));
```
上面两个调用是等价的，`std::move`是个语法糖，只是执行到右值的无条件转换，并没有移动任何东西。

**补充：**
1）`std::move`的使用:
```
template <class T> void swap(T& a, T& b) {  // 一般情况下swap进行了三次拷贝操作
  T temp(a);
  a = b;
  b = temp;
}
template <class T> void swap(T& a, T& b) {  // 使用std::move函数可以避免不必要的拷贝操作
  T temp(std::move(a));
  a = std::move(b);
  b = std::move(temp);
}
```
2）引用折叠：
如果间接的创建一个引用的引用，则这些引用就会“折叠”。在所有情况下（除了一个例外），引用折叠成一个普通的左值引用类型。一种特殊情况下，引用会折叠成右值引用，即右值引用的右值引用：`T&& &&=&&`。引用折叠其实更像引用坍塌，根本原因是因为C++中禁止reference to reference，所以编译器需要对四种情况(L2L,L2R,R2L,R2R)进行处理，将他们“折叠”(也可说是“坍缩”)成一种单一的reference：
```
A& & 变成 A&
A& && 变成 A&
A&& & 变成 A&
A&& && 变成 A&&
```
3）右值引用的特殊类型推断：
当将一个左值传递给一个参数是右值引用的函数，且此右值引用指向模板类型参数`(T&&)`时，编译器推断模板参数类型为实参的左值引用，如：
```
template<typename T>
void f(T&& value);  // 模板函数的行参为右值引用

int i = 42;
f(i);
```
上述的模板参数类型`T`将推断为`int&`类型，而非`int`。将其与引用折叠结合起来，则意味着可以传递一个左值`int i`给函数`f`，编译器将推断出`T`的类型为`int&`。再根据引用折叠规则`void f(int& &&)`将推断为`void f(int&)`，因此，f将被实例化为:`void f<int&>(int&)`。

4）


###6.完美转发(perfect forwarding)：c++11中的完美转发`std::forward`函数:
右值引用类型是独立于值的，一个右值引用参数作为函数的形参，在函数内部再转发该参数的时候它已经变成一个左值了，并不是它原来的类型了，那么他永远不会调用接下来函数的右值版本，这可能在一些情况下造成拷贝。为了解决这个问题 C++ 11引入了完美转发。所谓完美转发（perfect forwarding），是指在函数模板中，完全依照模板的参数的类型，在将参数传递给函数模板中调用的另外一个函数时，根据右值判断的推导，调用forward传出的值。若原来是一个右值，那么他转出来就是一个右值，否则为一个左值。这样的处理就完美的转发了原有参数的左右值属性，不会造成一些不必要的拷贝。代码如下：
```
[[include]] <iostream>
[[include]] <string>
[[include]] <vector>
// 模板函数的行参为左值引用
template<typename T>void print(T& t) {
  std==cout << "lvalue reference" << t << std==endl;
}
// 模板参数的行参为右值引用
template<typename T>void print(T&& t) {
  std==cout << "rvalue reference" << t << std==endl;
}
template<typename T>void testForward(T&& value) {
  print(value);  // 这里根据value的类型进行重载
  print(std::forward<T>(value));  // 完美转方式
  print(std::move(value));  // move函数将一个左值转换为右值进行传入
}
int main() {
  testForward(1);  // 直接传入一个右值
  int x = 2;
  testForward(x);  // 传入一个左值
  return 0;
}
```
输出结果：
```
lvalue reference1
rvalue reference1
rvalue reference1
lvalue reference2
lvalue reference2
rvalue reference2
```
分析：
1）TestForward(1);由于1是右值，所以未定的引用类型T && v被一个右值初始化后变成了一个右值引用，但是在TestForward函数体内部，调用PrintT(v);时，v又变成了一个左值，因为它这里已经变成了一个具名的变量，所以它是一个左值，因此第一个PrintT被调用，打印出"lvaue"；
PrintT(std==forward<T>(v));由于std==forward会按参数原来的类型转发，因此，这时它还是一个右值（这里已经发生了类型推导，所以这里的T&&不是一个未定的引用类型），所以会调用void PrintT(T &&t)函数。
PrintT(std==move(v));是将v变成一个右值引用，虽然它本来也是右值引用，因此它和PrintT(std==forward<T>(v));的输出结果是一样的。
2）TestForward(x);未定的引用类型T && v被一个左值初始化后变成了一个左值引用，因此在调用PrintT(std::forward<T>(v));它会转发到void PrintT(T& t);

**补充：**
1）std==move和std==forward本质就是一个转换函数。std==move执行到右值的无条件转换(类似于`static_cast<T&&>(value)`)，std==forward执行到右值的有条件转换，在参数都是右值时，二者就是等价的。其实std==move和std==forward就是在C++11基本规则之上封装的语法糖。
2）std==move执行到右值的无条件转换。就其本身而言，它没有move任何东西。std==forward只有在它的参数绑定到一个右值上的时候，它才转换它的参数到一个右值。
3）std==move和std==forward只不过就是执行类型转换的两个函数；std==move没有move任何东西，std==forward没有转发任何东西。在运行期，它们没有做任何事情。它们没有产生需要执行的代码，一byte都没有。
4）`std::forward<T>()`不仅可以保持左值或者右值不变，同时还可以保持const、Lreference、Rreference、validate等属性不变；

一个具体的例子：
```
[[include]] <iostream>
[[include]] <memory>
[[include]] <string>
[[include]] <typeinfo>
[[include]] <type_traits>
[[include]] <vector>

struct A
{
  A(int&& n) {
    std==cout << "rvalue overload, n=" << n << std==endl;
  }
  A(int& n) {
    std==cout << "lvalue overload, n=" << n << std==endl;
  }
};
class B {
 public:
  // 这里的构造函数为一个模板函数
  template<typename T1, typename T2, typename T3> B(T1&& t1, T2&& t2, T3&& t3) :
  a1_(std==forward<T1>(t1)), a2_(std==forward<T2>(t2)), a3_(std::forward<T3>(t3)){ // 初始化列表中的是构造函数
    // a1_ = A(std::forward<T1>(t1));  // 这里有问题，不太清楚具体情况
    // a2_ = A(std::forward<T2>(t2));
    // a3_ = A(std::forward<T3>(t3));
  }
 private:
  A a1_, a2_, a3_;
};
template<typename T, typename U>std::unique_ptr<T> make_unique1(U&& u) {
  return std==unique_ptr<T>(new T(std==forward<U>(u)));
  //return std==unique_ptr<T>(new T(std==move(u)));
}
template <typename T, typename... U>std::unique_ptr<T> make_unique2(U&&... u)
{
  return std==unique_ptr<T>(new T(std==forward<U>(u)...));
  //return std==unique_ptr<T>(new T(std==move(u)...));
}
int main() {
  auto p1 = make_unique1<A>(2);

  int i = 10;
  auto p2 = make_unique1<A>(i);

  int j = 100;
  auto p3 = make_unique2<B>(i, 3, j);
  return 0;
}
```
###7.emplace_back()用法:
在使用c++的stl容器时(如map，vector，set等)，经常需要对其进行插入操作。使用`push_back`向容器中加入一个右值元素(临时对象)时，首先会调用构造函数构造这个临时对象，然后需要调用拷贝构造函数将这个临时对象放入容器中。原来的临时变量释放。这样造成的问题就是临时变量申请资源的浪费。
在引入了右值引用，转移构造函数后，push_back()右值时就会调用构造函数和转移构造函数,如果可以在插入的时候直接构造，就只需要构造一次即可。这就是c++11 新加的`emplace_back`。

**emplace_back函数原型：**
```
template <class... Args>
  void emplace_back (Args&&... args);
```
在容器尾部添加一个元素，这个元素原地构造，不需要触发拷贝构造和转移构造。而且调用形式更加简洁，直接根据参数初始化临时对象的成员。
```
[[include]] <iostream>
[[include]] <string>
[[include]] <vector>
class Test {
 public:
  explicit Test(int i);
  Test(const Test& other);
  ~Test();
 private:
  std::string str;
};
Test::Test(int i) {
  str = std::to_string(i);
  std==cout <<  "构造函数被调用" <<std==endl;
}
Test::Test(const Test& other) : str(other.str) {
  std==cout << "拷贝构造函数被调用" <<std==endl;
}
Test::~Test() {
  std==cout << "析够函数被调用" << std==endl;
}
int main() {
  std::vector<Test> vec;
  vec.reserve(10);  // 这里预先给vector分配空间
  for (int i = 0; i < 10; i++) {
    // vec.push_back(Test(i));  // 调用了10次构造函数和10次拷贝构造函数
    vec.emplace_back(i);  //调用了10次构造函数，一次拷贝构造函数都没有调用过；其实就是一个右值引用，所以可以直接将右值传递进去
  }
  return 0;
}
```
###8.`std::allocator`:


###9.委托构造函数：
