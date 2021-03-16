# 条款2：尽量以const, enum, inline替换#define

尽量以编译器替换预处理器

## 一：#define的缺点

#define ASPECT_DARIO 1.653

使用#define有个缺点就是，符号ASPECT_RADIO也许不会被编译器发现，预处理器有可能会在编译器处理源码之前移走该符号；这就导致符号名称ASPECT_DARIO 有可能没有进入**符号表(symbol table)**

于是当你运行此常量但是获得一个编译错误的时候，可能会有疑惑，因为编译错误报的是1.653，而不是ASPECT_DARIO 

#### 解决方法：使用一个常量替换上述的宏（#define）

const double AspectRadio = 1.653;

作为一个语言常量，AspectRadio 肯定会被编译器看到，也就会进入到符号表(symbol table)中

## 二：常量替换#define的两种特殊情况

#### 1. 定义常量指针

由于常量定义式通常被放在头文件中，以便被不同的源码包含，因此有必要将指针（而不只是指针所指之物）声明为const，例如要在头文件中定义一个常量的（不变的）char*字符串，必须写const两次：

`const char* const authorName = "Scott"`

在这里，string对象通常比char*更合适

`const string authorName = "Scott"`

#### 2. 类class的常量

为了将常量的作用域scope限制于class内，你必须让它成为class的一个成员；

***而为了确保此常量至多只有一份实体，你必须让它成为一个static成员***

```c++
class GamePlayer{
private:
	static const int NumTurns = 5;
	int scores[NumTurns];
	...
}
```

然而当前看到的是NumTurns的声明式，而非定义式。

**通常C++要求你对你所用的任何东西提供一个定义式，但如果它是个class专属常量，又是static，且为整数类（integral type，例如int, char, bool），则需要特殊处理。只要不取它们的地址，你就可以声明并使用它们而无需提供定义式。但是如果你取某个class的专属常量的地址，或纵使你不取而你的编译器要获取的话，你就必须额外提供定义式如下：**

`const int GamePlayer::NumTurns;`

请把这行代码放到一个实现文件.cpp，而非头文件.h。由于class常量在声明时获得初值，因此定义时不可以再赋值。

请注意，我们无法利用#define创建一个class的专属常量，**因为#define并不重视作用域，一旦宏被定义，它就在其后的编译过程中有效，除非在某处被#undef**



有些旧式编译器不允许static成员在其声明式上获得初值，此外，所谓的"in-class 初值设定"，也只允许对整数常量进行，如果你的编译器不支持上述语法，就可以将初值放在定义式：

```c++
class CostEstimate{
private:
	static const double FudgeFactor; //static class常量声明位于头文件内
    ...
};

const double CostEstimate::FudgeFactor = 1.35;  //static class常量定义位于实现文件内
```



## 三：enum替换#define

上述写法在绝大多数情况下都是普遍适用的，唯一例外时当你在class编译期间需要一个class常量值，例如在上述的GamePlayer::scores的数组声明式中（是的，编译器坚持必须要在编译期间知道数组的大小），而这时候万一你的编译器不允许static整数型class完成int class初值设定，这时候可以采用"the enum hack"的补偿做法，其理论基础是"一个属于枚举类型的数值可充当int被使用"，于是GamePalyer可定义如下：

```c++
class GamePlayer{
private:
    enum{
        NumTurns = 5
    };
    int scores[NumTurns];
}
```

enum hack的行为某方面比较想#define而不像const，有时候这正是你想要的，例如取一个const的地址是合法，但是取一个enum的地址就不合法，而取一个#define的地址通常也不合法



## 四：inline替换#define

另一个常见的#define的误用情况是以它来实现宏。宏看起来像是函数，但是不会招致函数调用带来的额外开销，更像是做字符串替换

比如：`#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))`

无论何时写出这些宏，必须记住要为宏中的所有实参加上小括号。但是即使加上小括号，也会遇到问题：

```c++
int a = 5, b = 0;
CALL_WITH_MAX(++a, b);        // a会自增两次，因为可以将宏函数理解为字符串替换  f((++a) > (b) ? (++a) : (b))，这里++a>b，所以又会++a
CALL_WITH_MAX(++a, b + 10);   // a自增一次，最后a = 8
```

#### 使用内联函数inline替换

```c++
template <typename T>
inline void callWithMax(const T& a, const T& b)
{
    f(a > b ? a : b);
}
```

