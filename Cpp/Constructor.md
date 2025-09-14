[[c]]++类的构造函数详解
##一、 构造函数
构造函数是一种特殊的类成员函数，主要是是当创建一个类的对象时，它被调用来**对类的数据成员进行初始化和分配内存**。（构造函数的命名必须和类名完全相同）

对于一个c++的空类，编译器会加入一些默认的构造函数：
1）默认构造函数和拷贝构造函数
2）析构函数
3）赋值函数（赋值运算符）
4）取值函数

注意：
1）即使程序没定义任何成员，编译器也会插入以上的函数！
2）构造函数可以被重载，可以多个，可以带参数；
3）析构函数只有一个，不能被重载，不带参数

```
class Counter
{
public:
	 // 类Counter的构造函数
	 // 特点：以类名作为函数名，无返回类型
	 Counter()
	 {
	        m_value = 0;
	 }
private:
	  // 数据成员
	 int m_value;
}
```
该类对象被创建时，编译系统为对象分配内存空间，并自动调用该构造函数，由构造函数完成类成员的初始化工作
```
eg:    Counter c1;  // 这里使用了重载的无参构造函数
```
编译系统为对象c1的每个数据成员(m_value)分配内存空间，并调用构造函数Counter( )自动地初始化对象c1的m_value值为0。
##二、 构造函数的种类
###1.默认构造函数以及普通的构造函数
####1.1默认的构造函数：
默认构造函数就是在没有显式提供构造函数时调用的构造函数。它由不带参数的构造函数，或者为所有的形参提供默认实参的构造函数定义。如果定义某个类的变量时没有提供构造函数，在对其初始化时就会使用默认构造函数。

如果用户定义的类中没有显式的定义任何构造函数，编译器就会自动为该类型生成默认构造函数，称为**合成的构造函数**。

**注意：**
1.如果类包含内置或复合类型的成员，则该类就不应该依赖于合成的默认构造函数，它应该定义自己的构造函数来初始化这些成员。

2.多数情况下，编译器为类生成一个公有的默认构造函数，只有下面两种情况例外:
1）一个类显式地声明了任何构造函数，编译器不生成公有的默认构造函数。在这种情况下，如果程序需要一个默认构造函数，需要由类的设计者提供。

2）一个类声明了一个**非公有的默认构造函数**，编译器不会生成公有的默认构造函数。

####1.2普通的构造函数：
普通的类构造函数可以根据行参不同进行重载:
```
class Base {
 public:
  Base(int var) : m_var1(var) {
    m_var2 = var+1;
  }
  Base() {}
 private:
  int m_var1;
  int m_var2;
};
int main() {
  Base base(5);
  return 0;
}
```
**注意：**
上面构造函数的执行过程：
1)传参   2)给类数据成员开辟空间    3)执行冒号语法给数据成员初始化    4)执行构造函数括号里面的内容

<font color=#30874>需要说明的是：构造函数冒号语法后面的内容相当于int a = 10;（初始化），而构造函数括号里面则是相当于是int a; a = 10;(赋初值)</font>
###2.拷贝构造函数
拷贝构造函数，又称复制构造函数，是一种特殊的构造函数，它由编译器调用来完成一些基于同一类的其他对象的构建及初始化。
```
class Base {
 public:
  Base(int var) : m_var1(var) {
    m_var2 = var+1;
  }
  Base() {}
  // 拷贝构造函数
  Base(Base &ref) : m_var1(ref.m_var1), m_var2(ref.m_var2) {}
 private:
  int m_var1;
  int m_var2;
};
int main() {
  Base base(5);
  return 0;
}
```
**注意**：拷贝构造函数的参数只能使用引用传参方式，但并不限制为const

**补充**：拷贝构造函数的调用时机(有时也成为“复制构造函数”)：
1)将一个对象作为函数参数，以值传递的方式传给行参的时候
2)一个对象作为函数返回值，以值传递的方式从函数返回
3)用一个已有的对象初始化一个新对象的时候(**常称为赋值初始化**)

