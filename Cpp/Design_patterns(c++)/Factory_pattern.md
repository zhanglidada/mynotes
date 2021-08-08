#C++工厂模式
##一、简介
###1.什么是工厂模式
```
这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，而是通过使用一个共同的接口来指向新创建的对象。
```
简单来说，**工厂模式使用了C++多态的特性**，将存在继承关系的类，通过一个工厂类创建对应的子类（派生类）对象。在项目复杂的情况下，可以便于子类对象的创建。可以给系统带来等大的可扩展性以及更少的修改量。

###2.为什么需要工厂模式
####1). 工厂模式是为了解耦合.
把对象的创建和使用分开。比如`Class A`想要使用`Class B`,那么A只需要调用B的方法，至于B的实例化就交付给工厂类。
####2).工厂模式可以降低代码重复率。
如果创建对象B的过程很复杂，需要一定的代码量，而且很多地方都要用到，那么就会有很多的重复代码。我们可以这些创建对象B的代码放到工厂里统一管理。既减少了重复代码，也方便以后对B的创建过程的修改维护。(当然，也可以把这些创建过程的代码放到类的构造函数里，同样可以降低重复率，而且构造函数本身的作用也是初始化对象。不过，这样也会导致构造函数过于复杂，做的事太多，不符合设计原则。)

由于创建过程都由工厂统一管理，所以发生业务逻辑变化，不需要找到所有需要创建B的地方去逐个修正，只需要在工厂里修改即可，降低维护成本。同理，想把所有调用B的地方改成B的子类B1，只需要在对应生产B的工厂中或者工厂的方法中修改其生产的对象为B1即可，而不需要找到所有的`new B()`改为`new B1()`。
####3).工厂模式更加面向对象，增加了代码的封装性，减少逻辑的暴露。
因为工厂管理了对象的创建逻辑，使用者不需要知道创建的逻辑，只管使用即可，减少了使用的风险。

####4.)工厂模式使用场景
a.对象的创建过程/实例化准备工作很复杂，需要初始化很多参数、查询数据库等。

b.类本身有很多子类，这些类的创建过程在业务中容易发生改变，或者对类的调用容易发生改变。

**举例：**
如果一个类有多个构造方法（构造的重写），我们可以将它抽出来，放到工厂中，一个构造方法对应一个工厂方法，并命名一个友好的名字，这样我们就不再只是根据参数的不同来判断，而是可以根据工厂的方法名来直观判断将要创建的对象的特点。这对于使用者来说，体验比较好。
##二、工厂模式的使用
###1.工厂模式注意事项
作为一种创建类模式，在任何需要生成复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方就是复杂对象适合使用工厂模式，而简单对象，特别是只需要通过 new 就可以完成创建的对象，无需使用工厂模式。如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度。
###2.工厂模式实现方式
####1)简单工厂模式
#####1.具体情形：
鞋厂可以指定生产耐克、阿迪达斯和李宁牌子的鞋子。哪个鞋炒的火爆，老板就生产哪个，看形势生产。
#####2.UML图：
![1](/assets/1_g3o2n1mm5.jpg)

#####3.简单工厂模式的结构组成：
工厂类(`ShoesFactory`)：工厂模式的核心类，会定义一个用于创建指定的具体实例对象的接口。

抽象产品类(`Shoes`)：是具体产品类继承的父类或实现的接口。

具体产品类(`NiKeShoes\AdidasShoes\LiNingShoes`)：工厂类所创建的对象就是此具体产品实例。

#####4.简单工厂模式的特点：
工厂类(`ShoesFactory`)封装了创建具体产品对象的函数。

#####5.简单工厂模式的缺陷：
扩展性非常差，新增产品的时候，需要去修改工厂类。

#####6.简单工厂模式的代码：
1) `Shoes`为鞋子的抽象类（基类），接口函数为`Show()`，用于显示鞋子广告。

2) `NiKeShoes、AdidasShoes、LiNingShoes`为具体鞋子的类，分别是耐克、阿迪达斯和李宁鞋牌的鞋，它们都继承于Shoes抽象类。

