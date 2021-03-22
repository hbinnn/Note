

# lambda表达式

## 定义

 *lambda* 表达式具有一个返回类型、一个参数列表和一个函数体； *lambda* 表达式具有如下形式：

`[capture list] (parameter list) -> return type {function body}`

其中，*capture list* 是一个 *lambda* 所在函数中定义的局部变量的列表； *return type* 、 *parameter list* 和 *function body* 与任何函数一样，分别表示返回类型、参数列表和函数体。但是，*lambda* 必须使用尾置返回来指定返回类型；

可以忽略参数列表和返回类型，但是必须包含捕获列表和函数体

`auto f = [] { return 42; }`;

## 捕获列表

空捕获列表表示 *lambda* 不适用它所在函数中的任何局部变量；

另外 *lambda* 表达式不能有默认参数，因此一个 *lambda* 调用的实参数目永远与形参数目相等；

*lambda* 通过将局部变量包含在其捕获列表中来指出将会使用这些变量；

`[sz] (const string &a) { return a.size() >= sz; };`

只有在捕获列表中捕获一个它所在函数中的局部变量才能在 *lambda* 函数体中使用该变量；

如`[] (const string &a) { return a.size() >= sz; };` 则会编译错误；

## 值捕获

变量捕获的方式可以是值或引用；

采用值捕获的前提是变量可以拷贝；另外与参数不同，被捕获的变量的值是在 *lambda* 创建时拷贝，而不是调用时拷贝；

![image-20210322234633143](C:\Users\huangshibin\Desktop\Note\C++\lambda表达式.assets\image-20210322234633143.png)

## 引用捕获

![image-20210322234654348](C:\Users\huangshibin\Desktop\Note\C++\lambda表达式.assets\image-20210322234654348.png)

*v1* 之前的 *&* 指出 *v1* 应该以引用方式捕获；

当采用引用方式捕获一个变量，就必须保证被引用对象在 *lambda* 执行的时候时存在的；

## 隐式捕获

除了显式的在捕获列表中指出需要使用的变量，还可以让编译器根据 *lambda* 中的代码来推断我们要使用哪些变量；

应在捕获列表中写一个 *&* 或 *=* ； *&* 表示采用引用捕获方式， *=* 表示采用值捕获方式；

如果希望对一部分变量采用值捕获，一部分变量采用引用捕获，可以混合使用显式捕获和隐式捕获，此时捕获列表中的第一个元素必须是一个 *&* 或 *=* ；

另外在使用混合捕获时，显示捕获的变量必须使用与隐式捕获不同的方式，即如果隐式捕获使用了值捕获方式，则显式捕获必须采用引用捕获方式，同理如果隐式捕获采用了引用捕获方式，则显式捕获必须采用值捕获方式；

## 可变 *lambda*

如果要改变一个被捕获的变量的值，可以在参数列表首加上关键字 *mutable* ，此时可以省略参数列表

![image-20210322235713184](C:\Users\huangshibin\Desktop\Note\C++\lambda表达式.assets\image-20210322235713184.png)

一个引用捕获的变量是否可以修改取决于此引用指向的是一个 *const* 类型还是非 *const* 类型

![image-20210322235815491](C:\Users\huangshibin\Desktop\Note\C++\lambda表达式.assets\image-20210322235815491.png)

## 指定 *lambda* 返回类型

默认情况下。一个 *lambda* 体包含 *return* 之外的任何语句，则编译器假定此 *lambda* 返回 *void* ，被推断返回 *void* 的 *lambda* 不能返回值

![image-20210323000111059](C:\Users\huangshibin\Desktop\Note\C++\lambda表达式.assets\image-20210323000111059.png)

此时需要为 *lambda* 定义返回类型，需要使用尾置返回类型

![image-20210323000151219](C:\Users\huangshibin\Desktop\Note\C++\lambda表达式.assets\image-20210323000151219.png)

## 参数绑定

当需要在很多地方使用相同操作时，通常采用的是定义一个函数的方式，而不是 *lambda* 表达式；

如果 *lambda* 表达式的捕获列表为空，可以使用函数来代替；

但是对于捕获局部变量的 *labmda* ，用函数来代替就不是那么容易了

`[sz] (const string &a) { return a.size() >= sz; };` 可以转换成函数

`bool check_size (const string &s, string::size_type sz) { return s.size()  >= sz}`

但此时 *check_size* 不能作为 *find_if* 的一元谓词来使用，此时需要解决如何向 *sz* 形参传递一个参数

## *bind* 函数

*bind* 函数接受一个可调用对象，生产一个新的可调用对象来适应原对象的参数列表

一般形式为

`auto newCallable = bind (callable arg_list);`

其中， *newCallable* 本身是一个可调用对象， *arg_list* 是一个逗号分隔的参数列表， 对应给定的 *callable* 参数；

当我们调用 *newCallable* 时， *newCallcable* 会调用 *callable* 并传递给它 *arg_list* 中的参数

*arg_list* 中的参数可能包含形如 *_n* 的名字，其中 *n* 是一个整数， 这些参数是占位符，占据了传递给 *newCallable* 的参数的位置； *_1* 为 *newCallable* 的第一个参数，*_2* 为 *newCallable* 的第二个参数，以此类推

![image-20210323001948819](C:\Users\huangshibin\Desktop\Note\C++\lambda表达式.assets\image-20210323001948819.png)

![image-20210323002006223](C:\Users\huangshibin\Desktop\Note\C++\lambda表达式.assets\image-20210323002006223.png)

![image-20210323002021363](C:\Users\huangshibin\Desktop\Note\C++\lambda表达式.assets\image-20210323002021363.png)

## 使用 *placeholders* 名字

名字 *_n* 都定义在一个名为 *placeholders* 的命令空间中， 这个命名空间本身定义在 *std* 命名空间中

## 绑定引用参数

如果我们希望传递给 *bind* 一个对象又不拷贝它或无法拷贝，可以使用标准库的 *ref* 函数

![image-20210323002407246](C:\Users\huangshibin\Desktop\Note\C++\lambda表达式.assets\image-20210323002407246.png)

*ref* 返回一个对象，包含给定的引用，此对象是可以拷贝的，另外还有一个 *cref* 函数生成一个保存 *const* 引用的类；

*bind* 、 *ref*  和 *cref* 都定义在头文件 *functional* 中