---
layout: post
title: "题目小结：二叉树 ( 建树和遍历 )"
date: 2017-08-18
comments: true
tags: 
	- c++
	- 数据结构
	- 刷题
---


# 二叉树练习：

分为：
树的构建
树的同构
子树节点数
自定义遍历顺序
子树相同
Tree Travels again

<!-- more -->

## 1. 树的构建

有关二叉树的题目有以下几种输入建树方式：
1.前序+中序遍历序列
2.中序+后序遍历序列
3.前序+后序序列，同时说明该二叉树的所有节点子节点数为偶数
4.每个节点子节点数目以及子节点编号
5.先序序列，空节点以’,’代替

### 1. 前序+中序遍历序列

![前序+中序](http://ot1c7ttzm.bkt.clouddn.com/image/170818/hCiffgFFLD.JPG)

从图中可以看出，前序的第一个节点为根节点，第二个为左孩子，通过拿根节点在中序中搜索，可以将中序的序列分成左右两部分，由此可以得到左序列的长度，进而能够在前序中确定右孩子，而后在递归构建左子树和右子树。

用流程表述如下：
在前序中确定根节点->在中序中确定根节点->得知左右序列的长度->递归左右序列

你可能会疑惑，那左右节点呢？不是能够确定吗？
没错，但是会额外增加判断和细节确定，不如一次递归确定根节点，然后递归左右子树。

```
node *build( char *pre, char * ins, int n)
{
	node *s=NULL;
	if(n<=0) return s;
	if(n==1)
	{
		s=new node(*pre);
		return s;
	}
	int i;
	for(i=0; *(ins+i)!=*pre ; i++);
	s= new node (*pre);
	s->lc=build(pre+1,ins,i);
	s->rc=build(pre+i+1,ins+i+1,n-i-1);
	return s;
}
```

### 2. 中序+后序遍历序列

![中序+后序](http://ot1c7ttzm.bkt.clouddn.com/image/170818/lJ1GBIf81b.JPG)

和前序+中序一般，这次不过是根节点在最后，流程是类似的。

```
node *build(char *ins, char *post, int n)
{
	node *s=NULL;
	if(n<=0) return s;
	if(n==1)
	{
		s=new node (*(post+n-1));
		return s;
	}
	int i;
	for(i=0; *(ins+i)!=*(post+n-1); i++);
	s=new node (*(post+n-1));
	s->lc=build(ins,post,i);
	s->rc=build(ins+i+1,post+i,n-i-1);
	return s;
}
```

### 3. 前序+后序序列

![前序+后序](http://ot1c7ttzm.bkt.clouddn.com/image/170818/miHddAiGEf.JPG)

那在前序+后序中，根节点不是最前就是最后，怎么确定左右序列长度呢？
这时候就靠左孩子了。而根据给出的性质：**所有节点的子节点数均为偶数**可以判断，一个节点要么没有孩子，要么就有两个孩子，这也是我们能够大胆判定序列第二个是左孩子而不用担心只有左孩子而没有右孩子的情况。（没有左右孩子，只有根节点就直接返回了）

明确思路后修改一下细节就可以得到如下代码

```
node * build( char *pre , char *post, int len )
{ 
	node *s=NULL;
	if(len<=0 ) return s;
	if(len==1)
	{
		s=new node (*pre);
		return s;
	}
	int i;
	for(i=0; *(post+i)!=*(pre+1); i++);
	s=new node (*pre);
	s->lc=build(pre+1,post,i+1);
	s->rc=build(pre+i+2,post+i+1,len-i-2);
	return s;
}
```

### 4. 子节点编号

假设题目的输入如下：
第一行为节点总数n，节点编号1-n
紧接n行，每行三个数字，第一个数字代表该节点权值，第二三为左右孩子的编号，若无孩子，以0代替。
范例如下：

```
7 
1 6 3
2 0 4
1 7 0
3 0 0
1 2 1
2 0 0
2 0 0
```

显然，我们有四种数据需要处理：该节点编号，该节点权值，该节点左右孩子编号。而最重要的一点是：**没有给出根节点编号！**，除非题目明确表示几号为根节点，否则默认将1号当做根节点那可就从开始就错了。

我个人的做法是：用3个int数组(视情况可能1个char+2个int) +1个bool数组
3个int数组a[i],llc[i],rrc[i]分别保存编号为i的权值、左孩子编号、右孩子编号。
1个bool数组isroot用来确定根节点。题目保证必定能够构造一棵合法的二叉树，那么就不会出现1棵树+1个孤立节点的情况，那么，不被任何节点当做左右孩子的节点就自然是根节点了。只需要在输入左右孩子编号时额外在isroot里维护一下，最后扫描一遍isroot数组就能够确定root的编号了。

build函数很简单，若有孩子则递归构建。

```
node * build(int n)
{
	node *s=new node(a[n]);
	if(llc[n])
		s->lc=build(llc[n]);
	if(rrc[n])
		s->rc=build(rrc[n]);
	return s;
}
```

那输入要怎么处理？

```
memset(isroot,0,sizeof(isroot)); // 数组归零，很重要！
for(int i=1; i<=n; i++)
{
	scanf("%d",&a[i]);
	scanf("%d%d",&llc[i],&rrc[i]);
	isroot[llc[i]]=isroot[rrc[i]]=1;   // 此处题目已经说若无孩子则为0，而编号从1开始，因此无关紧要，若是规定为-1，则需要额外处理
}
for(nroot=1; isroot[nroot]==1; nroot++);
tree=build(nroot);
```

### 5. 先序序列

还有的时候，题目只给出先序序列，但是额外表明了空节点（一般以’,’标明）
如

```
abc,,de,g,,f,,,
```

很简单，只需要模仿先序遍历的过程即可，重要的在于要确定数组的秩

```
node *build( int &fir , int n) // 用引用很重要！否则在递归中会丢失自增的数据
// 而之所以不把传fir改为把数组的首地址传进来，因为判断是否到达数组尾部需要额外的功夫，不如一句if(fir>=n)方便
{
	node *s=NULL;
	if(fir>=n) return s;
	if(pre[fir]==',' ) return s;
	s=new node(pre[fir]);
	s->lc=build(++fir,n);
	s->rc=build(++fir,n);
	return s;
}
```

## 2. 树的同构

[SDUT 3340: 数据结构实验之二叉树一：树的同构](http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/3340.html)

题意：给定两棵树T1和T2。如果T1可以通过若干次左右孩子互换就变成T2，则我们称两棵树是“同构”的。

例如图1给出的两棵树就是同构的，因为我们把其中一棵树的结点A、B、G的左右孩子互换后，就得到另外一棵树。而图2就不是同构的。

![图1](http://ot1c7ttzm.bkt.clouddn.com/image/170818/5aAHAglgKi.png?imageslim)

![图2](http://ot1c7ttzm.bkt.clouddn.com/image/170818/dha1ah025f.png?imageslim)

难点：

1. 树的构建
2. 同构判断

坑：

1. n和m可能不同
   在n、m不等的情况下，你还是必须要把数据录入完，不然像这样连续输入的数据，上一组不录完直接出结果会影响下一组的录入。
2. n和m可能为0 (分别或同时)

判等思路：
1.粗糙版：贪心同构tree1，看看能不能变成tree2。
tree1和tree2同步递归，每到一个节点判断左右节点是否相同（可能不存在），若不同就交换左右节点。最后两棵树都出先序遍历的序列判断是不是一模一样

2.递归巧妙版：判断当前节点数值是否相等
若两个节点都不存在，返回正确。
若均存在且相等，递归查询（tree1左孩子+tree2左孩子）&&（tree1右孩子+tree2右孩子） 或者 （tree1左孩子+tree2右孩子）&&（tree1右孩子+tree2左孩子）。
若不等或者只有其中一个存在，返回错。

```
#include<iostream>
#include<cstdio>
#include<string.h>
#include<stdlib.h>
#include<cstring>
#include<ctime>
#include<cmath>
#include<math.h>
using namespace std;

char a[15];
int llc[15],rrc[15];
int n,m;
char tem1,tem2;
bool isroot[15];
int nroot;

struct node
{
	char data;
	node *lc,*rc;
	node(){ lc=rc=NULL;}
	node(char d, node * llc = NULL, node * rrc=NULL):data(d),lc(llc),rc(rrc){}
};

node * build(int i)
{
	node *s=new node (a[i]);
	if(llc[i]!=-1)
		s->lc=build(llc[i]);
	if(rrc[i]!=-1)
		s->rc=build(rrc[i]);
	return s;
}

node * rebuild(int n1)  // 坑点！ 需要把节点数传进去，若用全局的n或者m会错，因为nm不一定相等
{
	nroot=0;
	memset(isroot,0,sizeof(isroot));
	memset(llc,0,sizeof(llc));
	memset(rrc,0,sizeof(rrc));
	for(int i=0; i<n1; i++)
	{	
		cin>>a[i]>>tem1>>tem2;
		if(tem1=='-') 
			llc[i]=-1;
		else
		{
			llc[i]=tem1-'0';
			isroot[llc[i]]=1;
		}
		if(tem2=='-') 
			rrc[i]=-1;
		else
		{
			rrc[i]=tem2-'0';
			isroot[rrc[i]]=1;
		}
	}
	for( nroot=0; nroot<n1; nroot++)
		if(isroot[nroot]==0)
			break;
	return build(nroot);
}

bool change(node*r1, node *r2)
{
	if(!r1 && !r2) 
		return 1;
	if(r1&&r2)
	{
		if(r1->data==r2->data)
		{
			if( (change(r1->rc,r2->rc))&&(change(r1->lc,r2->lc))  || (change(r1->lc,r2->rc))&&(change(r1->rc,r2->lc))   )
				return 1;
		}
	}
	return 0;
}

int main()
{
while(cin>>n)
{
	node* tree1,*tree2;
	if(n)
		tree1=rebuild(n);
	cin>>m;
	if(m)
		tree2=rebuild(m);
	if(change(tree1,tree2))
		printf("Yes\n");
	else
		printf("No\n");
}
	return 0;
}
```

## 3. 子树节点数

[九度OJ 1113：二叉树](http://ac.jobdu.com/problem.php?pid=1113)

题意：一棵完全二叉树，层次遍历为1-n。给定一节点数值m和最后一个节点数值n，求m的子树节点数

一道数学题，不需要构造二叉树，但是需要对二叉树的各种性质比较了解。

个人想法（不一定为最简）：
先求得m和n所在二叉树的高度lenm和lenn，求得’<’lenn时的总子树节点数，问题就在于最后一层。我的做法是通过n求得最后一层有多少个节点，然后看m同层左边有多少个节点(count)，假设在最后一层m可以加上po个节点，那左边还有count*po个节点，最后一层的节点数减去count*po后，剩下在po范围里的则是最后一层m可以加上的节点数。

![解法图示](http://ot1c7ttzm.bkt.clouddn.com/image/170822/BEacbGD51k.png?imageslim)

cmath里的log函数是以e为底，因此需要手动用换底公式换成以2为底。

```
#include<iostream>
#include<cstdio>
#include<string.h>
#include<stdlib.h>
#include<cstring>
#include<ctime>
#include<cmath>
#include<math.h>
using namespace std;
 
int getheight(int num)
{
    return (int)(log(num)*1.0/(log(2)*1.0));
}
 
int main()
{   
    int m,n;
	while(  scanf("%d%d",&m,&n)&& m+n  )
	{   
		int lenm=getheight(m),lenn=getheight(n);
		int count=0;
		int po=1;
		int ans=0;
		for(int i=m-1; ; i--)
		{
			if(getheight(i)!=lenm)
				break;
			count++;  // 找出同层中m左边有多少个节点
		}
    
		for(int i=lenm; i<lenn; i++)
		{
			ans+=po;   // 在lenn层前的子树节点总数
			po*=2;
		}
		count*=po;
		int fir=pow(2,lenn);  // 第lenn层第一个节点的编号
		if(n<=count+fir-1)   // n数目很少，都没有进入m在这层的子树范围
			printf("%d\n",ans);
		else
		{
			n-= count+fir-1;
			n=min(n,po);    // 也可能n很大，超出了m的子树范围
			ans+=n;
			printf("%d\n",ans);
		}
	}
    return 0;
}
```

## 4. 自定义遍历顺序

[SDUT 3133: C要–二叉树中的秘密](http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/3133.html)

题意：在给定的二叉树中遍历查找某个点，给定遍历顺序，输出从根节点开始遍历到目标点需要多少步。

遍历顺序：
若根节点只有左子树或右子树，则遍历子树。
若左右子树均没有，则返回父节点
若左右子树均有
若左右节点高度不同，优先遍历高度小的
若左右节点高度相同，优先遍历储存的数据较小的

难点：
1.建树
2.按规则实现遍历

思路：
跟着题意走就行，注意细节。
每个节点储存数据外还维护一个值size，size为以该节点为根节点的子树的总结点数，用update函数递归地自顶向下求size

```
#include<iostream>
#include<cstdio>
#include<string.h>
#include<stdlib.h>
#include<cstring>
#include<ctime>
#include<cmath>
#include<math.h>
#include<algorithm>
using namespace std;
 
struct node
{
	int data;
	int size;
	node *pa,*lc,*rc;
	node(){ pa=lc=rc=NULL; size=1;}
	node(int d, node*ppa=NULL,node * llc = NULL, node * rrc=NULL):data(d),pa(ppa),lc(llc),rc(rrc){size=1;}
};
int x;
int llc[3010],rrc[3010];
int a[3010];
bool hasfound;
int ans;

int updateh(node *s)
{
	if(!s) return 0;
	return s->size=updateh(s->lc)+updateh(s->rc)+1;
}

node * build(int n)
{
	node *s=new node(a[n]);
	if(llc[a[n]])
	{	
		s->lc=build(llc[a[n]]);
		if(s->lc)
			s->lc->pa=s;
	}
	if(rrc[a[n]])
	{	
		s->rc=build(rrc[a[n]]);
		if(s->rc)
			s->rc->pa=s;
	}
	return s;
}

void found(node *s)
{
	if(hasfound==true) return;
	ans++;
	if(s->data==x)
	{
		hasfound=true;
		return;
	}
	if( !s->lc&&s->rc )
		found(s->rc);
	else if( !s->rc && s->lc )
		found(s->lc);
	else if( !s->rc && !s->lc)
		return;
	else
	{
		if(s->rc->size < s->lc->size || (s->rc->size==s->lc->size && s->rc->data < s->lc->data) )
		{
			found(s->rc);
			found(s->lc);
		}
		else
		{
			found(s->lc);
			found(s->rc);
		}
	}
}


int main()
{   
	int n;
	node *tree;
	int tem1;
while(~scanf("%d%d",&n,&x))
{
	hasfound=false;
	ans=0;
	memset(llc,0,sizeof(llc));
	memset(rrc,0,sizeof(rrc));
	for(int i=1; i<=n; i++)
	{
		a[i]=i;
		scanf("%d",&tem1);
		if(tem1==0) continue;
		else if(tem1==1) 
		{	
			scanf("%d",&llc[i]);
		}
		else
		{	
			scanf("%d%d",&llc[i],&rrc[i]);
		}
	}
	tree=build(1);
	updateh(tree);
	found(tree);
	printf("%d\n",ans);
}
    return 0;
}
```

## 5. 子树相同

[SDUT 3926: bLue的二叉树](http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/3926.html)

题意：给定两棵二叉树tree1和tree2，求tree1中有多少子树和tree2完全相同。

思路：
显然，直接暴力递归求等必定超时，我的简化方法是每个节点维护一个size值，如上题所示，size为以该节点为根节点的子树总节点数目。
在判断时，
若tree1中当前节点的size和tree2的根节点size不等或者data不等
若子树（左右均可）的size大于等于根节点的size，递归判断子树根节点
若子树的size小于tree2的size，就不判断了
若当前节点和tree2的根节点size相同并且数据相同，那就可以开始递归判断是否完全相等了，相等的话ans计数器自增，不等就返回。

```
#include<iostream>
#include<cstdio>
#include<string.h>
#include<stdlib.h>
#include<cstring>
#include<ctime>
#include<cmath>
#include<math.h>
#include<algorithm>
using namespace std;
 
struct node
{
	int data;
	int size;
	node *lc,*rc;
	node(){ lc=rc=NULL; size=1;}
	node(int d, node * llc = NULL, node * rrc=NULL):data(d),lc(llc),rc(rrc){size=1; }
};

int llc[100010],rrc[100010];
int a[100010];
bool isroot[100010];
int nroot;
int ans;
int pre1[100010],pre2[100010];
int npre1,npre2;

node *tree1,*tree2;

node * build(int n)
{
	node *s=new node(a[n]);
	if(llc[n])
		s->lc=build(llc[n]);
	if(rrc[n])
		s->rc=build(rrc[n]);
	return s;
}

void bbb(node *&tree, int n)
{
	memset(llc,0,sizeof(llc));
	memset(rrc,0,sizeof(rrc));
	memset(isroot,0,sizeof(isroot));
	for(int i=1; i<=n; i++)
	{
		scanf("%d",&a[i]);
		scanf("%d%d",&llc[i],&rrc[i]);
		isroot[llc[i]]=isroot[rrc[i]]=1;
	}
	for(nroot=1; isroot[nroot]==1; nroot++);
	tree=build(nroot);
}

int update(node *tree)
{
	if(!tree) return 0;
	return tree->size=1+update(tree->lc)+update(tree->rc);
}

int n,m;

bool equal(node *tree1, node *tree2)
{
	if( !tree1 && !tree2 ) return true;
	if( (!tree1 && tree2) || ( tree1&&!tree2 ) ) return false; 
	if(tree1->size!=tree2->size || tree1->data!=tree2->data)
		return false;
	return equal(tree1->lc,tree2->lc)&&equal(tree1->rc,tree2->rc);
}

void com(node *tree)
{
	if(tree->size!=tree2->size || tree->data!=tree2->data)
	{
		
		if( tree->lc && tree->lc->size>=tree2->size)
			com(tree->lc);
		if( tree->rc && tree->rc->size>=tree2->size)
			com(tree->rc);
	}
	else
		if(equal(tree,tree2)==true)
			ans++;
	return;
}

void release(node *tree)
{
	if(!tree) return;
	release(tree->lc);
	release(tree->rc);
	delete tree;
}

int main()
{   
while(~scanf("%d%d",&n,&m))
{
	ans=0;
	npre1=npre2=0;
	memset(pre1,0,sizeof(pre1));
	memset(pre2,0,sizeof(pre2));
	bbb(tree1,n);
	bbb(tree2,m);
	if(m>n)
	{
		printf("0\n");
		continue;
	}
	update(tree1);
	update(tree2);
	com(tree1);
	printf("%d\n",ans);
	release(tree1);
	release(tree2);
}
    return 0;
}
```


## 6. Tree Travels again

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170919/Ha2JdKll9I.JPG)

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170919/5e15450KD0.JPG)

题意：给定一棵二叉树迭代版中序遍历中栈的操作，求该二叉树的后序遍历序列

思路：经过观察可以发现，入栈的顺序其实就是**先序遍历的序列**，出栈顺序就是**中序遍历的序列**，那么就把题目化归成：已知二叉树先序和中序遍历序列求后序序列了

优化：
在得到序列后，可以不用建树，直接根据序列得到后序序列
思路：分治
先序序列首元素即是后序序列的尾元素，而后递归划分左右子树的序列即可

代码：

```
#include <iostream>
#include <string.h>
#include <string>
#include <cstring>
#include <cmath>
#include <algorithm>
#include <stdio.h>
#include <ctime>
#include <fstream>
using namespace std;


struct Node
{
	int data;
	Node * left,* right;
	Node() {left=right=NULL;}
	Node(int d , Node * l = NULL, Node * r = NULL ):data(d),left(l),right(r){}
};

int pre[100],in[100];
int stack[100];
int top=0;
int npre=0,nin=0;
int ans[100];

void solve(int prei , int ini , int posti, int n)
{
	if(n<=0) return ; 
	if(n==1)
	{
		ans[posti]=pre[prei];
		return;
	}
	ans[posti+n-1]=pre[prei];
	int i;
	for(i=0; in[i+ini]!=pre[prei] ; i++);
	solve( prei+1 , ini , posti , i );
	solve( prei+i+1 , ini+i+1 , posti+i , n-i-1 );
}

int main()
{
	int n;
	scanf("%d",&n);
	char c[10];
	int tem;
	// 根据出入栈构造序列
	while(cin>>c)
	{
		if(c[1]=='u')
		{
			cin>>tem;
			stack[top++]=tem;
			pre[npre++]=tem;
		}
		else
		{
			in[nin++]=stack[--top];
		}
	}
	solve(0,0,0,n);
	for(int i=0; i<n; i++)
	{
		if(i) printf(" ");
		printf("%d",ans[i]);
	}
	printf("\n");
	return 0;
}
```