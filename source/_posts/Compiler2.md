---
layout: post
title: "编译原理笔记（二）：词法分析"
date: 2017-08-24
comments: true
tags: 
	- 编译原理
	- 学习笔记
---


## 03-01. Lexical Analysis

**Goal **:
1 - Recognize substrings corresponding to tokens. ( **lexemes**: substrings )
2 - Identify the token class of each lexeme.
3 - Communicate tokens to the parser.

<!-- more -->

What the likes of human being will see:
![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170824/c2DJbIjL0F.JPG)
What the likes of analyzer will see:
![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170824/0bJg7Gfidl.JPG)

**Token classes**: the role of different elements
In a programming language: Identifiers、keywords、’(‘、’)’、numbers….
**Token classes correspond to sets of strings.**

- Identifier: 标识符

  > strings of letters or digits, starting with a letter
  > like “a1”,”A0o”

- Number:

  > a non-empty string of digits
  > like “9”,”001”

- Keyword: 关键字

  > “else” or “if” or “begin “…

- Whitespace:

  > a non-empty swquence of blanks, newlines, and tabs

- Operator:

  > “==”…

The output of the lexical analyzer: **a series of pairs **which are the token class.
![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170824/ci2A7H3E5i.JPG)

## 03-02. Lexical Analysis example

1 - The goal is to **partition the string** . This is implemented by reading left-to-right, recognizing **one token at a time**.

2 - **look ahead** may be required to decide where one token ends and the next token begins.

> but we would like to **minimize** the amount of look ahead.

## (extra) 03-02x 词法分析器的实现

1 - 手工编码实现
相对复杂，且容易出错
但是目前主流的实现方式
对词法分析器各部分掌握精确，效率高
如 GCC、LLVM

2 - 词法分析器的生成器
可快速生成原型、代码量小
难以控制实现细节
如 lex、flex、jlex

### 03-02x-01 手工编码实现

#### 方法一：转移图算法

根据需求画出状态转移图：
![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170830/c3GCd3ElG2.JPG)
而后实现该算法：

```
enum kind{IF , LPAREN , ID, INTLIT, ...};
struct token
{
	enum kind k;
	char *lexeme;
};
token nextToken()
{
	c=getchar();
	switch(c)
	{
		case '<': c=getchar();
			switch(c)
			{
				case '=': return LE;
				case '>': return NE;
				default: rollback(); return LT;
			}
		case '=': return EQ;
		case '>': c=getchar();
			switch(c)
			{
				case '=' return GE;
				default: rollback(); return GT;
			}
	}
}
```

#### 方法二：关键字表算法

对给定语言中所有关键字，构造关键字构成的哈希表H
对所有标识符和关键字，先统一按转移图识别
识别后，查看表H是否为关键字
通过合理构造哈希表H(完美哈希)，可以以O(1)时间完成

## 03-03. Regular Languages (正则语言)

We must say what set of strings is in a token class.

- The usual tool is to use regular languages( 正则语言 ).

  **Regular expression** (正则表达式) : define the regular language
  each regular expression is a set.

**1.Basic regular expression:**

- Single character

  > ‘c’= {“c”}

- Epsilon

  > $\epsilon$ = {“”}
  > It is a language that has a single string namely the empty string
  > $\epsilon$ != $\emptyset$

**2.Compound regular expression:**