**简单工厂的例子：**
```
#include <iostream>
#include <vector>
using namespace std;

// 抽象父类
class Shoes {
 public:
  // 声明一个虚析构函数
  virtual ~Shoes() {};
  virtual void Show() = 0;  // 此时show在子类中即使没有具体实现也不会报错
};
class NikeShoes : public Shoes {
 public:
  void Show() {
    cout << "我是耐克球鞋，我的广告语：Just do it" << std::endl;
  }
};
// 阿迪达斯鞋子
class AdidasShoes : public Shoes {
 public:
  void Show() {
    cout << "我是阿迪达斯球鞋，我的广告语:Impossible is nothing" << std::endl;
  }
};
// 李宁鞋子
class LiNingShoes : public Shoes {
 public:
  void Show() {
    cout << "我是李宁球鞋，我的广告语：Everything is possible" << std::endl;
  }
};
// 枚举类型
enum SHOES_TYPE {
  NIKE,
  LINING,
  ADIDAS
};

// shoes_factory为工厂类，类里实现根据鞋子类型创建对应鞋子产品对象的CreateShoes函数
class Shoes_Factory {
 public:
  // 根据鞋子类型创建对应的鞋子对象
  Shoes* Create_shoes(SHOES_TYPE type) {
    Shoes * shoes;
    switch (type) {
      case NIKE:
        shoes =  new NikeShoes();
        break;
      case LINING:
        shoes =  new LiNingShoes();
        break;
      case ADIDAS:
        shoes =  new AdidasShoes();
    default:
      shoes = nullptr;
      break;
    }
    return shoes;
  }
};
int main() {
  // 构造工厂对象
  Shoes_Factory shoesFactory;

  // 从鞋工厂生产NIKE鞋
  Shoes* NikeShoes = shoesFactory.Create_shoes(NIKE);
  if (NikeShoes != nullptr) {
    NikeShoes->Show();
    // 释放资源
    delete NikeShoes;
    NikeShoes = nullptr;
  }
  return 0;
}
```
####2)工厂方法模式
#####1.具体情形：
现各类鞋子抄的非常火热，于是为了大量生产每种类型的鞋子，则要针对不同品牌的鞋子开设独立的生产线，那么每个生产线就只能生产同类型品牌的鞋。
![factory2](/assets/factory2.jpg)

#####2.工厂方法模式的结构组成：
2.1 抽象工厂类(`ShoesFactory`)：工厂方法模式的核心类，提供创建具体产品的接口，由具体工厂类实现。
2.2 具体工厂类:(`NiKeProducer\AdidasProducer\LiNingProducer`):继承于抽象工厂，实现创建对应具体产品对象的方式。
2.3 抽象产品类(`Shoes`)：它是具体产品继承的父类（基类）。
2.4 具体产品类(`NiKeShoes\AdidasShoes\LiNingShoes`)：具体工厂所创建的对象，就是此类。

#####3.工厂方法模式的特点：
3.1 工厂方法模式抽象出了工厂类，提供创建具体产品的接口，交由子类去实现。
3.2 工厂方法模式的应用并不只是为了封装具体产品对象的创建，而是要**把具体产品对象的创建放到具体工厂类实现**。

#####4.工厂方法模式的缺陷：
4.1 每新增一个产品，就需要增加一个对应的产品的具体工厂类。相比简单工厂模式而言，工厂方法模式需要更多的类定义。
4.2 一条生产线只能一个产品。

#####5.工厂方法模式的代码：
`ShoesFactory`抽象工厂类，提供了创建具体鞋子产品的纯虚函数。
`NiKeProducer、AdidasProducer、LiNingProducer`具体工厂类，继承持续工厂类，实现对应具体鞋子产品对象的创建。

**工厂方法模式的例子：**
```
#include <iostream>
#include <vector>
using namespace std;

// 抽象父类
class Shoes {
 public:
  // 声明一个虚析构函数
  virtual ~Shoes() {};
  virtual void Show() = 0;  // 此时show在子类中即使没有具体实现也不会报错
};
class NikeShoes : public Shoes {
 public:
  void Show() {
    cout << "我是耐克球鞋，我的广告语：Just do it" << std::endl;
  }
};
// 阿迪达斯鞋子
class AdidasShoes : public Shoes {
 public:
  void Show() {
    cout << "我是阿迪达斯球鞋，我的广告语:Impossible is nothing" << std::endl;
  }
};
// 李宁鞋子
class LiNingShoes : public Shoes {
 public:
  void Show() {
    cout << "我是李宁球鞋，我的广告语：Everything is possible" << std::endl;
  }
};
// 枚举类型
enum SHOES_TYPE {
  NIKE,
  LINING,
  ADIDAS
};

// shoes_factory为工厂类，类里实现根据鞋子类型创建对应鞋子产品对象的CreateShoes函数
class Shoes_Factory {
 public:
  virtual Shoes* Create_Shoes() = 0;
  virtual ~Shoes_Factory() {};  // 虚析构函数用于删除子类中的指针
};

class NiKeProducer : public Shoes_Factory {
 public:
  Shoes* Create_Shoes() {
    return new NikeShoes();
  }
};
class LiNingProducer : public Shoes_Factory {
 public:
  Shoes* Create_Shoes() {
    return new LiNingShoes();
  }
};
class AdidasProducer : public Shoes_Factory {
 public:
  Shoes* Create_Shoes() {
    return new AdidasShoes();
  }
};
int main() {
  Shoes_Factory* NikeProdue = new NiKeProducer();
  Shoes* NikeShoes = NikeProdue->Create_Shoes();
  NikeShoes->Show();

  // 释放内存,先删除子类的实例
  delete NikeShoes;
  delete NikeProdue;
  return 0;
}
```
####3)抽象工厂模式
#####1.具体情形：
鞋厂为了扩大了业务，不仅只生产鞋子，把运动品牌的衣服也一起生产了。

