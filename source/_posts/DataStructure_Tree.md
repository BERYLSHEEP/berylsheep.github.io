---
layout: post
title: "数据结构笔记（三）：树"
date: 2017-08-15
comments: true
tags: 
	- 学习笔记
	- c++
	- 数据结构
---

# **树**

## **概述**

### 出现原因：

向量在静态操作速度快O(1)，在动态操作速度慢O(n)
列表在静态操作速度慢O(n)，在动态操作速度快O(1)
树结合了两者的优点，可看作是一种**半线性的结构**
应用：组织层次关系，如文件系统、学院层级关系等

<!-- more -->

### 树的分类：

![树的分类](http://ot1c7ttzm.bkt.clouddn.com/image/170820/B1l8I45KkG.png?imageslim)

### 概念1. 有根树rooted tree

指定任一结点为根
可以为一系列有根树指定一个结点并连接所有有根树，从而形成一棵更大的有根树，此时各有根树称为子树

![子树](http://ot1c7ttzm.bkt.clouddn.com/image/170815/5hgCeKb8ed.jpg?imageslim)

ri 为r的孩子( child )， ri之间互为兄弟( sibling )
r为父亲( parent ),
d=degree(r)为r的度=一结点拥有的孩子的数目
任何一棵树中边数e和结点数目n同阶

![边数和结点数同阶](http://ot1c7ttzm.bkt.clouddn.com/image/170815/CB0aKme08c.jpg?imageslim)

### 概念2. 有序树

指定Ti为T的第i棵子树，ri为r的第i个孩子

即是：兄弟之间有明显**次序**

### 概念3. 连通性和无环性：

任何两结点之间都有路径，则称为**连通图**（connected）
不含环路则称为**无环图**（acyclic）

树的特点：

> 1 .无环连通图：在无环性和连通性之间平衡
> 2 .极小连通图：在连通的情况下边数尽可能少
> 3 .极大无环图：在无环的情况下边数尽可能多

结论： 任何结点和根之间存在**唯一一条**路径
即 $path(v,r)=path(v)$
我们用一个指标 **深度( depth )** 记录下每个结点到根的距离
path(v)上的结点均为v的**祖先**( ancestor )
v是它们的**后代** ( descendent )
对于v而言，若**祖先存在，则必定唯一**。若**后代存在，却未必唯一**，因此是半线性结构。
而对于图而言，祖先和后代都未必唯一，因此图为非线性结构
**叶子结点**（leaf）：没有后代的结点

叶子深度最大的称为**树的高度**
**深度**是**结点v到总树根结点r**的距离
即height（v）=max( height (v_son) ) +1
结点v的**高度**为**以结点v为根结点**的子树的高度
即 height（v）=height（subtree（v））
约定俗成：空树的高度取作**-1**，一个结点的树高度取作**0**
对于任一结点：depth（v）+height（v）<=height(T)
（高度向下看，深度向上看）
![树的深度和高度](http://ot1c7ttzm.bkt.clouddn.com/image/170815/Lgcj768L1E.png?imageslim)

## **表示**

### 1. 接口

| 返回值   | 结点            | 功能            |
| ----- | ------------- | ------------- |
| node* | root()        | 根结点           |
| node* | parent()      | 父结点           |
| node* | firstchild()  | 长子            |
| node* | nextSibling() | 兄弟            |
| void  | insert( i,e ) | 将e作为第i个孩子插入   |
| int   | remove(i)     | 删除第i个孩子(及其后代) |
| void  | traverse      | 遍历            |

### 2. 构造方法

不妨用数组进行模拟

**长子兄弟法**：

每个结点除了储存数据外，还额外记录下父亲，长子和下一个兄弟的秩

```
template <typename T>
struct node
{
	T data;
	int father, child, nextSibling;
};
node  *tree;
```

# **二叉树**

## **概念**

结点度数**不超过2**（孩子数目<=2）的树

同一结点的孩子和子树以**左右**区分

二叉树是一种特殊的树。然而，二叉树却能够描述所有类型的树。

基于二叉树的概念，我们可以得到下列关系：
1.深度为k的结点最多有$2^k$个
2.高度为h的二叉树的结点数n满足$h’<’n’<’2^(h+1)$
1.当n=h+1时，二叉树退化为一条单链
2.当$n=2^(h+1)-1$时，二叉树为满二叉树

二叉树的宽度涨得非常快，高度为h的满二叉树总共有$2^(h+1)-1$个结点，第h层有$2^h$个结点

而高度h涨得很慢，与结点n的关系为$h=logn$

## **真二叉树(Proper binary tree)**

定义：所有结点的度数均为偶数的二叉树。

很多时候，一棵二叉树每个结点的度数可能在0、1、2中随机分布，为了对后续算法的简洁实现，我们给度数不足2的结点**虚拟地**补上孩子。

## **完全二叉树(complete binary tree)**

定义：叶结点只出现在**最后两层**，并且最底层的叶结点均在次底层叶结点的**左侧**。

![完全二叉树](http://ot1c7ttzm.bkt.clouddn.com/image/170817/I01506Gi5h.JPG)

## **满二叉树(full binary tree)**

定义：所有叶结点都在**最后一层**，每层结点都达到饱和。

特性：
1.结点数目n和高度h的关系：$n=2^(h+1)-1$
2.叶子结点数=内部结点数+1
3.满二叉树是**特殊**的**完全二叉树**

![满二叉树](http://ot1c7ttzm.bkt.clouddn.com/image/170817/JFcg7DgIHi.JPG)

## **用二叉树描述多叉树**

先上结论：**凡是有根且有序的树，均可以用二叉树实现**

为什么呢？让我们先来将一棵树用上文提及的长子兄弟法表示出来：

![树到二叉树](http://ot1c7ttzm.bkt.clouddn.com/image/170815/CF0IK1A4aB.JPG)

可以看到，左侧为树，右侧为长子兄弟表示法，在右侧中，竖直方向的是长子侧，而垂直方向的是兄弟侧，若是将长子和兄弟分别看作一个结点的左右子树，形象地说，就是将它提溜一下提起来

![树到二叉树](http://ot1c7ttzm.bkt.clouddn.com/image/170815/b7h5KJBHaH.JPG)

这就将一棵树变成了二叉树！

这也正是为什么研究二叉树就够了，因为它足以代表树这一类型。

## **基本定义**

**结点类的基本定义：**

```
template <typename T>
struct BinNode
{
	T data;
	BinNode<T>* lChild,*rChild,*parent;
	int height;

	BinNode() { lChild=rChild=parent= NULL ; height=0; }
	BinNode( T e , BinNode<T>* pa=NULL , BinNode<T>* lc=NULL, BinNode<T>* rc=NULL)
		:data(e),parent(pa),lChild(lc),rChild(rc) {}
	int size() const ;
	BinNode<T> * insertAsLC( T const & e ) //作为左孩子插入
	{ return lChild=new BinNode(e,this); }
	BinNode<T> * insertAsRC( T const & e ) //作为右孩子插入
	{ return rChild=new BinNode(e,this); }
	
	BinNode<T> * succ();   // 中序遍历时的直接后继
	
	template <typename V>
	void travLevel( V & visit );  // 子树层级遍历
	template <typename V>
	void travPre( V & visit );   // 子树先序遍历
	template <typename V>
	void travIn( V & visit );    // 子树中序遍历
	template <typename V>
	void travPost( V & visit );   // 子树后序遍历
};

template <typename T>
int BinNode<T>::size()
{
	int s=1;
	if(lChild)
		s+=lChild->size();
	if(rChild)
		s+=rChild->size();
	return s;
}
```

每个结点除了有自己的数据之外，还维护三个指针：父结点地址，左右孩子的地址，还有一个高度数据。此外，结点应维护的其它数据视情况而定。

size函数中，需要递归调用左右孩子的size，即是沿着树枝向下走，走到底后一路返回。

**树的基础定义：**

```
template <typename T>
struct BinTree
{
protected:
	int _size;
	BinNode<T>* _root;
	virtual int updateHeight( BinNode<T>* x); // 更新x结点的高度（用virtual适应不同树对高度的定义）
	void updateHeightAbove( BinNode<T>* x);  // 更新x及x的祖先的高度

public:
	BinTree() {_size=0; _root=NULL;}
	~BinTree() { remove(_root); }
	BinTree( T const& e) {_size=1; _root=new BinNode<T> (e);}
	int size() const {return _size;}
	bool empty() const {return !_root;}
	BinNode<T> * root() const {return _root;}
	BinNode<T> * insertAsRC ( BinNode<T>* x , T const& e );  // 构造右孩子
	BinNode<T> * insertAsLC ( BinNode<T>* x , T const& e );  // 构造左孩子
	BinNode<T> * attachAsLC ( BinNode<T>* x , BinTree<T>*& subtree ); // 接入子树作为左孩子
	BinNode<T> * attachAsRC ( BinNode<T>* x , BinTree<T>*& subtree ); // 接入子树作为右孩子
	int remove( BinNode<T>* x );   // 删除某个结点（及其子树）
	void removeAt( BinNode<T>* x );
	BinTree<T> * secede( BinNode<T>* x );    // 分离子树，返回子树头

	template <typename V>
	void travLevel( V & visit )  // 层级遍历
	{ _root->travLevel(visit); }
	template <typename V>
	void travPre( V & visit )  // 先序遍历
	{ _root->travPre(visit); }
	template <typename V>
	void travIn( V & visit )  // 中序遍历
	{ _root->travIn(visit); }
	template <typename V>
	void travPost( V & visit )   // 后序遍历
	{ _root->travPost(visit); }
};
```

## **遍历**

定义：按照某种次序访问所有的结点，使得所有结点恰好被访问一次

**遍历方式：**

> 1. 先序遍历( preorder )：中->左->右
> 2. 中序遍历( inorder )：左->中->右
> 3. 后序遍历( postorder )：左->右->中
> 4. 层次遍历：自上而下，自左而右
>
> 先序中序后序原则：必定先左后右，根结点访问次序如名字所示

### **1. 先序遍历( 中 左 右 )**

#### **1. 递归版本**

根据先序遍历的定义，我们很容易写出递归版本的遍历函数：

```
// V为函数模板类  visit是相应的函数对象
template <typename T> template <typename V>
void BinNode<T>::travPre_R( V & visit )
{
	visit(data);
	if(lChild) lChild->travPre_R(visit);
	if(rChild) rChild->travPre_R(visit);
}
```

然而，虽然递归和迭代的实现均是O(n)级的，但是它们在常数级所消耗的时间却不同，递归所要消耗的时间远远大，就如O(1)和O(100)均是O(1)，但还是有100倍的差距一般。

因此如果能够将**递归版**改成**迭代版**，就能够提高很多效率。

#### **2. 迭代版本1**

因为在递归版本中，向左右子树的递归出现在最后，即是**尾递归**，那么我们只需要引入一个**栈**，把左右子树的递归改成将左右子树入栈即可。

值得注意的是，由于栈的先进后出特性，在递归中我们先递归左子树，在迭代中入栈操作需要**先让右子树入栈，再让左子树入栈**

```
template <typename T> template <typename V>
void BinNode<T>::travPre_V1( V & visit )
{
	Stack <BinNode<T> *> s;
	BinNode<T>* x=this;
	if(x) 
		s.push(x);
	while(!s.empty())
	{
		x=s.pop();
		visit(x->data);
		if(x->rChild) s.push(x->rChild); // 先入后出
		if(x->lChild) s.push(x->lChild);
	}
}
```

#### **3. 迭代版本2**

看起来迭代版本1很好地完成了任务。然而，它借助了尾递归的特性却不易推广到中序和后序遍历的版本，由此我们需要回顾整个遍历的过程，通过观察找出规律，用另一种易于推广的方式进行迭代。

通过对先序遍历过程的观察，我们可以发现，每当指向一个根结点时，在自己被访问后，它会让目光转向自己的左孩子，而左孩子也同样会在被访问后让目光继续转向自己的左孩子。到最后，最小的左孩子无法转让，只好转到它的右孩子->它父亲的右孩子->父亲的父亲的右孩子…->根结点的右孩子。

于是乎，我们只需要每访问一个根结点时，一边沿着它的左侧链向下走，一边将沿途的右孩子入栈即可，当左孩子访问完后，就取栈中的右孩子访问。

![左侧链示意图](http://ot1c7ttzm.bkt.clouddn.com/image/170816/Jj0H03H434.JPG)

这样，我们就有了如下算法流程：

1.访问该结点
2.将右孩子入栈
3.目光转向它的左孩子
若左孩子存在，回到步骤1
4.若栈不为空，取出栈顶元素，回到步骤1

代码实现如下

```
template <typename T> template <typename V>
void BinNode<T>::travPre_I( V & visit )
{
	Stack<BinNode<T> *> s;
	BinNode<T> * x=this;
	s.push(x);
	while( !s.empty() )
	{
		x=s.pop();
		while(x)
		{
			visit(x->data);
			if(x->rChild) 
				s.push(x->rChild);
			x=x->lChild;
		}
	}
}
```

### **2. 中序遍历( 左 中 右 )**

#### **1. 迭代版本1**

先来观察一下中序遍历的流程：

每当指向一个根结点，它会立刻把目光转向自己的左孩子（自己不先被访问），左孩子也同样转向自己的左孩子。。最后的左孩子无法转让，只好让自己被访问，然后将目光转向自己的右孩子，右孩子访问完后返回自己的父亲结点，这时父亲只能被访问，然后转向它的右孩子。。。

中序和先序不同在于：**父结点不是立即被访问**，它要等自己的左子树访问完之后才会被访问，那么这一层层传递下来的左侧链，就跟一个个结点入栈一般。没错，在这个过程中，我们需要将根结点一个个入栈。访问完左结点之后，我们就把根取出来访问。

那右结点怎么办呢？

右结点在根访问完之后受到关注时，它就是自己子树的根结点了，它同样也要继续左侧链入栈的流程。

![中序遍历](http://ot1c7ttzm.bkt.clouddn.com/image/170816/amHbf583EI.JPG)

算法的流程如下：

1.将该结点入栈
2.转向该结点的左孩子
若左孩子存在，返回步骤1
3.若栈不为空，取出栈顶元素，访问后，转向它的右孩子
若右孩子存在，返回步骤1
若右孩子不存在，重新开始步骤3

现在，我们就可以在观察之后，将先序的迭代2版本沿用到中序遍历中了。

```
template <typename T> template <typename V>
void BinNode<T>::travIn_I( V & visit )
{
	Stack<BinNode<T> *> s;
	BinNode<T> * x=this;
	while(1)
	{
		while(x)
		{
			s.push(x);
			x=x->lChild;
		}
		if(s.empty()) 
			break;
		x=s.pop();
		visit(x->data);
		x=x->rChild;
	} 
}
```

此外，中序遍历还有一个特性，由此我们可以得到一个连栈都不需要用到(但时间消耗会上升)的迭代版本

#### **2. 迭代版本2**

当把二叉树横向伸展地足够开后，如下图所示
![中序遍历](http://ot1c7ttzm.bkt.clouddn.com/image/170817/JEkGbFc4E4.png?imageslim)
从左向右扫过去，各个结点被访问的次序就是中序遍历的顺序
左右规则如下:

左子树左孩子>左子树根结点>左子树右孩子>根结点>右子树左孩子>右子树根结点>右子树右孩子

根据这个规则，我们就可以得到每个结点在中序遍历时的**直接后继**，即是在空间上在其右边的第一个结点。(最右边的直接后继为NULL)

那么要怎么得到这个直接后继呢？
显然，如果一个结点有右子树，那么它的直接后继必然在右子树中，只需要沿着右子树的左侧链一直向下到底即可。
而要是它没有右子树，就要麻烦一些了，这时我们需要向上找，如果这个结点是它父结点的右孩子，那就还需要继续向上，直到找到一个**结点是它父结点左孩子**的结点，直接后继就是这个结点的父结点。
简单来说，这个没有右子树的结点不是最右边的结点，就必然是某棵左子树最右边的结点，它的直接后继自然是这棵左子树的父亲了。

将上述流程翻译成代码就是

```
template <typename T>
BinNode<T> *BinNode<T>::succ()
{
	BinNode<T>* s=this;
	if(rChild)   // 若有右孩子
	{
		s=rChild;
		while(s->lChild)   // 在右子树的左侧链一路到底
			s=s->lChild;
	}
	else
	{
		while( s->parent && s==s->parent->rChild )
			s=s->parent;    // 此时是左子树的根结点
		s=s->parent;   // 再向上到左子树的父亲结点(也可能是NULL)
	}
	return s;
}
```

有了直接后继函数之后，剩下的事情就简单了，我们只需要在最开始找到最左边的结点(左侧链最深的结点),然后把这火车开下去就好了。

```
template <typename T> template <typename V>
void BinNode<T>::travIn_S(  V & visit )
{
	BinNode<T>* x=this;
	while(x->lChild)
		x=x->lChild;
	do
	{
		visit(x->data);
		x=x->succ();
	}
	while(x);
}
```

### **3. 后序遍历( 左 右 中 )**

基于后序遍历的定义，我们可以导出如下流程：
1.先尽可能沿着左走，若是结点实在没有左孩子，只有右孩子，那么向右走一次也行，直到走到叶结点。
2.访问结点
3.若右兄弟存在，转向右兄弟，返回步骤1
4.向上回溯到父结点，返回步骤2，若无父结点，说明已经遍历完毕，退出流程

可以看到，在向当前结点的左孩子走时，我们需要记录下当前的结点以及它的右孩子（而且右孩子先），左孩子访问完后，取出右孩子遍历并访问，再取出父亲结点访问。由此我们需要一个栈。

```
template <typename T> template <typename V>
void BinNode<T>::travPost_I( V & visit )
{
	Stack<BinNode<T> *> s;
	BinNode<T> * x=this;
	s.push(x);
	BinNode<T> * c=s.top();
	while(1)
	{
		if(s.empty()) break;
		if(s.top()!=x->parent) // 不是父结点说明是栈顶是右兄弟，需要遍历它的子树
		{
			c=s.top();
			while(1)
			{
				if(c->rChild) 
					s.push(c->rChild);
				if(c->lChild) 
				{
					s.push(c->lChild);
					c=c->lChild;   // 尽可能向左走
				}
				else if(c->rChild)  // 如果实在只有右孩子，那就向右
					c=c->rChild;
				else  // 左右孩子都没有，走到底了，退出循环
					break;
			}
		}
		x=s.pop();
		visit(x->data);
	}
}
```

### **4. 层次遍历**

层次遍历的规则很简单：自上而下，自左而右

![层次遍历](http://ot1c7ttzm.bkt.clouddn.com/image/170817/a6h1hlafBm.jpg?imageslim)

对此，我们可以引入**队列**来解决它。

每当遇到一个结点，我们在访问它后将它的左右孩子(若存在)入栈，然后再取队首元素重复操作。

```
template <typename T> template <typename V>
void BinNode<T>::travLevel( V & e )
{
	Queue<BinNode<T>*> q;
	BinNode<T>* x=this;
	q.enqueue(x);
	while( !q.empty() )
	{
		x=q.dequeue();
		if(x->lChild)  q.enqueue(x->lChild);
		if(x->rChild)  q.enqueue(x->rChild);
		e(x->data);
	}
	return;
}
```

## **重构**

如果我们已经有了按某种方式遍历出的序列，那么如何通过序列重新构造出原本的二叉树？

### **一. 中序+(先序|后序)**

结论一：我们只需要 中序+(先序|后序)，即是中序遍历的序列加上先序或后序的任一序列即可构造出原本的二叉树。

现在我们来用数学归纳法证明一下：
假设结论在结点数n’<’N的情况下都成立
在n==N时
先假设我们有先序和中序遍历的序列，根结点为r，左右子树分别为L和R。
那么在先序遍历和中序遍历中分别如下图所示：

![重构二叉树](http://ot1c7ttzm.bkt.clouddn.com/image/170815/Hci6Ijk2bg.JPG)

可以看到，我们可以根据r来成功划分L和R序列，在中序遍历的序列中我们可以知道左右子树分别有哪些结点，进而在先序的序列中将它们划分开。
这时就形成了已知两棵子树的先序和中序遍历的序列来重构二叉树了，而因为结点数在n’<’N的左右子树中结论均成立，由此n==N时结论也成立。

而有后续和中序遍历的序列的证明也同理可得了。

```
// 先序+中序重构
template <typename T>
void BinTree<T>::rebuild_PI( T* pre, T* ins, int len )
{
	_root=rebuildSub_PI(pre,ins,len);
}
template <typename T>
BinNode<T> * rebuildSub_PI( T* pre, T* ins, int len ) 
{
	BinNode<T> *s=NULL;
	if(len<=0) return s;
	if(len==1)
	{
		s=new BinNode<T>(*pre);
		return s;
	}
	int i;
	for(i=0; *(ins+i)!=*(pre); i++);
	s=new BinNode<T>(*pre);
	s->lChild=rebuildSub_PI( pre+1, ins, i );
	s->rChild=rebuildSub_PI( pre+1+i, ins+i+1, len-i-1 );
	return s;
}
```

```
// 中序+后序重构
template <typename T>
void BinTree<T>::rebuild_IP( T* ins, T* post, int len )
{
	_root=rebuildSub_IP(ins,post,len);
}
template <typename T>
BinNode<T> * rebuildSub_IP( T* ins, T* post, int len ) 
{
	BinNode<T> *s=NULL;
	if(len<=0) return s;
	if(len==1)
	{
		s=new BinNode<T>(* (post+len-1) );
		return s;
	}
	int i;
	for(i=0; *(ins+i)!=*(post+len-1); i++);
	s=new BinNode<T>(*(post+len-1));
	s->lChild=rebuildSub_IP( ins, post, i );
	s->rChild=rebuildSub_IP( ins+1+i, post+i, len-i-2 );
	return s;
}
```

### **二. (先序+后序)\*真二叉树**

结论二：在只有先序和后序序列时，若是该二叉树是一棵**真二叉树**(所有结点的度数都是偶数),那么也可以构造出原本的二叉树来。

那么要怎么做呢？

在先序遍历中，若根结点有孩子，则必定左右都有。那么这个序列第一个必然是根结点，而第二个必然是左子树的根。
在后序遍历中，序列的最后必然是根，倒数第二个是右子树的根。
这样我们就知道了左右子树根结点是什么样的，进而可以完整分割出左右子树分别在先序和后序遍历的序列。而在这之后，不过是问题规模缩小的两个 (先序+后序)*真二叉树问题罢了。
![重构二叉树](http://ot1c7ttzm.bkt.clouddn.com/image/170815/C89dHdbAkF.JPG)

```
template <typename T>
void BinTree<T>::rebuild_PP( T *pre,T *post,int len )
{
	_root=new BinNode<T> (pre[0]);
	rebuildSub_PP( _root,pre,post,len );
	return;
}
template <typename T>
BinNode<T> * rebuildSub_PP( T *pre , T *post, int len )
{ 
	BinNode<T> *s=NULL;
	if( len<=0 ) return s;
	if(len==1)
	{
		s=new BinNode<T>(*pre);
		return s;
	}
	int i;
	for(i=0; *(post+i)!=*(pre+1); i++);
	s=new BinNode<T>(*pre);
	s->lChild=rebuildSub_PP(pre+1,post,i+1);
	s->rChild=rebuildSub_PP(pre+i+2,post+i+1,len-i-2);
	return s;
}
```

# **完整实现代码**

[二叉树的完整实现 ( 附带遍历和重构 )](https://github.com/zedom1/DSA/blob/master/tree/binary%20tree.cpp)

# 二叉搜索树BST ( Binary Search Tree )

## 概述

### 循关键码访问 call-by-key

数据项之间，依照各自的关键码(key)彼此区分

条件：关键码之间支持**大小比较**和**相等比对**

### 性质

二叉搜索树（又叫二叉排序树或二叉查找树），是一棵二叉树，可以为空，若不为空，则满足：

```
1. 非空左子树所有键值小于根结点的键值
2. 非空右子树所有键值大于根结点的键值
3. 左右子树都是二叉搜索树

```

基本框架：

```
template <typename T>
struct TreeNode
{
	T data;
	TreeNode<T> * left;
	TreeNode<T> * right;
};
```

## 接口

```
1. BinTree Find( T x , BinTree BST);

```

> 从BST中查找元素x,返回所在结点的地址

```
2. BinTree FindMin(BinTree BST);

```

> 从BST中查找并返回最小元素所在结点的地址

```
3. BinTree FindMax(BinTree BST);

```

> 从BST中查找并返回最大元素所在结点的地址

```
4. BinTree Insert( T x , BinTree BST);

```

> 将元素x插入BST中，返回插入后的BST

```
5. BinTree Delete( T x , BinTree BST);

```

> 从BST中删除元素x

### 查找操作： Find

```
- 查找从根结点开始，若树为空则返回NULL
- 若树非空，则将根结点键值和x比较：
    - 若 x < root.data ，在左子树中搜索
    - 若 x > root.data ，在右子树中搜索
    - 若两者相等，则返回指向该结点的指针
- 实现方式可以采用递归或迭代，因为是尾递归，因此很容易将递归形式改造成迭代形式
- 当树退化成链时，查找效率退为O(n)，因此最好组织成**平衡二叉树**

```

代码实现：

```
template <typename T>
TreeNode<T> * Find(T x , TreeNode<T> * BST)
{
	while(BST)
	{
		if(x>BST->data)
			BST=BST->right;
		else if(x<BST->data)
			BST=BST->left;
		else
			return BST;
	}
	return NULL;
}
```

### 查找最大和最小元素

根据二叉搜索树的性质：

```
- 最大元素一定在树的最右分支的端结点上
- 最小元素一定在树的最左分支的端结点上

```

同样的，查找最大最小元素也可以用递归和迭代的形式实现，鉴于递归同样也是尾递归，因此可以轻松地转化成迭代的版本

```
template <typename T>
TreeNode<T> * FindMax( TreeNode<T> * BST)
{
	if(BST)
		while(BST->right)
			BST=BST->right;
	return BST;
}
template <typename T>
TreeNode<T> * FindMin( TreeNode<T> * BST)
{
	if(BST)
		while(BST->left)
			BST=BST->left;
	return BST;
}
```

### 插入结点

关键：要找到元素应该插入的位置，可以采用和Find类似的方法

思路：

```
template <typename T>
TreeNode<T> * Insert( T x , TreeNode<T> * BST)
{
	if(!BST)
	{
		BST = new TreeNode<T>();
		BST->data=x;
		BST->left=BST->right=NULL;
	}
	else
	{
		if(x<BST->data)
			BST->left=Insert(x,BST->left);
		else if(x>BST->data)
			BST->right=Insert(x,BST->right);
	}
	return BST;
}
```

### 删除结点

考虑情况

```
1. 删除的是叶结点，则直接删除结点并将父结点指向该结点的指针修改为NULL
2. 要删除的结点只有一个孩子，则将父结点指向该结点的指针指向要删除结点的孩子
3. 要删除的结点有左右两棵子树
    用另一结点代替被删除结点：左子树最大元素或右子树最小元素

```

```
template <typename T>
TreeNode<T> * Delete( T x , TreeNode<T> * BST)
{
	TreeNode<T> *tmp;
	if(!BST) return NULL;
	else if(x<BST->data)
		BST->left=Delete(x,BST->left);
	else if(x>BST->data)
		BST->right=Delete(x,BST->right);
	else
	{
		if(BST->left && BST->right)
		{
			tmp=FindMin(BST->right);
			BST->data=tmp->data;
			BST->right=delete(BST->data,BST->right);
		}
		else
		{
			tmp=BST;
			if(!BST->left)
				BST=BST->right;
			else if(!BST->right)
				BST=BST->left;
			delete tmp;
		}
	}
	return BST;
}
```

# 平衡二叉树 AVL树 (Balanced Binary Tree)

定义：
空树，或者任一结点左右子树高度差绝对值不超过1，即 |BF(T)|<=1

**平衡因子（BF:Balance Factor）**: BF(T)=Hl - Hr
Hl和Hr分别为T的左右子树的高度

推算高度为h的平衡二叉树最少结点数：

```
高度为1时 h(1)=1;
高度为2时 h(2)=2;
高度为3时 h(3)=h(1)+h(2)+1=4;
...
高度为n时 h(n)=h(n-1)+h(n-2)+1;
原因：
    > 一个平衡二叉树的左右子树均是平衡二叉树，并且左右子树的高度差的绝对值<=1
    > 因此一棵高度为n的平衡二叉树可以由两棵高度为n-1的平衡二叉树构成或是一棵n-1+一棵n-2（不然不符合高度差的规则），又因为需要最少结点数，因此选择n-1 + n-2的组合，即高度为n的平衡二叉树的结点数 h(n)=h(n-1)+h(n-2)+1 ( 1是根结点本身 )
由此得到结论：高度为h的平衡二叉树的最少结点数为 $$ h(n) = h(n-1) + h(n-2) + 1 $$

```

结点数为n的平衡二叉树的最大高度为O(logn)

## 平衡二叉树的调整

**核心思想：选择中间值作为根结点**

**RR插入**：插入结点在右子树的右边，需要RR旋转（右单旋）

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170915/dlLJlGI0dh.JPG)

大小关系为： 根结点 < 右子树根结点 < 右子树的右边

因此选取右子树的根结点作为新的平衡二叉树的根结点

**LL插入**：插入结点在左子树的左边，需要LL旋转（左单旋）

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170915/57fehe3J04.JPG)

大小关系为： 左子树的左边 < 左子树根结点 < 根结点

因此选取左子树的根结点作为新的平衡二叉树的根结点

**LR插入**：插入结点在左子树的右边，需要LR旋转

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170915/E1l83GHjKg.JPG)

大小关系为： 左子树的根结点 < 左子树的右边 < 根结点

因此选取左子树的右子树的根结点作为新的平衡二叉树的根结点

**RL插入**：插入结点在右子树的左边，需要RL旋转

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170915/FEGfD0A16g.JPG)

大小关系为： 根结点 < 右子树的左边 < 右子树的根结点

因此选取右子树的左子树的根结点作为新的平衡二叉树的根结点

# 堆 heap

## 概述

**优先队列( Priority Queue )**
特殊的队列，取出元素的顺序是依照元素的**优先权（关键字）**的大小，而不是元素进入队列的先后顺序

堆的特性：
**结构性**：用数组表示的完全二叉树
**有序性**：任一结点的关键字是其子树所有结点的最大值（或最小值）
从根结点到任意结点路径上结点序列的有序性
最大堆（MaxHeap）：根结点为最大值
最小堆（MinHeap）：根结点为最小值

## 接口

数据对象集：完全二叉树

接口：
MaxHeap Create( int Maxsize );
创建一个空的最大堆
Boolean IsFull( MaxHeap H );
判断最大堆H是否满
void Insert( MaxHeap H , T data );
将data插入最大堆H
Boolean IsEmpty( MaxHeap H );
判断最大堆H是否为空
T DeleteMax( MaxHeap H );
返回H中最大元素（高优先级）

### Insert

思路：

> 将元素插入到数组最后，而后不断与其父结点进行比较，若新元素大于父结点，则与父结点互换，直至到根结点

```
template <typename T>
void Insert( Heap<T>* h, T item)
{
	int i;
	if( IsFull(h) ) // 若堆已满，则不做插入操作
		return;
	i = ++h->size;
	for( ; h->data[i/2]<item ; i/=2 )
		h->data[i] = h->data[i/2];
	h->data[i] = item;
}
```

### Delete

思路：

> 取出根结点元素后，将最后一个结点移至根结点，而后不断与左右结点进行比较及互换，最后完成删除操作

```
template <typename T>
void Delete( Heap<T>* h , T item )
{
	int parent , child;
	T maxn,tem;
	if( IsEmpty(h) ) 
		return;
	maxn = h->data[1];
	tem = h->data[h->size--];
	for( parent=1 ; parent*2<= h->size ; parent = child )
	{
		child = parent *2;
		if( child != h->size  && h->data[child]<h.data[child+1])
			child++;   // 寻找左右孩子中较大的那个
		if( tem>= h->data[child] ) 
			break;
		else
			h->data[parent] = h->data[child];
	}
	h->data[parent]=tem;
}
```

### 建立最大堆

将已经存在的N个元素按最大堆的要求存放在一个一维数组

方法1：
通过Insert函数把n个元素一个个插入堆中，时间代价为O(nlogn)

方法2：

```
1. n个元素按输入顺序存入，先满足完全二叉树
2. 调整各结点位置

```

#### 方法二：

自底向上调整

从倒数第二行的结点开始自右向左调整

而后再到倒数第三行、倒数第四行…直到根结点

这样每个结点在调整时它的左右均是堆，如同删除操作中那样不断比较即可

这样就只需要O(n)的时间即可完成建堆

# 哈夫曼树

带权路径长度（WPL）
设二叉树有n个叶子结点，每个叶子结点带有权值wk,从根结点到每个叶子结点的长度为lk，则每个叶子结点的带权路径长度之和为 WPL = $ \sum $ wk*lk

哈夫曼树（最优二叉树）：WPL值最小的二叉树

## 哈夫曼树的构造

每次把**权值最小**的两棵二叉树合并

## 特点

```
1. 没有度为1的结点
2. n个叶子结点的哈夫曼树有2n-1个结点
    设 n0: 叶子结点总数
    设 n1: 只有一个儿子的结点总数
    设 n2: 有2个儿子的结点总数
    则 n2=n0-1, 因为不存在只有一个儿子的结点，因此总结点数 n0+n2=2n0-1
3. 哈夫曼树任意非叶结点的左右子树交换后仍然是哈夫曼树
4. 对同一组权值，存在不同构的两棵哈夫曼树 （但WPL值一样）

```

## 哈夫曼编码

```
给定一段字符串，对字符进行编码，使得该字符串编码的存储空间最少

避免二义性：
    使用前缀码（prefix code）: 任何字符的编码都不是另一字符编码的前缀

利用二叉树进行编码：
    1. 左右分支：0、1
    2. 字符只在叶结点上

构造方法：根据字符的权值构造哈夫曼树

```

# 集合

```
集合运算：交、并、补、差、判定一个元素是否属于某个集合
并查集：集合并、查某元素属于什么集合
并查集实现：
    1. 利用树结构表示集合，每个结点代表一个集合元素（双亲表示法）
    2. 利用数组存储，两个一维数组分别存储数据和父亲下标

```

利用数组存储的定义：

```
template <typename T>
struct Node
{
	T data;
	int parent;
	Node( T d , int parent = -1 ):data(d){}
	Node(){parent=-1;}
};
```

每个结点除了维护自己的数据外，还额外维护了父结点在数组中的下标

## 查找当前结点所属集合

思路：
首先在数组中找到目标结点
而后顺着父结点下标这条链一直向上直到找到某个没有父结点的结点

```
template <typename T>
int Find( Node<T>* s , T x )
{
	/* 在s中查找值为x的元素所属的集合 */
	/* Maxsize为全局变量，s的最大长度 */
	int i;
	for( i=0; i<MaxSize&& s[i].data!=x; i++ );   // 在数组中寻找值为x的结点
	if( i>=MaxSize ) return -1;    // 没找到，返回-1
	for( ; s[i].parent>=0; i=s[i].parent );  // 顺着父结点指针一路向上找
	return i;
}
```

优化：
通过**路径压缩**，每次查找时把沿途所有节点的parent都设为根节点

```
template <typename T>
int Find( Node<T>* s , T x )
{
	/* 在s中查找值为x的元素所属的集合 */
	/* Maxsize为全局变量，s的最大长度 */
	int i;
	for( i=0; i<MaxSize&& s[i].data!=x; i++ );
	if( i>=MaxSize ) return -1;
	int tem=i,tem1=s[tem].parent;
	for( ; s[i].parent>=0; i=s[i].parent );
	while( tem1!=-1 && tem1!=i )
	{
		s[tem].parent = i;
		tem=tem1;
		tem1=s[tem].parent;
	}
	return i;
}
```

## 集合的并运算

思路：
首先分别找到两个元素所在的集合树的根结点
若根结点不同，则把其中一个根结点的父结点指针设置成另一个根结点数组下标

```
template <typename T>
void Union( Node<T>* s, T x1 , T x2)
{
	int root1,root2;
	root1=Find(s,x1);
	root2=Find(s,x2);
	if(root1!=root2)
		s[root2].parent = root1;
}
```

优化：
为了改善合并后的查找效率，把小的集合并入大的集合
方法1：每个结点额外维护一个值：以该结点为根的结点总数
缺点：只有根节点才需要用到该值，造成大量空间浪费
方法2：已知根节点的parent为-1，那么我们可以用负数来代表节点总数

```
template <typename T>
void Union1( Node<T>* s, T x1 , T x2)
{
	int root1,root2;
	root1=Find(s,x1);
	root2=Find(s,x2);
	if(root1!=root2)
	{	
		if( s[root1].parent<s[root2].parent )
		{	
			s[root1].parent += s[root2].parent;
			s[root2].parent = root1;
		}
		else
		{	
			s[root2].parent += s[root1].parent;
			s[root1].parent = root2;
		}
	}
}
```