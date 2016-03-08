---
layout: post
title: "Swift && Object-C"
date: 2016-01-06 15:55:20
tags: Coding
---
Swift语言本身并没有多线程支持，因此要使用多线程还是要调用NSThread、GCD等

Object-C中一个很方便的代码互斥方式@synchronized在swift中不支持，因此要实现代码互斥需要自己使用锁

Swift中可以自定义操作符，自定义一个操作符需要两个要素：

操作符特性描述，包括：操作符位置 prefix、infix、postfix等，操作符的结合性left、right以及操作符的优先级；
操作符功能函数，参数和返回值需要与操作符特性一致，比如 prefix的操作符接受一个参数，返回一个值；

在Swift中，函数是first-class，也就是说函数可以作为参数传入也可以作为返回值返回，也可以给变量赋值。函数式编程是一种很强大、很灵活的思想，在Object-C中也有block等函数式编程思想，但并不明显，在Swift中这一点得到了强化。

没有多线程：可以用 GCD 的 API，也被移植到 Swift 了。
没有私有属性：很多语言都没有，都靠使用者自觉。
只能与 Objective-C 互动：C 也是可以的。
没有异常处理：Objective-C 里基本也不用