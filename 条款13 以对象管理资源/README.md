## 条款13 以对象管理资源

### 1. 背景

```c++
void f()
{
    Investment* pInv = createInvestment();
    ...
    delete pInv;
}
```

上述情况在某种情况下可能会出现f()函数无法删除pInv，比如在...操作中，直接return掉，或者中途抛出异常，导致程序压根不会执行到delete语句

### 2. 解决方法一：将资源放进对象内，可依赖C++的“析构函数自动调用机制”确保资源被释放

比如使用**智能指针**，`std::auto_ptr`是一个“类指针对象”，其析构函数会自动对其所指对象调用delete

```c++
void f()
{
    std::auto_ptr<Investment> pInv(createInvestment());
}
```

- **获得资源后立刻放进管理对象内**。以上代码中`createInvestment`返回的资源被当作其管理者auto_ptr的初值，实际上，“以对象管理资源”的观念常被成为“资源获取时机便是初始化时机”（Resource Acquisition is Initialization。`RAII`）
- **管理对象运用析构函数确保资源被释放**。



#### auto_ptr的特性

为了避免多个`auto_ptr`指向同一个对象，`auto_ptr`有一个特性：若通过copy构造函数或者copy assignment操作符赋值它们，原本的`auto_ptr`就会变成null，而复制所得的指针将取得资源的唯一拥有权。

```c++
std::auto_ptr<Investment> pInv1(createInvestment());
std::auto_ptr<Investment> pInv2(pInv1);                  // 现在pInv2指向对象，pInv1被设为null
pInv1 = pInv2;                                           // 现在pInv1指向对象，pInv2被设为null
```



`auto_ptr`的特性同时也是它的缺点，当需要其发挥正常复制行为的时候，`auto_ptr`就不适合了，这时候应该使用“引用计数型智能指针” `referense-counting smart pointer RCSP`：**std::share_ptr**

```c++
void f()
{
	std::shared_ptr<Investment> pInv1(createInvestment());
    std::shared_ptr<Investment> PInv2(pInv1);              // pInv1和pInv2当前指向同一个对象
    pInv1 = pInv2;                                         // 同上
}
```



### auto_ptr和shared_ptr在其析构函数内做的是delete，而不是delete[]

这意味着在动态分配得到的array上使用auto_ptr和shared_ptr是有问题的。尽管能通过编译

```c++
std::auto_ptr<std::string> aps(new std::string[10]);       // 错误！
std::shared_ptr<int> spi(new int[1024])                    // 错误！
```

