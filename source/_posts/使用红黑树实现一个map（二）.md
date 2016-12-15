---
title: 使用红黑树实现一个map（二）
date: 2016-12-11 22:01:40
tags: [C++,map,数据结构,红黑树,RBTree]
categories: Data Structure
---

&#8195;&#8195;上一篇里把map的框架建好了，接下来几篇将会关注红黑树的实现。

<!-- more -->

# 简述
在分析红黑树的操作前，我们再回顾一下红黑树的定义  

1. 每个节点都是红色或者黑色
2. 根（root）节点和叶子节点（NIL空节点）都是黑色
3. 红节点的子节点不能为红色
4. 从任一节点出发，到达其任一叶子节点的路径中所包含的黑色节点数目相同 

为了维持这一定义，红黑树的插入和删除都需要进行额外的调整。
不过，虽然调整的平均复杂度低于AVL树，但是写起来真是费劲。

# 红黑树的查询
虽然给节点增加了颜色信息，但是查询一棵红黑树跟查询一棵普通的二叉树没什么区别——从根部开始向下，根据查询值和节点的值选择左儿子或者右儿子继续。
```C++
Node * find(Key key) 
{
	Node * ret = NULL;
	if (root == NULL)
		return ret;
	ret = root;
	while (ret != NULL && ret - > vt && ret - > type != NIL) {
		if (comp(key, ret - > vt - > first)) {
			ret = ret - > left;
		} else if (comp(ret - > vt - > first, key)) {
			ret = ret - > right;
		} else {
			break;
		}
	}
	if (ret - > vt && !(comp(ret - > vt - > first, key) && comp(key, ret - > vt - > first)))
		return ret;
	else
		return NULL;
}
```  
注意要用map模板中的comp函数进行比较，不要用'<','>'操作符哦，否则重定义的比较函数就没用了。  

相等的比较在这种情况下等价于:  
```
!(comp(ret - > vt - > first, key) && comp(key, ret - > vt - > first))
```

# 红黑树的插入
## 情况分析
插入可以算是树最基本的操作了，对于红黑树来说，插入新的节点可能会破坏红黑树的规则，所以需要对树的结构进行调整。  

我们记插入节点为C，C的父节点为P，叔叔为W，祖父节点为G。  

首先，新的节点将作为叶子节点被插入。考虑几种插入的情况：  
1. 插入节点为根节点： 直接将插入节点标记为黑色。
2. 插入节点的父节点为黑色： 将插入节点标记为红色，不影响红黑树的性质，结束。 