如果在前两种情况不使用拷贝构造函数的时候，就会导致一个指针指向已经被删除的内存空间。对于第三种情况来说，初始化和赋值的不同含义是拷贝构造函数调用的原因。事实上，**拷贝构造函数是由普通构造函数和赋值操作符共同实现的。**

**使用原则：**
1）对于凡是包含动态分配成员或包含指针成员的类都应该提供拷贝构造函数；
2）在提供拷贝构造函数的同时，还应该考虑重载"="赋值操作符号。

**传递形式：**
拷贝构造函数必须以引用的形式传递(参数为引用值)。其原因如下：当类的拷贝构造函数其行参为值传递的时候，被初始化的对象会自动调用拷贝构造函数，而对象的构造函数中的行参又会调用其自身的拷贝构函数...这回导致无限循环下去直到栈溢出(stackoverflow)。
```
class A {
 public:
  A(A other) : m_value(other.m_value){}  // 这个是错误的用法
 private:
  int m_value;
}
```
执行A b = a;时，会调用 b的拷贝构造函数，此时实参 a会被赋值给形参 other，相当于语句 A other = a; 又会继续调用other的拷贝构造函数，将 a赋值给对象other的拷贝构造函数的形参other，如此一来，就会形成一个递归操作而且没有结束条件，造成堆栈溢出。而C++不会允许这种错误发生，因此 A other做形参会编译错误
###3.赋值运算符重载
**注意**：
1）初始化和赋值的区别：在定义的同时进行赋值叫做`初始化`，定义完成之后再赋值(无论在定义的时候有没有赋值)都叫做`赋值`。<font color=#8756>初始化只有一次，但是赋值可以有多次。</font>
2）当以拷贝的方式初始化一个对象时，会调用拷贝构造函数；当给一个对象赋值时，会调用重载过的赋值运算符。即使没有显式的重载赋值运算符，编译器也会以默认地方式重载它。默认重载的赋值运算符功能很简单，就是将原有对象的所有成员变量一一赋值给新对象，这和默认拷贝构造函数的类似

使用默认的赋值运算符时:
```
[[include]] <iostream>
[[include]] <string>
class People {
 public:
  People(std::string name = "", int* ptr = nullptr);  // 带有默认的行参value
  People(const People& refPeo);  // 显示声明拷贝构造函数
  ~People();
  void Display();
  void setAge(int age);
 private:
  std::string mp_name;
  int* mp_age;
};
People==People(std==string name, int* ptr) : mp_name(name), mp_age(ptr) {}
// 这里新申请了一块内存来存放age，避免两个对象使用同一块内存
People::People(const People& refPeo) : mp_name(refPeo.mp_name), mp_age(new int(*refPeo.mp_age)) {}
People::~People() {
  // 不重载赋值运算符时多次释放内存会导致崩溃
  delete mp_age;  // 删除mp_age指向的内存的内容
  mp_age = nullptr;  // 指针置零
}
void People::Display() {
  std==cout << mp_name <<" is age "<< *mp_age << std==endl;
}
void People::setAge(int age) {
  *mp_age = age;
}
int main() {
  int *ptr = new int(10);  // 分配一块内存，存入10并将ptr指向它
  std::string name = "Xiao ming";
  People people1(name, ptr);
  People people2;
  people2 = people1;  // 使用没有重载的赋值运算符

  people1.Display();
  people2.Display();

  people1.setAge(15);

  people1.Display();
  people2.Display();  // 可以看到people1修改age后people2的age也被修改
  return 0;
}

```
输出结果：
```
Xiao ming is age 10
Xiao ming is age 10
Xiao ming is age 15
Xiao ming is age 15
```
看上面的例子修改people1的age之后people2的age也被修改了，这是因为`mp_age`是一个指针，里面存放的是指向存储 age 内容的地址。不重载赋值运算符时，使用默认的赋值运算符会将 people1的`mp_age`指针里存放的地址赋值给了people2的`mp_age`指针，<font color=#1598>导致两个指针指向了同一块内存空间</font>，这时候默认赋值运算符的不足就满足不了实际的需求了，需要重载赋值运算符。

