# c++知识点补充：
指针是一个变量，是用来指向内存地址的。一个程序运行时，所有和运行相关的物件都是需要加载到内存中，这就决定了程序运行时的任何物件都可以用指针来指向它。函数是存放在内存代码区域内的，它们同样有地址，因此同样可以用指针来存取函数，把这种指向函数入口地址的指针称为函数指针

##一、c++函数指针
###1.1函数指针：
####1.1什么是函数指针：
每个函数都占用一段内存单元，他们都有一个入口地址；指向函数入口地址的指针即函数指针。
####1.2函数指针语法：
`数据类型 (*指针变量名)(参数列表)`
####1.3说明：
1）注意区分如下两个语句：
`int (*p)(int a, int b)`  // p是一个指向函数的指针变量，所指函数的返回类行为整形
`int *p(int a, int b)`  // p是函数名，返回类型为整形指针

2）**指向函数的指针变量不是固定指向某一个函数，而只是定义了一个这样类型的变量，专门用来存放函数的入口地址；在程序中把哪一个函数的地址赋值给它，它就指向哪一个函数。**

3）在给函数指针变量赋值时，只需给出函数名，而不必给出参数：
<font color=#561847>注意：对于一个函数而言，函数名即这个函数在内存中的入口地址。</font>

如函数max的原型为：`int max(int x, int y)`; 指针p的定义为：`int (*p)(int a, int b)`; 则p = max;的作用是将函数max的入口地址赋给指针变量p。这时，p就是指向函数max的指针变量，也就是p和max都指向函数的开头。

4）定义一个函数指针并让其指向一个函数后，对函数的调用可以通过函数名调用，也可以通过函数指针调用(即用指向函数的指针变量调用)。
比如在上面定义过的函数指针p，可以有如下使用方式：
```
[[include]]<iostream>
// 声明一个函数指针
int (*p)(int, int);
int max(int a, int b) {
  return a > b ? a : b;
}
int main() {
  // 另函数指针指向一个函数(即指向函数的入口地址)
  p = max;
  // 两种用法相等
  std==cout << p(3, 5) << std==endl;
  std==cout << (*p)(7, 2) << std==endl;
  return 0;
}

```
结果：
```
5
7
```
5）函数指针只能指向函数的入口处，而不能指向函数中间的某一条命令。即：不能用*(p+1)来表示函数的下一条指令。

6） 函数指针变量常用的用途之一是把指针作为参数传递到其他函数。(即回调函数)
###1.2函数指针调用函数举例：
```
[[include]]<iostream>

int max(int, int);
int min(int, int);
int add(int, int);
// 调用函数指针的声明
int possess(int, int, int(*p)(int, int));

int main() {
  int a = 3;
  int b = 6;
  std::cout << "Max is : ";
  possess(a, b, max);
  std::cout << "Min is : ";
  possess(a, b, min);
  std::cout << "Sum is : ";
  possess(a, b, add);
  return 0;
}
int max(int a, int b) {
  return a > b ? a : b;
}
int min(int a, int b) {
  return a > b ? b : a;
}
int add(int a, int b) {
  return a + b;
}
int possess(int a, int b, int(*p)(int, int)) {
  std==cout << p(a, b) << std==endl;
}

```
输出结果：
```
Max is : 6
Min is : 3
Sum is : 9
```

##二、c++回调函数
<font color=red>回调函数就是一个通过函数指针调用的函数</font>。如果你把函数的指针（地址）作为参数传递给另一个函数，<font color=#45896>当这个指针被用来调用其所指向的函数时，我们就说这是回调函数。</font>**回调函数不是由该函数的实现方直接调用，而是在特定的事件或条件发生时由另外的一方调用的，用于对该事件或条件进行响应。**

###2.1回调函数机制：
1、定义一个函数（普通函数即可）；
2、将此函数的地址(即函数名)注册给调用者(即一个函数指针)；
3、特定的事件或条件发生时，调用者使用函数指针调用回调函数。


