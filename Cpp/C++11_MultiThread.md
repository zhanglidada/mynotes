#C++11 多线程详解
说到多线程编程，那么就不得不提并行和并发，多线程是实现并发（并行）的一种手段。并行是指两个或多个独立的操作同时进行。注意这里是同时进行，在一个时间段内执行多个操作，区别于并发。在单核时代，多个线程是并发的，在一个时间段内轮流执行；在多核时代，多个线程可以实现真正的并行，在多核上真正独立的并行执行。例如现在常见的4核4线程可以并行4个线程；4核8线程则使用了超线程技术，把一个物理核模拟为2个逻辑核心，可以并行8个线程。
##一、并发编程方法：
有两种并发编程方法：多线程和多进程
###1.1多进程并发：
使用多进程并发是将一个应用程序划分为多个独立的进程（每个进程只有一个线程），这些独立的进程间可以互相通信，共同完成任务。由于操作系统对进程提供了大量的保护机制，以避免一个进程修改了另一个进程的数据，<font color=red>使用多进程比多线程更容易写出安全的代码</font>，但这也造就了多进程并发的两个缺点：

**1). 在进程间的通信，无论是使用信号、套接字，还是文件、管道等方式，其使用要么比较复杂，要么就是速度较慢或者两者兼而有之。**
**2). 运行多个线程的开销很大，操作系统要分配很多的资源来对这些进程进行管理。**

由于多个进程并发完成同一个任务时，不可避免的是：操作同一个数据和进程间的相互通信，上述的两个缺点也就决定了多进程的并发不是一个好的选择。

###1.2多线程并发：
多线程并发指的是在同一个进程中执行多个线程。<font color=red>线程是轻量级的进程，每个线程可以独立的运行不同的指令序列，但是线程不独立的拥有资源，依赖于创建它的进程而存在</font>。也就是说，同一进程中的多个线程共享相同的地址空间，可以访问进程中的大部分数据，指针和引用可以在线程间进行传递。这样，同一进程内的多个线程能够很方便的进行数据共享以及通信，也就比进程更适用于并发操作。由于缺少操作系统提供的保护机制，在多线程共享数据及通信时，就需要程序员做更多的工作以保证对共享数据段的操作是以预想的操作顺序进行的，并且要极力的避免死锁(deadlock)。

##二、C++线程初步使用：
C++11的标准库中提供了多线程库，使用时需要`#include<thread>`头文件，该头文件主要包含了对线程的管理类`std::thread`以及其他管理线程相关的类。


<summary><font color=#600030><mark>thread_Test.hpp</mark></font></summary>

```
#ifndef HEAD_H
#define HEAD_H
#include<iostream>
#include<thread>
#endif
class Thread_Test{
    public:
        Thread_Test();
        void output(int input);
        void Recv(int from,int to);
        ~Thread_Test();
};
```

**在这里注意对创建的线程使用join和detach的区别！**

<summary><font color=#600030><mark>thread_Test1.cc</mark></font></summary>

```
#include"include/thread_Test.hpp"
using namespace std;
Thread_Test::Thread_Test(){
    cout<<"线程类被创建！"<<endl;
}
void Thread_Test::output(int input){
    cout<<"It is the "<<input<<"th "<<endl;
}
void Thread_Test::Recv(int from,int to){
    for(int i=from;i<to;i++){
        thread t(&Thread_Test::output,this,i);//通过创建thread的对象来创建线程,在类内部创建线程的时候需要指定`this`
        //t.join();//等待启动的线程完成后才会继续向下执行
        t.detach();//线程创建后进入后台并行运行
    }
}
Thread_Test::~Thread_Test(){//析构函数
    cout<<"the thread is over!"<<endl;
}
int main(){
    Thread_Test TT;
    TT.Recv(0,7);
    return 0;
}
```
使用join方式的结果：可以发现依次等待每个启动的线程完成后才会继续向下执行。
![15](/assets/15.png)
使用detach方式的结果：可以发现所有被创建的线程立刻进入后台并行，他们抢占式的使用控制台。
![16](/assets/16.png)

<font color=#006000>**由于控制台是系统资源，这里控制台拥有权的管理是操作系统完成的。但是，假如是多个线程共享进程空间的数据，这就需要自己写代码控制，每个线程何时能够拥有共享数据进行操作。共享数据的管理以及线程间的通信，是多线程编程的两大核心。**</font>

##三、线程管理：
每个应用程序至少有一个进程，而每个进程至少有一个主线程，除了主线程外，在一个进程中还可以创建多个线程。**每个线程都需要一个入口函数，入口函数返回退出，该线程也会退出**，主线程就是以`main`函数作为入口函数的线程。在C++ 11的线程库中，将线程的管理放在了类`std::thread`中，使用`std::thread`可以创建、启动一个线程，并可以将线程挂起、结束等操作。
###3.1启动一个线程：
C++ 11的线程库启动一个线程是非常简单的，只需要创建一个std::thread对象，就会启动一个线程，并使用该std::thread对象来管理该线程。

	func1();
	std::thread t(fnuc1);
这里创建`std::thread`传入的函数，实际上其构造函数需要的是可调用（callable）类型，只要是有函数调用类型的实例都是可以的。所有除了传递函数外，还可以使用：

