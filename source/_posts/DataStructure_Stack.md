---
layout: post
title: "数据结构笔记（二）：栈和队列"
date: 2017-08-14
comments: true
tags: 
	- 学习笔记
	- c++
	- 数据结构
---


# **概述**

**栈**(stack)和**队列**(queue)均属于**线性存储**结构

栈特性：

> 1. LIFO ( last in first out ) 后进先出
> 2. 元素插入和删除仅在一端进行

队列特性：

> 1. FIFO ( first in first out ) 先进先出
> 2. 元素在队尾插入，在队头删除

![栈和队列](http://ot1c7ttzm.bkt.clouddn.com/stack0.jpg)

这两种结构均可以用数组或链表实现

本文主要介绍栈，以下实现均为c++.

<!-- more -->

# **栈**

## **一. 常用接口：**

```
template <typename T>
```

| 返回值  | 函数名     | 功能        |
| ---- | ------- | --------- |
| bool | empty() | 判断是否为空    |
| int  | size()  | 获取栈的大小    |
| void | push(n) | 将n压入栈     |
| T&   | top()   | 取栈顶元素而不弹出 |
| T    | pop()   | 弹出栈顶元素    |

## **二. 基本定义：**

本文用数组实现栈。
在开始给栈初始化一定空间，每当插入数据达到一定规模则进行扩容，删除数据达到一定规模则进行缩容。

（ 扩容和缩容均采用加倍或减半策略，分摊时间复杂度为O(1) ）

栈顶在数据数组的末尾，增加元素和删除元素只需要O(1)的时间。

```
template <typename T>
struct Stack
{
private:
	T *data;
	int _size,maxsize;   // 当前规模和最大规模

protected:
	void expand();  // 扩容
	void shrink();  // 缩容

public:
	Stack(){ _size =0 ; maxsize=4; data=new T[maxsize]; };
	Stack( T * origin , int n );   // 可由数据数组进行初始化，origin[0]对应栈顶
	~Stack() { delete []data; }
	void push( T const & e ) { expand(); data[_size++]=e;  }
	T pop( ) { shrink(); return data[--_size];   }
	T & top( ) { return data[_size-1];};
	int empty()const {return _size==0;}
	int size()const {return _size;}
	int find( T const & e )const;  // 查询栈内是否有该元素，有则返回秩，用于n皇后问题
};
```

## **三. 典型应用：**

**一. 逆序输出**

输出次序和处理过程颠倒

例如：进制转换

**二. 递归嵌套**

具有自相似性 ( 总体和局部相似 )

例如：括号匹配、栈混洗(stack permutation)

**三. 延迟缓冲**

线性扫描算法模式

例如：中缀表达式求值

**四. 栈式计算**

例如：逆波兰表达式转换（RPN Reverse Polish Notation）

**五. 试探回溯**

例如：n皇后问题、寻路

下面开始一一介绍这些应用：

### **3-1. 进制转换：**

思路：短除法。对于目前的数字n和数制base，每次记录n%base的值，而后将n除以base，最后将之前记录的值逆向输出即是n在base数制下的值。

借用栈的特性我们能够很容易的实现将之前记录的值逆向输出这一功能，只需要不断将n%base入栈即可，在短除完毕后不断出栈直至栈为空。

```
void convert(  int n , int base ) 
{
	Stack<char>s;
	static char digit[]=
	{'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'};
	// 进制限定在2-16，有需要可以继续扩展
	while(n)
	{
		s.push( digit[n%base]);
		n/=base;
	}
	while( !s.empty() )
		cout<<s.pop();
	cout<<endl;
}
```

### **3-2. 括号匹配：**

判断某一表达式内的括号是否完全匹配

对于只有一种括号 () 的情况，我们可以用一个变量n记录括号的数目，当碰到左括号n自增，碰到右括号n自减，扫描完表达式之后若n==0同时在此期间n均大于等于0，则说明该表达式内的括号完全匹配。

然而，这仅适用于只需要检测一种括号的情况，若是不仅有小括号(),还有中括号[],乃至html中自定义的括号类型 ，则计数法将失效。

而借助栈我们能够很轻松地完成多括号匹配的任务。

思路：每当遇到一个左括号（无论是小括号还是中括号还是其它自定义类型），就将其入栈，每当遇到一个右括号，则取栈顶元素进行鉴定，若是和右括号类型匹配的左括号，则将其出栈，若是和其类型不匹配，则可返回匹配失败。当进行到表达式结束后，若栈为空，则说明表达式匹配成功，若栈不为空，则匹配失败。

显示为流程图即是

![流程图](http://ot1c7ttzm.bkt.clouddn.com/stack8.JPG)

下面来看一下代码

```
bool judge ( char *s , int n ) 
{
	Stack<char>a;
	for(int i=0 ; i<n; i++)
	{
		if( s[i]!='(' && s[i]!=')' && s[i]!='[' && s[i]!=']' )
			continue;
		if( s[i]=='('|| s[i]=='[')
			a.push(s[i]);
		else if(a.empty())
			return false;
		else if( (s[i]==')'&&a.top()=='(') || (s[i]==']'&&a.top()=='[') )
			a.pop();
	}
	if(	a.empty() )
		return true;
	return false;
}
```

### **3-3. 栈混洗：**

#### **3-3-0. 概念：**

栈混洗 ( Stack permutation )指的是：将栈A的所有元素借助中转栈S转移到栈B中。

栈混洗的过程中只允许以下两个操作：

> 1. A栈顶弹出并压入栈S S.push(A.pop())
> 2. S栈顶弹出并压入栈B B.push(S.pop())

#### **3-3-1. 排列种数：**

先来计算一下栈通过栈混洗操作后有多少种排列方式：

设总数为n的栈有SP(n)种排列方式

推导：对于第一个元素通过S进入栈B后，此时S为空，设B中有k个元素，则第一个元素后面有k-1个元素，而A中剩余n-k个元素，第一个元素可能第一个进入B，也可能最后一个进入B，所以k取值为[1,n]
对于第一个元素而言，它的排列方式有SP(n)种。

$$SP(n)=\sum_{k=1}^n SP(k-1)*SP(n-k)$$

解得$SP(n) = catalan(n) = \frac{(2n)!}{(n+1)!n!}$

#### **3-3-2. 甄别栈混洗：**

如何甄别一个序列是否是栈混洗呢？

三个元素栈混洗有5种，全排列有6种, <1,2,3] 非栈混洗的排列为 <2,1,3] ( ‘<’为栈顶方向 )