###2.2回调函数的使用方式：
1）调用函数地址：
```
[[include]]<iostream>

// 定义一个指向 int (int, int) 的函数指针别名为callback
typedef int (*callback)(int, int);
int add1(int, int, callback);
int add2(int, int);
int main() {
  int a = 5, b = 7;
  int res = add1(a, b, add2);
  std==cout << "返回结果： " << res << std==endl;
  return 0;
}
int add2(int a, int b) {
  return a + b;
}
int add1(int a, int b, callback p) {
  std==cout << "第一种使用方式" << p(a, b) << std==endl;
  std==cout << "第二种使用方式" << (*p)(a, b) << std==endl;
  return (*p)(a, b);
}

```
2）把回调函数和调用函数封装再调用
```
[[include]] <cstring>
[[include]] <iostream>
[[include]] <memory>
// 定义一个类型为 int (const void*, size_t, char*) 的函数指针别名为callback
typedef int (*callback)(const void* buffer, size_t size, char* p_out);

// 封装了一层，回调函数
int callBackFunc(const void* buffer, size_t size, char* p_out) {
  std==cout << "CallBackFunc" << std==endl;
  // 给p_out指向的内存赋初值(初始化操作)
  memset(p_out, 0x00, sizeof(char)*100);
  strcpy(p_out, "encoderCallback : this is string.");
  return 1;
}

void callFunc(callback consume_bytes, char* p_out) {
  std==cout << "CallFunction" << std==endl;
  const void* buffer = nullptr;
  // 函数指针用来调用其所指向的函数(回调函数)
  (*consume_bytes)(buffer, 0, p_out);
  // consume_bytes(buffer, 0, p_out);  // 同样的使用方式
}

int main(int argc, char* argv[]) {
  char p_out[100];
  // 令函数指针指向callBackFunc
  callFunc(callBackFunc, p_out);
  std==cout << p_out << std==endl;
  return 0;
}

```
**补充：**
c++中回调函数的用法：
我们经常会把一些耗时的操作放到线程中去执行，当任务执行完毕后就需要通知主线程，通知的方式有很多，在windows平台上可以使用消息机制，如果不想依赖平台API，让代码具有良好移植性，使用回调函数也是一种方法。

##三、c++变量的声明，定义以及初始化小结：
###3.1声明：
声明用于向程序表明变量的类型和名字，它告诉编译器“这个函数或变量在某处可找到，它的模样象什么”，但声明不一定引起内存的分配。定义也是声明（当定义变量的同时我们声明了它的类型和名字）
```
extern int x;  // 声明变量x，没有定义
std::size_t numDigits(int number); 	//函数声明
class widget;  //类的声明
template<typename T> class GraphNode;  //模板声明
```
###3.2定义：
变量名就是对相应的内存单元的命名，还可为变量指定初始值。
定义是说：“在这里建立变量”或“在这里建立函数”。它为名字分配存储空间。无论定义的是函数还是变量，编译器都要为它们在定义点分配存储空间。对于变量，编译器确定变量的大小，然后在内存中开辟空间来保存其数据，对于函数，编译器会生成代码，这些代码最终也要占用一定的内存。对于变量以及函数均只能够定义一次。
###3.3初始化：
初始化是给对象赋予初值的过程，初始化由构造函数执行。所谓`default`构造函数是一个可被调用而不带任何实际参数者，这样的构造函数要不没有参数，要不就是每个参数都有缺省值。

1）定义也是声明，`extern`声明不是定义，即不分配存储空间。`extern`告诉编译器变量在其他地方定义了。
```
extern int i; //声明，不是定义
int i; //声明，也是定义（但没有初始化）
int j = 1; // 声明，也是定义，同时进行初始化
```
2）如果声明有初始化式，就被当作定义，即使前面加了extern。只有当extern声明位于函数外部时，才可以被初始化。
```
extern double pi=3.1416; //定义
```
3.函数的声明和定义区别比较简单，带有{ }的就是定义，否则就是声明。
```
extern double max(double d1,double d2); //声明,此时extern可去掉
```
###3.4总结：
1）变量在使用前就要被定义或者声明。
2）在一个程序中，变量只能定义一次，却可以声明多次。
3）定义分配存储空间，而声明不会。

