# c++运算符重载
## 1.重载函数调用运算符‘()’：
<font color=#98568>1). 如果类重载了函数调用运算符，则我们可以像使用函数一样使用该类的对象。**注意：函数调用运算符必须是成员函数，且一个类可以定义多个不同版本的调用运算符。**</font>
```
[[include]]<iostream>
using namespace std;
class test{
public:
    int operator()(int i){
        cout<<i<<endl;
        return i;
    }
};
int main(){
    test t;
    int p = t(7);
    return 0;
}
```

<font color=#5974>2). 函数调用运算符 () 可以被用于重载类的对象。当重载 () 时，我们不是创造了一种新的调用函数的方式，相反地，是创建一个可以传递任意数目参数的运算符函数。</font>**即，当我们调用一个类的对象时，比如it()，其实编译器在内部会将其转换为it.operator()**
```
[[include]]<iostream>
using namespace std;
class Weapon{
public:
    Weapon(){
        arrow = 0;
        arch = 0;
        cout<<"here is nothing left!"<<endl;
    }
    Weapon(int get,int buy){
        arrow = get;
        arch = buy;
        cout<<"we have borrowed some..."<<" arrow is: "<<arrow<<" and arch is: "<<arch<<endl;
    }
    Weapon operator()(int get,int have,int borrow){  // 重载了调用运算符‘（）’
        Weapon WE;
        WE.arch = get + have + 20;
        WE.arrow = have + borrow +100;
        return WE;
    }
    void DisplayWeapons(){
        cout<<"arrow is : "<<arrow<<" and arch is : "<<arch<<endl;
    }
private:
    int arch;
    int arrow;
};
int main(){
    Weapon w1(12,13);//这里是构造函数
    cout<<"The original weapons : "<<endl;
    w1.DisplayWeapons();

    //这里其实先使用默认构造函数创建了w2对象，然后w1(a,b,c)其实是一个函数，返回一个对象并赋值给了w2
    Weapon w2 = w1(10,10,10);//这里重载类的对象，是一个可以传递指定数目参数的运算符函数
    cout<<"Another weapons : "<<endl;
    w2.DisplayWeapons();

    return 0;
}


```
![26](/assets/26.png)

## 2.重载输入输出运算符
### 2.1重载目的：
c++中标准库本身已经对右移操作符`>>`和左移操作符`<<`操作符进行了重载，使其能够用于不同数据的输入输出，但是输入输出的对象只能是 C++ 内置的数据类型（例如 bool、int、double 等）和标准库所包含的类类型（例如 string、complex、ofstream、ifstream 等）。
如果自己定义了一种新的数据类型，需要用输入输出运算符去处理，那么就必须对它们进行重载。以 complex 类为例来演示输入输出运算符的重载。

假设 num1、num2 是复数，那么输出形式就是`cout<<num1<<num2<<endl`;

输入形式就是：`cin>>num1>>num2`;

由于`cout`是`ostream`类的对象,`cin`是`istream`类的对象，要想达到这个目标，就必须以全局函数（友元函数）的形式重载`<<`和`>>`，否则就要修改标准库中的类，这显然不是我们所期望的。
###2.2重载输入运算符`>>`
我们以全局函数的形式重载`>>`，使它能够读入两个`double`类型的数据，并分别赋值给复数的实部和虚部：
```
istream & operator>>(istream &in, complex &A){
    in >> A.m_real >> A.m_imag;
    return in;
}
```
`istream`表示输入流，`cin`是 `istream` 类的对象，只不过这个对象是在标准库中定义的。之所以返回 `istream` 类对象的引用，是为了能够连续读取复数，让代码书写更加漂亮，例如：
```
complex c1, c2;
cin >> c1 >> c2;
```

