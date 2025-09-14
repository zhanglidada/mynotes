[[C]]++模板类和模板函数详解
![14](/assets/14.png)
##简介：
模板是C++支持**参数化多态的工具**，使用模板可以为类或者函数声明一种**一般模式**，<font color=red>使得类中的某些数据成员或者成员函数的参数、返回值取得任意类型。</font>

模板是一种对**类型**进行**参数化**的工具；
通常有两种形式：**函数模板**和**类模板**；
函数模板针对仅**参数类型**不同的**函数**；
类模板针对**仅数据成员**和**成员函数类型不同**的类。
<font color=#158349>在c++的模板定义过程中，typename和class没有任何区别(但是也仅仅是在这里没有区别)</font>

<font color=darkgreen>使用模板的目的就是能够让程序员编写与类型无关的代码</font>。比如编写了一个交换两个整型`int`类型的`swap`函数，这个函数就只能实现`int`型，对`double`，字符,这些类型无法实现，要实现这些类型的交换就要重新编写另一个`swap函数`。<font color=#A23400>使用模板的目的就是要让这程序的实现与类型无关</font>，比如一个`swap`模板函数，即可以实现`int`类型，又可以实现`double`型的交换。

**<font color=red>注意：模板的声明或定义只能在全局，命名空间或类范围内进行。即不能在局部范围，函数内进行，比如不能在main函数中声明或定义一个模板。</font>**
##一.函数模板通式
###1.函数模板的格式
```
template <class 形参名，class 形参名，......>
返回类型 函数名(参数列表) {
  //函数体
}
```
其中**template**和**class**是关键字，**class**可以用**typename**关键字代替，<font color=#930000>在这里typename 和class没区别，</font><font color=#336666> `<>`括号中的参数叫**模板形参**，</font>模板形参和函数形参很相像，**模板形参不能为空**。 **一但声明了模板函数就可以用模板函数的<font color=red>形参名</font>声明函数中的变量，即在该函数中可以使用内置类型的地方都可以使用模板形参名。** 模板形参需要在调用该模板函数时提供的模板实参来初始化模板形参，<font color=red>一旦编译器确定了实际的模板实参类型就称它实例化了函数模板的一个实例。</font>比如swap的模板函数形式为：
```
template <class T>
void swap(T& a, T& b) {
  // ......
}
```
当调用这样的模板函数时**类型T就会被被调用时的类型所代替**，比如`swap(a,b)`其中a和b是int 型，**这时模板函数swap中的形参T就会被`int`所代替**，模板函数就变为`swap(int &a, int &b)`。而当`swap(c,d)`其中c和d是double类型时，模板函数会被替换为`swap(double &a, double &b)`，这样就实现了函数的实现与类型无关的代码。
###2.注意：
对于函数模板而言不存在`h(int,int)`这样的调用，<font color=3a006f>**不能在函数调用的参数中指定模板形参的类型**</font>，<font color=red>对函数模板的调用应使用实参推演来进行，即只能进行`h(2,3)`这样的调用，或者`int a, b;h(a,b)`。</font>

##二.类模板通式
###1.类模板的格式为：
```
template <class 形参名，class 形参名，...>
class 类名 {
  //......
}
```
　　类模板和函数模板都是以**template**开始后接**模板形参列表**组成，模板形参不能为空，<font color=#600000>一但声明了类模板就可以用类模板的**形参名**声明类中的成员变量和成员函数</font>，即可以在类中使用内置类型的地方都可以使用模板形参名来声明。比如：
```
template<class T> class A {
public:
  T a; T b;
  T thy(T &c, T &d);
};
```
在类A中声明了两个类型为T的成员变量a和b，还声明了一个返回类型为T,带两个形参类型为T的函数thy。
###2.类模板对象的创建：
比如一个模板类A，则使用类模板创建对象的方法为`A<int> m`;在类A后面跟上一个`<>`尖括号并在里面填上相应的类型，<font color=red>这样的话类A中凡是用到模板形参的地方都会被`int`所代替。</font>当类模板有两个模板形参时，创建对象的方法为`A<int, double> m`;类型之间用逗号隔开。

