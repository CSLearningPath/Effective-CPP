### Use `std::shared_ptr` for shared-ownership resource management







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



还需要强调一点，`std::move`不保证他执行转换的对象可以被移动，也就是说移动操作并不永远比复制操作更廉价。下面看一个例子。

```C++
class Annotation{
	public:
    	explicit Annotation(const std::string text):value(std::move(text)){...}
    private:
    	std::string value;
}
```

这段代码看起来毫无问题，事实上也确实可以正常运行，但是我需要告诉你的是`text`并不是被移动到`value`，而是被拷贝。虽然`text`通过`std::move`转换成了一个右值，但是`text`的被申明成了`const std::string`，在转换前`text`是一个左值的`const std::string`,转换后的结果是一个右值`const std::string`，所以在转换前后，`const`的属性一直被保留着。但是对于`std::string`的移动构造函数`string(string&& rhs)`来说，他不能接受`const std::string`的右值，但是这个右值却可以传递给拷贝构造函数`string(const string& rhs)`，因为`lvalue-reference-to-const`可以被绑定到一个`const`右值上。