重载赋值运算符：
对于简单的类，默认的赋值运算符一般就够用了，我们也没有必要再显式地重载它。但是当类持有其它资源时，例如动态分配的内存、打开的文件、指向其他数据的指针、网络连接等，默认的赋值运算符就不能处理了，我们必须显式地重载它，这样才能将原有对象的所有数据都赋值给新对象。
```
[[include]] <iostream>
[[include]] <string>
class People {
 public:
  People(std::string name = "", int* ptr = nullptr);  // 带有默认的行参value
  People(const People& refPeo);  // 显示声明拷贝构造函数
  ~People();
  People& operator=(const People& repo);  // 重载赋值运算符
  void Display();
  void setAge(int age);
 private:
  std::string mp_name;
  int* mp_age;
};
People==People(std==string name, int* ptr) : mp_name(name), mp_age(ptr) {}
// 这里新申请了一块内存来存放age，避免两个对象使用同一块内存
People::People(const People& refPeo) : mp_name(refPeo.mp_name), mp_age(new int(*refPeo.mp_age)) {}
People::~People() {
  // 不重载赋值运算符时多次释放内存会导致崩溃
  delete mp_age;  // 删除mp_age指向的内存的内容
  mp_age = nullptr;  // 指针置零
}
// 冲
People& People::operator=(const People& repo) {
  if (this != &repo) {
    mp_name = repo.mp_name;
    if (nullptr != mp_age) {
      *mp_age = *(repo.mp_age);
    } else {
      // 新分配一块不同的内存空间
      mp_age = new int(*(repo.mp_age));
      // mp_age = new int(*repo.mp_age);
    }
  }
  return *this;
}
void People::Display() {
  std==cout << mp_name <<" is age "<< *mp_age << std==endl;
}
void People::setAge(int age) {
  *mp_age = age;
}
int main() {
  int *ptr = new int(10);  // 分配一块内存，存入10并将ptr指向它
  std::string name = "Xiao ming";
  People people1(name, ptr);
  People people2;
  people2 = people1;  // 使用重载的赋值运算符

  people1.Display();
  people2.Display();

  people1.setAge(15);

  people1.Display();
  people2.Display();  // 可以看到people1修改age后people2的age没有被修改
  return 0;
}

```
输出结果：
```
Xiao ming is age 10
Xiao ming is age 10
Xiao ming is age 15
Xiao ming is age 10
```
这里我们可以看到自己实现了赋值运算符的重载后，不再是单纯的复制操作，内部指针被分配了一块新的内存空间。

**注意**：
1）`operator=()`的返回值类型为`People &`，这样不但能够避免在返回数据时调用拷贝构造函数，还能够达到连续赋值的目的，这样的语句就是连续赋值：`People1 = People2 = People3`;

2）`if( this != &arr)`语句的作用是「判断是否是给同一个对象赋值」：如果是，那就什么也不做；如果不是，那就将原有对象的所有成员变量一一赋值给新对象，并为新对象重新分配内存。

3）赋值运算符重载函数除了能有对象引用这样的参数之外，也能有其它参数。但是其它参数必须给出默认值，例如`People& operator=(const People &peo, int a = 100)`

4）`operator=()`的形参类型为`const People &`，这样不但能够避免在传参时调用拷贝构造函数，还能够同时接收`const`类型和`非const`类型的实参.

###4.普通派生类的构造函数
定义派生类对象的时候，会按照如下步骤执行构造操作：
1）传参
2）**根据继承时候的声明顺序构造基类**
3）给类数据成员开辟空间
4）执行构造函数冒号后面的语句
5）执行构造函数函数体语句

