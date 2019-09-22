---
title: 'CPP Tips: const char* & char* const 区别'
tags:
  - C++
  - Tips
category: CPP Tips
date: 2019-09-22 18:47:01
---
## char* 
`char*` 是我们常用的一种, 也是最简单的一种, 他代表的是指向`可变`字符串的`可变`指针，也就是说我们既可以修改字符串(字符数组)的具体内容，也可以修改指针指向的位置，使其指向新的字符串地址
<!-- More -->
``` c++
#include <iostream>
int main() {
  char *a = new char[4];
  sprintf(a,"123"); // 设置内容为 123\0
  std::cout << a<< std::endl; // output: 123
  a=new char[4]; // 可以修改指针指向
  std::cout << a<< std::endl; // output: (empty)
  a[0]='4';      //可以修改字符数组
  std::cout << a<< std::endl; // output: 4
  delete[] a;
  retutn 0;
}
```
## const char* a
此处 `const` 修饰的是字符,也就是所`a`指向的的字符串数组内容是不可变的,`a`代表是 **指向`不可变`字符串的`可变`指针**，c++ 默认的字符串字面值就是`const char*`
``` c++
#include <iostream>
int main() {
  const char *a= "123";
  a = "78945"; // 指向新的字符串地址, ok
  std::cout << a<< std::endl; 
  a[0]='1'; // 此处尝试修改a[0]的值->read-only variable is not assignable 
}
```
## char* const a
此处 `const` 修饰的是指针, 也就是说`a`的值(指向的地址)是不可变的，a为指向`可变`字符串的`不可变`指针
``` c++
#include <iostream>

int main() {
  char *const a= new char[4];
  a[1]='7'; // ok
  a = new char[5]; //cannot assign to variable 'a' with const-qualified type 'char *const'
}
```
## const char* const a
这个就简单了，什么都不能变


> 简单记忆 `const` 靠近哪边,哪边就是不可变的 `const char *`就是`char`不可修改，`char * const a`靠近指针则`a`不可以修改(赋值)
---
Ref:
1. [Difference between char* and const char*?](https://stackoverflow.com/questions/9834067/difference-between-char-and-const-char)
2. [Difference between const char *p, char * const p and const char * const p](https://www.geeksforgeeks.org/difference-const-char-p-char-const-p-const-char-const-p/)