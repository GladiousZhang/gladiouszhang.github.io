---
title: Java学习笔记——异常
date: 2024-04-12 19:44:46
tags:
  - 学习笔记
  - Java
---

为了维护健壮性。
Idea捕获异常快捷键ctrl+alt+t。语法try{语句}catch(Exception e){异常处理}
异常事件:Error:JVM无法处理的严重问题，如栈溢出，内存不足。Exception:编程错误或偶然因素导致的一般问题。如空指针，文件不存在。Exception分为运行时异常和编译时异常。运行时异常可以暂时不用处理(太多)，编译时异常必须处理(如文件不存在，类不存在)。
空指针异常:运行时异常，使用对象对象为空。
数学运算异常:例如除以0。
数组越界异常
类型转换异常:试图把对象强制转化为不是实例的子类。
数字格式异常:字符串转化为数字，但是该字符串不满足时。
异常处理方式:try-catch-finally:程序员自行处理异常。throws:将异常抛出，由调用者处理。最顶级的调用者是JVM(main)
try-catch-finally过程:异常发生时，系统将异常封装为Exception对象e，传递给catch。有没有发生异常都执行finally。故经常将释放资源的代码放在finally。
throws机制:层层往上扔。throws是放在方法的后边，如果出错就会throws。
finally一定会执行，就算前面有return了，finally也会执行。如果没有catch，那么finally执行完就结束了(因为没有捕捉到错误，错误发生，程序崩溃)
