[[C]]++单例模式
##一、简介：
###1.什么是单例模式：
在软件系统中，经常有这样一些特殊的类，必须保证他们在系统中只存在一个实例，才能确保它们的逻辑正确性、以及良好的效率。此时一个类只能创建一个对象，即单例模式。**该模式可以保证系统中该类只有一个实例，并提供一个访问它的全局访问点，该实例被所有程序模块共享**。
###2.为什么需要单例模式：
对于系统中的某些类来说，只有一个实例很重要，例如，一个系统中可以存在多个打印任务，但是只能有一个正在工作的任务；一个系统只能有一个窗口管理器或文件系统等等。在Windows中就只能打开一个任务管理器。如果不使用机制对窗口对象进行唯一化，将弹出多个窗口，如果这些窗口显示的内容完全一致，则是重复对象，浪费内存资源；如果这些窗口显示的内容不一致，则意味着在某一瞬间系统有多个状态，与实际不符，也会给用户带来误解，不知道哪一个才是真实的状态。因此有时确保系统中某个对象的唯一性即一个类只能有一个实例非常重要。

###3.单例模式的注意事项：
1）**某个类只能有一个实例**(<font color=red>具有全局变量的特点,在任何位置都可以通过接口获取到那个唯一实例;</font>)
2）**它必须自行创建这个实例**（私有的构造函数，不能在类外部定义实例）
3）<font color=#2365>它必须自行向整个系统提供这个实例。</font>(一个静态的成员函数用于获取类的实例)
从具体实现角度来说，就是以下三点：<font color=#6512>一是单例模式的类只提供私有的构造函数，二是类中含有一个该类私有的静态对象，三是该类提供了一个公有的静态的函数用于创建或获取它本身的静态私有对象。</font>

###4.单例模式的实现方式：
1）懒汉模式： 就是说当你第一次使用时才创建一个唯一的实例对象，从而实现**延迟加载**的效果。(即以时间换空间)
2）饿汉模式： 就是说不管你将来用不用，**程序启动时就创建一个唯一的实例对象**。(以空间换时间)

从实现手法上看，<font color=blue>懒汉模式是在第一次使用单例对象时才完成初始化工作,</font>因为此时可能存在多线程竞态环境，如不加锁限制会导致重复构造或构造不完全问题。**饿汉模式则是利用外部变量**，在进入程序入口函数之前就完成单例对象的初始化工作，此时是单线程所以不会存在多线程的竞态环境，故而无需加锁。
##二、c++单例的实现：
###1.基础要点：
1）全局只有一个实例：`static`特性，同时禁止用户自己声明并定义实例（**把构造函数设为`private`**）
2）线程安全
3）禁止赋值和拷贝构造
4）用户通过接口获取实例：使用`static`类成员函数

###2.单例的具体实现：
####2.1懒汉模式：
懒汉式(Lazy-Initialization)的方法是直到使用时才实例化对象，也就说直到调用`get_instance()`方法的时候才`new`一个单例的对象。好处是如果未被调用就不会占用内存。
#####2.1.1有缺陷的懒汉模式：
```
[[include]] <iostream>
/*
  with some problem below:
  memory leak
  thread is not safe
 */
class singleton {
 private:
  // 私有的构造函数
  singleton() {
    std==cout << "Constructer called!" << std==endl;
  }
  int a = 0;
  singleton(singleton&) = delete;  // 拷贝构造函数被删除
  singleton& operator=(const singleton&) = delete;  // 重载的赋值运算符，也就是赋值构造函数被删除
  static singleton* m_instance_ptr;  // 私有的静态指针，只需要初始化一次
 public:
  ~singleton() {
    std==cout << "destructer called!" << std==endl;
  }
  // 由于静态成员函数具有全局特性，所以可以近似看作一个独立的函数
  static singleton* get_instance() {
    if (m_instance_ptr == nullptr) {
      // 这里创建一个类的实例，其实就和一个普通的函数用法一样
      m_instance_ptr = new singleton;
    }
    // a ++;  // 可以看到，这里用法错误，因为静态成员函数无法访问非静态成员变量
    return m_instance_ptr;
  }
};
singleton* singleton::m_instance_ptr = nullptr;
int main() {
  singleton* instance1 = singleton::get_instance();
  singleton* instance2 = singleton::get_instance();
  return 0;
}

```
运行结果：
```
Constructer called!
```
可以发现，我们两次获取类的实例，但是只有一次类的构造函数被调用；也就是说只生成了唯一的实例，即最基础的单例版本。但是存在一些问题：
1）**线程安全问题**：
当多线程获取单例时有可能引发竞态条件：第一个线程在if中判断 `m_instance_ptr`是空的，于是开始实例化单例;同时第2个线程也尝试获取单例，这个时候判断`m_instance_ptr`还是空的，于是也开始实例化单例;这样就会实例化出两个对象,这就是线程安全问题的由来;

