## g++

### 一些参数说明

参考：
http://www.cnblogs.com/lidan/archive/2011/05/25/2239517.html

1. -O：有五种设置：-O0、-O1、-O2、-O3和-Os。表示优化等级。
2. -Wall：一般使用该选项，允许发出GCC能够提供的所有有用的警告。也可以用-W{warning}来标记指定的警告。
3. -c：只激活预处理,编译,和汇编,也就是他只把程序做成obj文件(.o文件)。注意是小写c。
4. -fmessage-length=0：默认情况下，GNU工具链编译过程中，控制台输出的一行信息是不换行的，这样，当输出信息过长时(如编译错误时的信息)，会导致你无法看到完整的输出信息，加入-fmessage-length=0后，输出信息会根据控制台的宽度自动换行，这样就能看全输出信息了。
5. -o：输出文件名。-o test.o或-otest.o，也就是没空格也可以。注意是小写o。
6. -glevel：请求生成调试信息，同时用level指出需要多少信息，默认的level值是2。
7. -Idir：指定#include的查找目录(dir)。
8. -Ldir：连接时查找库的目录(dir)。
9. -llibrary：指定要使用的库，注意库的命名规则。

## 指针数组和数组指针

### 指针数组
```cpp
int *q[3]; //q是数组，数组元素是指针，3个int指针
```

### 数组指针
```cpp
int m[3][4]={0,1,2,3,4,5,6,7,8,9,10,11};
int (*p)[4]; //p是指针，指向一维数组,每个一维数组有4个int元素
int p[][4]; //这个和上面一样意思
```

example：
```cpp
#include <iostream>

using namespace std;

int main()
{
    int m[3][4]={0,1,2,3,4,5,6,7,8,9,10,11};
    int (*p)[4];
    p = &m[0];
    cout << (*p)[3]; //3
    return 0;
}
```

## const

### 指针使用const
1. 指针本身是常量不可变(const修饰指针本身)：`char * const pContent;`
2. 指针所指向的内容是常量不可变，并不是说所指向的值是个常量，而是对指针而言，这个值是常量不可变，即不可以用这个指针改变这个值(const修饰所指变量)：`const char * pContent;`
3. 两者都不可变：`const char * const pContent;`

## 模版
编译器帮你做了替换的工作，编译器在编译时，把模版中的类型替换成和调用相应的类型，生成相应函数。是编译时的多态。在多文件编译链接时，声明和定义分开时，需要注意，看下面例子。

例子：
```cpp
//Example.h  
template<class T>  
class Test {  
    public:  
        Test(T a);  
};  
  
//Example.cpp  
#include "Example.h"  
  
template<class T>  
Test<T>::Test(T a) {  
    //do nothing...  
}  
  
template class Test<int>; //显式实例化

//main.cpp
//省略，使用了Test<int>
```

Test类的声明和定义不在同一个文件，如果没有Example.cpp最后一句的显式实例化，会出错。在编译Example.cpp时，编译器根本不知道在main.cpp中使用了`Test<int>`类（因为编译是各个文件独立进行的），也就没有实例化它。

## 智能指针
1. auto_ptr和unique_ptr在赋值时会转让所有权，导致原本的指针不再指向那个内存。当把一个unique_ptr赋给另一个时，如果源unique_ptr是个临时右值，编译器允许这样做；如果源unique_ptr将存在一段时间，编译器将禁止。(C++ Primer Plus(第六版) P673)
2. auto_ptr和shared_ptr只能用new，而unique_ptr可以用new[];

## 不归类问题集合

### 返回函数的局部变量的引用或指针会出问题

例1：
```cpp
int & test()
{
    int a = 1;
    return a;
}
```

例2：
```cpp
int * test()
{
    int a = 1;
    return &a;
}
```

例3：
```cpp
int & test()
{
    int a = 1;
    int & ra = a;
    return ra;
}
```

例4：
```cpp
int * test()
{
    int a = 1;
    int * pa = &a;
    return pa;
}
```

例1和例2会警告，并且执行时就会死掉。例3和例4可以正常运行，并且得到的结果可能是正确的，但其实也是错误的。看下面例子，这里在调用例3的test后再调用test2，栈就被重新改写了，这时打印出的pint是不可预测的（虽然这里可能会打印出三，猜测是test2中的b），期望打印出的是test中的1。

```cpp
int test2()
{
    int a = 2;
    int b = 3;
    int c = 4;
    return 5;
}

int main()
{
    int & pint = test();  //例3的test函数
    test2();
    cout <<  pint << endl;
}
```

### cin.getline(char*, int)和cin.get(char*, int)、cin.get()
1. cin.getline(char*, int)：输入的字符串如果大于等于（等于也不行，因为回车还要占个换行符）设定的长度（第二个参数），程序会直接退出。如果输入的字符串小于指定长度，那么会读入换行符，并且把换行符舍弃。
2. cin.get(char*, int)：如果大于等于指定长度，不会像getline那样程序直接退出，再次调用get的时候，会从上次的输入处继续读取。见下面例一。如果小于指定长度，那么get会读到换行符前面的那个字符为止，换行符会留在输入队列，下次再调用get，它就读不到东西了，因为它一开始就遇到换行符了。
3. cin.get()：无参数。读取下一个字符，即使是换行符。可以循环调用，以便舍弃输入队列中过多的字符。如果肯定上次的cin.get(char*, int)调用时，输入没有超出长度，那么调用一次cin.get()就行了。

例一：
```cpp
//输入12345
cin.get(name1, 5); //name1 = "1234"
cin.get(name2, 5); //name2 = "5"
```