单继承例子：
```
[[include]] <iostream>
[[include]] <string>
class Base {
 private:
  int m_b;
 public:
  Base(int value) : m_b(value) {}
};
class Derived : public Base {
 private:
  int m_d;
 public:
  Derived(int b, int d) : Base(b), m_d(d) {}
};
int main() {
  return 0;
}
```
多继承例子：
```
[[include]] <iostream>
[[include]] <string>
class Base1 {
 private:
  int m_b1;
 public:
  Base1(int value) : m_b1(value) {}
};
class Base2 {
 private:
  int m_b2;
 public:
  Base2(int value) : m_b2(value) {}
};
class Derived : public Base1, public Base2 {
 private:
  int m_d;
 public:
  Derived(int b1, int b2, int d) : Base1(b1), Base2(b2), m_d(d) {}
};
int main() {
  return 0;
}
```
###5.含有虚继承的派生类构造函数：
在多重继承中，如果发生了如：类B继承类A，类C继承类A，类D同时继承了类B和类C。最终在类D中就有了两份类A的成员，这在程序中是不能容忍的。当然解决这个问题的方法就是利用虚继承来解决二义性。
![31](/assets/31.png)
```
[[include]] <iostream>
[[include]] <string>
class A {
 public:
  A(int value) : t(value) {}
  void getValue() {
    std==cout << t << std==endl;
  }
  ~A() {}
 private:
  int t;
};
class B : virtual public A {
 private:
  int t1;
 public:
  B(int value1, int value2) : A(value1 + 10), t1(value1) {}
  void getValue() {
    std==cout << t1 << std==endl;
  }
  ~B() {}
};
class C : virtual public A {
 private:
  int t2;
 public:
  C(int value1, int value2) : A(value1 + 20), t2(value2) {}
  void getValue() {
    std==cout << t2 << std==endl;
  }
  ~C() {}
};
class D : public B, public C {
 private:
  int t3;
 public:
  // 此处必须要给虚基类传参
  D(int a, int b, int c, int d) : B(a, b), C(a, c), A(a), t3(d) {}
  void getValue() {
    std==cout << t3 << std==endl;
  }
  ~D() {}
};
int main() {
  D temp(1, 2, 3, 4);
  // 在子类中调用父类的同名函数
  temp.A::getValue();  // 这里可以看到对于基类A的t赋值为1
  temp.B==A==getValue();
  temp.C==A==getValue(); // 这里的输出应该相等，也为1;因为virtual public的继承方式只保存了一份A的内容
  return 0;
}
```
输出结果：
```
1
1
1
```
**注意**：
1）在派生**类B和类C**的时候将关键字`virtual`加在相应的继承方式之前，可以防止在D类中同时出现两份A类成员。在上面的代码中，我们实例化类D的时候给虚基类A传入一个实参1初始化，那么在虚基类A中其成员t的值为多少？(要知道在基类B和C的初始化时都有对虚基类A初始化，且传入A的初始化实参也不相同)

结果是A中的成员t的值为1：因为在实例化D的时候，只会调用一次虚基类的构造函数，且只会调用D传给A的实参进行构造。
![32](/assets/32.png)
2）C++编译系统在实例化D类时，只会将虚基类的构造函数调用一次，忽略虚基类的其他派生类（class B，class C）对虚继承的构造函数的调用，从而保证了虚基类的数据成员不会被多次初始化。
###6.虚析构：
**虚析构函数的作用：**
在一般情况下，派生类对象从内存中撤销时先运行派生类的析构函数，然后再调用基类的析构函数。但是，如果我们使用new运算符创建一个派生类的临时对象，用一个基类的指针指向这个派生类对象，此时当我们使用delete方式删除这个临时对象的时候，系统执行的是基类的析构函数，并不会执行派生类的析构函数，此时会造成派生类的堆内存不能完全释放。