###3.类模板形参类型的指定：
**<font color=#006030>对于类模板，模板形参的类型必须在类名后的尖括号中明确指定。**</font>
比如`A<2>m`;用这种方法把模板形参设置为`int`是错误的（编译错误：error C2079: 'a' uses undefined class 'A<int>'），**类模板形参不存在实参推演的问题**也就是说不能把整型值2推演为`int`型传递给模板形参。要把类模板形参调置为`int`型必须这样指定:`A<int>m`。

###4.在类模板外部定义成员函数的方法为：
```
template< 模板形参列表 >函数返回类型 类名<模板形参名>::函数名(参数列表) {
  //函数体
};
```
比如有两个模板形参T1，T2的类A中含有一个void h()函数，则定义该函数的语法为：
```
template<class T1,class T2>void A<T1,T2>::h() {
  //......
}
```
**注意：当在类外面定义类的成员时template后面的模板形参应与要定义的类的模板形参一致。**

###5.再次提醒：
模板的声明或定义只能在<font color=red>全局，命名空间或类范围内进行</font>。即不能在局部范围，函数内进行;比如不能在main函数中声明或定义一个模板。

##三.模板的形参
**有三种类型的模板形参：类型形参，非类型形参和模板形参。在实例化模板的时候，编译器会自动根据实参的类型对模板行参进行推导。**
###1.类型模板参数
类型模板参数是我们使用模板的主要目的。我们可以定义多个类型模板参数:
```
template<typename T,typename Container> 
class Grid {
  //...
}
```
同样，也可以为类型模板参数指定默认值： 
```
[[include]] <iostream> 
using std::vector; 
template<typename T,typename Contianer=vector<T> > //注意空格 
class Grid 
{
  //...
} 
```
####1.1.类型模板形参：
**类型形参由关键字class或typename后接说明符构成**，如:
```
template<class T>
void h(T a) {
  //......
}; //函数模板
```
其中T就是一个类型形参，类型形参的名字由用户自已确定。注意：**模板形参表示的是一个未知的类型**。<font color=red>模板类型形参可作为类型说明符用在模板中的任何地方，与内置类型说明符或类类型说明符的使用方式完全相同</font>，即可以用于指定返回类型，变量声明等。
####1.2.不能为同一个模板类型形参指定两种不同的类型：
比如:
```
//函数模板
template<class T>void h(T a, T b){
//......
}
```
语句调用h(2, 3.2)将出错，因为该语句给同一模板形参T指定了两种类型，第一个实参2把模板形参T指定为int，而第二个实参3.2把模板形参指定为double，两种类型的形参不一致，会出错。**(针对函数模板)**
<font color=#d94600>注意：1.2的说法针对函数模板是正确的，但是忽略了类模板。下面将对类模板的情况进行补充。</font>
####1.3补充(针对于类模板):
当我们声明并定义类对象为：`A<int> a`，比如:
```
//定义对象A的一个成员函数
template<class T>void A<T>::g(T a, T b){
//......
```
当我们语句调用a.g(2, 3.2) 在编译时不会出错，但会有警告，**因为在声明类对象的时候已经将T转换为int类型**，而第二个实参3.2把模板形参指定为double，在运行时，会对3.2进行强制类型转换为3。
当我们声明类的对象为：`A<double> a`,此时就不会有上述的警告，因为**从int到double是自动类型转换**。

<mark><font color=#bb3d00>Template_head.hpp</font></mark>
```
[[ifndef]] TEMP_HEAD
[[define]] TEMP_HEAD

template<class T> class A {
 public:
  A(void);//模板类的构造函数
  T g(T a,T b);  // 模板类的成员函数
 private:
  int number;
};
[[endif]]
```
这里，我们指定模板的形参类型为int，同时给成员函数传递一个int，一个double实参：
<mark><font color=#bb3d00>Template1.cc</font></mark>
```
[[include]]<iostream>
[[include]]<string>
[[include]]"../include/Template_head.hpp"
template<class T>
A <T>::A() {
  this->number = 10;
}
template<class T>
T A<T>::g(T a,T b) {
  return a + b;
}
int main() {
  A<int> a;
  //因为对类模板的形参类型已经指定为int，所以运行时将传递给成员函数的实参进行强制类型转换
  std==cout << a.g(2, 3.2) << std==endl;;
  return 0;
}
```
![9](/assets/9.png)
###2.非类型模板参数
2.1 非类型模板形参：<font color=red>**模板的非类型形参，也就是内置类型形参**</font>，如
```
template<class T, int a>
class B {
  // ...
};
```
其中int a就是模板的非类型形参。

