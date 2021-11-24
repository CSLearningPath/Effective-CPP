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

返回类型有`&&`说明`std::move`函数返回的是一个右值引用（但是如果类型`T`恰好是一个左值引用，那么`T&&`将会变成一个左值引用）
