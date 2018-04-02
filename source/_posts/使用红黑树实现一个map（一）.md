---
title: 使用红黑树实现一个map（一）
date: 2016-12-10 10:48:48
tags: [C++,map,数据结构,红黑树,RBTree]
categories: Data Structure
---

&#8195;&#8195;map是一种很重要的容器，它以键值对的关联式数据存储给我们的许多编程提供了方便。从某erp公司实习回来以后，感觉许久没有写过复杂的算法，对C++的使用也生疏了不少，便想起了写个红黑树的map热热手。

<!--more-->

# map简介

## 实现方式
map的实现有多种方法，最主要的两种实现是哈希表（HashMap）和树（TreeMap），而使用树型结构的map一般是红黑树实现（AVL太过严格，限制了效率）。这两种方法各有利弊：  
* 哈希表虽然在查找、插入、删除的操作上复杂度为O(1)，但是它的数据的存放时无序的，因此在排序，获取最大Key，最小Key的元素等操作上较为复杂，并且也不易于扩展，但是对排序没有要求的unordered_map适合用哈希表来实现
* 红黑树是一种特殊的平衡二叉树，它使用红黑节点的特性让平衡二叉树在绝对平衡和效率直接获得了很好的平衡。红黑树的查找、插入和删除均为O(logN)的复杂度，并且树按Key值的大小组织结构，具有良好的适用性和扩展性。红黑树也是现在多数标准库中map的实现方式（如C++，java等）  

## 功能
map为外部提供了许多接口，在C++的stl库中，map为模板类，提供键值对(key, value)的存储方式。但是要注意的是，map的key必须要实现了“<”操作符，因为map的定义是：
```C++
template <class Key,
          class T,
          class Compare=less<Key>,
          class Alloc = allocator<pair<const key, T> >> class map:
          {
            ......
          }
```
map在组织数据结构时，需要按照Key的大小来排序，默认使用std:less，我们再来看std:less的定义：
```C++
template <Class T> struct less : binary_function <T, T, bool>
{
  bool operator() (const T& x, const T& y) const { return x < y; }
};
```
而map对外部提供的接口也有很多，包括插入、删除、查找、交换、清空、迭代器的操作等，具体可以参照C++的[reference](http://www.cplusplus.com/reference/map/map/)  

# 起步
## 目标
我希望使用C++，实现一个类似STL库中map的关联式模板容器，使用红黑书作为数据结构，能够实现插入、删除、查找、清空、迭代器等功能。
## 框架
简要介绍完了map，正式开始代码的编写，首先的工作是把map的模板架构搭好，仿照STL
```C++
template<
    class Key,
    class T,
    class Compare = std::less<Key>
> class map {
public:
  typedef pair<const Key, T> value_type;

  map() {}
  map(const map &other) {}
  ~map {}
}
```
stl中map的第四个参数Alloc会影响map对内存空间的使用机制，我们这里省略。定义pair类型的value_type是为了适应键值对作为数据的需求
## 红黑树
### 定义
map的核心，就是它的数据结构：红黑树了。我们先看看红黑树的定义：  
1. 每个节点都是红色或者黑色
2. 根（root）节点和叶子节点（NIL空节点）都是黑色
3. 红节点的父节点或子节点不能为红色
4. 从任一节点出发，到达其任一叶子节点的路径中所包含的黑色节点数目相同 

![红黑树](http://ohvmg8dgt.bkt.clouddn.com/rbt.png)

单纯从定义的角度理解定义总是有些困难，我们不妨从红黑树的目的来试着理解为什么要有这些限定  

在红黑树出现前，`AVL树`是人们普遍使用的一种数据结构。但是渐渐的，人们发现AVL为了保证其所要求的“左右子树高度差不超过1”的严格要求，
在插入和删除时，常常需要复杂的调整操作（可能达到O(logN)的调整次数）。于是，`红黑树`便出现了，相比于AVL树，红黑树希望降低平衡性的要求，
并且同时保证其查找的快速性。  

红黑树把节点分为两种类型：红色和黑色，通过黑色节点的数量来保证平衡性，而红色节点不计入平衡性控制中，就如同调节器，穿插在黑色节点中，可以让树在生长过程中有更大的自由度。
而红色节点不能连续出现，则保证了红黑树作为一个平衡树的基本原则：没有一条路径比其他路径长一倍以上,在最坏情况下，它的高度也不会超过2log(N)，而平均高度仍然保持在log(N)。
### 框架
我们先把红黑树的一些声明写好  
```C++
#define RED   0
#define BLACK 1
#define NIL   2
// 红黑树的节点
class Node {
public:
  Node() { vt = NULL; left = right = parent = NULL; type = NIL; } // 默认构造器
  Node(Node *other) // 拷贝构造器
  {
    if (other->vt != NULL)
    	vt = new value_type(*other->vt);
    else
      vt = NULL;
      type = other->type;
      left = right = parent = NULL;
  }
  Node(const int &color) // 用于生成带颜色的默认节点
  { 
    vt = NULL; left = right = parent = NULL; type = color; 
  } 
  Node(const value_type &value) // 传值构造器
  {
    vt = new value_type(value);
    parent = NULL;
    type = RED;
    left = new Node(NIL);
    right = new Node(NIL);
    left->parent = right->parent = this;
  }
  ~Node() // 析构器
  {
    // 在delete前注意判断对象是否指向合法地址
    if(vt)
      delete vt;
  }
public:
  value_type *vt;
  Node *left;
  Node *right;
  Node *parent;
  int type;
}
```
这里我对红黑树节点颜色的定义其实不好，判断红黑树是否为叶子节点（NIL）只需判断其是否为未初始化状态，不需要再定义一种颜色（因为叶子节点其实也是黑色的）。
```C++
class RBTree {
public:
  RBTree() { root = NULL; comp = Compare(); _size = 0; }
  RBTree(const RBTree &other) { //[TODO] }
  ~RBTree() { //[TODO] }
private:
  Node *root;
  size_t _size;
  Compare comp;
}
```
声明了红黑树的构造器和析构器，注意记得将模板类map中，Key的比较函数作为RBTree的对象，因为在查找、插入和删除中都需要比较Key的大小。 

红黑树的拷贝和析构并不简单，我们先不急着写，接下来把RBTree对外，也就是对map提供的接口先列出来  
```C++
size_t size() {}
bool empty() {}
void clear() {}
Node* find(Key key) {}
Node* insert(const value_type &value) {}
void remove(const value_type &value) {}
```
考虑到还要为map的迭代器（iterator）提供接口  
```C++
Node* begin() {}
Node* end() {}
```
其实就是为iterator提供树的头部和尾部。  

未完待续......