2.2 非类型形参在模板定义的内部是常量值，也就是说：<font color=red>非类型形参在模板的内部是常量</font>。

2.3 **模板的非类型形参只能是整型，enmu，指针和引用**，像`double`，`String`, `String **`这样的类型是不允许的。<font color=#ff0080>但是`double &`，`double *`，对象的引用或指针是正确的</font>。

2.4 **传递给非类型模板形参的实参必须是一个常量表达式**，<font color=red>即它必须能在编译时计算出结果</font>。

2.5 **注意：<font color=#707038>任何局部对象，局部变量，局部对象的地址，局部变量的地址都不是一个常量表达式，都不能用作非类型模板形参的实参。全局指针类型，全局变量，全局对象也不是一个常量表达式，不能用作非类型模板形参的实参**。</font>

2.6 <font color=red>全局变量的地址或引用，全局对象的地址或引用，const类型变量是常量表达式，可以用作非类型模板形参的实参</font>。

2.7 sizeof表达式的结果是一个常量表达式，也能用作非类型模板形参的实参。

2.8 **当模板的形参是整型时，调用该模板时的实参必须是整型的**，且在编译期间是常量，比如
```
template<class T, int a>
class A {
  // ......
};
```
如果有`int b`，这时`A<int, b> m`;将出错，因为b不是常量，如果`const int b`，这时`A<int, b> m`;就是正确的，因为这时b是常量。

2.9 **非类型形参一般不应用于函数模板中**，比如有函数模板
```
template<class T, int a>
void h(T b) {
  // ......
};
```
若使用h(2)调用会出现无法为非类型形参a推演出参数的错误，**对这种模板函数可以用显示模板实参来解决**，如用`h<int, 3>(2)`这样就把非类型形参a设置为整数3。

2.10 **非类型**模板形参的形参和实参间所允许的转换:
1)允许从数组到指针，从函数到指针的转换。如：
```
template<int *a> class A {
  // ......
};
int b[1];
A<b> m;
//即数组到指针的转换
```
2)const修饰符的转换。如：
```
template <typename T, const int *a>
class A {
  // ......
};
int b;
A<int, &b> m;
//即从int *到const int *的转换。
```
3)提升转换。如：
```
template<int a> class A {
  // ......
};
const short b=2;
A<b> m;
//即从short到int 的提升转换
```
4)整值转换。如：
```
template<unsigned int a> class A {
  // ......
};
A<3> m;
//即从int 到unsigned int的转换。
```
5)常规转换。

####非类型形参demo1：
<mark><font color=#4d0000>TemplateDemo.hpp</font></mark>
```
[[ifndef]] TEMP_DEMO_HEAD
[[define]] TEMP_DEMO_HEAD
template<class T,int MAXSIZE>
class Stack {
 public:
  Stack();                 //构造函数
  void push(T const &);    //元素入栈,采用常量引用方式
  void pop();              //元素出栈
  /*
    声明const函数，即成员函数不对成员变量做任何修改
   */
  T top() const;           //获得栈顶元素，这里的const表示成员函数是const类型，不能够修改类成员变量以及调用非const成员函数
  bool empty() const;      //返回栈是否为空
  bool full() const;       //返回栈是否为满
 private:
  T elems[MAXSIZE];        //包含元素的数组
  int numElems;            //元素的当前总个数
};
[[endif]]
```