1). lambda表达式：

	for(int i=0;i<4;i++){
        thread t([i]{                     //创建一个新线程，并将一个lambda函数给它调用
            cout<<i<<endl;
        });
        t.detach();
    }

![17](/assets/17.png)

2). 重载了()运算符的实例：
<summary><font color=#600030><mark>Task.hpp</mark></font></summary>

```
#ifndef HEAD_H
#define HEAD_H
  `xq`#include<thread>
#endif
//使用重载了()运算符的类实现多线程数字输出
class Task{
    public:
	/*重载了运算符()，重载的运算符'()'其实就是一个拥有特殊名称的函数*/
	void operator()(int i);
};
```
<summary><font color=#600030><mark>thread_test.cc</mark></font></summary>

```
#include"include/Task.hpp"
using namespace std;
void Task::operator()(int i){
    cout<<i<<endl;
}
int main(){
    for(int i=0;i<7;i++){
	Task task;
	//在向std::thread的构造函数传递函数对象的时候，注意要传入一个命名变量，而不能是一个临时变量
	//如果不是一个命名变量就会出现语法解析错误
	thread t(task,i);//有参方式构建线程，在创建线程t的时候，参数i被传递给了类task,我们就像使用函数一样使用了类的对象
	//t.detach();//启动的线程在后台自动运行，当前代码继续向下执行，主线程main不必等待所有线程执行结束才结束
	t.join();
    }
    return 0;
}
```

![18](/assets/18_uthe229yt.png)

**注意：**
把函数对象传入`std::thread`的构造函数时，要注意一个C++的语法解析错误（C++'s most vexing parse）。如果向`std::thread`的构造函数中传入的是一个临时变量，而不是命名变量就会出现语法解析错误。

注意如下代码：
```
std::thread t(Task());  // Task()的目的是创建一个临时变量
```
这里相当于声明了一个函数t，其返回类型为`thread`，而不是启动了一个新的线程。可以使用新的初始化语法避免这种情况:
```
std::thread t{Task()};
```

**当线程启动后，一定要在和线程相关联的`thread`销毁前，确定以何种方式等待线程执行结束**。C++11有两种方式来等待线程结束:

1). detach方式，<font color=red>启动的线程自主在后台运行，当前的代码继续往下执行，不等待新线程结束</font>。

2). join方式，<font color=red>等待启动的线程完成，才会继续往下执行</font>。假如前面的代码使用这种方式，其输出就会0,1,2,3，因为每次都是前一个线程输出完成了才会进行下一个循环，启动下一个新线程。

**<font color=#642100>无论在何种情形，一定要在`thread`销毁前，调用`t.join`或者`t.detach`，来决定线程以何种方式运行</font>**。当使用join方式时，会阻塞当前代码，等待线程完成退出后，才会继续向下执行；而使用detach
方式则不会对当前代码造成影响，当前代码继续向下执行，创建的新线程同时并发执行，这时候**<font color=#642100>需要特别注意一点：即创建的新线程对当前作用域的变量的使用情况</font>**。创建新线程的作用域结束后，有可能线程
仍然在执行，这时局部变量随着作用域的完成都已销毁，<font color=#336666>**如果线程继续使用局部变量的引用或者指针，会出现意想不到的错误，并且这种错误很难排查**</font>。例如：

我们在代码块中，使用fn启动了一个新的线程，在这个新的线程中使用了局部变量a的指针，并且将该线程的运行方式设置为detach。这样，在代码块执行结束后，变量a被销毁，但是在后台运行的线程仍然在
使用已销毁变量a的指针，其输出结果如下：(**由于是detach方式运行线程，所以线程可能还没执行结束main函数，即主线程就结束了！**)

<summary><font color=#600030><mark>thread_test2.cc</mark></font></summary>

```
#include<iostream>
#include<string>
#include<thread>
using namespace std;
auto fn = [](int *a){
    for(int i=0;i<10;i++)
        cout<<*a<<endl;
};
int main(){
    //一个代码块内部的变量只在这个代码块运行的时候有效，代码块执行完后就释放内存并销毁
    {
        int a=100;
        thread t(fn,&a);//注意，线程创建后，要在和线程相关的thread销毁前确定以何种方式结束线程的执行
        t.detach();//非阻塞方式运行,此时线程已经进入了系统后台（我们并没有拿到线程句柄，所以此时无法对线程进行操作）
    };
    cout<<"用来拖延时间。。。"<<endl;
    //getchar(); //在使用getchar的时候会发现由于main线程一直未结束，临时变量并没有释放，所以全部正确输出
    return 0;
}
```
<font color=red>可以看到，并没有固定的输出结果。</font>
![19](/assets/19_96jfio0k0.png)

**<font color=#616030>所以在以detach的方式执行线程时，要将线程访问的局部数据复制到线程的空间（使用值传递），一定要确保线程没有使用局部变量的引用或者指针，除非你能肯定该线程会在局部作用域结束前执行结束。</font>**
<font color=#6c3365>当然，使用join方式的话就不会出现这种问题，它会在作用域结束前完成退出。</font>

