---
layout: post
title: "刷题选集（一）"
date: 2017-08-18
comments: true
tags: 
	- c++
	- 刷题
---

## 目录

第一个选集，选的比较基础比较水，主要是思维+细节


### 贪心

SDUT 2072 删数问题
SDUT 2072 活动选择

### 栈

SDUT 3334 出栈序列判定
SDUT 2132 后缀表达式转换

### 动态规划

最大子列和
最大子列和及区间

<!-- more -->

## 贪心

### 删数问题

[SDUT 2072 删数问题](http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Contest/contestproblem/cid/2187/pid/2072)

题意：给一串100位以内的数字s和一个数字n，在s中删除n个数使得剩下的数字按原来次序自左向右排最小

思维+细节题

思路：自左向右扫n轮，每轮灭掉一个比它直接后继大的字符

细节：
1.可能有前导0， 比如 9023 删掉1个数字后 -> 023 -> 23
解决：用一个bool变量firstZero判断是否有前导0,遇到非0则归true
2.可能剩下全是0，比如 9002 删掉2个数字后 -> 00 -> 0
解决：用细节1解决中的那个变量，若输出完整个字符串后firstZero还是false，则额外输出0加换行
3.如何灭掉，可以每次都让后面的字符都向前移一个，也可以直接让当前字符归’0’-‘9’之外的字符，但是每次判断时都需要找在0-9之内的字符

```
#include <iostream>
#include <cmath>
#include <stdio.h>
#include <cstdio>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <string>
#include <cstring>
#include <algorithm>
using namespace std;
// 解题代码丑了些，见谅
int main()
{
	char s[110];
	int n;
	while(cin>>s>>n)
	{
		int l=strlen(s);
		s[l]=0;
		if(n>=l)
		{	
			printf("0\n");
			continue;
		}
		for(int i=0; i<n; i++)
		{
			int c;
			for(int j=0; j<l; j++)
			{	
				for(int k=j+1; (c=s[k])==0 ; k++);
				if(s[j]>c)
				{	
					s[j]=0;
					break;
				}
			}
		}
		int firstzero=0;
		for(int i=0; i<l; i++)
		{	
			if(s[i]=='0'&&firstzero==0)
				s[i]=0;
			if(s[i])
			{	
				firstzero=1;
				printf("%c",s[i]);
			}
		}
		if(firstzero)
			printf("\n");
		else
			printf("0\n");
	}
}
```

### 活动选择

[SDUT 2072 活动选择](http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/2073.html)

题意：每天都有n个活动申请举办，已知每个活动的开始时间b，结束时间e，求出每天最多能举办多少活动。

贪心题

思路：贪心，所有活动按结束时间e自小到大排序，结束时间相同的开始时间越大越好，然后从第一个活动开始扫到最后一个活动
若后一个活动的开始时间晚于现在活动的结束时间，则计数器自增，允许后一个活动举办。
若后一个活动的开始时间早于当前活动，则无视，扫下一个。

```
#include <iostream>
#include <cmath>
#include <stdio.h>
#include <cstdio>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <string>
#include <cstring>
#include <algorithm>
using namespace std;

struct aaa
{
	int l,r;
}a[110];

bool com(aaa a1, aaa a2)
{
	if(a1.r==a2.r)
		return a1.l>a2.l;
	return a1.r<a2.r;
}

int main()
{
	int n;
while(~scanf("%d",&n))
{
	memset(a,0,sizeof(a));
	for(int i=0; i<n; i++)
		scanf("%d%d",&a[i].l,&a[i].r);
	sort(a,a+n,com);
	int nowend=a[0].r,ans=1;
	for(int i=1; i<n; i++)
	{
		if(a[i].l<nowend)
			continue;
		nowend=a[i].r;
		ans++;
	}
	printf("%d\n",ans);
}
	return 0;
}
```

## 栈

### 栈混洗

[SDUT 3334 出栈序列判定](http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/3334.html)

题意：给定一个初始序列，序列依次入栈，栈顶在此期间可以任意出栈，问能否构成下列给出序列。

栈混洗 ( 贪心 )

思路：用一个中转栈S，将初始序列依次入栈，每次比较栈顶和目标序列头，若相同，则出栈，若不同则继续入栈，最后若栈为空则匹配

```
#include <iostream>
#include <cmath>
#include <stdio.h>
#include <cstdio>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <string>
#include <cstring>
#include <algorithm>
using namespace std;

int a[10010];
int stack[10010];
int tem[10010];
int m=0;
int n;
int t;

bool check ()
{
	int j=0,i=0;
	while(i<n&&m<n)
	{
		stack[m++]=a[j++];
		while( m-1>=0 && stack[m-1]==tem[i])
		{
			m--;
			i++;
		}
	}
	if(m>0)
		return false;
	return true;
}

int main()
{
	
	scanf("%d",&n);
	for(int i=0; i<n; i++)
		scanf("%d",&a[i]);
	scanf("%d",&t);
	while(t--)
	{
		m=0;
		for(int i=0; i<n; i++)
			scanf("%d",&tem[i]);
		if(check()==true)
			printf("yes\n");
		else
			printf("no\n");
	}
		return 0;
	}
```

### 后缀表达式转换

[SDUT 2132 一般算术表达式转换成后缀式](http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/2132.html)

题意：给出一个基于二元运算符的算术表达式，转换为对应的后缀式，并输出。