<mark><font color=#4d0000>TemplateDemo.cc</font></mark>
```
[[include]]<iostream>
[[include]]<string>
[[include]]<cstdlib>
[[include]]"../include/TemplateDemo.hpp"
template<class T,int MAXSIZE> Stack<T,MAXSIZE>::Stack(){//初始时栈不含任何元素
    numElems = 0;
}
template<class T,int MAXSIZE>void Stack<T,MAXSIZE>::push(T const &elem){
    if(numElems == MAXSIZE)
        throw std==out_of_range("Stack<>==push(): stack is full");
    elems[numElems] = elem;//附加元素
    ++numElems;//元素个数增加
}
template<class T,int MAXSIZE>void Stack<T,MAXSIZE>::pop(){
    if(numElems <= 0)
        throw std==out_of_range("Stack<>==pop(): empty stack");
    --numElems;//元素个数增加
}
template<class T,int MAXSIZE>T Stack<T,MAXSIZE>::top() const{
    if(numElems <= 0)
        throw std==out_of_range("Stack<>==top(): empty stack");
    return elems[numElems-1];//返回最后一个元素
}
template<class T,int MAXSIZE>bool Stack<T,MAXSIZE>::empty() const{
    return numElems == 0;
}
template<class T,int MAXSIZE>bool Stack<T,MAXSIZE>::full() const{
    return numElems == MAXSIZE;
}
int main(){
    try{
        Stack<int,20> int20Stack;               //可以存储20个元素
        Stack<int,40> int40Stack;               //可以存40个元素
        Stack<std::string,20> stringStack;      //可以存20个字符串

        // 使用可存储20个int元素的栈
        int20Stack.push(7);
        std==cout << int20Stack.top() << std==endl;    //7
        int20Stack.pop();

        // 使用可存储40个string的栈
        stringStack.push("hello");
        std==cout << stringStack.top() << std==endl;    //hello
        stringStack.pop();
        stringStack.pop();    //Exception: Stack<>::pop<>: empty stack
        return 0;

    }catch (std::exception const &ex){
        std==cerr << "Exception: " <<ex.what() <<std==endl;
        return EXIT_FAILURE;                    //退出且有ERROR标记
    }
    return 0;
}
```
![10](/assets/10.png)
####非类型形参demo2：
<mark><font color=#460046>TemplateDemo2.hpp</font></mark>
```
[[ifndef]] TEMP_DEMO2_HEAD
[[define]] TEMP_DEMO2_HEAD
template<typename T>class CompareDemo{
public:
    CompareDemo();
    int compare(const T&,const T&) const;
};
[[endif]]
```
<mark><font color=#460046>TemplateDemo2.cc</font></mark>
```
[[include]]<iostream>
[[include]]"../include/TemplateDemo2.hpp"
//typename的用法类似于class
template<typename T>CompareDemo<T>::CompareDemo(){
    //nothing to do.
}
template<typename T>int CompareDemo<T>::compare(const T &a,const T &b) const{
    if((a-b)>0)
        return 1;
    else if((a-b)==0)
        return 0;
    else
        return -1;
}
int main(){
    CompareDemo<double> cd;
    std==cout<<cd.compare(2,3)<<std==endl;
    return 0;
}
```
![11](/assets/11.png)

####非类型形参demo3：
<mark><font color=#4d0000>TemplateDemo3.cc</font></mark>
```
[[include]]<iostream>
using namespace std;
//定义了一个函数模板，返回值为const类型，不可被修改
template<typename T>const T& Max(const T &a,const T &b){
    return a > b ? a : b;
}
int main(){
    cout<<Max(2.1,2.2)<<endl;//模板形参被隐式推演为double
    cout<<Max<double>(2.3,2.05)<<endl;//显示指定模板参数
    cout<<Max<int>(2.5,2.7)<<endl;//显示指定的模板参数，会将函数函数直接转换为int。
    return 0;
}
```
![12](/assets/12.png)