- Union
  ![union](http://ot1c7ttzm.bkt.clouddn.com/image/170824/hJfgaJGKEm.JPG)

- Concatenation
  choose a string from A and a string from B and then combine, put them together with the string from a first and choosing strings at all possible ways from all possible combined strings.
  ![Concatenation](http://ot1c7ttzm.bkt.clouddn.com/image/170824/LDGkL285mA.JPG)

- Iteration
  It means one concatenated with itself i times.
  ![Iteration](http://ot1c7ttzm.bkt.clouddn.com/image/170824/BfBClc54ha.JPG)

  Besides:
  ![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170824/6ffL3l8eII.JPG)
  A+：there has to be at least one A

  There is **more than one way** to write down **the same set**.

  ![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170824/225dBIdEgb.JPG)

  Quiz :
  ![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170824/c29161Cfd1.JPG)

Explanation:

> 1. The given regular expression requires every string have at least one 1.
> 2. (10 + 11 + 1) can be replaced by 1
> 3. (0+1)* equals to (1+0)*

## 03-04. Formal Languages (形式语言)

Let $\Sigma$ be a set of characters
A language over $\Sigma$ is a set of string of characters drawn from $\Sigma$

### Meaning function: L

L($\epsilon$)= {“”}
L: Expression -> Sets of Strings
We apply L to deconpose the compund expressions into several expressions that we compute the meaning of and then computed the sets from those separate smaller sets.

Why use a meaning function?

- Makes clear what is syntax, what is semantics.
- Allows us to consider notation as a seperate issue
- Because expressions and meanings are not 1-1

## 03-05. Lexical Specifications

### write regular expressions

1 . Keyword: “if” or “else” or “then”…
![Keyword](http://ot1c7ttzm.bkt.clouddn.com/image/170824/mH7DDJeB3l.JPG)

2 . Integer: a non-empty string of digits
![Integer](http://ot1c7ttzm.bkt.clouddn.com/image/170824/a1i4ebijJe.JPG)

3 . Identifier: strings of letters or digits, starting with a letter
use square brackets to write a range of characters.
have the starting character and an ending character and then separate them by a hyphen.

> [ a - z ] = ‘a’+’b’+’c’…..+’z’
> ![Identifier](http://ot1c7ttzm.bkt.clouddn.com/image/170824/a044F7C3f1.JPG)

4 . Whitespace: a non-empty sequence of blanks, newlines, and tabs.
![Whitespace](http://ot1c7ttzm.bkt.clouddn.com/image/170824/Ehe5KDm9DK.JPG)

![digits](http://ot1c7ttzm.bkt.clouddn.com/image/170824/lGg51K9GmI.JPG)

- Regular expressions describe many useful languages
- Regular languages are a language specification

## 04-01. Lexical Specification

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170824/60A270jmd5.JPG)

1 . Write a regular expression for the lexemes of each token class.

> Number = digit+
> Keyword = ‘if’ + ‘else’…
> Identifier = letter ( letter + digit )*
> OpenPar=’(‘….
> 2 . Construct R, matching all lexemes for all tokens.
> R= Keyword + Identifier + Number +…
> = R1 + R2 …
> 3 . Let input be x1..xn
> For 1<=i<=n check x1..xi$\in$L(R)?
> 4 . If success , then we know that
> x1..xi $\in$ L(Rj) for some j
> 5 . Remove x1..xn from input and go to (3)

### Questions:

1 . How much input is used?
if x1..xi $\in$ L(R)
x1..xj $\in$ L(R) (i!=j)

> like “=” and “==”
> **Maximal Munch**: choose the longer one

2 . Whick token is used?

> like “if” is both a keyword and an identifier

Use a priority ordering (优先排序)
the rule is to **choose the one listed first**

3 . What if no rule matches?

**do good error handling.**
Compilers can’t simply crash, it should be able to give the user **a feedback** about where the error is and what kind of error it is.

Solution: **write a category of error strings**and put it in the last of the priority list.

### Summary:

- Use in lexical analysis requires small extensions

  > 1. To resolve ambiguities
  >
  >    > 1. matches as long as possible
  >    > 2. highest priority match
  >
  > 2. To handle errors
  >
  >    > 1. write a category of error strings and put it in the last of the priority list.

- Good algorithms known

  > 1. Require only single pass over the input
  > 2. Few operations per character (table lookup)

- Regular expression

  > $\epsilon$
  > a
  > a|b
  > ab
  > a*[a-z] == a|b|c…|za+ == a(a)*
  > a? == $\epsilon$|a
  > “a*“ != a*
  > e{1,3} == (e)|(ee)|(eee)
  > . == [^\n]

## 04-02. Finite Automata (有限状态自动机)

**Finite Automata**: are a very convenience as an implementation mechanism for regular expressions.

### A finite automaton consists of :

- An input alphabet **$\Sigma$**
- A finite set of states **S**
- A start state **n**
- A set of accepting states **F \subseteq S**
- A set of transitions state -> (input) state

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170830/Dm5hjkKeGC.JPG)

Transition s1 ->(a) -> s2
If end of input and in accepting state => accept state
Otherwise => reject state

> terminates in state S \not\in $ Final state
> gets stuck. (no transition)

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170824/FKD6FFmgJ2.JPG)

Another kind of transition: **$\epsilon$-moves**

The automaton can move to a different state without consuming any input.

### Deterministic Finite Automata (DFA) (确定的有穷状态自动机)

- One transition per input per state
- No $\epsilon$-moves
  A DFA takes only** one path **through the state graph
  **DFAs are faster to execute**
  **Every DFA is also an NFA**

### Nondeterministic Finite Automata (NFA) (非确定的有穷状态自动机)

- Can have multiple transitions for one input int a given state

- Can have $\epsilon$-moves
  An NFA can choose
  It accepts if some choices lead to an accepting state

  ![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170824/gdLcfhGJKB.JPG)
  In every step an NFA is in a set of states of the machine.
  We can consider all the possible moves.
  We look at the last set of states after the last input character is read and if there’s any final state in that set, then the machine accepts.
  **NFAs are much more smaller than DFAs.**

## 04-03. Regular expressions into NFAs

![steps](http://ot1c7ttzm.bkt.clouddn.com/image/170825/bKF0cmjGKd.JPG)

For each kind of regular expression, define an equivalent NFA.

**Thompson Algorithm:**

![Transition](http://ot1c7ttzm.bkt.clouddn.com/image/170825/i3FI550glK.JPG)
![Transition](http://ot1c7ttzm.bkt.clouddn.com/image/170825/EgEe117kFh.JPG)
![Transition](http://ot1c7ttzm.bkt.clouddn.com/image/170825/3E2G1150hL.JPG)

![quiz](http://ot1c7ttzm.bkt.clouddn.com/image/170825/9AlcACHLHD.JPG)

## 04-04. NFA to DFA

**$\epsilon$-closure **
choose a state and look at the states that can be reached by following **only epsilon moves**.

![closure](http://ot1c7ttzm.bkt.clouddn.com/image/170825/4H05J22cg1.JPG)

How many different states can an NFA get into ?

> if there are N states
> The states NFA can get into at most n different states.
> |S|<=N

How to convert an NFA to a DFA?

> There are $2^n -1 $ possible subsets of n states.
> **This is a finite set of possible configurations.**
> Use DFA to **simulate the behaviour** of the NFA.
> O($2^n$)

Define an NFA:
![NFA](http://ot1c7ttzm.bkt.clouddn.com/image/170825/744hcDeEhc.JPG)

Define a DFA:
![DFA](http://ot1c7ttzm.bkt.clouddn.com/image/170825/050HKEm80c.JPG)

Transition
![Transition](http://ot1c7ttzm.bkt.clouddn.com/image/170825/JA4Eaa9917.JPG)

**子集构造算法：**
工作表算法伪代码：

```
q0 <- eps_closure(s0)   // s0为NFA起始状态，q0为新构造的DFA的状态
// eps_closure 为找到s0的所有$\epsilon$可到达的状态
Q <- {q0}    // Q 为DFA的所有状态集合
workList <- q0   // 队列
while( workList != [] )    
{
	remove q from workList // 队列不为空则取队首元素
	foreach (character c)   // 循环判断每个字符
	{
		t <- e-closure( delta(q,c) ) 
		  // delta函数 计算q中每个状态通过c所能到的状态
		  // t为q通过c所能到的下一个状态集合
		D[q ,c ] <- t   // 在DFA中记录：q通过c可以转换到t
		if( t \not \in Q)  // t在队列中未出现的话，还需要继续扩展
			add t to Q and workList
	}
}
```

## 04-05. Implementing Finite Automata

### DFA -> Table-driven Implemetation

NFA -> DFA -> list a 2-dimensional state table -> translate into code
![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/j94m246b37.JPG)

We can also use 1-dimensional tables save space.
The table contains a pointer to a vector of moves for that particular state.
**share the rows that are duplicated in the automaton**
![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/0bCIjdDL14.JPG)

#### DFA 最小化

**Hopcroft 最小化算法**
伪代码

```
split(S)  // 集合切分
{	
	foreach (character c )
		if(c can split S) // S中某些状态通过c转移到同样的状态
			split S into T1 , ... Tk
}
hopcroft()
{
	split all nodes into N,A  // 先把大状态集合根据是否为接受状态分成两份
	while( set is still changes ) // S是否刚刚被切分
		split(S)
}
```

#### DFA 代码实现

DFA是一个有向图
可以根据转移表（类似邻接矩阵），哈希表，跳转表

##### 转移表

将字符和状态先填入一个二维数组table
例如：

```
table [0]['a']=1;
table [1]['b']=1;
....
```

伪代码

```
nextToken()
{
	state=0;  // 当前到了哪个状态，如q0
	stack=[];  // 为了实现最长匹配机制
	while( state!=ERROR )
	{
		c=getchar();
		if(state is ACCEPT)
			clear(stack) 
		push(state)
		// 保留最接近的接受状态，然后尽可能向前看进行尝试，若向前看失败，则可通过下面的循环不断弹出直到最接近的接受状态
		state = table [state][c]  // 查表知当前状态通过c能到哪个状态
		// 若当前状态无法通过c到达别的状态，则赋值为ERROR，退出循环
	}
	while( state is not ACCEPT )
	{
		state=pop();
		rollback();   // 把刚刚读入的那个字符扔回流中(多读了一个字符)
	}
}
```

##### 跳转表 jump table

好处：不需要维护一个状态和字符的二维数组并且效率高
伪代码

```
nextToken()
{
	state =0
	stack = []
	goto q0

q0:
	c=getchar()
	if(state is ACCEPT)
		clear(stack)
	push (state)
	if(c=='a')
	goto q1

q1:
	c=getchar()
	if(state is ACCEPT)
		clear(stack)
	push(state)
	if(c=='b'||c=='c')
		goto q1
}
```

### NFA -> Table-driven Implemetation

It is also possible that we might not want to convert to a DFA.
It might be that the particular specification we gave is very expensive to turn into a DFA.

Implement transition via a table
![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/F9dfJ3Kb7b.JPG)

## Quiz

### Quiz 1

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/2GkiL3207F.JPG)

> We have 16 distinct strings of length 4,
> 8 distinct strings of length 3,
> 4 distinct strings of length 2,
> 2 distinct strings of length 1,
> and one empty string
> Ans=16+8+4+2+1=31

### Quiz 2

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/EJ4F29cJHA.JPG)

> This is a priority ordering, we should check the list from the beginning to the end.
> We also have to choose the **Maximal Munch**

### Quiz 3

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/49847aGhCJ.JPG)

> **Maximal Munch**: both rule 3 and 4 match the whole string
> But the rule 3 has a higher priority.

### Quiz 4

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/EEaIjgb23C.JPG)

### Quiz 5

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/lmIf5H9dIF.JPG)

> We need n states to culculate the number of 0 and 2n states to culculate the number of 1
> n+2n+1(start state)=3n+1

### Quiz 6

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/LEj2kKElCi.JPG)

### Quiz 7

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/16lIAb0iL5.JPG)
![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/Ij6i8jLa8C.JPG)

### Quiz 8

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/ILKimAdblB.JPG)

> b*(ab)* -> abab match better than a(ba)*

### Quiz 9

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/h7D7JjaL27.JPG)

### Quiz 10

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/c0J8K5GjJL.JPG)

> “11” and “1”
>
> ### Quiz 11
>
> ![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/icagcIlIEF.png?imageslim)
>
> DFAs must have finite states, no ϵ-moves, and one transition per input per state.
> The last option has infinite states.

### Quiz 12

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170825/ak0L0mdIeA.png?imageslim)

> NFAs must have a finite set of states, which rules out the last option. **Every DFA is also an NFA**, which is why the first option is included.