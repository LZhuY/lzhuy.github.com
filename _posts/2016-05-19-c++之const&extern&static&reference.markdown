---
layout: post
title:  "c++ const&extern&static"
date:   2016-05-18 22:21:49
categories: cplusplus
tags: cplusplush
---

c++很重要的三个修饰关键字，const&extern&static，这次是记录这几个关键字的作用。

1、const
(1)定义的时候必须同时赋值初始化，因为const变量的值是不能改变的。
(2)修饰文件中定义的全局变量，变成文件的私有变量，其他文件不能通过extern声明访问到。
(3)修饰引用。
   int i = 100;
   const int& ref1 = i;
   ref1++;//非法的操作，不能通过const的引用修改非const变量。

   const int j = 100;
   int& ref2 = j; //非法，const变量不能被const引用引用。

   int& ref3 = 10;//非法
   const int& ref4 = 10;//ok,因为其实是引用了一个临时变量。 int temp = 10; const int& ref = temp;
   
   float f = 1.0f;
   const int& ref5 = f;//ok,其实是有一个中间值。
   int& ref6 = f;//非法类型不一样

(4)被exern修饰的const变量还是可以被别的文件通过extern声明访问到的。
(5)const的使用原则，const变量应该是一个常量表达式。这样在编译阶段就可以替换成常量表达式的内容。
如果不是一个常量表达式那可能就不适合用const修饰。并且不适合放在头文件见中。

2、extern
(1)可以用于声明变量，不初始化。声明后可以访问别的文件中定义的全局变量。

3、static
(1)文件中修饰全局变量可以在别的文件中通过extern声明访问到。