####补充：关于线程运行时候的一点疑问和总结：
**1.在线程创建之后的线程运行情况：**（在使用join和detach方式决定线程结束方式之前）
在线程创建之后，线程就开始运行，在决定执行结束方式之前，被创建的线程就已经开始运行并和main函数这个主线程一起抢占cpu以及io，所以这个时候当我们要输出到屏幕时就会和主函数线程进行抢占。

<summary><font color=#600030><mark>thread_test.cc</mark></font></summary>

```
#include <iostream>
#include <string>
#include <thread>
using namespace std;
auto fn = [](int* a) {
  for (int i = 0; i < 5; i++)
    cout << *a << endl;
};
int main() {
  // 一个代码块内部的变量只在这个代码块运行的时候有效，代码块执行完后就释放内存并销毁
  {
      int a = 100;
      thread t(fn, &a);  // 注意，线程创建后就已经开始执行;但是要在创建线程的作用域结束之前确定线程以何种方式结束运行
      /*
        在这里线程会和main函数这个主线程抢占输出io
       */
      cout << "test1" << endl;
      cout << "test2" << endl;
      cout << "test3" << endl;
      cout << "test4" << endl;
      cout << "test5" << endl;
      cout << "test6" << endl;
      cout << "test7" << endl;
      t.detach();//非阻塞方式运行,此时线程已经进入了系统后台（我们并没有拿到线程句柄，所以此时无法对线程进行操作）
      //t.join();
  };
  cout << "用来拖延时间。。。" << endl;
  //getchar(); //在使用getchar的时候会发现由于main线程一直未结束，临时变量并没有释放，所以全部正确输出
  return 0;
}

```

1). 可以看到在使用join方式的时候，main函数中在线程join之后的语句肯定比线程中的输出靠后，但是在线程创建到线程join之间main函数的输出语句会和线程产生抢占。
![20](/assets/20.png)

2). 在使用detach方式后，线程挂在后台运行，main函数中在线程detach之后的语句仍然和线程进行资源的抢占。但是由于此时代码块中的临时变量已经释放，所以此时线程中指针指向的内存内容不存在(会输出一些非法内容，很危险)
![21](/assets/21.png)

###3.2异常情况下等待线程完成
当决定以`detach`方式让线程在后台运行时，**可以在创建线程的实例后立即调用`detach`，这样线程就会实现线程的实例分离**，即使出现了异常，使得线程的实例被销毁，仍然能保证创建的线程在后台继续运行。

<font color=#2F0000>但线程以`join`方式运行时，需要在主线程的合适位置调用`join`方法，如果调用`join`前出现了异常，线程被销毁，线程就会被异常所终结。</font>为了避免因为异常将线程终结，或者由于某些原因，例如线程访问了局部变量，就要保证线程一定要在函数或者作用域退出前完成，即保证要在函数或者作用域退出前调用`join`。
```
void func() {
    thread t([]{
	cout << "hello C++ 11" << endl;
    });
    try
    {
	//do_something_else
    }
    catch (...)
    {
	t.join();
	throw;
    }
    t.join();
}
```
上面代码能够保证在正常或者异常的情况下，都会调用join方法，这样线程一定会在函数`func`退出前完成。但是使用这种方法，不但代码冗长，而且会出现一些作用域的问题，并不是一个很好的解决方法。
一种比较好的方法是资源获
取即初始化（RAII,Resource Acquisition Is Initialization)，该方法提供一个类，**在析构函数中调用join。**

<summary><font color=#600030><mark>thread_test3.hpp</mark></font></summary>

```
#ifndef THREAD_HEAD3
#define THREAD_HEAD3

#include<thread>
class Thread_guard {
 public:
  // 如果为引用形参，需要加上 const，否则使用指针形参（但是这里需要对线程join，所以只能使用指针行参）
  explicit Thread_guard(std::thread *_t);
  ~Thread_guard();
  // 表示禁止使用编译器默认的构造函数
  Thread_guard(const Thread_guard&) = delete;
  Thread_guard& operator=(const Thread_guard&) = delete;
 private:
  std::thread *t;
};
#endif
```

<summary><font color=#600030><mark>thread_test3.cc</mark></font></summary>

```
#include<iostream>
#include"../include/thread_test3.hpp"
Thread_guard::Thread_guard(std::thread *_t):t(_t) {
}
Thread_guard::~Thread_guard() {
    // 代表线程仍然可以join，即在函数析构之前线程没有使用join或者detach方式
    if ((*t).joinable()) {
        (*t).join();
    }
}
void func() {
  // 用lambda表达式对线程初始化
  std::thread t([]{
    std::cout << "thread test ..." << std::endl;
  });
  Thread_guard g(&t);
}
int main() {
  func();
  return 0;
}

```

<font color=r#58975>在这里，无论是何种情况，当函数退出时，局部变量g调用其析构函数，从而能够保证join一定会被调用。</font>