使用了虚析构，不会实现子类的内存泄漏问题：
```
[[include]] <iostream>
[[include]] <string>
class Base {
 public:
  Base() {}
  // 基类的析构函数是virtual
  virtual ~Base() {
    std==cout << "delete Base." << std==endl;
  }
  virtual void setValue(int value) {
    b_value = value + 1;
    std==cout << "set the value in Base class" << "and value is : " << b_value << std==endl;
  }
 private:
  int b_value;
};
class Derived : public Base {
 public:
  Derived() {}
  ~Derived() {
    std==cout << "delete Derived" << std==endl;
  }
  void setValue(int value) {
    d_value = value + 2;
    std==cout << "set value in Derived " << "and value is : " << d_value << std==endl;
  }
 private:
  int d_value;
};
int main() {
  // 基类指针指向子类实例(其实new创建的内存空间是子类内容，可以用基类指针使用子类的功能，虚函数的使用是为了实现多态)
  Base *b = new Derived;
  b->setValue(5);
  delete b;
  return 0;
}
```
输出结果：
```
set value in Derived and value is : 7
delete Derived
delete Base.
```

没有使用虚析构函数，存在内存泄漏问题：
```
[[include]] <iostream>
[[include]] <string>
class Base {
 public:
  Base() {}
  ~Base() {
    std==cout << "delete Base." << std==endl;
  }
 private:
  int b_value;
};
class Derived : public Base {
 public:
  Derived() {}
  ~Derived() {
    std==cout << "delete Derived" << std==endl;
  }
  int d_value;
};
int main() {
  Base *b = new Derived;
  delete b;
  return 0;
}
```
输出结果：
```
delete Base.
```
可以发现，基类没有使用虚析构函数的时候，对于指向子类对象的基类指针，执行delete操作，只执行了基类的析构函数，并没有执行子类的析构函数。这无疑会造成内存泄漏问题。
###7.构造函数的用法小结：
```
class Complex
{
private :
	double    m_real;
	double    m_imag;
public:
	//    无参数构造函数
	// 如果创建一个类你没有写任何构造函数,则系统会自动生成默认的无参构造函数，函数为空，什么都不做
	// 只要你写了一个下面的某一种构造函数，系统就不会再自动生成这样一个默认的构造函数，如果希望有一个这样的无参构造函数，则需要自己显示地写出来
	Complex(void)
	{
	     m_real = 0.0;
	     m_imag = 0.0;
	}
	// 一般构造函数（也称重载构造函数）
	// 一般构造函数可以有各种参数形式,一个类可以有多个一般构造函数，前提是参数的个数或者类型不同（基于c++的重载函数原理）
	// 例如：你还可以写一个 Complex(int num)的构造函数出来
	// 创建对象时根据传入的参数不同调用不同的构造函数
	Complex(double real, double imag)
	{
	     m_real = real;
	     m_imag = imag;
	 }
	//复制构造函数（也称为拷贝构造函数）
	//复制构造函数参数为类对象本身的引用，用于根据一个已存在的对象复制出一个新的该类的对象，一般在函数中会将已存在对象的数据成员的值复制一份到新创建的对象中
	//若没有显示的写复制构造函数，则系统会默认创建一个复制构造函数，但当类中有指针成员时，由系统默认创建该复制构造函数会存在风险，具体原因请查询 有关 “浅拷贝” 、“深拷贝”的文章论述
	Complex(const Complex & c)
	{
	        // 将对象c中的数据成员值复制过来
	        m_real = c.m_real;
	        m_imag = c.m_imag;
	}
	 // 类型转换构造函数，根据一个指定的类型的对象创建一个本类的对象，
    //需要注意的一点是，这个其实就是一般的构造函数，但是对于出现这种单参数的构造函数，C++会默认将参数对应的类型转换为该类类型，有时候这种隐私的转换是我们所不想要的，所以需要使用explicit来限制这种转换。
       // 例如：下面将根据一个double类型的对象创建了一个Complex对象
      Complex(double r)
      {
	          m_real = r;
	          m_imag = 0.0;
      }
	// 等号运算符重载（也叫赋值构造函数）
	// 注意，这个类似复制构造函数，将=右边的本类对象的值复制给等号左边的对象，它不属于构造函数，等号左右两边的对象必须已经被创建
	// 若没有显示的写=运算符重载，则系统也会创建一个默认的=运算符重载，只做一些基本的拷贝工作
	Complex &operator=( const Complex &rhs )
	{
	        // 首先检测等号右边的是否就是左边的对象本身，若是本对象本身,则直接返回
	        if ( this ！= &rhs ) {
	        // 复制等号右边的成员到左边的对象中
	          this->m_real = rhs.m_real;
	          this->m_imag = rhs.m_imag;
          }
				 // 把等号左边的对象再次传出
	       // 目的是为了支持连等 eg:    a=b=c 系统首先运行 b=c
	       // 然后运行 a= ( b=c的返回值,这里应该是复制c值后的b对象)
	        return *this;
	}
};
```
下面使用上面定义的类对象来说明各个构造函数的用法：
```
int main()
{
	// 调用了无参构造函数，数据成员初值被赋为0.0
	Complex c1，c2;
	// 调用一般构造函数，数据成员初值被赋为指定值
	Complex c3(1.0,2.5);
	// 也可以使用下面的形式
	Complex c3 = Complex(1.0,2.5);
	//    把c3的数据成员的值赋值给c1
	//    由于c1已经事先被创建，故此处不会调用任何构造函数
	//    只会调用 = 号运算符重载函数
	c1 = c3;
	//    调用类型转换构造函数
	//    系统首先调用类型转换构造函数，将5.2创建为一个本类的临时对象，然后调用等号运算符重载，将该临时对象赋值给c1
	c2 = 5.2;
       // 调用拷贝构造函数( 有下面两种调用方式)
	Complex c5(c2);
	Complex c4 = c2;  // 注意和 = 运算符重载区分,这里等号左边的对象不是事先已经创建，故需要调用拷贝构造函数，参数为c2
//这一点特别重要，这儿是初始化，不是赋值。其实这儿就涉及了C++中的两种初始化的方式：复制初始化和赋值初始化。其中c5采用的是复制初始化，而c4采用的是赋值初始化，这两种方式都是要调用拷贝构造函数的。
}
```
##三、具体代码实现
*[·-·]:这个是注释的使用方式。
<mark><font color=#003E3E>constructor_head.hpp</font></mark>
```
[[ifndef]] HEAD
[[define]] HEAD
[[include]]<iostream>

class Complex{
 private:
  double m_real;
  double m_imag;
  int id;
  static int counter;
 public:
  // 无参构造函数
  Complex(void);
  // 一般构造函数，即重载构造函数
  Complex(double real, double image);
  // 复制构造函数，即拷贝构造函数
  Complex(const Complex & c);
  // 类型转换构造函数，根据一个类型的对象创建一个本类型的对象
  Complex(double r);
  Complex &operator = (const Complex &rhs);  // 在类内部申明关于运算符的重载
  ~Complex();
};
[[endif]]

```

