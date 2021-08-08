#c++ 命名空间初步了解
##命名空间初步了解：
###简单入门：
C++语言引入命名空间（Namespace）这一概念主要是为了避免命名冲突，其关键字为 namespace。

科技发展到如今，一个系统通常都不会仅由一个人来开发完成，不同的人开发同一个系统，不可避免地会出现变量或函数的命名冲突，当所有人的代码测试通过，没有问题时，将所有人的代码结合到一起，因为变量或函数重名而导致的问题将会造成一定的混乱，例如：

    int flag = 1; //小李声明的变量
    // …… //中间间隔若干行代码
    bool flag = true; //小韩声明的变量


注意：此例仅为解释命名空间所用，在公司的系统开发中并非如此中所述，完全仅靠命名空间来解决命名冲突的。

如上所示，因为个人习惯不同，小李喜欢声明int型变量用于逻辑判断，而小韩则更喜欢采用bool类型变量。但两个声明放到同一个函数中的时候，很明显编译器会提示出flag变量重新定义的错误。这种问题若不加以处理是无法编译通过的。

可以使用命名空间解决类似上面的命名冲突问题，例如：

    namespace Li{ //小李的变量声明
    int flag = 1;
    }
    namespace Han{ //小韩的变量声明
    bool flag = true;
    }


小李与小韩各自定义了以自己姓氏为名的命名空间，此时将小李与小韩的flag变量定义再置于同一个函数体中，则不会有任何问题，当然在使用这两个变量的时候需要指明所采用的是哪一个命名空间中的flag变量。

指定所使用的变量时需要用到“::”操作符，**“::”操作符是域解析操作符。**例如：

    Li::flag = 0; //使用小李定义的变量flag
    Han::flag = false; //使用小韩定义的变量flag


我们已经定义了两个命名空间 Li 和 Han，并在其中各自声明flag变量，使用的时候则需要分别用域解析操作符指明此时用的flag变量是谁定义出来的flag变量，是小韩还是小李定义的。

除了直接使用域解析操作符，还可以采用using声明（using declaration），例如：

    using Li::flag;
    flag = 0; //使用小李定义的变量flag
    Han::flag = false; //使用小韩定义的变量flag

**在代码的开头用using声明了Li::flag，其含义是using声明以后的程序中如果出现未指明的flag时，则使用Li::flag，**但是若要使用小韩定义的flag，则仍需要Han::flag。
<font color="#FF0000">using声明不仅仅可以针对命名空间中的一个变量，也可以用于声明整个命名空间(**即释放整个命名空间**)</font>，例如：

    using namespace Li;
    flag = 0; //使用小李定义的变量flag
    Han::flag = false; //使用小韩定义的变量flag

**如果命名空间Li中还定义了其他的变量，则同样具有flag变量的效果，在using声明后，若出现未具体指定命名空间的命名冲突变量，则*默认采用Li命名空间中的变量*。**

**命名空间内部不仅可以声明或定义变量，对于其它能在命名空间以外声明或定义的实体，同样也都能在命名空间内部进行声明或定义，例如变量的声明或定义、函数的声明或定义、类的申明和定义、typedef等都可以出现在命名空间中。**
###命名空间可以不连续：
命名空间可以定义在几个不同的部分，如下：
```
namespace nsp{
//相关申明
......
}
```

**此处的用法既可能是定义了一个名为nsp的新命名空间，也可能是为已存在的命名空间添加一些新成员。**

即：你引用了一个头文件，包含一个已经定义的命名空间，但是你可以在当前文件中继续定义一个同名的命名空间，在其中进行的声明即对原来命名空间声明的补充。
<font color="#AG1234">**注：**你在当前文件下对头文件命名空间的补充只具有当前可用性，并不会对头文件中命名空间造成影响。</font>