和[数据结构笔记（二）：栈和队列](https://zedom1.github.io/2017/08/14/stack/) 中的逆波兰表达式转换方法类似但要简单的多。

1.运算符基于二元 ( “ +-*/ ()# “ )
2.不涉及数字，只需要记录变量名a、b、c，默认为一个字符，无需做转换
3.表达式以#号结尾

鉴于是在题目中碰到，我们也不好花许多时间来依次实现栈、列出优先级数组，运算符也只有加减乘除、小括号和井号，因此就直接在程序里判断好了。

思想是一样的，用两个栈分别保存答案和运算符。
遇到变量就直接入答案栈，遇到运算符先和运算符栈的栈顶进行优先级比较，若栈顶的大，则栈顶的运算符进入答案栈，直到栈顶优先级小，则将该运算符入栈。

优先级设置：
左括号最大，但不碰到右括号不出栈
右括号和#最小并且不入栈
同级运算符如+和-比较时栈顶的大

细节：
左右括号和#出运算符栈后不进入答案栈
两个#号看作一对括号，在开始人为加入一个#号和最后的配对

最后答案栈保存的是要输出答案的**逆序**，但若用数组模拟的话就可以直接从头输出到尾

```
#include <iostream>
#include <cmath>
#include <stdio.h>
#include <cstdio>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <string>
#include <cstring>
#include <algorithm>
using namespace std;

int main()
{
	char s[1000];  // 输入表达式
	char op[1000]; int m=0;   // 运算符栈，用数组模拟，m为栈顶
	char ans[1000]; int ansm=0;  // 答案栈，用数组模拟，ansm为栈顶
	int l=0;   // 记录输入表达式的长度
	op[m++]='#';   // 栈底加入一个#用来和最后的#配对
	while(scanf("%c",&s[l])&&s[l++]!='#');
	for(int i=0; i<l; i++)
	{
		if(s[i]!='+'&&s[i]!='-'&&s[i]!='*'&&s[i]!='/'&&s[i]!='('&&s[i]!=')'&&s[i]!='#')    // 该字符为变量
		{	
			ans[ansm++]=s[i];
			continue;
		}
		switch(s[i])
		{
		case '+': 
		case '-':
			{
				if(op[m-1]=='('||op[m-1]=='#')
				{	
					op[m++]=s[i];
					break;
				}
				else  // 除非为左括号或井号，不然在这道题中其它栈内的运算符优先级都要更大
				{
					ans[ansm++]=op[--m];
					op[m++]=s[i];
					break;
				}
			}
		case '*': case '/': case '(':
			{
				op[m++]=s[i];
				break;
			}
		case ')':
			{
				while(op[m-1]!='(')  // 把左右括号之间的都依次入答案栈
				{
					ans[ansm++]=op[--m];
				}
				--m;  // 左括号出栈
				break;
			}
		case '#':
			{
				while(op[m-1]!='#')
				{
					ans[ansm++]=op[--m];
				}
				--m;
				break;
			}
		}
	}
	for(int i=0; i<ansm; i++)
		printf("%c",ans[i]);
	printf("\n");
	return 0;
}
```

## 动态规划

### 最大子列和

题意：给定n个数的序列，求其中和最大的连续子列

网上从O(n2)的暴力到O(nlogn)的分治再到O(n)的动态规划的优化都有，因此不再赘述，直接来O(n)的在线的动态规划方法

思路：若某个元素或某段元素和为负数，那么它就不可能是最终答案的开头部分，因为它对那个最大的子序列的贡献是负的。（也可能所有数字都是负数，那么某个元素就是最终答案本身） 那么我们从最开头开始扫到最尾，用一个累加器累加沿途的值，一旦累加器超过目前的最大值，那么更新最大值，一旦累加器小于0，那么继续加下去的话，贡献是负数，还不如清空累加器从0开始，直到处理到数组末尾

```
#include <iostream>
#include <cmath>
#include <stdio.h>
#include <cstdio>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <string>
#include <cstring>
#include <algorithm>
using namespace std;

int main()
{
	int n;
	scanf("%d",&n);
	int now,maxn;
	int tem;
	scanf("%d",&tem);  // 先读入第一个数字，并把now和maxn置位tem而非0，防止出现所有数字都是负数的情况
	now=maxn=tem;
	for(int i=1; i<n; i++)
	{
		now=max(0,now);
		scanf("%d",&tem);
		now+=tem;
		maxn=max(maxn,now);
	}
	printf("%d\n",maxn);
	return 0;
}
```

### 最大子列和及区间

我们在最大子列和的基础上做一点修改，现在我们不仅要得到最大的和，还需要那个最大的区间，要怎么改呢？

在理解了上一题的动态规划解法之后，要求得最大区间，我们只需要增加三个变量，分别记录当前这段可能是最大子列的开头first、和最大的子列的开头ansfirst、和最大的子列的结尾last

思路：

```
每当now累加器小于0时，说明前面一段将对后面没有贡献，要舍弃，那要以下一个读入的数字作为开头，以此更新first。

一旦now累加器大于最大和maxn，说明我们找到了一个可能是答案的序列，而first此时正指着这个序列的头，因此需要把它正式保存下来给ansfirst，此时的数字tem也是当前最大序列的结尾，也把它保存下来给last

#include <iostream>
#include <cmath>
#include <stdio.h>
#include <cstdio>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <string>
#include <cstring>
#include <algorithm>
using namespace std;

int main()
{
	int n;
	scanf("%d",&n);
	int now=0,maxn=0;
	int tem;
	int first,last,ansfirst;
	scanf("%d",&tem);
	first=last=ansfirst=now=maxn=tem;
	for(int i=1; i<n; i++)
	{
		scanf("%d",&tem);
		if(now<0)
		{ 
			now=0;
			first=tem;
		}
		now+=tem;
		if(now>maxn)
		{
			last=tem;
			maxn=now;
			ansfirst=first;
		}
	}
	printf("%d %d %d\n",maxn,ansfirst,last);
	return 0;
}
```