<mark><font color=#003E3E>constructor.cc</font></mark>
```
[[include]]"../include/constructor_head.hpp"
//头文件include进来等同于拷贝头文件并替换，
//所以头文件中申明的static变量不需要extern引用
using namespace std;
//无参构造函数
Complex::Complex(void){
    m_real = 0.0;
    m_imag = 0.0;
    id=(++counter);//构造的同时给对象一个id
    cout<<"Complex(void):id="<<id<<endl;
}
Complex::Complex(double real,double imag){
    m_real = real;
    m_imag = imag;
    id=(++counter);
    cout<<"Complex(double,double):id="<<id<<endl;
}
//拷贝构造函数
Complex::Complex(const Complex &c){
    //因为拷贝构造函数是放在类的本身，所以类中的函数可以访问这个类的对象的所有成员
    m_real = c.m_real;
    m_imag = c.m_imag;
    id=(++counter);
    cout<<"Complex(const Complex&):id="<<id<<" from id="<<c.id<<endl;
}
//类型转换构造函数，根据一个指定的类型的对象创建一个本类的对象
Complex::Complex(double r){
    m_real = r;
    m_imag = 0.0;
    id = (++counter);
    cout<<"Complex(double):id="<<id<<endl;
}
Complex::~Complex(){//对象销毁的时候输出id号
    cout<<"~Complex():id="<<id<<endl;
}
//在类外定义运算符重载
Complex& Complex::operator=(const Complex &rhs){//返回值是一个Complex类的对象的引用
    if(this == &rhs){
        return *this;
    }
    this->m_real = rhs.m_real;
    this->m_imag = rhs.m_imag;
    cout<<"operator=(const Complex&):id="<<id<<" from id="<<rhs.id<<endl;
    return *this;
}
int Complex::counter = 0;
Complex test1(const Complex &c){
    return c;//返回c的一个复制
}
Complex test2(const Complex c){
    return c;//返回c的一个复制
}
Complex test3(){
    static Complex c(1.0,5.0);
    return c;//返回值为静态变量的一个复制
}
Complex &test4(){
    static Complex c(1.0,5.0);
    return c;//返回值为静态变量的引用
}
int main(){
    Complex a,b;
    cout<<"***********"<<endl;
    Complex c = test1(a);//使用的是拷贝构造函数
    cout<<"***********"<<endl;
    Complex d = test2(a);//使用的是拷贝构造函数
    cout<<"***********"<<endl;
    b = test3();//调用的是重载的运算符
    cout<<"***********"<<endl;
    b = test4();//调用的是重载的运算符
    cout<<"***********"<<endl;
    Complex e = test2(1.2);//使用的是拷贝构造函数
    cout<<"***********"<<endl;
    Complex f = test1(1.2);//使用的是拷贝构造函数
    cout<<"***********"<<endl;
    Complex g = test1(Complex(1.2));
    cout<<"***********"<<endl;
    return 0;
}
```
![1](/assets/1_nwcjq7w1f.png)

