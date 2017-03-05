---
title: 使用红黑树实现一个map（三）
date: 2016-12-18 16:45:47
tags: [C++,map,数据结构,红黑树,RBTree]
categories: Data Structure
---

&#8195;&#8195;写好了红黑树的插入，剩下最关键的一个操作就是删除了。红黑树的删除比它的插入更复杂一些，需要更加细致地分析了。

<!--more-->

# 删除情况分析
红黑树的删除，可以看作平衡二叉树删除的一种特殊情况，它们在调整的步骤上不同。删除节点如果有一个以上的儿子，则找到删除节点下一个节点T，用这个节点代替删除节点，删除T。

## 红色节点
最简单的一种情况，直接删除，无需额外调整。 

![pic1](http://ohvmg8dgt.bkt.clouddn.com/rbt_delete_pic1.png)

## 黑色单支节点
单支指删除的节点只有一个儿子，这种情况可以算作第三种情况的特殊情况  

![pic2](http://ohvmg8dgt.bkt.clouddn.com/rbt_delete_pic2.png)

## 黑色双子节点
若删除的节点为黑色，切有两个儿子，即左右子树。在树种寻找大于删除节点z的最小节点y（该节点至多有一个子树，切为右子树），并将y的右儿子记为x。删除时，用y替换z，并且交换y和z的颜色，直接删除y即可。这时我们便回到了上面这种情况  

所以，删除的问题被我们规约到了一种情况，删除一个至多有右子树的节点y。 

1. 若y为红色，直接删除不影响平衡性。
2. 若y为黑色，则删除后x替换了y的位置，相当于少了一个黑色节点，平衡性受到影响，需要进行调整。

我们记x的兄弟节点为w，现在x子树相比w少了一个黑色节点，需要在x子树中找到一个合适位置的红色节点染为黑色。此时又出现了几种情况，而最简单的一种是x为红色：  

* x为红色，直接将x置为黑色
* x为黑色，情况又变得复杂了起来，不妨看看x为左子树的情况。

1. w为红色，则w的儿子全黑，x与w的父亲p也为黑。  
![pic3](http://ohvmg8dgt.bkt.clouddn.com/rbt_delete_reb_1.jpg)  
x比w少一个黑色节点，显然此时无法简单地通过染色解决这个失衡问题。只好再借助旋转：交换p与w的颜色，同时对p做一次左旋

2. w为黑色，则p可能为红色也可能为黑色。
![pic4](http://ohvmg8dgt.bkt.clouddn.com/rbt_delete_reb_2.jpg)  
把w变为红色，则p的两个子树黑色节点回归平衡。但是对p来说，子树p少了一个黑色节点，此时将p作为新的x，如果新的x为红色，可以简单地将其置黑，则整棵树平衡。否则进行递归调整。

3. w为黑色，w左孩子为红色，右孩子为黑色。  
![pic5](http://ohvmg8dgt.bkt.clouddn.com/rbt_delete_reb_3.jpg)
交换w与左孩子的颜色，同时对w进行右旋，此情况变为情况4

4. w为黑色，w右孩子为红色。
![pic6](http://ohvmg8dgt.bkt.clouddn.com/rbt_delete_reb_4.jpg)
最接近成功的一种情况，交换w与p的颜色，同时对p进行左旋，再把w的右儿子置黑，则此时左右平衡。  

# 代码
删除的代码
```C++
void remove(const value_type & value) 
{
    Node * tNode;
    Node * sNode;
    Node * cNode = find(value.first);
    if (cNode == NULL)
        return;
    if (cNode - > left - > type == NIL || cNode - > right - > type == NIL) {
        tNode = cNode;
    } else {
        tNode = nextNode(cNode);
    }

    if (tNode - > left && tNode - > left - > type != NIL) {
        sNode = tNode - > left;
        tNode - > left = NULL;
    } else {
        sNode = tNode - > right;
        tNode - > right = NULL;
    }

    if (tNode == root) {
        root = sNode;
    } else {
        if (tNode == tNode - > parent - > left)
            tNode - > parent - > left = sNode;
        else
            tNode - > parent - > right = sNode;
    }
    sNode - > parent = tNode - > parent;

    if (tNode != cNode) {
        if (tNode - > type != NIL) {
            delete cNode - > vt;
            cNode - > vt = new value_type( * tNode - > vt);
        }
    }
    if (tNode - > type == BLACK)
        delete_fixed_up(sNode);
    delete tNode;
    tNode = NULL;
    // 删除了根节点
    if (root - > type == NIL) {
        delete root;
        root = NULL;
    }
    _size--;
}
```
```C++
void delete_fixed_up(Node * cNode) 
{
    Node * wNode;
    Node * pNode;
    // set black if its red
    if (cNode - > type == RED)
        cNode - > type = BLACK;
    else {
        while (cNode != root && cNode - > type != RED) {
            pNode = cNode - > parent;
            // left case
            if (cNode == cNode - > parent - > left) {
                wNode = cNode - > parent - > right;
                // case1: sibling is red. change to case 2-4
                if (wNode - > type == RED) {
                    wNode - > type = BLACK;
                    cNode - > parent - > type = RED;
                    leftRotate(pNode);
                    wNode = cNode - > parent - > right;
                }
                // case2:
                if ((!wNode - > left || wNode - > left - > type != RED) && (!wNode - > right || wNode - > right - > type != RED)) {
                    if (wNode - > type != NIL)
                        wNode - > type = RED;
                    cNode = wNode - > parent;
                } else {
                    // case 3: to 4
                    if (!wNode - > right || wNode - > right - > type != RED) {
                        if (wNode - > left && wNode - > left - > type != NIL)
                            wNode - > left - > type = BLACK;
                        if (wNode - > type != NIL)
                            wNode - > type = RED;
                        rightRotate(wNode);
                        wNode = cNode - > parent - > right;
                    }
                    // case 4:
                    wNode - > type = cNode - > parent - > type;
                    cNode - > parent - > type = BLACK;
                    if (wNode - > right && wNode - > right - > type != NIL)
                        wNode - > right - > type = BLACK;
                    pNode = cNode - > parent;
                    leftRotate(pNode);
                    break;
                }
            }
            // right case
            else {
                wNode = cNode - > parent - > left;
                if (wNode - > type == RED) {
                    wNode - > type = BLACK;
                    cNode - > parent - > type = RED;
                    rightRotate(pNode);
                    wNode = cNode - > parent - > left;
                }
                if ((!wNode - > left || wNode - > left - > type != RED) && (!wNode - > right || wNode - > right - > type != RED)) {
                    if (wNode - > type != NIL)
                        wNode - > type = RED;
                    cNode = wNode - > parent;
                } else {
                    if (!wNode - > left || wNode - > left - > type != RED) {
                        if (wNode - > right && wNode - > right - > type != NIL)
                            wNode - > right - > type = BLACK;
                        if (wNode - > type != NIL)
                            wNode - > type = RED;
                        leftRotate(wNode);
                        wNode = cNode - > parent - > left;
                    }
                    wNode - > type = cNode - > parent - > type;
                    cNode - > parent - > type = BLACK;
                    if (wNode - > left && wNode - > left - > type != NIL)
                        wNode - > left - > type = BLACK;
                    pNode = cNode - > parent;
                    rightRotate(pNode);
                    break;
                }
            }
        } // end while
    } // end else
    if (cNode) {
        if (cNode - > type != NIL)
            cNode - > type = BLACK;
        else
            cNode - > type = NIL;
    }
}
```

未完待续......