###3.3向线程传递参数：
向线程调用的函数传递参数也是很简单的，只需要在构造`thread`的实例时，依次传入即可。例如：
```
void func(int *a,int n){}

int buffer[10];
thread t(func,buffer,10);
t.join();
```
**需要注意**，<font color=red>向线程所调用的函数传递参数时，默认会将传递的参数以拷贝的方式复制到线程空间，即使参数的类型是引用。</font>例如：
```
void func(int a,const string& str);
thread t(func,3,"hello");  // func为线程t的执行函数
```
func的第二个参数是`string &`，而传入的是一个字符串字面量。该字面量以`const char*`类型传入线程空间后，在线程的空间内转换为`string`。
1). 在线程中使用引用来更新对象：
```
#include<iostream>
#include<thread>
class _tagNode {
 public:
  int a = 0;
  int b = 1;
};
// 这里采用的是引用传值方式
void setValue(_tagNode &node) {
  node.a = 10;
  node.b = 12;
}
_tagNode node;
void thread_func() {
  // 这里对引用方式传递对象需要使用ref方式对其进行包装
  std::thread t(setValue, std::ref(node));
  t.join();
}
int main() {
  thread_func();
  std::cout << "The value of node is : " << node.a << " " << node.b << std::endl;
  return 0;
}
```

**注意：** 当给 thread 的执行函数传递指针参数时，没有任何问题，但是如果想传递引用，按照普通函数的调用方法会遇到编译失败。**这里类似于 std::bind方法，std::thread 和 std::bind 采用了相同的机制，必须使用 std::ref 来包装。**

**输出的结果：**
```
zhangli@zhangli-GS63VR-7RF:~/code/Cpp/thread/test4/build$ ./threadtest4
The value of node is : 10 12
```
2). 使用类的成员函数作为线程参数：
这里有几种不同的传递方式：
```
#include<iostream>
#include<thread>
class _tagNode {
 public:
  int a = 0;
  int b = 1;
  void do_some_work(int number) {
    std::cout << "线程调用类内部的函数。非静态调用方式" << number << std::endl;
  }
  static void do_some_static_work(int number) {
    std::cout << "线程调用类内部的函数。静态调用方式" << number << std::endl;
  }
};
void thread_func1() {
  // 静态成员函数直接传递，因为类的静态成员函数和普通函数没有区别
  std::thread t1(&_tagNode::do_some_static_work,15);
  t1.join();

  _tagNode node;
  // 非静态成员函数需要传递对象或者对象的地址进入
  std::thread t2(&_tagNode::do_some_work, node, 20);
  t2.join();
  std::thread t3(&_tagNode::do_some_work, &node, 45);
  t3.join();
}
int main() {
  thread_func1();
  return 0;
}
```

###3.4线程的暂停和停止：
1). 线程的暂停：
从外部让线程暂停，会引发很多并发问题，这大概也是std::thread并没有直接提供pause函数的原因。但有时线程在运行时，确实需要“停顿”一段时间怎么办呢？可以使用`std::this_thread::sleep_for`或`std::this_thread::sleep_until`方法。
```
#include<chrono>
#include<iostream>
#include<thread>
// 传递给线程的函数，在函数内部使用暂停功能
void Pauseable() {
  //sleep 500ms
  std::this_thread::sleep_for(std::chrono::milliseconds(500));
  // sleep until pointed time
  std::this_thread::sleep_until(std::chrono::steady_clock::now() + std::chrono::milliseconds(500));
}
int main() {
  std::thread t(Pauseable);
  if(t.joinable())
    t.join();
  return 0;
}
```
2). 线程的停止：
一般情况下当线程函数执行完成后，线程“自然”停止。但在std::thread中有一种情况会造成线程异常终止，**那就是：析构**。当std::thread实例析构时，如果线程还在运行，则线程会被强行终止掉，这可能会造成资源的泄漏，因此尽量在析构前join一下，以确保线程成功结束。
如果确实想提前让线程结束怎么办呢？一个简单的方法是使用“共享变量”，线程定期地去检测该量，如果需要退出，则停止执行，退出线程函数。使用“共享变量”需要注意，在多核、多CPU的情况下需要使用“原子”操作。

###3.5转移线程的所有权：
注意，**拷贝构造函数和拷贝赋值运算符被禁用，意味着std::thread对象不能够被拷贝和赋值到别的thread对象**；
```
拷贝构造函数                     thread(const thread&) = delete;
拷贝赋值运算符                  thread& operator=(const thread&) = delete;
```
thread是可移动的(movable)的，但不可复制(copyable)。可以通过`move`来改变线程的所有权，灵活的决定线程在什么时候`join`或者`detach`：
```
thread t1(f1);
thread t2(move(t1));
```
这里将线程从t1转移给t2,这时候t1就不再拥有线程的所有权，调用t1.join或t1.detach会出现异常，要使用t2来管理线程。这也就意味着`thread`可以作为函数的返回类型，或者作为参数传递给函数，能够更为方便的管理线程。

**线程的标识类型为`std::thread::id`，有两种方式可以得到线程的id**：
线程ID是一个线程的标识符，C++标准中提供两种方式获取线程ID；
```
1). thread_obj.get_id();
2). std::this_thread::get_id()
```
有一点需要注意，就是空thread对象，也就是不表示任何线程的线程对象调用`get_id`返回值为0；
此外当一个线程被`detach`或者`joinable() == false`时，调用`get_id`的返回结果也为0。