**补充：**
对于类而言，当类中定义自己的构造函数之后，由于默认无参构造被隐藏，所以定义一个类的实例的时候不能使用默认构造方式初始化类的实例，必须显示使用定义的构造函数初始化类的实例。

##四、typedef用法
**为现有类型创建一个新的名字(即别名)。typedef能隐藏笨拙的语法构造，平台相关的数据类型**
###4.1定义一种类型的别名：
typedef定义类型的别名不只是简单的宏替换。可以用作同时声明指针型或者数组类型的多个对象。

typedef + 指针：
```
char* pa, pb;    //不符合我们的意图，它只声明了一个指向字符变量和一个字符变量；

typedef char* PCHAR;   //一般用大写
PCHAR pa, pb;          //可行，同时声明了指向字符变量的指针
```
typedef + 数组：
```
// 可以不用如下面这样重复定义有 81 个字符元素的数组：
char line[81]; char text[81];

// 定义一个 typedef，每当要用到相同类型和大小的数组时，可以这样：
typedef char typearray[81];
typearray text, secondline;
```
###4.2简化struct结构体用法
在旧的c代码中，声明struct类型对象时，必须带上struct，即`struct 结构名 对象名`，如:
```
struct tagPOINT1 {
  int x;
  int y;
};

struct tagPOINT1 p1;  // 声明结构体类型变量p1
```
而在C++中，则可以直接写：结构名 对象名，即：`tagPOINT1 p1;`

同时，我们也可以给结构体定义一个别名：
```
typedef struct tagPOINT {
  int x;
  int y;
} POINT;  // POINT为struct结构体的一个别名

POINT p1;  // 这样就比原来的方式少写了一个struct，比较省事，尤其在大量使用的时候
```
###4.3定义平台无关类型
比如定义一个叫`REAL`的浮点类型：
在目标平台一上，让它表示最高精度的类型为：`typedef long double REAL;`
在不支持 long double 的平台二上，改为：`typedef double REAL;`
在连 double 都不支持的平台三上，改为：`typedef float REAL;`
也就是说，当跨平台时，只要改下 typedef 本身就行，不用对其他源码做任何修改。
标准库就广泛使用了这个技巧，比如size_t。
另外，因为typedef是定义了一种类型的新别名，不是简单的字符串替换，所以它比宏来得稳健（虽然用宏有时也可以完成以上的用途）。

###4.4为复杂的声明定义一个简单的别名
在原来的声明里逐步用别名替换一部分复杂声明，如此循环，**把带变量名的部分留到最后替换**，得到的就是原声明的最简化版
1.原声明：`int *(*a[5])(int, char*);`
变量名为a，直接用一个新别名pFun替换a就可以：
`typedef int *(*pFun)(int, char*);`
原声明的最简化版：
`pFun a[5];`
2.原声明：`void (*b[10]) (void (*)());`
变量名为b，先替换右边部分括号里的，`pFunParam`为别名一：
`typedef void (*pFunParam)();`
再替换左边的变量b，`pFunx`为别名二：
`typedef void (*pFunx)(pFunParam);`
原声明的最简化版：
`pFunx b[10];`
3.原声明： `doube(*)()(*e)[9]`;
变量名为e，先替换左边部分，pFuny为别名一：`typedef double(*pFuny)();`
再替换右边的变量e，pFunParamy为别名二:`typedef pFuny (*pFunParamy)[9];`
原声明的最简化版：
pFunParamy e;