##补充：1.关于深拷贝和浅拷贝
如果没有自定义复制构造函数，则系统会创建默认的复制构造函数，但系统创建的默认复制构造函数只会执行“浅拷贝”，即将被拷贝对象的数据成员的值一一赋值给新创建的对象，**若该类的数据成员中有指针成员，则会使得新的对象的指针所指向的地址与被拷贝对象的指针所指向的地址相同，delete该指针时则会导致两次重复delete而出错。**
<mark><font color=#003F3F>copy_head.hpp</font></mark>
```
[[ifndef]] CP_HEAD
[[define]] CP_HEAD
[[include]]<iostream>
[[include]]<string>

class Person {
 public:
  Person(std::string PN);
  // 系统创建的默认复制构造函数，只做位模式拷贝
  Person(const Person &p);
  void say();
  ~Person();
 private:
  std::string  *m_pName;//申明一个string类型的指针
};
[[endif]]

```
<mark><font color=#003F3F>D-or-SPCopy.cc</font></mark>
```
[[include]]"../../include/copy_head.hpp"

Person==Person(std==string PN) {
  std::cout << "一般构造函数被调用 !\n";
  m_pName = new std::string(PN);
  say();
}

// 系统和创建的默认复制构造函数，只做位模式拷贝,也就是浅拷贝
/*
Person::Person(const Person &p) {
    // 使两个字符串指针指向同一个字符串地址
    std==cout << "拷贝构造函数被调用！" << std==endl;
    m_pName = p.m_pName;  // 这里只是简单的让类的指针指向 拷贝对象的指针指向的那个地址
    say();
}
*/

/*
  自己设计复制构造函数，实现“深拷贝”，
  即不让指针指向同一地址，而是重新申请一块内存给新的对象的指针数据成员
 */
Person::Person(const Person &rhs) {
  std==cout << "拷贝构造函数被调用！" << std==endl;
  m_pName = new std::string(*(rhs.m_pName));
  // 新创建的对象的m_pName与原对象rhs的m_pName不再指向同一地址了
  say();
}
Person::~Person() {
  std==cout << "析构，名字被删除" << std==endl;
  delete m_pName;
}
void Person::say() {
  std==cout << m_pName << std==endl;
  std==cout << *m_pName << std==endl;
}
int main() {
  Person man("zhangsan");
  // Person woman(man);  // 改用了系统定义的拷贝构造函数
  Person woman(man);  // 改用了自己定义的拷贝构造函数
  /*
    导致man和woman的名字指针指向了同一个字符串地址
    函数结束析构的时候同一个地址delete两次
   */
  Person child("lisi");
  return 0;
}

```