###3.模板模板参数(以模板作为模板的参数)
即将一个模板作为另一个模板的参数，如下：
```
template<typename T, template<typename U>typename Container>
class template_template_test {
 private:
  Container<T> c;
};
```
模板的第一个参数是T类型，第二个参数是一个Container，他是一个可以指定一个U类型的变量。**下面是简单使用：**
```
template<typename T, template<typename U>typename Container>
class template_template_test {
 private:
  // Container本身就是一个模板，需要接受一个模板参数
  Container<T> c;
};
template<typename T>
class test {
 private:
  T t1;
};
int main() {
  template_template_test<std::string, test> mytest;
  return 0;
}
```
我们定义了一个模板类，并将其作为参数传入另一个参数为模板的模板类中。但是如果我们传入一个容器，会产生一些小问题：
```
template_template_test<std==string, std==list> mylist1;  // 这里编译会产生错误
```
如果我们将`string`和`list`传入到类`template_template_test`中，然后就会定义一个`list<string>`的变量，虽然这样看起来是可以的，但是因为list容器实质上是有第二参数的，虽然第二参数有默认的参数，正如我们平常使用的那样，只需要指定一个参数，但是在这里无法通过编译，因此，我们使用如下解决办法：
```
template<typename T>using Lst = std==list<T, std==allocator<T>>;  // 使用using定义一个类型的别名
// 一般stl中大多数默认第二参数都是allocator
```
接下来，我们可以正常将list传入模板类中：
```
template_template_test<std::string, LIST> mylist1;
```

接下来，我们参考stl模板的**原型**,比如vector：
```
template<typename E,typename Allocator=allocator<E> > 
class vector {
	...
}; 
```
之后就可以定义：
```
template<typename T,template<typename E,typename Allocator=allocator<E> >typename Container>class Grid {
 // 这里模板类Grid的模板行参有两个，分别为T和Container;其中Container是一个模板类，它有一个模板参数为E类型
 public: 
  Grid();
  //Omitted for brevity 
  Container<T>* mCells; 
}; 
```

模板模板参数的一般用法：
```
template<other params,...,template<Template TypeParams>typename ParameterName,other params,...> class A{
  //...
};
```
举例子，Grid的一个构造函数：
```
template<typename T,template<typename E,typename Allocator=allocator<E> >typename Container=vector> 
Grid<T,Container>::Grid(int inWidth,int inHeight): mWidth(inWidth),mHeight(inHeight) 
{ 
  mCells=new Container<T> [mWidth]; //注意此处Container<T>说明，由于mWidth是一个int类型数据，所以此处T的类型为int，所以实际上还是说明 Grid<int,vector<int> > 
  for(int i=0;i<mWidth;++i) 
    mCells[i].resize(mHeight); 
} 
```
使用的时候，与一般的函数没有什么区别：
```
Grid<int,vector> myGrid; 
myGrid.getElement(2,3); 
```
**注意：不要拘泥于它的语法实现，只要记住可以使用模板作为模板的一个参数。**
<mark><font color=#3a006f>TemplateDemo3.cc</font></mark>
```
[[include]]<iostream>
[[include]]<vector>
// 这里默认Container的类型为vector（如果我们没有指定Container的类型）
template<typename T, template<typename E, typename Allocator = std::allocator<E> >typename Container>class Grid{
public:
    Grid(int inWidth, int mHeight);
    Container<T>* mCells;  // 声名一个Container类型的指针
    void getElement();
private:
    int mWidth;
    int mHeight;
};
// 这里的Grid<t,Container>中Container其实就是一个模板模板形参
template<typename T, template<typename E, typename Allocator = std::allocator<E> >typename Container>
Grid<T, Container>::Grid(int inWidth, int inHeight) : mWidth(inWidth), mHeight(inHeight) {
    // 注意此处Container<T>说明，由于mWidth是一个int类型数据，所以此处T的类型为int，所以实际上还是说明 Grid<int,vector<int> >
    mCells = new Container<T> [mWidth];
    for (int i = 0; i < mWidth; ++i)
        mCells[i].resize(mHeight);
}
template<typename T, template<typename E, typename Allocator = std::allocator<E> >typename Container>
void Grid<T, Container>::getElement() {
    std==cout << "mWidth is : " << mWidth << " and mHeight is : " << mHeight << std==endl;
}
int main() {
    Grid<int, std::vector> myGrid(2, 3);
    myGrid.getElement();
    return 0;
}
```
![13](/assets/13.png)
###4.指针和引用模板参数
指针和引用模板参数必须指向**所有翻译单元中都可用的全局变量**。对于这些类型的变量，相应的术语就是<font color=red>带有外部连接的数据</font>。对此，我们使用extern声明即可。
```
template<typename T ,const T& EMPTY>
class Grid {
	//...
}; 
extern const int emptyInt=0; 
Grid<int,emptyInt> myIntGrid; 
```