**交换thread表示的线程：**
除了上面的detach可以分离thread对象及其所表示的线程，或者move方法移动到别的线程之外，还可以使用swap来交换两个thread对象表示的线程。
```
#include <chrono>
#include <iostream>
#include <thread>
int tstart(const std::string& name) {
  std::cout << "Thread test! " << name << std::endl;
  return 0;
}
int main() {
 /*
  std::thread t(Pauseable);
  std::thread t(tstart);
  if(t.joinable())
    t.join();
  */
  std::thread t1(tstart, "C++ 11 thread_1!");
  std::thread t2(tstart, "C++ 11 thread_2!");
  // it is just the main thread's id
  std::cout << "current thread id: " << std::this_thread::get_id() << std::endl;
  std::cout << "before swap: "<< " thread_1 id: " << t1.get_id() << " thread_2 id: " << t2.get_id() << std::endl;
  t1.swap(t2);
  std::cout << "after swap: " << " thread_1 id: " << t1.get_id() << " thread_2 id: " << t2.get_id() << std::endl;
  // t.detach();
  t1.join();
  t2.join();
  return 0;
}
```
**结果：**
```
current thread id: 140466635372352
before swap:  thread_1 id: 140466635368192 thread_2 id: 140466626975488
after swap:  thread_1 id: 140466626975488 thread_2 id: 140466635368192
Thread test! C++ 11 thread_1!
Thread test! C++ 11 thread_2!
```
**关于thread swap函数的实现：**
```
void
    swap(thread& __t) noexcept
    { std::swap(_M_id, __t._M_id); }
```
**可以看到交换的过程仅仅是互换了thread对象所持有的线程id.**


##四、线程使用进阶：
###thread_local:
**C++11的`thread_local`来自于boost中的`thread_specific_ptr`:**
thread_specific_ptr代表了一个全局的变量，<font color=#25687>而在每个线程中都各自new一个线程本地的对象交给它进行管理</font>，这样，各个线程就可以各自独立地访问这个全局变量的本地存储版本，线程之间就不会因为访问同一全局对象而引起资源竞争导致性能下降。而线程结束时，这个资源会被自动释放。

1.thread_local变量是C++ 11新引入的一种存储类型。它会影响变量的存储周期(Storage duration)，C++中有4种存储周期：
```
automatic
static
dynamic
thread
```
有且只有thread_local关键字修饰的变量具有线程周期(thread duration)，这些变量在线程创建时生成(不同编译器实现略有差异，但在线程内变量第一次使用前必然已构造完毕)，线程结束时被销毁(析构，利用析构特性，thread_local变量可以感知线程销毁事件)。并且每一个线程都拥有一个独立的变量实例(Each thread has its own instance of the object)。`thread_local`可以和`static`与`extern`关键字联合使用，这将影响变量的链接属性(to adjust linkage)。

2.以下三类变量都可以被声明为`thread_local`类型：
```
命名空间下的全局变量
类的static成员变量
本地变量

thread_local int x; // a thread_local variable at namespace global scope
class test {
  static thread_local std::string s; // a thread_local static class member
};
thread_local std::string test::s = "a string test";
std::vector<std::string> foo() {
  thread_local std::vector<std::string> s; // a thread_local local variable
  return s;
}
```
3.既然每个线程都拥有一份独立的thread_local变量，那么就有2个问题需要考虑：

1)各线程的thread_local变量是如何初始化的

2)各线程的thread_local变量在初始化之后拥有怎样的生命周期，特别是被声明为thread_local的本地变量(local variables)

这里输出（id值是每次运行时变的，多线程输出结果顺序是随机的，下面是理想的代码顺序输出结果）：
```
#include<iostream>
#include<thread>
#include<vector>
thread_local int g_n = 1;  // 声明thread_local
void f() {
  g_n++;
  std::cout<< "the thread id: " << std::this_thread::get_id() << "and value of g_n: " << g_n << std::endl;
}
void foo() {
  thread_local int i = 0;
  std::cout<< "the thread id: " << std::this_thread::get_id() << "and value of i: " << i << std::endl;
  i++;
}
void f2() {
  foo();
  foo();
}
int main() {
  // 初始进入main主线程的时候thread_local g_n的值为1
  g_n++;
  f();  // 这里因为都是在主线程main中，所以g_n的值为3
  std::cout<< "***********************************************************************" << std::endl;
  std::thread t1(f);  // 进入线程t1中，g_n的值为初始值1
  t1.join();
  std::cout<< "***********************************************************************" << std::endl;
  std::thread t2(f);
  t2.join();
  std::cout<< "***********************************************************************" << std::endl;
  /*在主线程main中，f2函数调用了foo两次，第一次调用foo创建了一个thread_local i，
    值为0,i++，第二次调用foo的时候由于上次的值已经增加，所以输出1.
   */
  f2();
  std::cout<< "***********************************************************************" << std::endl;
  std::thread t3(f2);  // 进入t3线程，在t3中重新开始一个初始的thread_local i.
  t3.join();
  std::cout<< "***********************************************************************" << std::endl;
  std::thread t4(f2);
  t4.join();

  return 0;
}
```
![27](/assets/27_a6sx5ch6x.png)
输出的前3行打印能帮助解答thread_local变量是如何初始化的，可以看到每个线程都会进行一次初始化，例子中的g_n在主线程中最早被初始化为1，随后被修改为2和3，但这些修改操作并不影响g_n在线程t1和t2中的初始值(值为1)，虽然t2和t3线程启动的时候主线程中的变量值已经被更新为3，所以主线程、thread1、thread2打印结果分别为3，2，2。

