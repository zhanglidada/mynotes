#C++ 强制转换运算符用法小结
##补充知识点：
向上类型转换：指的是子类向基类进行的强制类型转换
向下类型转换：指的是基类向子类进行的强制类型转换

**强制类型转换的时候其实并没有改变对应内存的内容，其实只是改变了那块内存位置的读取方式。其实对应内存中的内容只和new时候的类型有关。**
![30](/assets/30.png)
我们知道，派生类不仅有自己的方法和属性，同时它还包括从父类继承来的方法和属性。当我们从派生类向基类转换时，不管用传统的c语言还是c++转换方式都可以百分百转换成功。但是可怕是向下转换类型，也就是我们从基类向派生类转换，当我们采用传统的C语言和c++转换时，就会出现意想不到的情况，因为**转换后派生类自己的方法和属性丢失了**，一旦我们去调用派生类的方法和属性那就糟糕了，这就是对类继承关系和内存分配理解不清晰导致的。好在c++增加了static_cast和dynamic_cast运用于继承关系类间的强制转化。

##前言：
**C++引入了4种类型转换运算符，更加严格的限制允许的类型转换，使转换过程更加规范.**
1). dynamic_cast : 用于多态类型的转换，即有继承关系的类指针之间、或者有交叉关系的类指针之间的转换，并且具有类型检查的功能，同时**需要虚函数的支持**。

2). static_cast: **用于非多态类型的转换**，即基本类型之间、有继承关系的类对象之间、类指针之间的转换，**不能用于基本类型指针之间的转换**

3). const_cast: **用于删除const ,volatile 和 __unaligned 属性**，<font color=#5787543>注意，强制转换的目标类型必须是指针或引用</font>

4). reinterpret_cast: 用于指针类型之间、整数和指针类型之间的转换，即位的简单重新解释

<font color =#548c00>其中，const_cast和reinterpret_cast的使用会带来更大的风险，因此不到万不得已，不推荐使用。</font>

##dynamic_cast
用法： `dynamic_cast<type>(expression);`
转换方式：
`dynamic_cast< type* >(e)`:type必须是一个类类型且必须是一个有效的指针。
`dynamic_cast< type& >(e)`：type必须是一个类类型且必须是一个左值
`dynamic_cast< type&& >(e)`：type必须是一个类类型且必须是一个右值

e的类型必须符合以下三个条件中的任何一个：
1）e的类型是目标类型type的公有派生类
2）e的类型是目标type的共有基类
3）e的类型就是目标type的类型。

备注：
　　① <font color=#235684>转换类型必须是一个指针、引用或者void*，用于将基类的指针或引用安全地转换成派生类的指针或引用</font>；
　　② dynamic_cast在运行期间强制转换，运行时进行类型转换检查；
　　③ 对指针进行转换，失败返回null，成功返回type类型的对象指针；对于引用的转换，失败抛出一个bad_cast ，成功返回type类型的引用；
　　④ **dynamic_cast不能用于内置类型的转换**
　　⑤ **用于类的转换，基类中一定要有virtual定义的虚函数（保证多态性），不然会编译错误。**