如果不返回引用，那就只能一个一个地读取了：
```
complex c1, c2;
cin >> c1;
cin >> c2;
```
另外，运算符重载函数中用到了 `complex` 类的 `private` 成员变量，必须在 `complex` 类中将该函数声明为友元函数：
```
friend istream & operator>>(istream & in , complex &a);
```
`>>`运算符可以按照下面的方式使用：
```
complex c;
cin >> c;
```
当输入`1.45 2.34`后，这两个小数就分别成为对象 `c` 的实部和虚部了。`cin >> c`;这一语句其实可以理解为：
```
operator<<(cin , c);
```
###2.3重载输出运算符`<<`
同样地，我们也可以模仿上面的形式对输出运算符`>>`进行重载，让它能够输出复数，请看下面的代码：
```
ostream & operator<<(ostream &out, complex &A){
    out << A.m_real <<" + "<< A.m_imag <<" i ";
    return out;
}
```
`ostream` 表示输出流，`cout` 是 `ostream` 类的对象。由于采用了引用的方式进行参数传递，并且也返回了对象的引用，所以重载后的运算符可以实现连续输出。比如：
```
cout << d1 << d2 << endl;
```
此时`cout << d1`;等价于：
```
operator<<(cout, d1);
```
为了能够直接访问 `complex` 类的 `private` 成员变量，同样需要将该函数声明为 `complex` 类的友元函数：
```
friend ostream & operator<<(ostream &out, complex &A);
```
代码：
```
[[include]] <iostream>
using namespace std;

class complex{
 public:
  complex(double real = 0.0, double imag = 0.0) : m_real(real), m_imag(imag){ }
  /*
    重载最基本的+，-，*，/
    返回的是新创建的对象的拷贝
   */
  friend complex operator+(const complex &A, const complex &B);
  friend complex operator-(const complex &A, const complex &B);
  friend complex operator*(const complex &A, const complex &B);
  friend complex operator/(const complex &A, const complex &B);
  /*
    重载+=，-=，*=，/=
    返回的是第一个对象本身
   */
  friend complex& operator+=(complex& A, const complex& B);
  friend complex& operator-=(complex& A, const complex& B);
  friend complex& operator*=(complex& A, const complex& B);
  friend complex& operator/=(complex& A, const complex& B);
  /*
    重载输入输出运算符
   */
  friend istream& operator>>(istream &in, complex &A);
  friend ostream& operator<<(ostream &out, const complex &A);
 private:
  double m_real;  // 复数的实部
  double m_imag;  // 复数的虚部
};
complex operator+(const complex &A, const complex &B) {
  complex C;
  C.m_real = A.m_real + B.m_real;
  C.m_imag = A.m_imag + B.m_imag;
  return C;
}
complex operator-(const complex &A, const complex &B) {
  complex C;
  C.m_real = A.m_real - B.m_real;
  C.m_imag = A.m_imag - B.m_imag;
  return C;
}
complex operator*(const complex &A, const complex &B) {
  complex C;
  C.m_real = A.m_real * B.m_real - A.m_imag * B.m_imag;
  C.m_imag = A.m_imag * B.m_real + A.m_real * B.m_imag;
  return C;
}
complex operator/(const complex &A, const complex &B) {
  complex C;
  double square = A.m_real * A.m_real + A.m_imag * A.m_imag;
  C.m_real = (A.m_real * B.m_real + A.m_imag * B.m_imag) / square;
  C.m_imag = (A.m_imag * B.m_real - A.m_real * B.m_imag) / square;
  return C;
}
complex& operator+=(complex &A, const complex &B){
  // A.m_real = A.m_real + B.m_real;
  // A.m_imag = A.m_imag + B.m_imag;
  A.m_real += B.m_real;
  A.m_imag += B.m_imag;
  return A;
}
istream& operator>>(istream &in, complex &A) {
  in >> A.m_real >> A.m_imag;
  return in;  // 为了可以连续输入
}
ostream& operator<<(ostream &out, const complex &A) {
  out << A.m_real << " + " << A.m_imag << " i " << endl;
  return out;
}
int main() {
  complex c1, c2, c3;
  cin >> c1 >> c2;

  c3 = c1 + c2;
  cout << "c1 + c2 = " << c3 << endl;
  c3 = c1 - c2;
  cout << "c1 - c2 = " << c3 << endl;
  c3 = c1 * c2;
  cout << "c1 * c2 = " << c3 << endl;
  c3 = c1 / c2;
  cout << "c1 / c2 = " << c3 << endl;

  c1 += c2;
  cout << "c1 += c2, c1 is : " << c1 << endl;

  return 0;
}
```

输入数据：
```
2.4 3.6
4.8 1.7
```
输出结果：
```
c1 + c2 = 7.2 + 5.3 i

c1 - c2 = -2.4 + 1.9 i

c1 * c2 = 5.4 + 21.36 i

c1 / c2 = 0.942308 + 0.705128 i

c1 += c2, c1 is : 7.2 + 5.3 i
```