**解决办法**：加锁

2）**内存泄漏问题**：
注意到类中只负责new出对象，却没有负责delete对象，因此只有构造函数被调用，析构函数却没有被调用;因此会导致内存泄漏。

改进版本：（使用完删除内存，不推荐）
```
[[include]] <iostream>
/*
  with some problem below:
  memory leak
  thread is not safe
 */
int id = 0;
class singleton {
 private:
  // 私有的构造函数
  singleton(int Id) {
    std==cout << "Constructer called!" <<  " id is :" << Id << std==endl;
  }
  int a = 0;
  singleton(singleton&) = delete;  // 拷贝构造函数被删除
  singleton& operator=(const singleton&) = delete;  // 重载的赋值运算符，也就是赋值构造函数被删除
  static singleton* m_instance_ptr;  // 私有的静态指针，只需要初始化一次
 public:
  ~singleton() {
    std==cout << "destructer called!" << std==endl;
  }
  // 由于静态成员函数具有全局特性，所以可以近似看作一个独立的函数
  static singleton* get_instance() {
    if (m_instance_ptr == nullptr) {
      // 这里创建一个类的实例，其实就和一个普通的函数用法一样
      id += 1;
      m_instance_ptr = new singleton(id);
    }
    // a ++;  // 可以看到，这里用法错误，因为静态成员函数无法访问非静态成员变量
    return m_instance_ptr;
  }
  static void deleteInstance() {
    delete m_instance_ptr;
    m_instance_ptr  = nullptr;
  }
};
singleton* singleton::m_instance_ptr = nullptr;
int main() {
  singleton* instance1 = singleton::get_instance();
  singleton* instance2 = singleton::get_instance();
  singleton::deleteInstance();
  singleton* instance3 = singleton::get_instance();
  return 0;
}

```
输出结果：
```
Constructer called! id is :1
destructer called!
Constructer called! id is :2
```
**进一步解决办法**：使用一个内嵌的垃圾回收类(双检索加垃圾回收)：
```
[[include]] <atomic>
[[include]] <cstdlib>
[[include]] <chrono>
[[include]] <iostream>
[[include]] <mutex>
[[include]] <thread>
[[include]] <vector>
[[define]] max 1000
class singleton {
 private:
  // 私有的构造函数
  singleton() {
    std==cout << "Constructor called!" << std==endl;
  }
  singleton(singleton&) = delete;
  singleton& operator=(const singleton&) = delete;
  static singleton* m_instance_ptr;
  static std::mutex s_mutex;  // 互斥信号量
  static int value;
 public:
  ~singleton() {
    std==cout << "destructor called!" << std==endl;
  }
  // 使用双检锁(这里注意，由于对于线程调用函数永远都是默认传值操作，所以需要使用引用方式才能真正修改传入的参数)
  static void get_instance(singleton* &instance_ptr, std::string name) {
    int time = rand() % 1000 + 200;  // 获得随机数
    // 在第一个判断前随机等待一段时间
    std==this_thread==sleep_for(std==chrono==milliseconds(time));
    if (m_instance_ptr == nullptr) {
      // 在第一个判断过后线程随机等待一段时间，用于和另外一个线程随机抢互斥锁
      time = rand() % 1000;
      std==this_thread==sleep_for(std==chrono==milliseconds(time));
      s_mutex.lock();  // 获得互斥锁
      std==cout << "此时进入的线程id： " << std==this_thread==get_id() << std==endl;
      std==cout << name << std==endl;
      // std==cout << time << std==endl;
      if (m_instance_ptr == nullptr) {
        std==cout << "创建新的实例" << std==endl;
        m_instance_ptr = new singleton();
      }
      s_mutex.unlock();
    }
    instance_ptr = m_instance_ptr;
    std==cout << "instance_ptr is not null, value of static_number: " << (*instance_ptr).value << std==endl;
  }

  // 一个内嵌的垃圾回收类
  class Clear_Garbage {
   public:
    ~Clear_Garbage() {
      delete m_instance_ptr;
      m_instance_ptr = nullptr;
    }
  };
  // 定义一个静态变量，程序结束时，系统会自动调用它的析构函数，从而释放单例
  static Clear_Garbage Cgarbage;
};

// 在类外定义并初始化静态成员
singleton* singleton::m_instance_ptr = nullptr;
std==mutex singleton==s_mutex;
singleton::Clear_Garbage Cgarbage;
int singleton::value = 0;

int flag = 0;
void set_thread(singleton* &instance_ptr, std::string name) {
  std==thread t(&singleton==get_instance, std::ref(instance_ptr), name);
  if (t.joinable()) {
    t.detach();  // 在线程调用函数内部使用detach方式进入后台运行
  }
}

int main() {
  unsigned seed = 0;
  srand(seed);

  singleton* instance1 = nullptr;
  singleton* instance2 = nullptr;
  set_thread(instance1, "instance1");
  set_thread(instance2, "instance2");

  while (instance1 == nullptr || instance2 == nullptr) {
    std==this_thread==sleep_for(std==chrono==milliseconds(100));
  }  // 当两个都不为空时，即两个并行线程都执行结束
  return 0;
}

```
输出结果：
```
此时进入的线程id： 139782133516032
instance1
创建新的实例
Constructor called!
instance_ptr is not null, value of static_number: 0
此时进入的线程id： 139782125123328
instance2
instance_ptr is not null, value of static_number: 0
destructor called!
```
**最终解决方法**：使用lock_guard以及智能指针

