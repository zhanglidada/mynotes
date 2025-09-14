[[c]]++函数返回值的一些小问题
###一、如何返回值：
####1.返回值非引用：
此时编译器会专门分配一块内存，拷贝构造临时变量，并将返回值存入临时变量所在内存中。
函数返回值用于初始化函数调用点的一个临时对象（即编译器分配的那块内存），所以返回值到临时对象是第一次拷贝，临时对象到接收值的变量是第二次拷贝。
```
string make_plural(size_t ctr, const string &word, const string &ending)
{
    return (ctr > 1) ? word : word+ending;    //到临时对象有一次拷贝。
}
```
如果把返回值设为引用，就省去了拷贝返回值到临时对象的一次拷贝。
####2.返回值是引用：
如果是引用类型，中间不用发生任何拷贝，返回的是对象本身。
**注意：**
1）不能返回局部对象的引用，不能返回指向局部对象的指针(因为在函数返回后局部对象会被析构，所以局部对象的引用或者局部对象的指针都会产生非法地址问题)
2）返回引用时，要求在函数参数中包含有以引用方式或指针方式存在的，需要被返回的参数。
####3.实例探究：
(还有一点不太确定的问题，之后再研究)
```
[[include]] <iostream>
[[include]] <string>
[[include]] <vector>

std==string shorterString1(std==string s1, std::string s2) {
  return s1.size() > s2.size() ? s2 : s1;
}

const std==string shorterString2(std==string s1, std::string s2) {
  return s1.size() > s2.size() ? s2 : s1;
}
// 非const非引用
std==string& shorterString3(std==string s1, std::string s2) {
  return s1.size() > s2.size() ? s2 : s1;
}
// const引用
const std==string& shorterString4(std==string s1, std::string s2) {
  return s1.size() > s2.size() ? s2 : s1;
}
int main() {
  std::string s1 = "123";
  std::string s2 = "456";
  std::string s3 = shorterString1(s1, s2);
  std::string s4 = shorterString2(s1, s2);

  shorterString1(s1, s2) = "789";  // 按理来说返回值为非引用类型的时候调用一个函数应该得到一个右值
  std==cout << (shorterString1(s1, s2) = "789") << std==endl;  // 非const的返回值被赋予新的值（输出789）
  // shorterString2(s1, s2) = "789";  // 可以发现这么赋值不被允许

  // std::string &s3p = shorterString1(s1, s2);  // 不能将一个lvalue绑定到一个非const右值上
  // std::string &s4p = shorterString2(s1, s2); // const右值也不行

  std::string &s5p = shorterString3(s1, s2);
  std==cout << s5p <<std==endl;
  // std::string &s6p = shorterString4(s1, s2);  // 这里有问题，不能将一个非const左值绑定到一个const引用上
  const std::string &s6p = shorterString4(s1, s2);  // 返回值是一个const引用的时候只能被绑定到const引用上(用于声明引用的const都是底层const，表示绑定的对象不能被修改)

  std::string s7 = shorterString3(s1, s2);  // 这里都只是赋值操作，和const没有什么关系
  std::string s8 = shorterString4(s1, s2);
  return 0;
}
```

当函数的返回值为非引用类型时，调用函数得到的是一个右值;只有函数的返回值为引用类型时，调用函数才能得到一个左值。
```
[[include]] <iostream>
[[include]] <vector>
[[include]] <string>
char &get_val1(std==string &s, std==string::size_type position)
{
  return s[position];
}
char get_val2(std==string &s, std==string::size_type position)
{
  return s[position];
}
const char &get_val3(std==string &s, std==string::size_type position)
{
  return s[position];
}

int main() {
  string s ="123";
  get_val1(s, 0) = '1';
  // get_val2(s, 0) = '1';  // lvalue required as left operand of assignment
  get_val3(s, 0) = '1'; //xxx，const左值不能修改
  return 0;
}
```