后6行打印说明了一个事实，<font color=#459>声明为thread_local的本地变量在线程中是持续存在的，不同于普通临时变量的生命周期，它具有static变量一样的初始化特征和生命周期，虽然它并没有被声明为static。</font>例子中foo函数中的thread_local变量 i 在每个线程第一次执行到的时候初始化，在每个线程各自累加，在线程结束时释放。

4.thread_local使用场景：
dre是一个全局的default_random_engine对象（全局对象可以是C风格的extern，也可是C++风格的static，如果在同一个源文件中，则在“代码的上面”）。同时开启多个线程运行，每个线程都需要使用这个全局的dre对象，不同的线程对dre设置不同的seed。但是，又不能让所有线程同时访问同一个dre，这样dre的seed变来变去，而且无法得到想要的随机值序列。

**此时有2个解决方案：**
1).改变声明dre的地方，把dre从全局对象变为线程的局部变量（<font color=#34568>比如一个函数中</font>）。在线程里面，再通过参数的传递，把dre传到需要它的地方。
2).声明dre的地方不变，依然是全局，只要加一个thread_local关键词，其他什么都不用改。

**方案1很丑陋，方案2很优雅。**
```
#include<chrono>
#include<future>
#include<iostream>
#include<random>
#include<vector>
thread_local std::default_random_engine dre;
std::vector<int> test_dre(int seed) {
  dre.seed(seed);  // 根据传入的种子值不同，创建的随机数也不同;但是调用同一队引擎以及种子时，生成的随机数都一样
  int vec_size = 10;
  std::vector<int> temp(vec_size);
  for (int i = 0; i < vec_size; i++) {
    temp[i] = dre();  // 获得一个随机数
    std::cout<< dre() <<std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
  }
  std::cout << std::endl;
  return temp;
}
bool equal(const std::vector<int>& v1, const std::vector<int>& v2) {
  int size1 = v1.size();
  int size2 = v2.size();
  if (size1 != size2)
    return false;

  for (int i = 0; i < size1; i++)
    if (v1[i] != v2[i])
      return false;

  return true;
}
int main() {
  int num_thread = 2;
  using return_type = std::vector<int>;
  std::future<return_type> future1 = std::async(test_dre, 0);
  return_type return1 = future1.get();
  return_type return2 = test_dre(0);
  if (equal(return1, return2))
    std::cout << "they are equal." <<std::endl;
  else
    std::cout << "they are not equal." <<std::endl;

  return 0;
}
```
![28](/assets/28_24370gldy.png)
##五、线程调用的补充知识点：
###1.std::async（使用async异步调用程序）：
在我们希望获取线程函数的返回结果的时候，我就不能直接通过`thread.join()`得到结果，这时就必须定义一个变量，在线程函数中去给这个变量赋值，然后join,最后得到结果，这个过程是比较繁琐的。
c++11还提供了异步接口`std::async`，通过这个异步接口可以很方便的获取线程函数的执行结果。`std::async`会自动创建一个线程去调用线程函数，它返回一个`std::future`，这个future中存储了线程函数返回的结果，当我们需要线程函数的结果时，直接从future中获取，非常方便。
但是，其实std::async给我们提供的便利可不仅仅是这一点，它首先解耦了线程的创建和执行，使得我们可以在需要的时候获取异步操作的结果；其次它还提供了线程的创建策略（比如可以通过延迟加载的方式去创建线程），使得我们可以以多种方式去创建线程。在介绍async具体用法以及为什么要用std::async代替线程的创建之前，先说一说`std::future`、`std::promise`和 `std::packaged_task`。
####std::future:
std::future是一个类模板(class template)，**一个std::future对象在内部存储一个将来会被赋值的值，并提供了一个访问该值的机制，通过get()成员函数实现**。但如果有人试图在get()函数可用之前通过它来访问相关的值，那么get()函数将会阻塞，直到该值可用。(在多线程中没办法控制输出的顺序，那么`std::future`就为所有会有输出的位置放一个占位符，认定这个位置会有一个输出，但是具体什么时间会有不确定)。`std::future`提供了一种访问异步操作结果的机制，可以通过查询future的状态`future_status`来获取异步操作的结果。`future_status`有三种状态：
```
deferred：异步操作还没开始
ready：异步操作已经完成
timeout：异步操作超时
```
```
//  查询future的状态
#include<chrono>
#include<future>
#include<iostream>
#include<thread>
#include<vector>
std::vector<int> test_dre(int number) {
  int vec_size = number;
  std::vector<int> temp(vec_size);
  for (int i = 0; i < vec_size; i++) {
    temp.push_back(i);
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
  }
  return temp;
}
int main() {
  std::future_status status;
  using return_type = std::vector<int>;
  std::future<return_type> future1 = std::async(test_dre, 10);  // 创建异步方式调用线程，之后就开始继续向下执行
  do {
    status = future1.wait_for(std::chrono::milliseconds(1000));  // 这里等待异步方式执行结束，并等待返回结果
    if (status == std::future_status::deferred) {
      std::cout << "deferred\n";
    } else if (status == std::future_status::timeout) {
      std::cout << "timeout\n";
    } else if (status == std::future_status::ready) {
      std::cout << "ready!\n";
    }
  } while(status != std::future_status::ready);
  return 0;
}
```
![29](/assets/29.png)
获取future结果有三种方式：`get`、`wait`、`wait_for`;其中get等待异步操作结束并返回结果;wait只是等待异步操作完成，没有返回值;wait_for是超时等待返回结果。

