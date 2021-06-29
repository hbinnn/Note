# c++工程师能力评估

## new知识点

```c++
int *p1 = new int[10]; 
int *p2 = new int[10]();
```

> 1、new当个对象
>
> new在自由空间分配内存，但其无法为其分配的对象命名，因次是无名的，分配之后返回一个指向该对象的指针。
>
> ```
> int *pi = new int; // pi指向一个动态分配的，未初始化的无名对象
> ```
>
> 此new表达式在自由空间构造一个int类型对象，并返回指向该对象的指针。
>
> 默认情况下，**动态分配的对象是默认初始化的，这意味着内置类型或组合类型的对象的值是无定义的，而类类型对象将用默认构造函数进行初始化**。
>
> 2、new(多个对象)数组
>
> new分配的对象，不管单个对象还是多个对象的分配，都是默认初始化。**但可以对数组进行值初始化，方法就是：在大小之后添加一对空括号。**
>
> ```
> int *pia = new int[10];    // 10个未初始化int
> int *pia2 = new int[10](); // 10个值初始化为0的int
> ```



```
哪种C/C++ 分配内存的方法会将分配的空间初始化为0
malloc()
calloc()
realloc()
new[ ]
```

> 1) malloc 函数： void *malloc(unsigned int size)
>
>    在内存的动态分配区域中分配一个长度为size的连续空间，如果分配成功，则返回所分配内存空间的首地址，否则返回NULL，申请的内存不会进行初始化。
>
> 2）calloc 函数： void *calloc(unsigned int num, unsigned int size)
>
>    按照所给的数据个数和数据类型所占字节数，分配一个 num * size 连续的空间。
>
>   calloc申请内存空间后，会自动初始化内存空间为 0，但是malloc不会进行初始化，其内存空间存储的是一些随机数据。
> 3）realloc 函数： void *realloc(void *ptr, unsigned int size)
>
>   动态分配一个长度为size的内存空间，并把内存空间的首地址赋值给ptr，把ptr内存空间调整为size。
>
>   申请的内存空间不会进行初始化。
> 4）new是动态分配内存的运算符，自动计算需要分配的空间，在分配类类型的内存空间时，同时调用类的构造函数，对内存空间进行初始化，即完成类的初始化工作。动态分配内置类型是否自动初始化取决于变量定义的位置，在函数体外定义的变量都初始化为0，在函数体内定义的内置类型变量都不进行初始化。

## 地址偏移计算

```c++
unsigned char *p1;
unsigned long *p2;
p1=(unsigned char *)0x801000;
p2=(unsigned long *)0x810000;
请问p1+5= 什么？
p2+5= 什么？
```

> p1指向字符型，一次移动一个字符型，1个字节；p1+5后移5个字节，16进制表示为5；
>
> p2指向长整型，一次移动一个长整型，4个字节，p2+5后移20字节，16进制表示为14。
>
>  **{ char每次移动1个字节；short移动2个字节 ；int , long ,float移动4个字节 ；double移动8个字节}**



```c++
下面程序运行后的结果为：
char str[] = "glad to test something";
char *p = str;
p++;
int *p1 = reinterpret_cast<int *>(p);
p1++;
p = reinterpret_cast<char *>(p1); 
printf("result is %s\n", p);
```

> charstr[] = "glad to test something";          //定义字符串
>
> char*p = str;                            //p指向字符串首地址，即字符'g'
>
> p++;                                  //p是char*类型，每次移动sizeof(char)字节，故此时p指向 'g'的下一个字符 'l'
>
> int*p1 = reinterpret_cast<int*>(p);           //指针p被重新解释为整型指针并被赋值给p1
>
> p1++;                                  //p1是int*类型， 每次移动sizeof(int)字节，故此时p1 指向 'l'后的第四个字符 't'
>
> p = reinterpret_cast<char*>(p1);             //指针p1被重新解释为字符型指针并被赋值给p
>
> printf("result is %s\n", p);                   //从't'开始输出字符串，即得到 "to test something"



## 指针、数组大小计算

```c++
void example(char acWelcome[]){
    printf("%d",sizeof(acWelcome));
    return;
}
void main(){
    char acWelcome[]="Welcome to Huawei Test";
    example(acWelcome);
    return;
}
的输出是？
```

> char str[20]="0123456789";
> int a=strlen(str); //a=10; >>>> strlen 计算字符串的长度，以结束符 0x00 为字符串结束。
> int b=sizeof(str); //而b=20; >>>> sizeof 计算的则是分配的数组 str[20] 所占的内存空间的大小，不受里面存储的内容改变。
>
> 上面是对静态数组处理的结果，如果是对指针，结果就不一样了
>
> char* ss = "0123456789";
> sizeof(ss) 结果 4 >>> ss是指向[字符串常量](https://www.baidu.com/s?wd=字符串常量&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)的字符指针，sizeof 获得的是一个指针的之所占的空间,应该是
>
> 长整型的，所以是4
> sizeof(*ss) 结果 1 >>> *ss是第一个字符 其实就是获得了字符串的第一位'0' 所占的内存空间，是char类
>
> 型的，占了 1 位
>
> strlen(ss)= 10 >>>> 如果要获得这个字符串的长度，则一定要使用 strlen



```c++
void Func(char str_arg[100])
{
       printf("%d\n",sizeof(str_arg));
}
int main(void)
{
     char str[]="Hello";
     printf("%d\n",sizeof(str));
    printf("%d\n",strlen(str));
    char*p=str;
    printf("%d\n",sizeof(p));
    Func(str);
}
32位系统下的的输出？
```

> 1. str是复合类型数组char[6]，维度6是其类型的一部分，sizeof取其 维度*sizeof(char)，故为6；
> 2. strlen 求c类型string 的长度，不含尾部的'\0'，故为5；
> 3. p只是个指针，32位机上为4；
> 4. c++中不允许隐式的数组拷贝，所以Func的参数会被隐式地转为char*，故为4；
>
> note：若Func的原型为void Func(char (&str_arg) [6])（若不为6则调用出错），则结果为6.



堆、栈的空间分配

```c++
设已经有A,B,C,D4个类的定义，程序中A,B,C,D析构函数调用顺序为？

C c;
void main()
{
    A*pa=new A();
    B b;
    static D d;
    delete pa;
}
```

> 这道题主要考察的知识点是 ：全局变量，静态局部变量，局部变量空间的堆分配和栈分配
>
> 其中全局变量和静态局部变量时从 静态存储区中划分的空间，
>
> 二者的区别在于作用域的不同，全局变量作用域大于静态局部变量（只用于声明它的函数中），
>
> 而之所以是先释放 D 在释放 C的原因是， 程序中首先调用的是 C的构造函数，然后调用的是 D 的构造函数，析构函数的调用与构造函数的调用顺序刚好相反。
>
> 局部变量A 是通过 new 从系统的堆空间中分配的，程序运行结束之后，系统是不会自动回收分配给它的空间的，需要程序员手动调用 delete 来释放。
>
> 局部变量 B 对象的空间来自于系统的栈空间，在该方法执行结束就会由系统自动通过调用析构方法将其空间释放。
>
> 之所以是 先 A  后 B 是因为，B 是在函数执行到 结尾 "}" 的时候才调用析构函数， 而语句 delete a ; 位于函数结尾 "}" 之前。



## 字节对齐

```c++
若char是一字节，int是4字节，指针类型是4字节，代码如下：
class CTest
{
    public:
        CTest():m_chData(‘\0’),m_nData(0)
        {
        }
        virtual void mem_fun(){}
    private:
        char m_chData;
        int m_nData;
        static char s_chData;
};
char CTest::s_chData=’\0’;
问：
（1）若按4字节对齐sizeof(CTest)的值是多少？
（2）若按1字节对齐sizeof(CTest)的值是多少？
请选择正确的答案。
```

> 答案 ：
>
> 若按4字节对齐sizeof(CTest)的值是12；
>
> 若按1字节对齐sizeof(CTest)的值是9
>
> 解释：
>
> **在类中，如果什么都没有，则类占用1个字节，一旦类中有其他的占用空间成员，则这1个字节就不在计算之内，如一个类只有一个int则占用4字节而不是5字节。**
>
> **如果只有成员函数，则还是只占用1个字节，因为类函数不占用空间**
>
> **虚函数因为存在一个虚函数表，需要4个字节，数据成员对象如果为指针则为4字节，注意有字节对齐，如果为13字节，则进位到16字节空间。**
>
> sizeof的本质是得到某个类型的大小，确切的来说就是当创建这个类型的一个对象(或变量)的时候，需要为它分配的空间的大小。而类也可以理解为类似于int、float这样的一种类型，当类中出现static成员变量的时候，static成员变量是存储在静态区当中的，它是一个共享的量，因此，在为这个类创建一个实例对象的时候，是无需再为static成员变量分配空间的，所以，这个类的实例对象所需要分配的空间是要排除static成员变量的，于是，当sizeof计算类的大小的时候会忽略static成员变量的大小
>
> 注意点：
> 1 先找有没有virtual 有的话就要建立虚函数表，+4
> 2 static的成员变量属于类域，不算入对象中   +0
> 3 什么成员都没有的类，或者只有成员函数    +1



## 拷贝构造

```c++
#include<iostream>
using namespace std;
class MyClass
{
public:
    MyClass(int i = 0)
    {
        cout << i;
    }
    MyClass(const MyClass &x)
    {
        cout << 2;
    }
    MyClass &operator=(const MyClass &x)
    {
        cout << 3;
        return *this;
    }
    ~MyClass()
    {
        cout << 4;
    }
};
int main()
{
    MyClass obj1(1), obj2(2);
    MyClass obj3 = obj1;
    return 0;
}
运行时的输出结果是（）122444
```

>关键是区分 浅/深拷贝操作 和 赋值操作:
>
>没有重载=之前：
>
>A a ;
>
>A b;
>
>a = b;
>
>这里是赋值操作。
>
>A a;
>
>A b = a; 
>
>这里是浅拷贝操作。
>
>重载 = 之后：
>
>A a ;
>
>A b;
>
>a = b;
>
>这里是深拷贝操作（当然这道题直接返回了，通常我们重载赋值运算符进行深拷贝操作）。
>
>A a;
>
>A b = a; 
>
>这里还是浅拷贝操作。
>
>所以 MyClass obj3 = obj1; 调用的是拷贝构造函数。
>
>如果写成 MyClass obj3; obj3 = obj1; 输出的结果就是 1203444
>
>注意点：
>
>**拷贝构造函数和赋值运算符的行为比较相似，都是将一个对象的值复制给另一个对象；但是其结果却有些不同，拷贝构造函数使用传入对象的值生成一个新的对象的实例，而赋值运算符是将对象的值复制给一个已经存在的实例。**



## 异常处理

```
在异常处理中，如释放资源，关闭数据库、关闭文件应由（ ）语句来完成。
try子句
catch子句
finally子句
throw子句
```

> - try:可能发生异常的语句
> - catch:捕获，并处理异常（printStackTrace()用来跟踪异常事件发生时执行堆栈的内容）
> - throw:方法内部抛异常
> - throws:声明方法异常
> - finaly：代码中无论是否有异常都会执行，清除资源