举例：
<font color=red>dynamic_cast最常用的场景就是将原本**指向派生对象的<font color=#5879>基类指针或引用</font>”升级为“派生类指针或引用**</font>
```
#include<iostream>
class Base {
 public:
  virtual void show() {//基类的虚函数
	  std::cout << "this is the base function!  Base::show()." << std::endl;
  }
  virtual ~Base(){}
};
//派生类
class Derived : public Base {
 public:
  virtual void show() {
	  std::cout << "this is the derived function!  Derived::show()." << std::endl;
  }
    virtual ~Derived(){}
};
void change_type(Base *&base) {
  if (Derived *derived = dynamic_cast<Derived *>(base)) {
	  std::cout << "基类指针转换为派生类指针成功！" << std::endl;
  } else {
	  std::cout << "基类指针转换为派生类指针失败！" << std::endl;
  }
}
int main() {
    Base *base = new Derived;  // 指向派生对象的基类指针
    change_type(base);  // 基类指针强制类型转换为派生类后指向了派生类对象的内存

    base = new Base;  // 指向基类对象的基类指针
    change_type(base);  // 基类指针强制类型转换为派生类后指向了基类对象的内存，存在问题
    return 0;
}
```
输出结果：
```
基类指针转换为派生类指针成功！
基类指针转换为派生类指针失败！
```
<font color=#9856>**注释**：
1) 第一种情况，先让基类的指针指向派生类对象，即派生类对象赋值给基类指针，这其实就是一种向上转换，可以直接隐式转换，合乎语法。然后令基类指针“升级为”派生类指针，原则上，这是一种“向下转换”（downcasting），需要显示强制转换，但由于基类指针所指向的内存是派生类的，所以转换后派生类指针指向派生类的内存，理论上不存在安全隐患，所以用dynamic_cast进行强转，没任何问题。

2) 第二种情况，先让基类指针指向基类对象，然后基类指针强制类型转换为派生类指针，并令派生类指针指向基类对象的内存，显然是存在安全隐患，dynamic_cast会返回一个null指针，告诉开发者转换失败。**但这个操作发生在运行时。**

3) dynamic_cast和传统的`(type)(expression)`强制转换的最大区别在于提供了运行时的类型检查，保证了类型安全，使用强制转换，会跳过编译器的类型检查，但可能会造成运行时异常，导致程序直接崩溃。
</font>

##static_cast
用法：`static_cast<type>(expression);`
作用：　与dynamic_cast作用类似，将expression转换为type类型，区别在于，<font color=#e1234>static_cast发生于编译时(即编译时检查)，dynamic_cast发生于运行时。</font>

备注：
1)用于类层次结构中基类和派生类之间指针或引用的转换，其中——**向上转换是安全的，向下转换是不安全的，但两者均可以通过编译**，也就是说开发者要负责强制转换运行时的安全性，这一点不如`dynamic_cast`安全；
```
#include<iostream>
class Another{};
class Base{};
class Derived : public Base{};

int main(){
    Base *base = static_cast<Base *>(new Derived);//向上类型转换，能通过编译而且安全
    Derived *derive = static_cast<Derived *>(new Base);//向下类型转换，能通过编译但是不安全
    //Another *a = static_cast<Another *>(new Base); //没有任何关系，不能进行强制类型转换
    return 0;
}
```
2)可以用于内置类型的转换。
C中内置类型隐式转换是有规则的。比如说，算术运算，低类型自动往高类型转换（如图），但转换的精度损失由开发者负责，编译器往往会提出警告。使用了static_cast运算符之后，等于告诉编译器，“我知道这里发生了类型转换，我会为转换的安全性负责，你不用管了”，编译器不会发出编译警告，除非你类型转换完全非法（比如 `int a = static_cast<int>("Hello world!"); `时 ），
`static_cast`才会报编译错误；
![22](/assets/22.png)

3)把`void*`转换成目标类型的指针；

4)把任意类型转换成`void`类型；

5)`static_cast`无法转换expression中的`const / volatile / __unaligned` 属性（会报编译时错误）。

##const_cast
用法：`const_cast<type>(expression);`
作用： 弥补了`static_cast`无法转换`const / volatile` 的不足，`将expression的const / volatile属性移除`，**仅限于底层const属性.**

备注：
1)顶层const：表示指针本身是常量，比如`int *const pointer`，此时指针自身的值不可修改，但是指针可以进行解引用修改指针所指向的变量的值。

底层const：表示指针所指向的变量是常量，比如`const int * pointer`，此时指针的值可以修改，但是指针所指向的值不能被指针进行解引用所修改。(<font color=#456712>一般，顶层const表示任意的对象是常量,对算数类型，类，指针均适用</font>)

2)**`const_cast`不能执行其他任何类型转换，只能用于<font color=#5897>同类型之间不同的`const/volatile`属性的移除</font>。否则会报编译时错误。**