####std::promise:
1）`std::promise`是一个模板类:
`template<class R> class promise`。其泛型参数R为`std::promise`对象保存的值的类型，R可以是void类型。<font color=#4589>`std::promise`为获取线程函数中的某个值提供便利</font>。`std::promise`保存的值可被与之关联的`std::future`读取，读取操作可以发生在其它线程。`std::promise`允许`move`语义(右值构造，右值赋值)，但不允许拷贝(拷贝构造、赋值)，`std::future`亦然。`std::promise`和`std::future`合作共同实现了多线程间通信。

2）设置std::promise的值：
通过成员函数set_value可以设置std::promise中保存的值，该值最终会被与之关联的std::future::get读取到。需要注意的是：set_value只能被调用一次，多次调用会抛出std::future_error异常。事实上std::promise::set_xxx函数会改变std::promise的状态为ready，再次调用时发现状态已要是reday了，则抛出异常。
```
#include <chrono>
#include <future>
#include <iostream>
#include <string>
#include <thread>
void read(std::future<std::string> *future) {
  // future 会一直阻塞，直到有值来（由于future和promise共享状态，所以promise中有值的时候即可）
  std::cout << future->get() << std::endl;
}
int main() {
  // promise相当于生产者
  std::promise<std::string> promise;
  // future 相当于消费者
  std::future<std::string> future = promise.get_future();
  // 另一个线程中通过future来读取promise的值
  std::thread thread1(read, &future);
  // 让read等一会儿
  std::this_thread::sleep_for(std::chrono::seconds(1));
  promise.set_value("Hello Future");
  // 等待线程执行完成
  if (thread1.joinable())
    thread1.join();
  return 0;
}
```
等待一秒后输出结果：
```
Hello Future
```
在这里，与`std::promise`关联的`std::future`是通过`std::promise::get_future`获取到的，自己构造出来的无效。一个`std::promise`实例只能与一个`std::future`关联共享状态，当在同一个`std::promise上反复调用`get_future`会抛出future_error异常。

**共享状态：**
在`std::promise`构造时，`std::promise`对象会关联一个共享状态，这个共享状态可以存储一个R类型的值或者一个由`std::exception`派生出来的异常值。`std::future`通过`std::promise::get_future`调用获得与`std::promise`相同的共享状态。

3）`std::promise`不设置值：
如果promise直到销毁时，都未设置过任何值，则promise会在析构时自动设置为std::future_error，这会造成std::future.get抛出std::future_error异常。
```
#include <chrono>
#include <future>
#include <iostream>
#include <string>
#include <thread>
void read(std::future<int> future) {
  try {
    future.get();
  } catch(std::future_error &e) {
    std::cerr << e.code() << "\n" << e.what() << std::endl;
  }
}
int main() {
  std::thread thread;
  {
    /*如果promise不设置任何值
     *则在promise析构时会自动设置为future_error
     *这会造成future.get抛出该异常
     */
    std::promise<int> promise;
    thread = std::thread(read, promise.get_future());
  }
  if (thread.joinable())
    thread.join();
  return 0;
}
```
**输出结果：**
```
future:4
std::future_error: Broken promise
```

4）通过`std::promise`让`std::future`抛出异常：
通过std::promise::set_exception函数可以设置自定义异常，该异常最终会被传递到std::future，并在其get函数中被抛出。
```
#include <future>
#include <iostream>
#include <thread>
#include <exception>
#include <stdexcept>
void catch_error(std::future<void> &future) {
  try {
    future.get();
  } catch(std::logic_error &e) {
    std::cerr << "logic error: " << e.what() << std::endl;
  }
}
int main() {
  std::promise<void> promise;
  std::future<void> future = promise.get_future();
  std::thread thread(catch_error, std::ref(future));
  // 自定义异常需要使用make_exception_ptr转换一下
  promise.set_exception(std::make_exception_ptr(std::logic_error("caught")));

  if (thread.joinable())
    thread.join();
  return 0;
}
```
**输出结果：**
```
logic error: caught
```
```
// std::promise::set_exception函数原型
void set_exception(std::exception_ptr p);
```
自定义的异常信息可以通过位于头文件exception下的std::make_exception_ptr函数转化为std::exception_ptr类型。

5）`std::promise<void>`:

通过上面可以看到，`std::promise<void>`是合法的。此时`std::promise.set_value`不接受任何参数，仅用于通知关联的`std::future.get()`解除阻塞。

6）std::promise退出：

std::async(异步运行)时，有时会对std::promise所在线程退出时间比较关注。std::promise支持定制线程退出时的行为：

`std::promise::set_value_at_thread_exit`线程退出时，`std::future`收到通过该函数设置的值。
`std::promise::set_exception_at_thread_exit`线程退出时，`std::future`则抛出该函数指定的异常。

####std::packaged_task:

`std::packaged_task`包装了一个可调用的目标（如`function, lambda expression, bind expression, or another function object`）,以便异步调用。它和promise在某种程度上有点像。promise保存了一个共享状态的值，而它允许传入一个函数，并将函数计算的结果传递给std::future，包括函数运行时产生的异常。

1）**packaged_task的一个例子：**
```
#include <chrono>
#include <future>
#include <iostream>
#include <thread>