![figure1](http://ohvmg8dgt.bkt.clouddn.com/rbt_insert_case1-2.png)

上面两种情况显然是我们乐意看见的，处理起来十分简单，但是现实往往不是那么美好：如果插入节点的父节点是红色应该怎么办？  

![figure2](http://ohvmg8dgt.bkt.clouddn.com/rbt_insert_case3-intro.png)  

这个时候的树已经违反了红黑树的第三条性质，出现了连续两个红色节点，需要调整。但是调整的同时还要注意维持红黑书本身的性质。  
  
我们调整的思路是将当前节点、父节点、叔叔节点和祖父节点看成一次调整的对象。
每次调整结束后，只要祖父节点及其以下，确保满足了红黑书的性质，即使这一部分的调整破坏了其对整棵树的平衡性影响，
我们也可以以祖父节点为“当前节点”向上递归调整，直到根节点。  

刚刚的情况，最直接的想法是将当前节点（3）或父节点（2）变为黑色，但是这样会影响到祖父节点（1）子树的高度，又可能造成规则的破坏。  

但是注意到由于父节点P是红色，祖父节点G一定是黑色。如果将父节点变为黑色后，将祖父节点变为红色，则不影响整棵树的平衡性。

......

除非叔叔节点是黑色。

如果叔叔节点是红色，那么将他跟父节点类似地染为黑色则解决了插入节点局部不符合红黑树性质的问题。
只需要将祖父节点当做一个插入节点，向上递归调用调整算法即可。
3. 插入节点的父节点为红色，叔叔节点也为红色：  
 交换父节点和祖父节点的颜色，将叔叔节点染为黑色，以祖父节点为新的插入节点向上递归。
 ![figure3](http://ohvmg8dgt.bkt.clouddn.com/rbt_inser_case3_1.png)  
 
那么如果叔叔节点是黑色呢？  

这个时候，只通过修改节点的颜色好像不能完成对树的调整了。但是不要忘了我们还可以调整红黑树的形态。 

红黑树最经典也是最重要的一种操作：旋转。旋转被对称地分为左旋和右旋，可以将一个局部失衡的状态调整为平衡的状态  
```C++
void leftRotate(Node * & parent) 
{
	Node * subR = parent - > right;
	Node * subRL = subR - > left;

	parent - > right = subRL;
	if (subRL) {
		subRL - > parent = parent;
	}

	subR - > left = parent;
	subR - > parent = parent - > parent;
	parent - > parent = subR;
	parent = subR;

	if (parent - > parent == NULL) {
		root = parent;
	} else if (parent - > parent - > left == parent - > left) {
		parent - > parent - > left = parent;
	} else if (parent - > parent - > right == parent - > left) {
		parent - > parent - > right = parent;
	}
}

void rightRotate(Node * & parent) 
{
	Node * subL = parent - > left;
	Node * subLR = subL - > right;

	parent - > left = subLR;
	if (subLR) {
		subLR - > parent = parent;
	}

	subL - > right = parent;
	subL - > parent = parent - > parent;
	parent - > parent = subL;

	parent = subL;

	if (parent - > parent == NULL) {
		root = parent;
	} else if (parent - > parent - > left == parent - > right) {
		parent - > parent - > left = parent;
	} else if (parent - > parent - > right == parent - > right) {
		parent - > parent - > right = parent;
	}
}
```  

如果将P提升到现在G的位置，将会变成下面这种状态。这个时候只需要对现在的祖父节点G（之前的P）和叔叔节点的颜色，
就保证了G的两侧平衡。  

## 代码实现
有了左旋和右旋，对红黑书插入的四种情况我们都可以处理了。不要忘记分左右两种情况处理。  
```C++
void insert_fixed_up(Node * cNode) 
{
	Node * pNode = cNode - > parent;
	while (cNode != root && pNode - > type == RED) {
		Node * gNode = pNode - > parent;
		Node * uNode = NULL;
		// left
		if (pNode == gNode - > left) {
			uNode = gNode - > right;
			/* case-1:
			 * curr    left
			 * father  left  red
			 * uncle   right red
			 */
			if (uNode && uNode - > type == RED) {
				pNode - > type = uNode - > type = BLACK;
				gNode - > type = RED;
				cNode = gNode;
				pNode = gNode - > parent;
				continue;
			}
			/* case-2:
			 * curr    left
			 * father  left  red
			 * uncle   right BLACK
			 */
			if (cNode == pNode - > right) {
				Node * tmp;
				leftRotate(pNode);
				tmp = pNode;
				pNode = cNode;
				cNode = tmp;
			}
			/* case-3:
			 * curr    right
			 * father  left  red
			 * uncle   right BLACK
			 */
			pNode - > type = BLACK;
			gNode - > type = RED;
			rightRotate(gNode);
		} else if (pNode == gNode - > right) {
		// right
			uNode = gNode - > left;
			if (uNode && uNode - > type == RED) {
				pNode - > type = uNode - > type = BLACK;
				gNode - > type = RED;
				cNode = gNode;
				pNode = gNode - > parent;
				continue;
			}
			if (cNode == pNode - > left) {
				Node * tmp;
				rightRotate(pNode);
				tmp = pNode;
				pNode = cNode;
				cNode = tmp;
			}
			pNode - > type = BLACK;
			gNode - > type = RED;
			leftRotate(gNode);
		}
		pNode = cNode - > parent;
	}
	root - > type = BLACK;
}

```
插入本身比较简单，记得处理一些特殊情况，如第一个节点,已经存在的节点等。  

为了方便操作，给Node增加一个初始化的方法：  
```C++
void initialNode(const value_type & value) 
{
	vt = new value_type(value);
	this - > type = RED;
	left = new Node(NIL);
	right = new Node(NIL);
	left - > parent = right - > parent = this;
}
```
插入的实现
```C++
Node * insert(const value_type & value) 
{
	if (root == NULL) {
		root = new Node(value);
		root - > type = BLACK;
		_size++;
		return root;
	}
	Node * pNode = NULL;
	Node * cNode = root;
	while (cNode != NULL && cNode - > type != NIL) {
		if (comp(cNode - > vt - > first, value.first)) {
			pNode = cNode;
			cNode = cNode - > right;
		} else if (comp(value.first, cNode - > vt - > first)) {
			pNode = cNode;
			cNode = cNode - > left;
		} else // value already exist
		{
			return NULL;
		}
	}
	// insert
	cNode - > initialNode(value);
	cNode - > parent = pNode;
	if (comp(value.first, pNode - > vt - > first)) {
		pNode - > left = cNode;
	} else if (comp(pNode - > vt - > first, value.first)) {
		pNode - > right = cNode;
	}

	// fixed up
	insert_fixed_up(cNode);
	_size++;
	return cNode;
}
```

# 小结
红黑树的插入较为复杂，初次实现，往往难以下手。
但是作为一个已经被广泛使用的数据结构，红黑树的教程也很多。
这里推荐一个网站，对许多数据结构做了可视化的演示实现，并且提供了Web在线展示功能，
对理解数据结构和小规模的调试还是很有帮助的  
[Data Structure Visualizations](cs.usfca.edu/~galles/visualization/)