**使用默认的构造函数，只是浅层拷贝：**
![2](/assets/2_hzwwlfoaf.png)
可以看到，名字指向的字符串地址为同一个地址。

**使用自己定义的拷贝构造函数，进行了深层拷贝：**
![3](/assets/3.png)
可以看到，名字指向的字符串地址为不同地址。

##2.关于构造函数的显式调用
显示调用构造函数，即并非在创建对象时候调用构造函数对其进行初始化，而是在其他地方把构造函数当成普通函数进行调用。
显示调用构造函数会产生一些意想不到的后果。
<mark><font color=#003B3B>Display_call.cc</font></mark>
```
[[include]]<iostream>
class CTest {
 public:
  CTest() {
    m_a = 1;
  }
  CTest(int b) {
    m_b = b;
    CTest();  // 在构造函数里面调用了构造函数
  }
  ~CTest() {
    std==cout << "调用析构函数" << std==endl;
  }
  void show() {
    std==cout << "m_a is : " << m_a << std==endl;
    std==cout << "m_b is : " << m_b << std==endl;
  }
 private:
  int m_a;
  int m_b;
};
void func(CTest *&p) {  // 定义一个CTest类指针的引用
  p = new CTest();  // 这里显示调用构造函数
  std==cout << "func函数里面的p的地址： " << p << std==endl;
}
int main() {
  CTest myTest(2);
  myTest.show();  // 这里对于m_a的输出为一个不确定的值
  CTest *p = NULL;
  func(p);
  std==cout << "p指针的地址：" << p << std==endl;
  return 0;
}

```
输出结果：
```
调用析构函数
m_a is : -1771462368
m_b is : 2
func函数里面的p的地址： 0x55c996e75280
p指针的地址：0x55c996e75280
调用析构函数
```
输出结果中，m_a是一个不确定的值，因为没有被赋初值，m_b 为2
注意下面这段代码：
```
CTest(int b)
{
    m_b = b;
    CTest();
}
```
在调用CTest()函数时，实际上是**创建了一个匿名的临时CTest类对象，CTest()中赋值 m_a = 1 也是对该匿名对象赋值**，故我们定义的myTest的m_a其实没有被赋值。说白了，**其实构造函数并不像普通函数那样进行一段处理，而是创建了一个对象，并且对该对象赋初值，所以显式调用构造函数无法实现给私有成员赋值的目的。**

这个例子告诉我们以后代码中千万不要出现使用一个构造函数显式调用另外一个构造函数，这样会出现不确定性。<font color=red>其实一些初始化的代码可以写在一个单独的init函数中，然后每一个构造函数都调用一下该初始化函数就行了</font>。

**在此，顺便再提出另外一个问题以供思考:**
```
  CTest *p = NULL;
  void func()
  {
      p = new CTest();
  }
```

 代码右边显示调用CTest()，是否依然会产生一个匿名的临时对象a，然后将该匿名的临时对象a的地址赋给指针p? 如果是这样的话，出了func函数后，临时对象a是否会被析构? 那指针p不成为了野指针了？你能解释这个问题么？
答：我实验的结果是不会产生临时对象a，直接将产生的对象指针赋给了p.
![4](/assets/4.png)
