[[c]]++观察者模式
##一、简介
###1.什么是观察者模式：
观察者模式算是应用最多，使用最广的模式之一。 `Observer`的一个实例 `Model/View/Control（MVC）` 结构在系统开发架构设计中有着很重要的地位和意义， MVC实现了业务逻辑和表示层的解耦。Observer 模式要解决的问题为： 建立一个一（Subject）对多（Observer） 的依赖关系， 并且做到当“一” 变化的时候， 依赖这个“一”的多也能够同步改变。
在GOF的《设计模式:可复用面向对象软件的基础》一书中对观察者模式是这样说的：定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。当一个对象发生了变化，关注它的对象就会得到通知；这种交互也称为发布-订阅(publish-subscribe)。目标是通知的发布者，它发出通知时并不需要知道谁是它的观察者。
###2.观察者模式的性质：
1、目标与观察者之间的关系：目标与观察者之间是一对多的关系。
2、单向依赖：只有目标知道什么时候通知观察者。
3、命名模式：又称为发布-订阅模式，目标接口定义后面跟subject，观察者接口定义后面跟observer，观察者接口的更新方法建议为update，方法的参数是根据需要定义的。
4、触发通知的时机：先改变后通知。
###3.观察者的实现方式：
1.推模型：目标对象主动向观察者推送目标的详细信息，推送的信息通常是目标对象的部分或者全部信息。
2.拉模型：目标对象在通知的时候只传递少量信息，如果观察者需要更具体的信息，由观察者主动到目标对象获取，相当于是观察者主动在目标对象中拉数据。

**两种模式比较：**
1.推模型是假设目标对象知道观察者所需的数据，而拉模型是直接通知观察者，观察者需要什么数据直接从目标对象获取。
2.推模型会使观察者对象难以复用，而拉模型下update方法的参数是模型本身，基本可以适应各种情况的需要。
###4.观察者模式的优缺点：
1.优点：观察者模式实现了观察者和目标之间的抽象耦合，有触发机制，支持广播通信
2.缺点：可能会引起无必要的操作1.

##观察者模式的实现：
1.简单的UML类图：
![observer](/assets/observer.png)
Subject（目标）
——目标知道它的观察者。可以有任意多个观察者观察同一个目标；
——提供注册和删除观察者对象的接口。

Observer（观察者）
——为那些在目标发生改变时需获得通知的对象定义一个更新接口。

ConcreteSubject（具体目标）
——将有关状态存入各ConcreteObserver对象；
——当它的状态发生改变时，向它的各个观察者发出通知。

ConcreteObserver（具体观察者）
——维护一个指向ConcreteSubject对象的引用；
——存储有关状态，这些状态应与目标的状态保持一致；
——实现Observer的更新接口以使自身状态与目标的状态保持一致。

**观察者模式的协作方式：**
1.当ConcreteSubject发生任何可能导致其观察者与其本身状态不一致的改变时，它将通知它的各个观察者；
2.在得到一个具体目标的改变通知后，ConcreteObserver对象可向目标对象查询信息。ConcreteObserver使用这些信息以使它的状态与目标对象的状态一致(拉模型)

示例1：
```
[[include]] <iostream>
[[include]] <list>
[[include]] <memory>
using namespace std;

// 定义observer和subkect的虚基类
class Observer;
class Subject {
 public:
  virtual void Attach(Observer*) = 0;
  virtual void Detach(Observer*) = 0;
  virtual void Notify() = 0;
};
class Observer {
 public:
  // 只有在subject的notify调用的时候，notify中对每个observer进行update
  virtual void Update(int) = 0;
};


class ConcreteObserver1 : public Observer {
 public:
  ConcreteObserver1(Subject* psubject) : m_pSubject(psubject){}
  void Update(int value)
  {
    cout << "ConcreteObserver1 get the update. New State:" << value << endl;
  }

 private:
  Subject* m_pSubject = nullptr;
};

class ConcreteObserver2 : public Observer {
 public:
  ConcreteObserver2(Subject* psubject) : m_pSubject(psubject){}
  void Update(int value)
  {
    cout << "ConcreteObserver2 get the update. New State:" << value << endl;
  }

 private:
  Subject* m_pSubject = nullptr;
};

class ConcreteSubject : public Subject {
 public:
  void Attach(Observer* pPbserver);
  void Detach(Observer* pObserver);
  void Notify();

  void setState(int state) {
    m_state = state;
  }
 private:
  list<Observer*> m_observer_list;
  int m_state = 0;
};
void ConcreteSubject::Attach(Observer* pObserver) {
  m_observer_list.push_back(pObserver);
}
void ConcreteSubject::Detach(Observer *pObserver)
{
  m_observer_list.remove(pObserver);
}
void ConcreteSubject::Notify() {
  list<Observer*>::iterator it = m_observer_list.begin();
  while (it != m_observer_list.end()) {
    (*it)->Update(m_state);
    it++;
  }
}
int main() {
  // create Subject
  ConcreteSubject *pSubject = new ConcreteSubject();
  // create observer
  ConcreteObserver1 *pObserver1 = new ConcreteObserver1(pSubject);
  ConcreteObserver2 *pObserver2 = new ConcreteObserver2(pSubject);

  // create the state
  pSubject->setState(13);

  // register the observer
  pSubject->Attach(pObserver1);
  pSubject->Attach(pObserver2);

  pSubject->Notify();

  pSubject->setState(18);
  pSubject->Notify();

  pSubject->Detach(pObserver1);
  pSubject->Notify();

  delete pSubject;
  delete pObserver1;
  delete pObserver2;
}
```
结果：
```
ConcreteObserver1 get the update. New State:13
ConcreteObserver2 get the update. New State:13
ConcreteObserver1 get the update. New State:18
ConcreteObserver2 get the update. New State:18
ConcreteObserver2 get the update. New State:18
```