3)<font color=#245679>需要注意的是，`const_cast`**通常对指针和引用进行转换**，而无法直接移除内置类型的`const/volatile`属性。变量本身的const属性是不能去除的，要想修改变量的值，一般是去除指针（或引用）的const属性，再进行间接修改。换言之，这种语法直接提供了一个具有写权限的指针或引用，可以通过间接访问的方式修改常量。</font>

####例子：
```
#include<iostream>
using namespace  std;
int main(){
    const int a = 10;//a是常量
    const int * const_pointer = &a;//指针所指的变量为常量，但是指针本事不是常量，即底层const
    int *b = const_cast<int *>(const_pointer);//const_cast的转换仅限于底层const
    cout<<"修改前*b的值： "<<*b<<endl;
    *b=20;
    cout<<"修改后*b的值： "<<*b<<endl;
    cout<<"a="<<a<<endl;
    cout<<"b="<<b<<endl;
    cout<<"&a="<<&a<<endl;
    return 0;
}
```
![23](/assets/23.png)

在这里可以看到，通过const_cast 常量a被修改。但是这里有个有趣的现象，就是，指向常量a的指针b解引用修改了值，但是a本身的值却还是没变，从地址上看，指针b确实指向了常量a，但是*b和a却不一样，这是为什么呢？
推测可能是编译器对字面型常量的引用，有自己的优化，所以a的值没有发生更改，但实际上已经改了。

####对此稍作修改：
```
#include<iostream>
using namespace  std;
int main(){
    int input;
    cout<<"请输入一个数字"<<endl;
    cin>>input;
    const int a = input;//a是常量
    const int * const_pointer = &a;//指针所指的变量为常量，但是指针本事不是常量，即底层const
    int *b = const_cast<int *>(const_pointer);//const_cast的转换仅限于底层const
    cout<<"修改前*b的值： "<<*b<<endl;
    *b=20;
    cout<<"修改后*b的值： "<<*b<<endl;
    cout<<"a="<<a<<endl;
    cout<<"b="<<b<<endl;
    cout<<"&a="<<&a<<endl;
    return 0;
}
```
![24](/assets/24.png)

此时，常量a的值也被修改。
##reinterpret_cast
用法：`reinterpret_cast<type>(expression);`
```
从指针类型到一个足够大的整数类型
从整数类型或者枚举类型到指针类型
从一个指向函数的指针到另一个不同类型的指向函数的指针
从一个指向对象的指针到另一个不同类型的指向对象的指针
从一个指向类函数成员的指针到另一个指向不同类型的函数成员的指针
从一个指向类数据成员的指针到另一个指向不同类型的数据成员的指针
```
作用： `reinterpret_cast` 允许将任何指针转换为任何其他指针类型。 也允许将任何整数类型转换为任何指针类型以及反向转换。

备注：
1)`reinterpret_cast`运算符可用于`char*`到`int*`或`One_class*` 到`Unrelated_class*`之类的转换，这本身并不安全，但可以通过编译；
2)`reinterpret_cast`用来处理无关类型转换,**本质是重新定义内存数据的解释方式**，即为操作数的位模式提供较低层次的重新解释。但是他仅仅是重新解释了给出的对象的比特模型，并没有进行二进制的转换！

```
int *pi;
char *pc = reinterpret_cast<char*>(pi);
```
可以看到reinterpret_cast的强大作用，但是要注意的是，它并没有进行二进制的转换，pc指向的真实对象其实还是int的，不是char~ 。

举个例子：
```
#include<iostream>
int main(){
    char s[]="hello world!";
    long l = reinterpret_cast<long>(s);//这里提取了字符串的地址，并转换为long类型。
    std::cout<<"l = "<<l<<std::endl;
    return 0;
}
```
![25](/assets/25.png)

**注意事项：**
错误的使用reinterpret_cast很容易导致程序的不安全，只有将转换后的类型值转换回到其原始类型，这样才是正确使用reinterpret_cast方式。

**补充说明：**
reinterpret_cast不能像const_cast那样去除const修饰符。
