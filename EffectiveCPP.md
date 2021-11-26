### Item 1 Understand template type deduction

C++ 中的特性`auto`是建立在模板类型推导的基础上的。考虑下面的函数模板

```c++
template<typename T>
void f(ParamType param);
```

他的调用如下:

```c++
f(exp)
```

在编译期间，编译器会对`expr`进行两个推导，一个是针对`T`的，另一个是针对`ParamType`的，这两个类型通常是不同的，因为`ParamType`通常会包含一些修饰，比如`const`和引用。

下面看一个例子，如果模板这样声明：

```c++
template<typename T>
void f(const T& param);
```

然后这样进行调用:

```c++
int x = 0;
f(x);
```

那么`T`会被推导成`int`,`ParamType`会被推导成`const int&`。

我们经常想当然的认为`T`和传递进来的参数的类型是一样的，但是经常事与愿违。`T`类型的推导跟`ParamType`息息相关。

我们将从下面三种情况来探讨：

* `ParamType`是一个指针或者引用，但是不是通用引用。

  1. 如果`expr`的类型是一个引用，忽略引用部分。
  2. 然后`expr`的类型与`ParamType`进行模式匹配来决定`T`

  举个例子：

  ```c++
  template<typename T>
  void f(T& param);
  ...
  
  int x = 1;
  const int cx = x;
  const int&rx = x;
  f(x);
  f(cx);
  f(rx);
  ```

  想一想，`f(x)`，`f(cx)`,`f(rx)`中的`T`和`ParamType`都被推导成什么了？答案如下：

  ```c++
  f(x);                           //T是int，param的类型是int&
  f(cx);                          //T是const int，param的类型是const int&
  f(rx);                          //T是const int，param的类型是const int&
  ```

  这里需要说明的是，对象的常量性通常会被保留为`T`的一部分，所以将一个`const`对象传递给`T&`类型为参数的模板是安全的。

  那么如果模板声明中有`const`呢？`const`将不再被推倒成`T`的一部分。

  ```c++
  template<typename T>
  void f(const T& param);
  ...
  
  int x = 1;
  const int cx = x;
  const int&rx = x;
  
  f(x);                           //T是int，param的类型是const int&
  f(cx);                          //T是int，param的类型是const int&
  f(rx);                          //T是int，param的类型是const int&
  ```

  









### Item 23 理解std::move和std::forward

我们可以从`std::move`和`std::forward`不做什么来了解他们。其实，`std::move`不移动任何东西，`std::forward`也不转发任何东西，他们仅仅只是进行一些转换。`std::move`将无条件将它的实参转换成右值，`std::forward`在特定的条件下进行转换。

一个C++11的`std::move`的示例如下所示：

```c++
template<typename T>
typename remove_reference<T>::type&& move(T&&param){
    using ReturnType = typename remove_reference<T>::type&&;
    return static_cast<ReturnType>(param);
}
```

返回类型有`&&`说明`std::move`函数返回的是一个右值引用（但是如果类型`T`恰好是一个左值引用，那么`T&&`将会变成一个左值引用,所以这里使用了`remove_reference<T>`）。因为`std::move`除了转换类型以外并没有做其他的事情，所以很多人提议将`std::move`改名成`rvalue_cast`可能更适合，但是既然已经确定了，就这样理解吧。



还需要强调一点，`std::move`不保证他执行转换的对象可以被移动，下面看一个例子。

```C++
class Annotation{
	public:
    	explicit Annotation(const std::string text):value(std::move(text)){...}
    private:
    	std::string value;
}
```

这段代码看起来毫无问题，事实上也确实可以正常运行，但是我需要告诉你的是`text`并不是被移动到`value`，而是被拷贝。虽然`text`通过`std::move`转换成了一个右值，但是`text`的被申明成了`const std::string`，在转换前`text`是一个左值的`const std::string`,转换后的结果是一个右值`const std::string`，所以在转换前后，`const`的属性一直被保留着。但是对于`std::string`的移动构造函数`string(string&& rhs)`来说，他不能接受`const std::string`的右值，但是这个右值却可以传递给拷贝构造函数`string(const string& rhs)`，因为`lvalue-reference-to-const`可以被绑定到一个`const`右值上。

