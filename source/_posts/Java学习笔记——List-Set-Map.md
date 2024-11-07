---
title: Java学习笔记——List, Set, Map
date: 2024-04-18 19:47:16
tags:
  - 学习笔记
  - Java
---

![集合框架图](watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5a2mSkFWQeeahOengOeQtA==,size_15,color_FFFFFF,t_70,g_se,x_16-1713407102148-3.png)

# List(包含ArrayList, Vector, LinkedList)

ArrayList底层使用数组实现数据存储
ArrayList基本等同与Vector，但是Vector线程安全（因此ArrayList更快）
ArrayList源码分析：
维护一个Object类型的数组elementData，如果使用无参构造器，初始化数组大小为0，第一次添加扩容为10，再次扩容是每次扩容1.5倍。如果使用指定大小的构造器，则直接初始化为指定大小，每次扩容1.5倍。elementData用transient修饰，表示该属性不会被序列化。
Vector底层也是elementData数组，多线程操作时使用Vector。无参构造时默认长度为10，满了以后每次增加2倍，如果指定大小，满后则直接扩容两倍（其实可以设置每次增长量，如果没有设置，就默认翻倍）
LinkedList底层实现了双向链表和双端队列，没有实现同步，线程不安全。增删较多用LinkedList，改查较多用ArrayList

# Set(包含HashSet, LinkedHashSet)

hashset底层是hashmap，hashmap底层是数组+链表+红黑树，hashmap初始化长度为16，数据到一定量(JAVA8中链表长度≥8，table大小≥64)后变成红黑树。如果链表到了8，table没到64，那么table扩容为2倍。此外，table中所有**节点的数量**到threshold(0.75乘以table长度，初始是16*0.75=12)时进行扩容，扩容也是扩为2倍。扩容的同时，原有的元素会被重新放置（因为数组大小改变）。

LinkedHashSet底层是一个LinkedHashMap，其底层维护了一个数组+双向链表，使用hashCode决定在table中的位置，双向链表维护插入顺序，使得元素看起来是按顺序插入的。相当于在HashTable（HashMap）的Node外面套了一层壳（Entry），加上了before和after字段。其目的就是为了维护顺序，并且减小查询开销（可以直接计算hash）。遍历时使用LinkedHashMap的head字段进行遍历。扩容等机制和HashMap相同。

# Map(包含HashMap, LinkedHashMap, Hashtable, Priorities)

HashMap在加入相同键时会发生替换，而替换时并不会改变modCount，modCount是一个用于记录HashMap结构修改次数（改变HashMap中映射数量或者修改内部结构）的变量。HashMap是线程不安全的。

Hashtable底层维护一个table数组，数组存储Hashtable$Entry对象，形成链表。初始化table大小11，threshold为大小乘以0.75(向下取整)。扩容机制:左移一位(乘以二)加一。Hashtable的效率比HashMap低。HashTable的键值都不能为空，是线程安全的

properties可以读取xx.properties文件，获取文件中的配置(举例:数据库)

# 如何选择

- 插入单列元素：Collection(List、Set)
  - 允许重复：List
    - 不用线程安全，效率高：
      - 具有大量增删操作：LinkedList
      - 具有大量改查操作：ArrayList
    - 线程安全，效率低：Vector
  - 不允许重复：Set
    - 无序：HashSet（底层使用HashMap）
    - 排序：TreeSet
    - 按插入顺序：LinkedHashSet（底层使用LinkedHashMap）
- 插入双列元素：Map
  - 无需线程安全：
    - 无序：HashMap
    - 排序：TreeMap
    - 按插入顺序：LinkedHashMap
  - 线程安全：
    - 读取文件：Properties
    - 其他：Hashtable
