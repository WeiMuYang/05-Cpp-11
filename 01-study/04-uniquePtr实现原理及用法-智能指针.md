# [智能指针的用法](./)  [img](./img)     

> ######  _标签:_   ![](https://img.shields.io/badge/技术类-yellowgreen.svg) ![](https://img.shields.io/badge/C++11/14-用户手册-blue.svg) [![](https://img.shields.io/badge/链接-C++用法详解-brightgreen.svg)](https://www.cnblogs.com/tenosdoit/p/3456704.html/) [![](https://img.shields.io/badge/链接-代码文件-orange.svg)](../02-code/)        
>

## 0 auto_ptr    [官方文档](http://www.cplusplus.com/reference/memory/auto_ptr/?kw=auto_ptr)   

自动指针 [已弃用]
注意：从 C++11 开始不推荐使用此类模板。 unique_ptr 是一种具有类似功能的新工具，但具有改进的安全性（无虚假复制分配）、添加的功能（删除器）和对数组的支持。有关其他信息，请参阅 unique_ptr。

这个类模板为指针提供了一个有限的垃圾收集设施，通过允许指针在 auto_ptr 对象本身被销毁时自动销毁它们指向的元素。

auto_ptr 对象具有获取分配给它们的指针的所有权的特性：拥有一个元素的所有权的 auto_ptr 对象负责销毁它指向的元素，并在其自身被销毁时释放分配给它的内存。析构函数通过自动调用 operator delete 来完成此操作。

因此，没有两个 auto_ptr 对象应该拥有相同的元素，因为它们都会在某个时候试图破坏它们。当两个 auto_ptr 对象之间发生赋值操作时，所有权被转移，这意味着失去所有权的对象被设置为不再指向元素（它被设置为空指针）。   
<center>
<img src="./img/02-1.png" alt="02-1" style="zoom:50%;" />  
</center>  

## 1 unique_ptr   [官方文档](http://www.cplusplus.com/reference/memory/unique_ptr/)     

管理指针的存储，提供有限的垃圾收集设施，几乎没有内置指针的开销（取决于使用的删除器）。   

这些对象具有获取指针所有权的能力：一旦它们获得所有权，它们就会通过在某个时刻负责其删除来管理所指向的对象。    

unique_ptr 对象一旦自身被销毁，或者当它们的值因赋值操作或对 unique_ptr::reset 的显式调用而发生变化时，就会自动删除它们管理的对象（使用删除器）。   

unique_ptr 对象唯一地拥有它们的指针：没有其他设施应负责删除该对象，因此其他托管指针不应指向其托管对象，因为一旦需要，unique_ptr 对象就会删除其托管对象，而不考虑是否其他指针仍然指向同一个对象或不指向同一对象，从而使指向那里的任何其他指针指向无效位置。   

unique_ptr 对象有两个组件：

- 存储指针：指向它管理的对象的指针。这是在构造时设置的，可以通过赋值操作或调用成员重置来更改，并且可以使用成员 get 或 release 单独访问以进行读取。   
- 存储删除器：一个可调用对象，它采用与存储指针相同类型的参数，并被调用以删除托管对象。它在构造时设置，可以通过赋值操作更改，并且可以使用成员 get_deleter 单独访问。     

unique_ptr 对象通过运算符 * 和 ->（对于单个对象）或运算符 []（对于数组对象）提供对其托管对象的访问，从而复制有限的指针功能。出于安全原因，它们不支持指针算术，只支持移动赋值（禁用复制赋值）。       

常用的接口：

> ### std::[unique_ptr](http://www.cplusplus.com/reference/memory/unique_ptr/)::operator=  
>
> ```C++
> move assignment (1)	  unique_ptr& operator= (unique_ptr&& x) noexcept;
> assign null pointer (2)	   unique_ptr& operator= (nullptr_t) noexcept;
> type-cast assignment (3)	template <class U, class E>    unique_ptr& operator= (unique_ptr<U,E>&& x) noexcept; 
> copy assignment (deleted!) (4)	unique_ptr& operator= (const unique_ptr&) = delete;
> ```
>
>  ```C++
>  // unique_ptr::operator= example
>  #include <iostream>
>  #include <memory>
>  
>  int main () {
>    std::unique_ptr<int> foo;
>    std::unique_ptr<int> bar;
>  
>    foo = std::unique_ptr<int>(new int (101));  // rvalue
>  
>    bar = std::move(foo);                       // using std::move
>  
>    std::cout << "foo: ";
>    if (foo) std::cout << *foo << '\n'; else std::cout << "empty\n";
>  
>    std::cout << "bar: ";
>    if (bar) std::cout << *bar << '\n'; else std::cout << "empty\n";
>  
>    return 0;
>  }
>  ```
>
> 



```C++
#include <iostream> 
#include <string>
using namespace std;

class Test  {
public:
    Test(string s)
    {
        str = s;
        cout << "Test create\n";
    }
    ~Test()
    {
        cout << "Test delete:" << str << endl;
    }
    string& getStr()
    {
        return str;
    }
    void setStr(string s)
    {
        str = s;
    }
    void print()
    {
        cout << str << endl;
    }
private:
    string str;
};

int main()
{
    auto_ptr<Test> ptest(new Test("123"));
    ptest->setStr("hello ");
    ptest->print();
    ptest.get()->print();
    ptest->getStr() += "world !";
    (*ptest).print();
    ptest.reset(new Test("123"));
    ptest->print();
    return 0;
}
```