###4.5 typedef与#define区别
####区别1：
通常讲，typedef要比#define要好，特别是在有指针的场合：
```
typedef char* pStr1;  // 定义char* 类型别名为pStr1
[[define]] pStr2 char *;  // 定义pStr2为char*,只是简单的字符替换
pStr1 s1, s2;
pStr2 s3, s4;
```
6 在上述的变量定义中，s1、s2、s3都被定义为`char *`，而s4则定义成了char，不是我们所预期的指针变量，根本原因就在于#define只是简单的字符串替换而typedef则是为一个类型起新名字。

####区别2：
下面的代码中编译器会报一个错误：
```
typedef char* pStr;
char string[4] = "abc";
const char * p1 = string;  // 底层const，指向常量的指针
const pStr p2 = string;
p1++;
p2++;
```
是p2++出错了。这个问题再一次提醒我们：`typedef`和`#define`不同，它不是简单的文本替换。上述代码中`const pStr p2`并不等于`const char * p2`。`const pStr p2`和`const int x`本质上没有区别，都是对变量进行只读限制，只不过此处变量p2的数据类型是我们自己定义的而不是系统固有类型而已。因此，`const pStr p2`的含义是：限定数据类型为`char *`的变量p2为只读，因此p2++错误。

###4.6 注意事项
1.对于4.4中复杂声明的理解可以用<font color = red>“右左法则”</font>：
**从变量名看起，先往右，再往左**，碰到一个完整的圆括号就调转阅读的方向；括号内分析完就跳出括号，还是按先右后左的顺序，如此循环，直到整个声明分析完。举例：

1.1`int (*func)(int *p);`
1）首先找到变量名func，外面有一对圆括号，而且左边是一个`*`号，这说明func是一个指针；
2）然后跳出这个圆括号，先看右边，又遇到圆括号，这说明`(*func)`是一个函数，所以func是一个指向这类函数的指针，即函数指针，这类函数具有int*类型的形参，返回值类型是int。

1.2.`int (*func[5])(int *);`
1）`func`右边是一个[]运算符，说明func是具有5个元素的数组；
2）`func`的左边有一个`*`，说明func的元素是指针（注意这里的`*`不是修饰func，而是修饰 func[5]的，原因是[]运算符优先级比`*`高，func先跟`[]`结合）。跳出这个括号，看右边，又遇到圆括号，说明func数组的元素是函数类型的指针，它指向的函数具有`int*`类型的形参，返回值类型为int。

也可以记住2个模式：
```
type (*)(....)  // 函数指针
type (*)[]  // 数组指针
```

2.typedef无法改变关键字的存储类型
typedef在语法上是一个存储类的关键字（如auto、extern、mutable、static、register等一样），虽然它并不真正影响对象的存储特性，如：
`typedef static int INT32; //不可行`
编译将失败，会提示“指定了一个以上的存储类”。

##五、虚函数、纯虚函数、抽象类以及虚基类
###5.1虚函数

###5.2纯虚函数

###5.3抽象类

###5.4虚基类

##六、c++的一些关键字
###6.1 mutable 关键字
mutable是为了突破const的限制而设置的。被mutable修饰的变量，将永远处于可变的状态，即使在一个const函数中。
一般来说，const函数是不可以修改类成员变量的，但是有时候我们需要知道该const函数调用的次数，那么就需要一个mutable关键字来修饰变量。（因为此时const函数对其余成员变量仍然是不可修改的）
```
class Person {
 public:
  Person(int age, string name) : age_(age) , name_(name) {}
  ~Person() {}
  int getAge() const{
    count_++;
    return age_;
  }
  int getAgeCallingTimes() const{
    return count_;
  }
 private:
  int age_;
  string name_;
  mutable int count_ = 0;  // 表示此变量可以一直修改
};
int main() {
  Person jack(12, "jack");
  for (int i = 0; i < 10; i++) {
    jack.getAge();
  }
  cout << "Calling Times : " << jack.getAgeCallingTimes();
}
```
