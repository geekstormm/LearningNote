# C++ tips

## C++ split

```C++
#include <sstream>
#include <string>
stringstream str("bla,bla,asdd,asdfgf");
string s;
vector<string> res;

while (getline(str, s, ' ')) {
    res.push_bacl(s);
    cout << s << endl;
}
```

## C++重载、重写（覆盖）、重定义

- 重载：在同一作用域里面，某个函数或者运算符进行多次定义。子类无法重载父类的函数，父类重名函数将会被覆盖。重载在**编译阶段**根据函数签名决定调用哪个函数。
- 重写和重定义：
  - 1、肯定发生在子类和父类之间。
  - 2、父类和子类的函数必须有完全相同的原型
  - 3、使用virtual声明后产生多态，没用virtual就称为重定义
  - 多态是在**运行期间**根据具体对象的类型决定函数调用

## 虚函数的实现原理