头文件中定义了一个命名空间：
<mark><font color=#003B3B>namespace.hpp</font></mark>
```
#ifndef NAME_SPACE
#define NAME_SPACE
//命名空间的定义
namespace CPlusPlus_Primier{
    int parameter1 = 1;
    int parameter2 = 2;
    class Query{
        public:
            Query(void){
                number = parameter1;
        }
        void show(){
            std::cout<<"the number is: "<<number<<std::endl;
        }
        int getnum(){
            return this->number;
        }
        private:
            int number;
    };
    //在命名空间中定义另一个命名空间，即命名空间的嵌套
    namespace SubNamespace{
        class SubQuery{
            public:
                SubQuery(void){//内嵌作用域可以直接使用上级命名空间中的类，
                    Query q = Query();
                    Subnumber = q.getnum();
                }
            void show(){
                std::cout<<"the Subnumber is: "<<Subnumber<<std::endl;
            }
            private:
                int Subnumber;
        };
    }
}
#endif
```
在当前.cc文件中对头文件包含进来的namespace进行补充，添加了一个变量c并在main函数中输出：
<mark><font color=#003B3B>namespace.cc</font></mark>
```
#include<iostream>
#include<string>
#include"../include/name_space.hpp"
namespace CPlusPlus_Primier{
    int c=67;
}
int main(){
    std::cout<<"the new paramrter in namespace is :"<<CPlusPlus_Primier::c<<std::endl;
    return 0;
}
```
![7](/assets/7.png)
另外一个文件试图访问上一个文件中对头文件进行的修改会报错：
<mark><font color=#003B3B>namespace1.cc</font></mark>
```
#include<iostream>
#include<string>
#include"../include/name_space.hpp"
int main(){
    std::cout<<"the new paramrter in namespace is :"<<CPlusPlus_Primier::c<<std::endl;
    return 0;
}
```
![8](/assets/8.png)
###命名空间的使用：
####1)初步：
下面我们来看一个简单的C++程序的示例：
```
#include<iostream>
using namespace std;
int main() {
  cout<<"hello world!"<<endl;
  return 0;
}

```
这是一个简单的C++程序hello world示例，在程序中采用了using声明命名空间std，using namespace std; 这一语句涵盖了std命名空间中的所有标识符，而该命名空间包含C++所有标准库。
**头文件iostream文件中定义的所有变量、函数等都位于std命名空间中，**每次使用iostream中的变量或函数都需要在前面加上std::是非常麻烦的一件事，为此可直接用using声明将std中的所有变量或函数等都声明出来。

如果不用using namespace std;这一语句，则程序应该像下面这样：
```
#include<iostream>
int main() {
  std::cout << "hello world!" << std::endl;
  return 0;
}

```
这样看起来是相当麻烦的，如果在某次使用iostream中变量或函数时漏掉std则会导致为定义标识符错误。

**C++语言是以C语言为基础的，它继承了所有的C语言函数库，但C++对这些标准库都重新命名了。标准C头文件（如math.h）重命名为cmath，去掉头文件的.h，并在前面加上c。因此在C++中如需使用math.h头文件则可以按照如下方式使用:**
```
#include<cmath>
using namespace std;
```
namespace 头文件的代码
```
#ifndef NAMESPACE_H
#define NAMESPACE_H
namespace Li {  // 小李的变量声明
  int flag = 1;
}
namespace Han {  // 小韩的变量声明
  bool flag = true;
}
#endif  // NAMESPACE_H

```
主函数的代码
```
#include <iostream>
#include "namespace.h"
using namespace std;
using namespace Li;

int main(int argc, char *argv[]) {
  cout << Li::flag << endl;
  Li::flag = 9;
  cout << Li::flag << endl;
  return 0;
}

```
输出的结果是
```
1
9
```

####2)进阶使用：
每个命名空间都是一个作用域：和其它作用域类似，命名空间中的每个名字都必须表示该空间内的唯一实体。因为不同命名空间的作用域不同，所以在不同命名空间内可以有相同名字的成员。
定义在某个命名空间中的名字可以被该命名空间内的其它成员直接访问，也可以被这些成员内嵌作用域中的任何单位访问。位于该命名空间之外的代码则必须明确指出所用的名字属于哪个命名空间。
<mark><font color=#003B3B>namespace.cc</font></mark>
```
#include<iostream>
#include<string>
#include"../include/name_space.hpp"
int main() {
  CPlusPlus_Primier::Query Q = CPlusPlus_Primier::Query();
  Q.show();
  CPlusPlus_Primier::SubNamespace::SubQuery SQ = CPlusPlus_Primier::SubNamespace::SubQuery();
  SQ.show();
  return 0;
}

```
<mark><font color=#003B3B>namespace.hpp</font></mark>
```
#ifndef NAME_SPACE
#define NAME_SPACE
//命名空间的定义
namespace CPlusPlus_Primier{
    int parameter1 = 1;
    int parameter2 = 2;
    class Query{
        public:
            Query(void){
                number = parameter1;
        }
        void show(){
            std::cout<<"the number is: "<<number<<std::endl;
        }
        int getnum(){
            return this->number;
        }
        private:
            int number;
    };
    //在命名空间中定义另一个命名空间，即命名空间的嵌套
    namespace SubNamespace{
        class SubQuery{
            public:
                SubQuery(void){//内嵌作用域可以直接使用上级命名空间中的类，
                    Query q = Query();
                    Subnumber = q.getnum();
                }
            void show(){
                std::cout<<"the Subnumber is: "<<Subnumber<<std::endl;
            }
            private:
                int Subnumber;
        };
    }
}
#endif
```
![6](/assets/6.png)