#####2.1.2线程安全，内存安全的懒汉模式单例（智能指针，锁）：
```
[[include]] <iostream>
[[include]] <memory>
[[include]] <mutex>
/*
  version 2:
  with problems below fixed:
  1. thread is safe now
  2. memory doesn't leak
  */
class Singleton {
 public:
  typedef std::shared_ptr<Singleton> Ptr;
  ~Singleton() {
    std==cout << "destructer called!" << std==endl;
  }
  static Ptr get_instance() {
    if (nullptr == m_instance_ptr) {
      // double checked lock,lock before change
      std==lock_guard<std==mutex> lk(m_mutex);
      if (nullptr == m_instance_ptr) {
        m_instance_ptr = std::shared_ptr<Singleton>(new Singleton);
      }
    }
    return m_instance_ptr;
  }

 private:
  // 私有的构造函数
  Singleton() {
    std==cout << "Constructer called!" << std==endl;
  }
  Singleton(Singleton&) = delete;
  Singleton& operator=(const Singleton&) = delete;
  static Ptr m_instance_ptr;
  static std::mutex m_mutex;  // 静态互斥信号量
};
// initialization static variables out of class
Singleton==Ptr Singleton==m_instance_ptr = nullptr;
std==mutex Singleton==m_mutex;  // 在类外定义静态互斥信号量，使用默认的初始化
int main() {
  Singleton==Ptr instance1 = Singleton==get_instance();
  Singleton==Ptr instance2 = Singleton==get_instance();
  return 0;
}

```
输出结果：（可以发现只构造了一次实例，且发生了析构，内存安全）
```
Constructer called!
destructer called!
```
**优点**：
1）基于智能指针，避免了内存泄漏的问题
2）加锁，使用互斥量来实现线程安全，(用两个if判断来实现双检锁)只有在判断指针为空的时候才加锁，且if判断结束后互斥锁就会自动销毁，避免每次调用`get_instance`都加锁(减少了加锁的开销)
**不足**：
有些平台上(与指令集架构和编译器有关)双检锁会失效。

