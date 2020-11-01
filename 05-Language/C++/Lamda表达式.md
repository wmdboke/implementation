#### Lamda表达式(C++11):

本质是匿名函数。具体形式如下：

```cpp
[capture list] (params list) mutable exception-> return type { function body 
} 
capture list：捕获外部变量列表
params list：形参列表
mutable指示符：用来说用是否可以修改捕获的变量,能修改传递进来的拷贝的变量。
exception：异常设定
return type：返回类型
function body：函数体
```

最简单的是：[](){};
