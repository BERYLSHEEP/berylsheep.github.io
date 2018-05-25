---
layout: post
title: "编译原理笔记（一）：概述"
date: 2017-08-23
comments: true
tags: 
	- 编译原理
	- 学习笔记
---


Stanford的CS1 Compilers 课程学习笔记

学习编译知识并最终制作COOL语言的编译器

<!-- more -->

## 01-01. Introduction

Two major approaches to implementing programming languages:
**Compilers(编译器)** 、 **Interpreters(解释器)**

### 1. Interpreters(解释器)

![Interpreters](http://ot1c7ttzm.bkt.clouddn.com/image/170823/5908CiF8j3.JPG)

**Online**: the work it does is all part of running your program.

The Interpreter produces the output with data and program directly

It doesn’t do any processing of the program before it executes the program on the input.

### 2. Compilers(编译器)

![Compilers](http://ot1c7ttzm.bkt.clouddn.com/image/170823/Af9BI1GDC1.JPG)

**Offline**: produces an executable
The executable is **another program**, might be another language or bytecode, it can be **run separately** on data.

### 01-02. FORTRAN:

FORTRAN ( Formula Translation Project )：**The first successful high level language**.
Meanings : The formulas were translated into a form that the machine could execute directly.

Programming languages = fairly deep theory + good engineering skills

The structure of FORTRAN one:
1.Lexical Analysis (词法分析)
2.Parsing (语法分析)
3.Semantic Analysis (语义分析)
4.Optimization (优化)
5.Code Generation (代码生成)

#### 1.Lexical Analysis (词法分析)

Goal : **divide** the program text into its **tokens** (words)(符号).

Example of tokens :
1.key words 关键词 ( like “if”)
2.variable names 变量名 (“x”,”y”,”z”)
3.constants 常量 (“1”,”2”)
4.operators 运算符(“=”,”==”)
5.punctuation 标点符号(“;” “,”)

![tokens](http://ot1c7ttzm.bkt.clouddn.com/image/170823/I3eDhhmCgf.JPG)

#### 2.Parsing (语法分析)

Goal : **diagramming sentences **(分析句子)

The first step : Identify the role of each word
![structure tree](http://ot1c7ttzm.bkt.clouddn.com/image/170823/EkedKA1ch4.JPG)

![structure tree](http://ot1c7ttzm.bkt.clouddn.com/image/170823/kAjm7Bi5B2.JPG)

#### 3.Semantic Analysis (语义分析)

The compilers can only do very** limited** kinds of semantic analysis.

It generally try to catch inconsistencies. (捕捉异常)

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170823/6bhGdDehIL.JPG)

![lexically scoped](http://ot1c7ttzm.bkt.clouddn.com/image/170823/d07LaK2mgh.JPG)

#### 4.Optimization (优化)

Goal : To modify the program so that it **uses less of some resource** ,(use less time, run faster or use less space)

if(x and y are integers)
![Optimization](http://ot1c7ttzm.bkt.clouddn.com/image/170823/lKCdIdhI1K.JPG)

#### 5.Code Generation (代码生成)

The most common goal : produce assembly code (生成汇编代码)

General goal : do a translation into some other languages.

#### The proportions (比例)

![proportions](http://ot1c7ttzm.bkt.clouddn.com/image/170823/CAJ9Dbc8jJ.JPG)

FORTRAN I :
complex lexical analysis phase and parsing phase,
small semantic anaysis phase (very weak)

Today:
very little in parsing and lexical analysis. ( we have good tools to help us )
very large optimization phase
a small code-generation phase ( we also have good tools )

### 01-03. The Economy of Programming Languages

1.Why are there so many programming languages
2.Why are there new programming languages
3.What is a good programming language

#### 1. Why are there so many programming languages

1.**application domains**(应用域) have distinctive and conflicting needs.

![application domains](http://ot1c7ttzm.bkt.clouddn.com/image/170823/gcEe09H4AH.JPG)

It would be** difficult to integrate all of these into one system **that would do good job on all of these things.

#### 2. Why are there new programming languages

**Programmer training** is the dominant cost for a programming language

Here are some facts:

1. Widely-used languages are slow to change

   > Design and build a compiler for a new language are not actually taht expensive.
   > The real cost is in all the **users and educating them**.

2. Easy to start a new language

   > Zero or low training cost
   > Adapt quickly to changing situations, not very costly to experiment.

3. Languages adopted to fill a void

   > Programming languages exist for **purpose**.
   > There are **new application domains** coming along all the time.

#### 3. What is a good programming language

There is **no universally accepted metric**(普遍接受的标准) for language design.

### 02-(01-03). Cool Overview

**COOL**: Classroom Object Oriented Language

COOL -> MIPS assembly language

```
class Main
{
	main():Int { 1 };
};
class Main
{
	i : IO <- new IO;
	main():IO {  i.out_string("Hello World!\n") };
};
class Main
{
	i : IO <- new IO;
	main():Object {  i.out_string("Hello World!\n"); 1; };
};
class Main
{
	main():IO {  (new IO).out_string("Hello World!\n") };
};
class Main inherits IO
{
	main():IO {  self.out_string("Hello World!\n") };
};
class Main inherits IO
{
	main():IO {  out_string("Hello World!\n") };
};



class Main
{
	main():Object 
	{
		(new IO).out_string( (new IO).in_string.concat("\n\") )
	};
};
// in_string : 接受输入，返回string
// concat :  连接字符串
class Main inherits A2I    // ascii to integer
{
	main():Object 
	{
		(new IO).out_string(  i2a( a2i(new IO).in_string)+1 ).concat("\n\")       )
	};
};
class Main inherits A2I
{
	main():Object 
	{
		(new IO).out_string(  i2a( fact(a2i(new IO).in_string) ).concat("\n\")       )
	};
	fact( i: Int ) : Int { if(i=0) then 1 else i*fact(i-1) fi };
};

class Main inherits A2I
{
	main():Object 
	{
		(new IO).out_string(  i2a( fact(a2i(new IO).in_string) ).concat("\n\")       )
	};
	fact( i: Int ) : Int 
	{ 
		let fact: Int <- 1 in  // 声明局部变量
		{
			while ( not (i=0) ) loop
			{
				fact <- fact*i;
				i <- i-1;
			}
			pool;
			fact;
		}
	};
};


class List inherits A2I    //链表类
{
	item : Object;    // 数据
	next : List ;    // 指向下一个
	init ( i: Object , n: List ): List
	{
		item <- i;
		next <- n;
		self;    // 返回值为List 
	}
	flatten (): String
	{
		let string : String <-
			case item of       // 相当于switch    关键词为 case .. of 
				i : Int => i2a(i);     //  变量名+“:”+类型+操作
				s : String => s;
				o : Object => { abort(); ""; };    // 相当于default分支，因abort返回Object类，为避免编译错误所以在最后加了"" 以返回String
			esac;    // case的结束标志
		in    // 局部变量范围
			if ( isvoid next ) then    // 若next为空
				string
			else
				string.concat(next.flatte())
			fi     // if的结束标志
	};
};
	
class Main inherits IO
{
	main(): Object
	{
		let 	hello : String <- "Hello ",
			world : String <- "World!",
			newline: String <- "\n",
			nil : List,    // 不初始化则自动为空，因没有NULL标记
			list : List <-
				(new List).init(hello, 
					(new List).init(world,
						(new List).init(newline, nil ) ) )
		in
			out_string(list.flatten())
	};
};
```

语法：
class 是一系列method(函数)的集合，每个method以大括号+分号结尾
必有Main class和main函数
函数后用 “：”+”type” 声明函数的返回类型
单个表达式无需分号，多个表达式以分号分隔，无需写明返回值为哪个，返回值为最后一个表达式
所有类继承自Object, self相当于其它语言中的this
用“=”作判断，用”<-“作赋值
if以fi作为结尾
loop以pool结尾
case..of以esac结尾，每个分支都要以分号结尾