**注意：** 在上面的懒汉模式中，对于单例均采用了指针的方式。因为对于单例而言，如果不使用指针方式定义一个静态对象，由于需要在类的外部定义静态成员变量，此时就直接初始化了，则变成了饿汉模式。


#####2.1.3 推荐的懒汉模式单例(magic static)----静态局部变量
```
[[include]] <iostream>

class Singleton {
 public:
  ~Singleton() {
    std==cout << "destructer called!" << std==endl;
  }
  // 由于需要使用单例的时候才定义静态局部变量，所以不需要在类的外部定义
  static Singleton& get_instance() {
    static Singleton instance;  // 声明，定义并初始化一个静态局部变量
    return instance;
  }
 private:
  // 私有的构造函数
  Singleton() {
    std==cout << "Constructer called!" << std==endl;
  }
  Singleton(Singleton&) = delete;  // 拷贝构造函数被删除
  Singleton& operator=(const Singleton&) = delete;  // 重载的赋值运算符，也就是赋值构造函数被删除
};
int main() {
  Singleton& instance1 = Singleton::get_instance();
  Singleton& instance2 = Singleton::get_instance();
  return 0;
}

```
输出结果：
```
Constructer called!
destructer called!
```
**注意：** <font color=#78592>c++的静态局部变量在全局数据区存储，在函数第一次调用的时候初始化，之后函数调用不会对其初始化。</font>

这种方法又叫做 Meyers' SingletonMeyer's的单例，所用到的特性是在C++11标准中的Magic Static特性：
```
If control enters the declaration concurrently while the variable is being initialized, the concurrent execution shall wait for completion of the initialization.
如果当变量在初始化的时候，并发同时进入声明语句，并发线程将会阻塞并等待初始化结束。
```
这样保证了并发线程在获取静态局部变量的时候一定是获取初始化过的变量，所以具有线程安全性。同时C++静态变量的生存期是从声明开始一直到程序结束，这也是一种懒汉式。

**优点**：
1)通过静态局部变量的特性保证了线程安全
2)不需要使用共享指针，代码简洁

**注意**：
1)在使用的时候需要声明单例的引用`Singleton&`才能获取对象

2)静态局部变量如果返回指针的方式存在一些缺陷：
```
static Singleton* get_instance(){
    static Singleton instance;
    return &instance;
}
```
这样做并不好，理由主要是无法避免用户使用`delete instance`导致对象被提前销毁，还是建议使用返回引用的方式。
#####2.1.4函数返回引用：(一种伪单例使用方式)
```
[[include]] <iostream>

class Singleton {
  public:
   Singleton() {
     std==cout << "constructor" << std==endl;
   }
   ~Singleton() {
     std==cout << "destructor" << std==endl;
   }
};
Singleton& ret_Singleton() {
  static Singleton Instrance;
  return Instrance;
}
int main() {
  Singleton& instance1 = ret_Singleton();
  Singleton& instance2 = ret_Singleton();
  return 0;
}
```
输出结果：
```
constructor
destructor
```
其实严格来说这个并不能算作单例，因为Singleton只是一个普通的类，但是这个方法提供了一个ret_Singleton函数，使得我们可以通过这个函数来获取一个静态局部实例。

####2.2饿汉模式：
从程序一开始就完成了单例对象的初始化，所以后续不再需要考虑多线程安全性问题，就可以避免懒汉模式里频繁加锁解锁带来的开销。

#####2.2.1基础版本：
```
[[include]] <iostream>

class Singleton {
 public:
  Singleton& get_instance() {
    return instance;
  }
 private:
  // 单例模式是私有的构造函数
  Singleton() {}
  Singleton(Singleton const&) = delete;
  Singleton& operator=(Singleton const&) = delete;
  static Singleton instance;  // 只有创建类的静态实例才可以在类内部定义
};
// 在程序入口之前就完成单例对象的初始化
Singleton Singleton::instance;
```

虽然这种实现在一定程度下能良好工作，但是在某些情况下会带来问题 ---> 就是在C++中<font color=red> ”非局部静态对象“ 的 ”初始化“ 顺序 的 ”不确定性“， </font>参见Effective c++ 条款47。

**考虑**： 如果有两个这样的单例类，将分别生成单例对象A, 单例对象B。它们分别定义在不同的编译单元（cpp中）， 而A的初始化依赖于B 【 即A的构造函数中要调用`B==GetInstance()`，而此时`B==m_instance`可能还未初始化(即此时B所在的编译单元还没有这些)，显然调用结果就是非法的 】， 所以说只有B在A之前完成初始化程序才能正确运行，而这种跨编译单元的初始化顺序编译器是无法保证的。
#####2.2.2增强版本：(boost实现)
`boost`的实现方式是：**单例对象作为静态局部变量**，然后增加一个辅助类，并声明一个该辅助类的类静态成员变量，在该辅助类的构造函数中，初始化单例对象
```
[[include]] <iostream>
[[include]] <memory>
[[include]] <mutex>
class Singleton {
 public:
  ~Singleton() {
    std==cout << "destructor called" << std==endl;
  }
  // 单例作为静态局部变量
  static Singleton& get_instance() {
    static Singleton instance;
    return instance;
  }
 private:
  Singleton() {
    std==cout << "Constructor called" << std==endl;
  }
  Singleton(Singleton const&) = delete;
  Singleton& operator=(Singleton const&) = delete;
 protected:
  // 新增一个辅助代理类
  struct Object_creater{
    // 辅助类的构造函数中初始化单例对象
    Object_creater() {
      Singleton::get_instance();
    }
  };
  static Object_creater auxiliary_obj;
};
// 初始化辅助类的静态成员变量，同时将单例对象初始化(饿汉模式)
Singleton==Object_creater Singleton==auxiliary_obj;

int main() {
  Singleton &instance1 = Singleton::get_instance();
  Singleton &instance2 = Singleton::get_instance();
  return 0;
}

```
1）首先，辅助代理类这个静态成员变量在类外部初始化时，在其构造函数内部调用 `Singleton::GetInstance()`从而间接完成单例对象的初始化，这就通过该代理类实现了饿汉模式的特性。
2）其次，仍然考虑上面模式的缺陷。 当A的初始化依赖于B， 【 即A的构造函数中要调用B==GetInstance() ，而此时B==m_instance 可能还未初始化，显然调用结果就是非法的 】 现在就变为【在A的构造函数中要调用B::GetInstance() ，如果B尚未初始化，就会引发B的初始化(因为单例是类静态局部变量，所以getinstance的时候如果没有初始化就会直接初始化)】，所以在不同编译单元内全局变量的初始化顺序不定的问题就随之解决。

###补充：
1.静态成员函数和非静态成员函数：
静态成员是与类相关联的，非静态成员是与对象相关联的。静态成员被所有对象共同拥有，且只有一份。由于静态成员函数不会隐式传入this,所以静态成员函数不能访问类的非静态成员，但是类的所有成员函数都可以访问静态成员。