#####2.UML图：
![factory3](/assets/factory3.jpg)

#####3.抽象工厂模式的结构组成（和工厂方法模式一样）：
3.1 抽象工厂类（`ShoesFactory`）：工厂方法模式的核心类，提供创建具体产品的接口，由具体工厂类实现。

3.2 具体工厂类（`NiKeProducer`）：继承于抽象工厂，实现创建对应具体产品对象的方式。

3.3 抽象产品类（`Shoes\Clothe`）：它是具体产品继承的父类（基类）。

3.4 具体产品类（`NiKeShoes\NiKeClothe`）：具体工厂所创建的对象，就是此类。

#####3.抽象工厂模式的特点：
提供一个接口，可以创建多个**产品族**中的产品对象。如创建耐克工厂，则可以创建耐克鞋子产品、衣服产品、裤子产品等。

#####4.抽象工厂模式的缺陷：
同工厂方法模式一样，新增产品时，都需要增加一个对应的产品的具体工厂类。

#####5.抽象工厂模式的代码：
`Clothe`和`Shoes`，分别为衣服和鞋子的抽象产品类。
`NiKeClothe`和`NiKeShoes`，分别是耐克衣服和耐克衣服的具体产品类。

**具体代码实现：**
```
#include <iostream>
#include <vector>
using namespace std;

// 抽象父类
class Shoes {
 public:
  // 声明一个虚析构函数
  virtual ~Shoes() {};
  virtual void Show() = 0;  // 此时show在子类中即使没有具体实现也不会报错
};
class Clothe {
 public:
  virtual ~Clothe() {};
  virtual void Show()  = 0;
};

// nike鞋子
class NikeShoes : public Shoes {
 public:
  void Show() {
    cout << "我是耐克球鞋Just do it" << std::endl;
  }
};
// 耐克衣服
class NiKeClothe : public Clothe {
 public:
  void Show() {
    std::cout << "我是耐克衣服hhhhh" << std::endl;
  }
};


// Factory为抽象工厂，提供了创建鞋子CreateShoes()和衣服产品CreateClothe()对象的接口。
class Factory {
 public:
  virtual Shoes* CreateShoes() = 0;
  virtual Clothe* CreateClothe() = 0;
  virtual ~Factory() {};
};
// NiKeProducer为具体工厂，实现了创建耐克鞋子和耐克衣服的方式。
class NikeProducer : public Factory{
 public:
  Shoes *CreateShoes() {
    return new NikeShoes();
  }

  Clothe *CreateClothe() {
    return new NiKeClothe();
  }
};

int main() {
  // ================ 生产耐克流程 ==================== //
  // 鞋厂开设耐克生产线
  Factory* NikeProduce = new NikeProducer();

  // 耐克生产线产出球鞋
  Shoes* Nikeshoes = NikeProduce->CreateShoes();
  // 耐克生产线产出衣服
  Clothe* NikeClothe = NikeProduce->CreateClothe();

  NikeClothe->Show();
  Nikeshoes->Show();

  // 释放资源
  delete Nikeshoes;
  delete NikeClothe;
  delete NikeProduce;
  return 0;
}
```

##三、总结：
**以上三种工厂模式，在新增产品时，都存在一定的缺陷。**
简单工厂模式，，需要去修改工厂类，这违背了开闭法则。
工厂方式模式和抽象工厂模式，都需要增加一个对应的产品的具体工厂类，这就会增大了代码的编写量。
