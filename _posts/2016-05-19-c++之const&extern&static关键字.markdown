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
(3)修饰引用，引用是变量的别名。const

2、extern
(1)可以用于声明变量，不初始化。声明后可以访问别的文件中定义的全局变量。

3、static
(1)文件中修饰全局变量可以在别的文件中通过extern声明访问到。