int sum(int a, int b) {
  std::this_thread::sleep_for(std::chrono::seconds(2));  // 线程在这里睡两秒
  return a + b;
}
int main() {
  std::packaged_task<int(int, int)> task(sum);
  std::future<int> future = task.get_future();  // future和task绑定共享参数

  /*和std::promise一样，std::packaged_task支持move，但不支持拷贝
    std::thread的第一个参数不止是函数，还可以是一个可调用对象，
    即支持operator()(Args...)操作
   */
  // 注意，thread传进去的是一个右值引用，所以对于左值task需要使用move方式
  std::thread t(std::move(task), 1, 2);  // 创建线程以后，线程就开始执行
  t.detach();  // 在这里只是决定线程的结束方式，我们让其在后台运行

  // 等待异步计算结果,因为std::future在用get方式获得数据时，在未获得的时候会一直阻塞并等待线程执行结束
  std::cout << "1 + 2 => " << future.get() << std::endl;  // 可以看到，这里等待了2s才等到线程中的计算结束，获得数据
  return 0;
}
```
输出结果：
```
1 + 2 => 3
```

2）**std::packaged_task详解：**
`std::packaged_task`位于头文件`#include <future>`中，是一个模板类
```
template <class R, class... ArgTypes>
class packaged_task<R(ArgTypes...)>
```
其中R是一个函数或可调用对象，ArgTypes是R的形参。与`std::promise`一样，`std::packaged_task`支持move，但不支持拷贝(copy)。`std::packaged_task`封装的函数的计算结果会通过与之联系的`std::future::get`获取(**当然，可以在其它线程中异步获取**)。关联的`std::future`可以通过`std::packaged_task::get_future`方式与其绑定，`get_future`仅能调用一次，多次调用会触发`std::future_error`异常。`std::package_task`除了可以通过可调用对象构造外，还支持缺省构造(无参构造)。但此时构造的对象不能直接使用，需通过右值赋值操作设置了可调用对象或函数后才可使用。判断一个`std::packaged_task`是否可使用，可通过其成员函数valid来判断。

3）`std::packaged_task::valid`：
该函数用于判断std::packaged_task对象是否是有效状态。当通过缺省构造初始化时，由于其未设置任何可调用对象或函数，valid会返回false。只有当std::packaged_task设置了有效的函数或可调用对象，valid才返回true。
```
  std::packaged_task<void()> task1; // 缺省构造
  // std::boolalpha使bool型变量按照false、true的格式输出。如不使用该标识符，那么结果会按照1、0的格式输出
  std::cout << std::boolalpha << task1.valid() <<std::endl;
  std::packaged_task<void()> task3([](){});  // 右值赋值，可调用对象
  std::cout << std::boolalpha << task3.valid() <<std::endl;
```
输出结果：
```
false
true
```
4）`std::packaged_task::operator()(ArgTypes...)`：
该函数会调用std::packaged_task对象所封装的可调用对象R，但其函数原型与R稍有不同:
```
void operator()(ArgTypes... );
```
operator()的返回值是void，即无返回值。因为std::packaged_task的设计主要是用来进行异步调用，因此R(ArgTypes...)的计算结果是通过std::future::get来获取的。该函数会忠实地将R的计算结果反馈给std::future，即使R抛出异常(此时std::future::get也会抛出同样的异常)。
```
  std::packaged_task<void()> convert([](){
    throw std::logic_error("will catch in future");});  // 创建一个packaged_task的对象，绑定的参数是一个lambda表达式

  std::future<void> future1 = convert.get_future();

  convert();  // 异常不会在此处抛出,这里其实是使用了函数调用运算符

  try {
    future1.get();
  } catch(std::logic_error &e) {
    std::cerr << typeid(e).name() << ": " << e.what() << std::endl;
  }
```

输出结果：
```
St11logic_error: will catch in future
```
5）补充：
`std::packaged_task::make_ready_at_thread_exit`函数接收的参数与`operator()(_ArgTypes...)`一样，行为也一样。只有一点差别，那就是不会将计算结果立刻反馈给`std::future`，而是在其执行时所在的线程结束后`std::future::get`才会取得结果。

`std::packaged_task::reset`与`std::promise`不一样，`std::promise`仅可以执行一次`set_value`或`set_exception`函数，但`std::packagged_task`可以执行多次，其奥秘就是reset函数
```
template<class _Rp, class ..._ArgTypes>
void packaged_task<_Rp(_ArgTypes...)>::reset()
{
    if (!valid())
        __throw_future_error(future_errc::no_state);
    __p_ = promise<result_type>();
}
```
通过重新构造一个promise来达到多次调用的目的。显然调用reset后，需要重新get_future，以便获取下次operator()执行的结果。**由于是重新构造了promise，因此reset操作并不会影响之前调用的make_ready_at_thread_exit结果，也即之前的定制的行为在线程退出时仍会发生。**

####std::promise、std::packaged_task和std::future的关系:
