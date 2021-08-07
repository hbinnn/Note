# string

## 初始化

```c++
string s1;
string s2 = s1;
string s3 = "hello";	// 拷贝初始化
string s4(10, 'a');		// 直接初始化
string s5("hello");
```

## 常用内置函数

```
string s;
s.empty()	// s是否为空	
s.size()	// s中字符的个数
s.find('a') == s.npos	// s中查找 'a', 未找到返回 s.npos
string::size_type	// s.size()的返回类型，无符号整型
s = s1 + s2;	// s1与s2拼接后赋值给s
s == s1		// s与s1相比较
s.append();	// z
```