而通过观察可得，对于任意三个元素，能否按某种相对次序出现在混洗中与其他元素无关

对于 1<=i < j < k<=n,若出现 k i j的排列，则非栈混洗 （称为**禁形**）

![禁形1](http://ot1c7ttzm.bkt.clouddn.com/stack2.jpg)

通过证明，禁形对于是否是栈混洗为 **充要条件**

![禁形2](http://ot1c7ttzm.bkt.clouddn.com/stack3.jpg)

那么我们可以得到如下的判别方法：

O(n3)判别方法：分别枚举i，j，k

O(n2)判别方法：枚举 i , j , j+1

O(n)判别方法：贪心模拟栈混洗过程，看是否能够形成输出序列，若需要pop时S为空或者需要的元素在S中但不是栈顶，则判定不是栈混洗

以下为O(n)的判别方法的实现：

```
template <typename T>
bool stackPermutation( T *origin, T * b , int n )
{
	Stack<T> s;
	Stack<T> ori( origin ,n );
	for( int j=0; j<n ; j++ )
	{	
		if( s.empty() )
		{	
			if( ori.empty() ) 
				return false;
			s.push( ori.pop() );
			cout<<"push\n";
		}
		while( !s.empty() )
		{
			if( s.top()==b[j] )
			{
				s.pop();
				cout<<"pop\n";
				break;
			}
			else
			{
				if( ori.empty() ) 
					return false;
				s.push( ori.pop() );
				cout<<"push\n";
			}
		}
	}
	if( !s.empty() || !ori.empty() )
		return false;
	return true;
}
```

#### **3-3-3. 栈混洗与括号匹配：**

值得注意的是，合法的栈混洗的过程中，同一元素的入栈和出栈操作和括号匹配相同，需要出栈的时候栈顶却不是对应的元素，则不是合法的栈混洗序列，在括号匹配中则是匹配失败。

![栈混洗与括号匹配](http://ot1c7ttzm.bkt.clouddn.com/stack4.jpg)

结论：合法的括号匹配相当于合法的栈混洗，若是输出不合法的混洗序列则说明表达式的括号不匹配

同时，n对括号所能构成的合法表达式同为$SP(n) = catalan(n) = \frac{(2n)!}{(n+1)!n!}$种

### **3-4. 中缀表达式求值：**

#### **3-4-0. 概述：**

中缀表达式即是我们最常在数学中使用的形式：运算符在运算数中间，使用约定俗成的运算符优先级和使用括号来强调优先级。
例如 1+1 即是一个中缀表达式

#### **3-4-1. 算法：**

主体思路：用两个栈分别保存**操作数**和**运算符**，每当有一个新的运算符，则判断栈顶运算符和当前运算符的优先级，若栈顶的优先级高，则进行栈顶的运算，若当前优先级高，则将该运算符压入栈。

此外，值得注意的是

一：为了方便判断优先级，在优先级表中**同一阶级的运算符根据出现次序不同优先级也不同**，如‘+’和‘-’，当‘+’在栈顶而’-‘作为新运算符时，’+’优先级大于’-‘，反之若’-‘在栈顶，则’-‘优先级大于’+’.

二：默认表达式为合法，因此不会出现栈顶为右括号的情况(因为右括号想要入栈时，因优先级小于除左括号和\0之外的运算符，所以它和左括号之间的运算符都会出栈)

三：同时，对于优先级为’=’的运算符不予入栈处理，反而是将栈顶元素出栈，此时是将一对对子(左右括号或\0)处理完毕。

四：表达式开始前先将’\0‘压入运算符栈，和表达式字符串末尾的\0形成对子，可以看作一对括号。

```
const char Prior[9][9] = 
{  
//        +   -   *   /   ^   !   (   )  \0
/* + */  '>','>','<','<','<','<','<','>','>',
/* - */  '>','>','<','<','<','<','<','>','>',
/* * */  '>','>','>','>','<','<','<','>','>',
/* / */  '>','>','>','>','<','<','<','>','>',
/* ^ */  '>','>','>','>','<','<','<','>','>',
/* ! */  '>','>','>','>','>','>',' ','>','>',
/* ( */  '<','<','<','<','<','<','<','=',' ',
/* ) */  ' ',' ',' ',' ',' ',' ',' ',' ',' ',
/* \0*/  '<','<','<','<','<','<','<',' ','='
};

void readNumber( char *&s , Stack<float>& opnd)
{
	float ans=0;
	opnd.push( *s-'0' );
	while( *(++s)<='9'&& *s>='0' )
		opnd.push( opnd.pop()*10 + *s-'0' );
	if(*s!='.')   // 可能是小数
		return;
	float f=1;
	while( *(++s)<='9'&& *s>='0' )
		opnd.push( opnd.pop() + (*s-'0')*(f/=10.0) );
	return;
}
void getnum( char a, int & num)
{
	switch(a)
	{
	case '+': num=0;return;
	case '-': num=1;return;
	case '*': num=2;return;
	case '/': num=3;return;
	case '^': num=4;return;
	case '!': num=5;return;
	case '(': num=6;return;
	case ')': num=7;return;
	case '\0': num=8;return;
	}
}
char orderBetween ( char a, char b)
{
	int num1,num2;
	getnum( a, num1 );
	getnum( b, num2 );
	return Prior[num1][num2];
}
float cal( float num1, char op , float num2)
{
	switch (op)
	{
	case '+': return num1+num2;
	case '-': return num1-num2;
	case '*': return num1*num2;
	case '/': return num1/num2;
	case '^': return pow(num1,num2);
	default : exit(1);
	}
}
float cal( char op, float num1)
{
	if( abs(num1-0)<1e-6 )
		return 1;
	for(int i=1; i<num1 ; i++)
		num1*=i;
	return num1;
}
float evaluate ( char * s )
{
	Stack<float> opnd;
	Stack<char> optr;
	float num1,num2;
	s[strlen(s)]='\0';
	optr.push('\0'); // 结尾操作符
	while( !optr.empty() )
	{
		if( (*s)<='9'&&(*s)>='0' ) //当前是操作数
		{	
			readNumber( s, opnd );  // 可能是多位数
		}
		else
			switch( orderBetween( optr.top(),*s ) )
			{
			case '<': optr.push(*s); s++; break;
			case '=': optr.pop(); s++; break; // () \0
			case '>': 
			{
				char op=optr.pop();
				if(op=='!')
					opnd.push( cal( op, opnd.pop() ) );
				else
				{
				// 逆向取数，可能是'-'号这些和次序相关的运算符
					num2=opnd.pop();
					num1=opnd.pop();
					opnd.push( cal(num1,op,num2));
				}
			}
			}
	}
	return opnd.pop();  // 表达式处理完毕后运算数栈最后剩下的就是表达式结果了
}
```

### **3-5. 逆波兰表达式：**

中缀表达式虽然符合人的使用和思维习惯，但是对于计算机处理而言过于繁琐了，于是就诞生了其它类型的表达式来简化计算机的运算，而逆波兰表达式就是其中一种。

#### **3-5-0. 概念：**

逆波兰表达式 ( RPN : Reverse Polish Notation )

没有括号和约定俗成的优先级，从左至右扫描表达式，运算符**谁先出现算谁**。

运算符位于参与运算的运算数之后。

例如：

0！1+2 3！*4+5-^

值得注意的是：不同运算数之间可能仅是以空格分格，同一运算数数字之间没有空格。

以上表达式翻译成中缀表达式即是：

( 0! + 1 )^( 2 * 3! +4 - 5 )

和中缀表达式相比，逆波兰表达式的优点在于**计算速度更快**。

在得到一个函数表达式后，若是之后的调用仅是修改数字而不修改运算符，那么对于中缀表达式则需要算n次，对于逆波兰表达式需要1次转换+算n次，效率将高很多，当然代价就是转换的速度很慢。

#### **3-5-2. 计算：**

计算逆波兰表达式相比中缀表达式要简单的多，只需要一个运算数栈，遇到数字则入栈，遇到运算符就取数字计算。

```
float rpnEvaluate( char * RPN )
{
	Stack<float> opnd;
	float num1,num2;
	int len=strlen(RPN);
	RPN[len++]='\0';
	while(len--)
	{
		if( (*RPN)<='9'&&(*RPN)>='0' ) //当前是操作数
		{	
			readNumber( RPN, opnd );  // 可能是多位数
		}
		else
		{
			char op=*RPN;
			if(op=='!')
				opnd.push( cal( op, opnd.pop() ) );
			else
			{
				num2=opnd.pop();
				num1=opnd.pop();
				opnd.push( cal(num1,op,num2));
			}
		}
	}
	return opnd.pop();
}
```

#### **3-5-2. 手工转换：**

手工将中缀表达式转换成逆波兰表达式的方法如下：

1.用括号显式界定所有运算符

2.将所有运算符移到对应的右括号后面

3.去掉所有括号

4.稍加整理

![手工转换](http://ot1c7ttzm.bkt.clouddn.com/stack6.jpg)

#### **3-5-3. 转换算法：**

非常有意思的是，计算中缀表达式的算法流程和转换成RPN的算法流程大致相同，只需要在将数字入栈的同时接入RPN的末尾，弹出运算符时接入RPN末尾即可。

```
// evaluate函数中调用的其它函数和中缀表达式中的一样，在此不重复附上
void append( char *&RPN, float num )
{
	int len=strlen(RPN);
	char buf[64];
	if( num!= (float)(int)num )   // 判断是否为整数
		sprintf( buf, "%.2f \0",num); // 小数暂取两位
	else
		sprintf( buf, "%d \0",(int)num);
	RPN=(char *)realloc( RPN, sizeof( char )*( len+strlen( buf )+1 ) );   // 扩容
	strcat( RPN,buf );
}
void append( char *&RPN, char op )
{
	int len=strlen( RPN );
	RPN=(char *)realloc( RPN, sizeof( char )*( len + 3 ) );
	sprintf( RPN+len, "%c \0",op );
}
float evaluate ( char * s , char *&RPN )
{ // s：中缀表达式， RPN：需要存放转换后表达式的位置
	Stack<float> opnd;
	Stack<char> optr;
	float num1,num2;
	s[strlen(s)]='\0';
	RPN[0]='\0';
	optr.push('\0'); // 结尾操作符
	while( !optr.empty() )
	{
		if( (*s)<='9'&&(*s)>='0' ) //当前是操作数
		{	
			readNumber( s, opnd );  // 可能是多位数
//////// RPN转换 ////////
			append( RPN, opnd.top());
//////// RPN转换 ////////
		}
		else
			switch( orderBetween( optr.top(),*s ) )
			{
			case '<': optr.push(*s); s++; break;
			case '=': optr.pop(); s++; break; // () \0
			case '>': 
			{
				char op=optr.pop();
//////// RPN转换 ////////
				append( RPN, op );
//////// RPN转换 ////////
				if(op=='!')
					opnd.push( cal( op, opnd.pop() ) );
				else
				{
					num2=opnd.pop();
					num1=opnd.pop();
					opnd.push( cal(num1,op,num2));
				}
			}
			}
	}
	return opnd.pop();
}
```

### **3-6. n皇后问题：**

#### **3-6-0. 概述：**

试探：逐步增加候选解的长度 ，逐步向目标解靠近

回溯：一旦发现和目标解不同，则回溯到上一步继续试探

一个形象的比方：拿着绳子探索迷宫，绳子一段绑在入口，人随意选择一方向前进，一旦遇到死路，做好标记，沿着绳子后退到上一个分叉口，选择另一个方向，若是该分叉口所有前进方向均是死路，则沿着绳子返回再上一个分支，以此直到找到出口或是试完所有可能。

n皇后问题：在n*n的棋盘中放置n个皇后，n个皇后间两两不冲突。冲突的条件是一个皇后在另一个皇后同一行或同一列或正反对角线上。

![n皇后问题](http://ot1c7ttzm.bkt.clouddn.com/stack7.jpg)

#### **3-6-1. 算法：**

用栈保存当前探索进度，逐行放置皇后，每当确定皇后可以在当前列放置时，将其入栈，若所有列均不可放置，则取出栈顶元素，继续向后面的列尝试放置。 若是栈的大小等于n，说明已经构成一个解，此时可以输出栈内元素也可将计数器自增。

```
template <typename T>
int Stack<T>::find( T const & e ) const
{
	for( int i=0 ; i<_size ; i++)
		if( data[i] == e )
			return i;
	return -1;
}
struct Queen
{
	int x,y;
	Queen( int xx=0, int yy=0 ):x(xx),y(yy){}
	bool operator == ( Queen const & a ) const
	{
		if( x==a.x || y==a.y || x+y==a.x+a.y || x-y==a.x-a.y )
			return true;
		return false;
	}
	bool operator != ( Queen const & a ) const { return !( *this == a );}
};
int placeQueens( int n ) // n皇后
{
	int num_solution=0;
	Stack< Queen >a; 
	Queen q(0,0);  //当前寻求位置放的皇后
	do
	{
		if( a.size()>=n || q.y>=n ) // 若是列数超出边界或已经构成一个解
		{	
			q=a.pop();   // 取出栈顶元素，列数自增继续尝试
			q.y++;
		}
		while( q.y<n && a.find(q)>=0 ) // 若和其它皇后发生冲突，则find函数会返回冲突的皇后的秩 [0,n) 若是-1则无冲突
			q.y++;
		if( q.y<n )  // 找到位置放置皇后
		{
			a.push( q );
			if( a.size()>=n )
				num_solution++;
			q.x++;  // 前往下一行找解，列数重置
			q.y=0;
		}
	}
	while( !( q.x==0 && q.y>=n ) );
	return num_solution;
}
```

# **队列**

## **一. 常用接口：**

```
template <typename T>
```

| 返回值  | 函数名        | 功能      |
| ---- | ---------- | ------- |
| bool | empty()    | 判断是否为空  |
| int  | size()     | 获取队列的大小 |
| void | enqueue(n) | 将n入队    |
| T&   | front()    | 查看队头元素  |
| T    | dequeue()  | 删除队头元素  |
| T&   | rear()     | 查看队尾元素  |

## **二. 基本定义：**

队列同样可以用数组或链表实现

### **2-1. 版本一：继承List类**

在这里选择继承[笔记一](https://zedom1.github.io/2017/08/12/list/)中实现的双向链表List类

```
template <typename T>
struct queue:public List<T> //利用双向链表版
{ // size和empty直接沿用
	void enqueue ( T const& e ) {insertAsLast(e);}
	T dequeue() { return remove(first()); }
	T& front() { return first()->data; }
	T& rear() { return last()->data; }
};
```

### **2-2. 版本二：从零开始的数组版**

用数组模拟队列时，为方便起见，用first和last作为队头和队尾的秩，而不是在删除元素时将后面所有的元素迁移一位。只是需要注意在last逼近数组边界时需要扩容或者重新安排位置。

```
template <typename T>
struct queue  // 数组模拟版
{
private:
	T *data;
	int first,last,maxsize;

protected:
	void expand();
// expand()判断 last 或 last-first 是否接近maxsize
public:
	queue();
	~queue() {delete []data;}
	bool empty()const {return first==last;}
	int size()const {return last-first;}
	T& front()const { if(!empty())return data[first]; else exit(1);}
	T& rear()const { if(!empty())return data[last-1]; else exit(1);}
	void enqueue( T const & e) { expand(); data[last++]=e; }
	T dequeue() { return data[(first++)-1]; }
};
```

### **2-3. 版本三：环状数组版**

与版本二中用first和last作为队头和队尾类似，但不同点在于，数组在逻辑上是首尾相接的环状，这样在足够大的数组下就可以不考虑队列移动到数组末尾的情况了

适用于已知数据的规模的情况

数组在逻辑上相连而在物理上不相连，用取余size实现

```
template <typename T>
struct Queue
{
private:
	T *data;
	int maxsize;
	int first;
	int last;
	int size;

public:
	Queue(int size=0)
	{
		maxsize=size+1;
		last=0;
		first=1;
		data= new T[maxsize];
	}
	~Queue()
	{
		delete []data;
	}
	void clear()
	{
		last=first=0;
	}
	void enqueue(const T & it)
	{
		last=(last+1)%maxsize;
		data[last]=it;
	}
	T dequeue()
	{
		T it = data[first];
		first = (first+1)%maxsize;
		return it;
	}
	const T firstValue()
	{
		return data[first];
	}
	int length()
	{
		return ((last+maxsize)-first+1)%maxsize;
	}

};
```

# **完整实现代码**

[栈的完整实现 ( 附带应用的实现 )](https://github.com/zedom1/DSA/blob/master/stack%20and%20queue/stack.cpp)

[队列的完整实现](https://github.com/zedom1/DSA/blob/master/stack%20and%20queue/queue.cpp)