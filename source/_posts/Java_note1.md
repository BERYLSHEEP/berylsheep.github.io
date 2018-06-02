---
layout: post
title: "JAVA笔记：基础篇"
date: 2017-09-11
comments: true
tags: 
  - 学习笔记
  - JAVA
---


## 概述

JAVA学习过程中的简要笔记

从已有C++基础的前提下开始学习java

内容包括java的特性和机制、基本的语法、数组、几个容器、面向对象三大特性、内部类、异常处理、常见的类和简单的内存分析等

<!-- more -->

### 各版本和体系架构

- J2EE
  - JAVA 2(to) Enterprise Edition:
  - 定位在服务器端的应用
- J2SE
  - JAVA 2(to) Standard Edition:
  - 定位在个人计算机的应用
- J2EE
  - JAVA 2(to) Micro Edition:
  - 定位在消费性电子产品的应用

### 概念

```
- JDK
    - Java Development Kit Java开发工具包
- JRE
    - Java Runtime Environment java运行环境
- JVM
    - Java Virtual Machine java虚拟机

```

### 数据类型

![数据类型](http://ot1c7ttzm.bkt.clouddn.com/image/170908/gg4hmL0CDi.JPG)

### 整型变量

```
- 表示形式
    - 十进制
    - 八进制：以0开头
    - 十六进制：以0x或0X开头

```

整型默认为int型，long型常量后加”l”或”L”

| 类型    | 占用空间 | 表示范围             |
| ----- | ---- | ---------------- |
| byte  | 1字节  | -128~127         |
| short | 2字节  | -$2^15$~$2^15-1$ |
| int   | 4字节  | -$2^31$~$2^31-1$ |
| long  | 8字节  | -$2^63$~$2^63-1$ |

- 超过long的数字使用**BigInteger**类

### 浮点型

| 类型     | 占用空间 | 表示范围                 |
| ------ | ---- | -------------------- |
| float  | 4字节  | -3.403E38~3.403E38   |
| double | 8字节  | -1.798E308~1.798E308 |

- 默认为double
- 若需要不产生舍入误差的精确计算，需要用**BigDecimal**类
- double变float：在数字后加”f”或”F”

### 字符型

- char采用Unicode编码表，，占用2个字节
- 同c++,可以和整数相互转型，可以使用转义字符

### boolean

- 占1位(非字节)
- java中boolean无法和整数之间转换

### 变量

```
- 变量使用前需要声明

```

#### final

类似c++的const

```
- final修饰变量，则该变量无法修改
- final修饰类，则说明该类不能被继承，不能有子类
    - 如 Math、String
- final修饰方法，则该方法不能被子类重写，但是可以被重载

```

**命名规范**：

- 变量、方法名：
  - 首字母小写+驼峰原则
- 常量：
  - 大写字母+下划线
- 类名：
  - 首字母大写+驼峰原则

### 运算符

```
- 算术运算符： +,-,*,/,%,++,--
    - 不同于c++, java中浮点数可以使用"%"运算符！
    - 加号两边只要有一个字符串，则为字符串连接符，整个结果为字符串
- 赋值运算符： =
- 关系运算符： <,>,<=,>=,==,!=, instanceof
    - instanceof 判断内存中实际对象是否属于某个类 （通常用于造型cast）
        - 返回值：boolean  true代表属于
        - 使用： boolean ret= a instanceof A;
- 逻辑运算符： &&,||,!
    - 同c++,逻辑与和或采用**短路**方式，从左至右，若确定了值则不会继续计算
- 位运算符： &,|,^,~,>>,<<,>>>
    - >> 表示有符号右移：左边以该数的符号位补充，移出的部分将抛弃
        - 01110>>1 右移一位： 00111
        - 10010>>1 右移一位： 11001
    - >>> 表示无符号右移：左边以0补充，移出的部分将抛弃
        - 01110>>>1 右移一位：00111   (正数时和">>"相同)
        - 10010>>>1 右移一位：01001
    - << 没有 "<<<"
- 条件运算符：  ?:
- 扩展赋值运算符： +=,-=,*=,/=,%=

```

### switch语句

- 在JDK7之前，switch表达式结果只能是Int(可以转Int的byte,char,short),枚举类型
- JDK7之后，switch表达式结果还可以是字符串
- 使用方法同c++,进入一条语句后若未遇到break则自动执行下面的所有语句

```
switch(a)
{
	case 1: break;
	case 2: System.out.println(1);
	default: break;
}
```

### 语句

- if,while语句同c++
- while中可以使用break,continue关键字
- goto作为保留字无法作为变量，但也不能使用，取而代之的是带标签的continue和break
- 可以使用带标签的continue和break
  - 带标签的continue：跳转继续执行标签指向的循环
  - 带标签的break：停止标签指向的循环，跳至标签外的那层循环（若存在）

### 方法 method，function

[修饰符1 修饰符2..] 返回值 方法名(形参列表){
语句
}

```
- **Java中只有值传递！**
- 基本类型传递数据值本身，引用类型传递对象的引用而非对象本身
    - 即：基本类型传进去副本，不改变原来的值，引用类型传进去的为引用，会改变原来的值

```

### 内存分析

```
- 栈
    - 存放：局部变量
    - 自动分配连续的空间
- 堆
    - 存放new出来的对象
    - 空间不连续
- 方法区
    - 包含于堆区
    - 存放：类的信息(代码)、static变量、常量池(字符串常量)等

```

```
class Teacher{
	String name;
	int age;
	void teach() {}
}

public class Student {
	String name;
	int age;
	
	void study() {}
	
	public static void main(String[] args){
		Student st1=new Student();
		Student st2=new Student();
		st1.age=18;
		st1.name="张三";
		st2.age=30;
		st2.name="李四";
		
		Teacher t1=new Teacher();
		t1.age=30;
		t1.name="李四";
	}
}
```

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170908/c3j416JG4A.JPG)

## 包 package

定义：Java的类库管理机制，借助文件系统的目录

- 为什么需要使用Package

  - 为了解决类的重名问题
  - 为了便于管理类：合适的类位于合适的包

- 怎么使用package

  - 通常是类的第一句非注释性语句

    - 规则：通常域名倒过来写，加上模块名

      > com.qq.test

一个包内的所有的类必须放在一个目录下，那个目录的名字必须是包的名字

包名称内可以带有”.”，每个”.”代表文件系统的下一级目录

如

```
package p1;
package p1.s1;
```

则在文件系统中，p1文件夹下有个名为s1的文件夹包

### 使用其它包里的类:

1 . 使用**import**预先声明

```
import package1.T;  // 引入package1包里的T
import package2.*   // 引入package2包里的所有类
public class T2
{
	private T t1;
}
```

2 . 每次使用时附带说明包的名

```
public class T2
{
	private package1.T t1;
}
```

## API文档

使用**JAVADOC**生成API文档

```
- 解决代码文档分离的问题

```

- 特殊的注释
  - 文档注释： “/**”
- 常用的java注释标签
  - @Author 作者
  - @version 版本
  - @param 参数
  - @return 返回值含义
  - @throws 抛出异常描述
  - @deprecated 废弃，建议用户不再使用该方法

```
/**
 * 类的描述、功能
 * @author aa
 * @version 1.0
 */
```

### 垃圾回收机制 (Garbage Collection)

```
- 对象空间的分配
    - 使用"new"关键字创建对象
- 对象空间的释放：
    - 将对象赋值为null即可，垃圾回收器将负责回收所有不可达的对象的内存空间
- 要点：
    - 程序员无权调用垃圾回收器
    - 程序员可以通过System.gc()通知GC运行，但JAVA规范并不能保证立刻运行
    - finalize方法是JAVA提供给程序员用来释放对象或资源的方法，但尽量少用

```

## 数组

### 定义数组变量

1 - <类型>[]<数组名>=new <类型>[元素个数];

> 例如：
>
> int [] grade = new int [100];
>
> 这样创建的数组会是默认的0值

2 - <类型>[]<数组名>={元素1，元素2…};

> 例如： int []a={1,2,3};

3 - <类型>[]b = a; (a为同类型的数组)

> 与c++中数组名的含义类似，数组名变量管理数组空间
>
> b和a管理同一个数组

### 特点：

1 - 所有元素的数据类型相同
2 - 一旦创建则不能改变大小

### 要求：

- 元素个数必须是整数
- 元素个数必须给出
- 元素个数可以是变量

### 内部成员：

length : 返回元素个数

> for(int i=0; i<grade.length ; ++i)

### 数组和数组变量：

- 数组变量为数组的管理者，而非其本身
- 数组创建出来后交给数组变量管理
- 数组变量之间的赋值为管理权限的赋予
- 数组变量的比较是判断是否管理同一个数组

### 常用操作：

1. 数组复制
   System的arraycopy函数

   ```
   public static void arraycopy(Object src,
                                int srcPos,
                                Object dest,
                                int destPos,
                                int length)
   // src 源数组
   // srcPos 源数组开始复制的位置
   // dest 目标数组
   // destPos 目标数组开始复制的位置
   // length 复制长度
   // [srcPos,srcPos+length-1)
   ```

2. 二分查找
   Arrays.sort(Object obj); // 非基本类型需要实现Comparable接口
   Arrays.binarySearch(Object obj, Object goal);

3. 填充
   Arrays.fill(Object obj, int start, int end, Object goal)
   将目标数组[start,end)填充为goal

### for-each循环：

例如：for( int k: data )

对于data数组的每一个元素，循环每一轮取一个元素作为k

第一轮 k=data[0] , 第二轮 k=data[1]

**for-each循环复制基础类型数组元素，无法修改基础类型数组元素的值**

## 包裹类型

- 每种基础类型都有对应的包裹类型
- 出现原因：基本数据类型不面向对象，但我们经常需要将基本数据转化成对象

| 基础类型    | 包裹类型      |
| ------- | --------- |
| boolean | Boolean   |
| char    | Character |
| int     | Integer   |
| double  | Double    |

### 用处：

- 声明变量时包裹类型同基础类型

> int a=1; <=> Integer a=1;

- 可以获取该类型的信息，如最大值最小值

> 如 System.out.println(Integer.MAX_VALUE);
>
> ( Integer.MAX_VALUE=2147483647 )

- Character

| 函数声明                           | 参数      | 用途            |
| ------------------------------ | ------- | ------------- |
| static boolean isDigit         | char ch | 判断该字符是否是数字    |
| static boolean isLetter        | char ch | 判断该字符是否是字母    |
| static boolean isLetterOrDigit | char ch | 判断该字符是否是数字或字母 |
| static boolean isLowerCase     | char ch | 判断该字符是否是小写字母  |
| static boolean isUpperCase     | char ch | 判断该字符是否是大写字母  |
| static boolean isWhitespace    | char ch | 判断该字符是否是一种空格  |
| static char toLowerCase        | char ch | 把该字符转换成小写     |
| static char toUpperCase        | char ch | 把该字符转换成大写     |

**转换仅体现在返回值，原变量的值没有改变**

#### 自动装箱和自动拆箱 auto-boxing & unboxing

- 自动装箱
  - 基本类型自动封装到和它相同类型的包装里
  - 本质： Integer i=100; 编译器编译时： Integer i = new Integer(100);
- 自动拆箱
  - 包装类对象自动转换成基本数据类型
  - 本质： int a = new Integer(100); 编译器编译：int a = new Integer(100).intValue();

#### **缓存处理**

```
-  [-128,127]之间的数，仍然当做基本数据类型处理（增加效率）

```

```
Integer d1=1234;
Integer d2=1234;

System.out.println(d1==d2);
System.out.println(d1.equals(d2));

Integer d3=123;
Integer d4=123;

System.out.println(d3==d4);
System.out.println(d3.equals(d4));
```

结果：

> false (d1==d2)
> true (d1.equals(d2))
> true (d3==d4)
> true (d3.equals(d4))

## 字符串 String

- String是一个类，String变量是对象的管理者而非所有者

### 创建：

1 - String s = new String(“hello world”);

2 - String s = “hello”;

### 字符串连接

- 用 “+” 连接两个字符串

> “hello “+”world” -> “hello world”

- 若“+”一边为字符串一边非字符串，则将另一边表达为字符串而后进行连接

> “I am “+18 -> “I am 18”
>
> 1+2+”age” -> “3age”
>
> “age”+1+2 -> “age12”

### 字符串输入

- in.next();
  读入一个单词，以空格结束
- in.nextLine();
  读入一整行

### 字符串比较

- 用 “==” ：比较两个对象是同一个字符串

> if(input == “hello”)

- 用 “.equals” ：比较两个字符串的内容是否相同

> if(input.equals(“hello”))

### 字符串操作

- 字符串是**对象**，所有操作是通过“.”进行的
- 表示对“.”左边的字符串做右边的操作
- 字符串可以是**变量**也可以是**常量**

### 常用操作

**所有操作都不会对字符串本身进行修改**

- s1.compareTo(String s2)

> 比较两个字符串大小
>
> s1>s2 返回正数
>
> s1<s2 返回负数
>
> s1和s2内容相同，返回0
>
> 字符串长者大
>
> 字符串长度相同时，从第一位开始以字典序比较

- s1.length()

> 获得String的长度

- s1.charAt(int index)

> 访问String里的字符
>
> 返回index上的单个字符
>
> 0<=index<=s1.length()-1
>
> **不能用for-each循环遍历字符串**

- s1.substring(int begin ,int end)

> 获得 s1 下标于[begin,end)的子串
>
> end可省略，默认为s1.length()

- s1.indexOf( (char)|(String) c, int n)

> 寻找c字符或字符串第一个所在位置，-1为不存在
>
> 从n开始找(包括n)，n可以省略，默认为0
>
> s1.lastIndexOf(c,n) 从n开始向右边找，此时n默认为s1.length()
>
> 寻找字符串中第二个的方法： s1.indexOf(c, s1.indexOf(c)+1);

- s1.startsWith( String s2)

> 判断s1的前缀是否为s2

- s1.endsWith( String s2)

> 判断s1的后缀是否为s2

- s1.trim()

> 清除s1左右两端的空格
>
> 要保留清除可以 s1=s1.trim();

- s1.replace( (char)|(String) c1,(char)|(String) c2) (c1、c2同类型)

> 将s1中所有字符或字符串c1替换成c2

- s1.toLowerCase() s1.toUpperCase()

> s1大小写切换

- s1.split(char a)

> 以a作为分隔符分割字符串s1，将结果作为字符串数组返回
>
> String [] s= s1.split(“ “);
>
> 多种分隔符用”|”分割，(“a”|”b”)
>
> 以”.””|”作为分隔符时要加”\\“，即(\\.)(\\|)

### StringBuilder 和 StringBuffer

StringBuilder：线程不安全，效率高
StringBuffer：线程安全，效率低
两者基本相同

String无法进行修改，若用”+=”操作符时，系统会自动创建一个新的字符串，因此开销很大。

用StringBuffer/StringBuilder能够很好解决string的扩展问题

- StringBuilder sb= new StringBuilder();

- sb.append(“1”);

  > 扩展字符串
  > 其返回值为StringBuilder,因此可以连续调用
  >
  > ```
  > // JDK源码
  > @Override
  > public StringBuilder append(String str) {
  >     super.append(str);
  >     return this;
  > }
  >
  > // 因此可以使用
  > sb.append("1").append("2");
  > ```

- sb.toString();

  > 将StringBuilder形成字符串，可作为函数的String返回值

- sb.delete(int start , int end);

  > 删除下标为[start,end)的字符

- sb.reverse();

  > 反转sb

#### 容量机制

```
- StringBuffer和StringBuilder继承自AbstractStringBuilder，AbstractStringBuilder有非final的value字符串数组（String的value字符串数组为final）
- 默认容量为16，或是给定初始字符串长度+16或是给定初始长度
- 扩容为当前容量*2+2

```

## 类

### 创建对象

对象变量是对象的**管理者**

- T tem = new T();

### 成员变量

- 定义：类内部的变量
- 特点：成员变量的生存期是对象的生存期，作用域为类内部的成员函数
- 若没有初始化，则会自动被赋值(0)
- 初始化
  - 定义时就可以给出初始值
  - 对象变量的0值表示没有管理任何对象，也可以主动给null值
  - 定义初始化可以调用函数，可以使用已经定义的成员变量

```
public class T
{
	int a=1;
	int b=f();
	int f() {return 1;}
}
```

### 构造函数

- 名字和类名字完全相同，创建对象时会自动调用
- 没有返回值
- 调用时先完成外部成员变量初始化，而后再逐步执行构造函数内的语句

### 函数重载

- 定义：同名但参数列表不同的函数
- 可以有多个构造函数，但参数列表要不同
- 构造函数内可以通过this()调用其它构造函数,但只能是第一句，只能在构造函数内使用，只能使用一次

```
public class T
{
	int a=1;
	T() {a=2;}
	T(int a)
	{
		this();
		this.a=a;
	}
}
```

若调用带参数a的构造函数，a的最终值为a，但赋值过程为：1->2->3，调用过程为：T(int a)->T()->int a=1;->执行T()->执行T(int a)

### 访问属性

- public (同c++)
- private (同c++)
- protected (同c++)
- friendly

#### public

- 任何人都可以直接使用
  - 任何函数或定义初始化中可以使用
  - 使用是指调用、访问、定义变量
- public类必须定义在自己的文件里
- 一个编译单元只能有一个public类 (一个编译单元：一个.java文件，一次对这一个文件进行编译)

#### private

- 只有类的内部能够访问
  - 类的成员函数
  - 定义初始化
- 限制是针对类而非对象
  - 对象之间可以相互访问private成员变量(a对象的成员函数可以直接访问b对象的私有变量)

#### friendly

- 未加private或public声明的成员 (即**默认值**)
- 属于同一个包内的所有类都可以访问

### System.out.println

正常情况下，调用System.out.println直接输出类的对象，则会输出其地址

若是在类中包含 public String toString 函数，则在println时自动调用该函数

```
public class T{
	int c;
	public String toString () {return ""+c;}
}
```

## 常用类

### 时间处理：

#### java.util.Date

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170910/197ED2Bj8D.JPG)

- 对象表示一个特定的瞬间，精确到毫秒
- 时间的表示为数字：从 标准纪元1970.01.01 0点开始到某个时刻的毫秒数，类型为long

#### DateFormat, SimpleDateFormat

java.text.*

- 用来将字符串和时间相互转换
- DateFormat 是抽象类，无法实例化对象
- SimpleDateFormat非抽象类

使用方法（详见API文档）

1. 已知毫秒转相应时间和字符串

   ```
   DateFormat df = new SimpleDateFormat("yyyy年MM月dd日hh时mm分ss秒");
   DateFormat df1 = new SimpleDateFormat("yy-MM-dd hh:mm:ss");
   Date d = new Date(1321454564);
   String str=df1.format(d);
   System.out.println(str);
   ```

2. 已知字符串及输入格式转毫秒数

   ```
   String str2 = "1977-07-07";
   DateFormat df2 = new SimpleDateFormat("yyyy-MM-dd");
   try {
   	Date d2 = df2.parse(str2);
   	System.out.println(d2);
   } catch (ParseException e) {
   	// TODO Auto-generated catch block
   	e.printStackTrace();
   }
   ```

#### Calendar，GregorianCalendar 日历类

java.util.*

- Calendar是抽象类，无法实例化对象
- GregorianCalendar是Calendar的一个具体子类，提供了大多数国家和地区使用的标准日历系统
- Calendar类用来和Date类做切换，把计算机保存的时间转换成人能看懂的日期
- 注意
  - 月份：1月是0，2月是1，…12月是11
    - 星期：周日是1，周一是2…周六是7
    - 可以使用 Calendar.FEBRUARY 表示数字

使用方法：（详见API文档）

1. 整体设置

   ```
   Calendar c = new GregorianCalendar();
   c.set(2001, Calendar.FEBRUARY ,15);
   Date date = c.getTime();
   ```

2. 单独设置

   ```
   c.set(Calendar.YEAR , 2001);
   c.set(Calendar.MONTH , 1|Calendar.FEBRUARY);
   // 未设置参数则自动采用当前时间 (日期、时分秒等)
   System.out.println(c.get(Calendar.YEAR));
   ```

3. 用Date赋值

   ```
   c.setTime(new Date());
   ```

4. 日期计算

   ```
   c.add(Calendar.YEAR, 30);
   ```

### File 类

java.io.File:文件和目录路径名的抽象表现形式

```
- 通过File对象可以访问、修改文件属性
- 可以创建空文件或目录

```

```
File f = new File("d:/aaa/bbb/ccc");
f.mkdir();  // 若父目录存在则创建
f.mkdirs(); // 若父目录不存在则自动创建父目录
```

## 容器

- 容器类的两个类型：

  1 . 容器的类型

  2 . 元素的类型

### ArrayList

- import java.util.ArrayList;

- 特点：元素可以相同，元素以与进入的顺序排序

- 基础类型：存放值

- 非基础类型：作为管理者，存放地址

- 定义：

  ArrayList notes = new ArrayList();

#### 常用操作

- notes.add(int location, String s);

> 向下标为location的位置中插入元素
>
> location可省略，默认为末尾

- notes.size();

> 返回容器中存放元素数目

- notes.get(int index);

> 返回下标为Index的元素

- nodes.remove(int index);

> 移除下标为Index的元素

- notes.toArray(String []a);

> 将notes里所有元素依次填入a中

#### for-each循环

若ArrayList存放的非基础类型，则for-each中可以修改值

### Set

- import java.util.HashSet
- 特点：元素均不相同，不排序，与进入的顺序无关

### Hash表(散列表)

- import java.util.HashMap;
- HashMap < Key, Value > a=new HashMap < Key,Value > ();
  数据以一对值放进去，一个为键(KEY),一个为值(value)
  值对应键
- 键不能重复，只留相同的最后一个，元素不以先后排序

#### 常用操作

- a.put(Key k1, Value v1);

> 向HashMap里添加元素，键k1对应值为v1

- a.get(Key k);

> 返回键k对应的值，若为空，则返回null

- a.containsKey(Key k);

> 查询是否存在键为k的数据

- a.keySet();

> 返回key的集合
>
> 可以通过 a.keySet().size()获取存放的键的数目
>
> 通过keySet使用for-each循环

## 继承

- extends

  ```
  public class son extends parent
  ```

- **java只允许单继承，一个类只能有一个父类**

| 父类成员访问属性  | 在父类中的含义           | 在子类中的含义                   |                    |
| --------- | ----------------- | ------------------------- | ------------------ |
| public    | 对所有人开放            | 对所有人开放                    |                    |
| protected | 只有包内其它类、自己和子类可以访问 | 只有包内其它类、自己和子类可以访问         |                    |
| 缺省        | 只有包内其它类可以访问       | 如果子类与父类在同一个包内：只有包内其它类可以访问 | 否则：相当于private，不能访问 |
| private   | 只有自己可以访问          | 不能访问                      |                    |

子类同样继承了父类的private成员，只是子类无法访问！
但子类可以通过调用父类的非private函数来间接修改父类的private成员变量

### 继承VS组合

- “is-a”关系使用继承
  - 如：Bird类继承自Animal类
- “has-a”关系使用组合
  - 如：Computer类包含CPU类，每个Computer对象都有一个CPU对象

### super

本质：一个**关键字**，类似this(this的本质为指针),是直接父类的引用

用法
1 . 直接引用

相当于指向当前对象的父类，用”.”访问父类的成员

2 . 子类成员变量或方法与父类同名时，用super加以区分

```
class son extends parent{
	void a();
	void b(){
		a();   // son的a函数
		super.a();  // parent的a函数
	}
}
```

3 . 调用父类的构造函数

规定：必须写在子类构造的第一行，不能和this同时出现在一个构造函数

子类的构造函数第一行都会隐含地包含super()函数，若此时父类没有不含参数的构造函数，则无法通过编译

```
class son extends parent{
	int a;
	son(int a1){
		super(a1);
	}
}
```

### 同名变量

若没有明确指明，则在谁的成员函数中就使用谁的成员变量

### 子类和子类型

- 类定义了类型
- 子类定义了子类型 (son是parent的一个子类型)
- 子类对象可以被当做父类的对象使用
  - 赋值给父类变量
  - 传递给需要父类对象的函数
  - 放进存放分类对象的容器里

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170903/gfj3geAc0f.JPG)

```
Vehicle v1 = new Vehicle();
Vehicle v2 = new Car();
Vehicle v3 = new Bicycle();
```

## 多态

主要用来实现**动态联编**

即程序的最终状态只有在执行过程中才被决定而非在编译时期就决定了

以此提高系统的灵活性和扩展性

### 如何实现多态

```
- 引用变量的两种类型

    - 编译时类型（一般是父类）
        - 由声明时的类型决定

    - 运行时类型（运行时具体的子类）
        - 由实际对应的对象类型决定

- 三个必要条件
    - 要有继承
    - 要有方法重写
    - 父类引用指向子类对象

```

### 多态变量

- 对象变量能够保存不止一种的对象
- 可以保存声明类型的对象或是声明类型的子类的对象
- 子类对象赋给父类变量，发生**向上造型**

### 造型 cast

造型(cast)：把一个类型的对象赋值给另一个类型的变量

- 子类对象可以复制给父类变量
  - 与C++不同：java不存在对象与对象的赋值，只能是管理者修改指向的对象
- 父类对象不能赋值给子类变量
- 可以使用造型强制赋值，但只有当父类变量实际管理的是子类的对象才行
- 类型转换 ！= 造型
  - 类型转换中对象发生改变，如 int i=(int)10.2;
  - 造型中对象本身没有改变，只是改变了看待它的方式

```
Vehicle v;
Car car = new Car();
v=car;  //可以
c=v;   //编译错误
c=(Car)v // 编译通过

若
Vehicle v=new Vehicle();
Car car=(Car)v; // 编辑器不报错，但会出现ClassCastException异常
```

#### 向上造型

- 子类对象当做父类对象使用
- 向上造型是默认的，不需要运算符
- 向上造型是安全的

### 绑定

同名函数调用父类对象时，会根据实际管理对象的类型调用相应的函数

- 静态绑定：根据变量声明类型决定
- 动态绑定：根据变量的动态类型决定 ( 实际管理的类型 )
- 默认使用动态绑定
- 成员函数中调用其它成员函数通过this变量调用

### 覆盖 override

- 覆盖关系：子类和父类中存在名称和参数表完全相同的函数
- 通过父类变量调用存在覆盖关系的函数时，会调用变量当前所管理的对象所属的类的函数

### Object类

- 所有的类都继承自Object类
- 几乎所有OOP都有Object类（除c++）

#### Object类的函数(部分)

- toString()
- equals()

### 多态的内存分析

1 . 基本的继承关系和方法调用

```
- 父类：Animal  有voice方法
- 子类：Cat 重写voice方法，自己有catchMouse方法
- 测试：分别声明父类和子类变量并指向子类对象

```

```
public class Test{
	
	public static void showVoice(Animal c) {
		c.voice();
		if(c instanceof Cat)
			((Cat) c).catchMouse();
	}
	public static void main(String[] args) {
		Animal a=new Cat();
		Cat cat=(Cat)a;
		Test.showVoice(cat);
		Test.showVoice(a);
	}
}

class Animal{
	String name;
	int age;
	void voice() {}
}

class Cat extends Animal{
	void voice() {
		System.out.println("miao");
	};
	void catchMouse() {}
}
```

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170909/Jd69EhEaBa.JPG)

2 . 深化多态

```
- 父类：HttpServlet 有service方法和doGet方法，其中service方法中调用doGet方法
- 子类：MyServlet 重写了doGet方法
- 测试：父类变量指向子类对象，调用父类的service方法观察doGet执行的版本

```

```
public class HttpServlet {
	public void service() {
		System.out.println("HttpServlet.service()");
		doGet();
	}
	public void doGet() {
		System.out.println("HttpServlet.doGet()");
	}
}


public class MyServlet extends HttpServlet {
	public void doGet() {
		System.out.println("MyServlet.doGet()");
	}
}

public class Test{
	
	public static void main(String[] args) {
		HttpServlet s = new MyServlet();
		s.service();
	}
}
// 输出结果：
// HttpServlet.service()
// MyServlet.doGet()
```

原因分析：
函数调用本身隐含着 this 引用， 在service中实际是 this.doGet()，此时this指向的为MyServlet即是子类的对象，因此调用子类重写的doGet函数

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170909/70jHJ3hJEi.JPG)

## 设计原则

### 消除代码复制

- 代码复制是不良设计的一种表现

  解决方法：使用函数封装重复的代码

### 增加可扩展性

- 后期的可维护性

  解决方法：用封装降低耦合，用接口实现聚合，借助类继承

  耦合：类和类之间的关系

  耦合越低越好

## 抽象和接口

关键词：**abstract** (抽象)

> public abstract class a

- 抽象函数：表达概念而**无法实现具体代码**的函数
- 抽象类：表达概念而**无法构造出实体**的类
- 为什么需要抽象类
  - **模板模式**，抽象类为子类提供了一个通用模板，子类可以在模板基础上进行扩展
  - **通过抽象类，可以避免子类设计的随意性**，严格限制子类的设计，使得子类之间更加通用。
- 抽象方法的意义：**使方法的设计和实现分离！**
- 要点：
  - 有抽象函数的类一定是抽象类
    - 抽象类**不能制造对象**，但可以定义变量
  - 任何继承了抽象类的非抽象类对象可以赋值给抽象类变量
  - 抽象类可以拥有非抽象函数、属性和构造方法，但构造方法只能用来被子类调用
  - 继承自抽象类的子类必须覆盖父类的抽象函数，不然也成为抽象类

### 接口

关键词：**interface**

> public interface a{}

- 接口是纯抽象类(接口中只有常量、抽象方法)
- 所有成员变量都是public abstract static final （默认的，可以不写）
- 接口意义：设计和实现分离，利于大项目制作
- 接口变量的含义为任何实现了接口的对象

关键词：**implements**

> public class b implements a{}

- 类可以实现很多接口
- 接口可以继承接口，不能继承类
- 接口不能实现接口
- 接口和抽象类
  - 接口是比”抽象类”还要”抽象”的”抽象类”，全面地专业地实现了**规范和具体实现的分离**
  - **接口就是规范，定义了一组规则**
  - **接口的本质是契约**，制定好后大家都遵守
  - 项目的具体需求是多变的，我们需要以不变应万变，这就是规范，因此需要**面向接口编程**

#### 面向接口的编程方式

- 设计程序时先定义接口，再实现类
- 任何需要在函数间传入传出的一定是接口而不是具体的类
- Java成功的关键之一，极适合多人同时写一个大程序
- Java被批评的要点之一，代码量膨胀地很快

## 控制反转与MVC模式

### Swing

容器、部件

- 容器继承自部件，因此可以作为部件被放入其它容器中

- 容器使用布局管理器管理内部的部件,能够根据不同的环境自动调整

- JFrame使用BorderLayout管理部件，并且将界面划分为五个区域

  ![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170907/8B7I3088Hh.JPG)

- 容器使用 add 函数添加部件，JFrame中若不指定放置区域，则默认为BorderLayout.CENTER,后添加的部件会覆盖前面的部件

### 控制反转

- 按钮公布一个listener接口和一对注册、注销函数
- 实现接口后将把listener对象注册在按钮上
- 一旦按钮被按下，就会反过来调用listener对象的函数

### 内部类 innerclasses

定义在别的类内部、函数内部的类

#### 作用：

```
- 提供了更好的封装，只能让外部类直接访问，不允许同一个包中其它类直接访问
- 内部类可以访问外部类的所有成员，但外部类不能访问内部类的内部属性

```

#### 使用场合

在内部类只为所在外部类提供服务的情况优先使用

#### 分类：

##### 成员内部类

（可以使用private,public,protected修饰）

###### 非静态内部类

`- 外部类使用非静态内部类和使用其它类相同 - 非静态内部类必须寄存在外部类的对象里，相当于外部类的一个属性。**非静态内部类对象单独属于外部类的某个对象**- 非静态内部类不能有静态方法、静态属性、静态初始化块- 静态成员不能访问非静态成员：外部类的静态方法、静态代码块补鞥呢访问非静态内部类，包括不能使用非静态内部类定义变量、创建实例- 成员变量访问要点：    1. 内部类里方法的局部变量：变量名    2. 内部类属性：this.变量名    3. 外部类属性：外部类名.this.变量名- 内部类的访问：    - 外部类中定义内部类： new innerClass()    - 外部类以外的地方使用非静态内部类        Outer.inner n = OuterObject.new Inner();`

```
public class Outer {
	public static void main(String[] args) {
		Face f=new Face();
		Face.Nose n= f.new Nose();
	}
}

class Face {
	int type;
	
	class Nose{
		String type;
		
		void breath() {
			Face.this.type=1;
		}
		
	}
}
```

###### 静态内部类

```
- 要点：
    - 静态内部类对象存在时，不一定存在对应的外部类对象
    - 静态内部类无法直接访问外部类实例方法
    - 静态内部类看作**外部类的一个静态成员**

```

```
Face f= new Face();
// Nose为非静态内部类
Face.Nose n=f.new Nose();
// Ear为静态内部类
Face.Ear e=new Face.Ear();
```

###### 局部内部类：定义在方法内部，作用于仅限于本方法，但用的极少

```
- 访问函数的本地变量时只能访问函数的final变量

```

##### 匿名内部类

适用于只需要使用一次的类，例如：键盘监听操作等

```
new 父类构造器(实参) 实现接口(){
	匿名内部类类体
}
```

- 匿名类可以继承某类，也可以实现接口
- Swing的消息机制广泛使用匿名类

### MVC设计模式

- 数据、表现和控制

  三者分离

  - M = Model 模型
  - V = View 表现
  - C = Control 控制

- 模型：保存和维护数据，提供接口让外部修改数据，通知表现需要刷新

- 表现：从模型获取数据，根据数据画出表现

- 控制：从用户得到输入，根据输入调整数据

### 异常机制 Exception

#### 常见的异常

```
用户输入错误
设备错误
硬件问题：如打印机关掉、服务器问题
磁盘满了

```

#### 异常Exception

```
Java提供的用来处理程序中错误的一种机制

Java采用**面向对象** 方式来处理异常。处理过程：
    抛出异常：执行方法时若发生异常，则这个方法生成代表异常的一个对象，停止当前执行路径，把异常对象提交给JRE
    捕获异常：JRE得到异常后寻找相应代码处理该异常。JRE在方法的调用栈中查找，从生成异常的方法开始回溯，直到找到相应的异常处理代码位置

```

#### 常见异常

```
1. ArithmeticException
    例如试图除以0
2. NullPointerException
    对象为null但调用了对象的方法或属性
3. ClassCastException
    转型错误，解决：使用istanceof判断
4. ArrayIndexOutOfBoundsException
    访问元素超出数组长度
5. NumberFormatException
    数字格式异常，如把String转换成int

```

#### 异常的处理方法：

`捕获异常（try，catch，finally）    try{// 可能出现异常的语句}catch(Exception e){// }finally{// }    未遇到异常时，执行完try内的语句后不执行catch，而后执行finally    出现异常时，跳转至catch，执行catch后，执行finally        try             try语句指定了一段代码，该段代码就是一次捕获并处理的范围。在执行过程中，当任意一条语句产生异常时，就会跳过该段中后面的代码。代码中可能会产生并抛出一种或几种类型的异常对象，它后面的catch语句要分别对这些异常做相应的处理            一个try语句必须带有至少一个catch语句块或一个finally语句块 。。                当异常处理的代码执行结束以后，是不会回到try语句去执行尚未执行的代码。        catch            每个try可以搭配多个catch，用来处理不同的异常            捕获异常时：越是顶层的类越是放在下面            常用方法：                toString() 显示异常类名和异常原因                getMessage() 只显示异常原因                printStackTrace() 跟踪异常发生时堆栈内容        finally            不管是否发生异常都需要执行的语句，一般是关闭资源            不要在finally中使用return        执行顺序：            1. 执行try、catch，给返回值赋值            2. 执行finally            3. return抛出异常 throws    方法声明中加throws，则谁调用该方法谁用try catch处理异常    可以throws多个异常    用**throw**(没有s)手动new异常对象并抛出异常`

#### 方法重写中声明异常原则：

```
**子类声明的异常范围不能超出父类的范围**
    - 父类没有声明异常，子类也不能
    - 不能抛出 原有方法抛出的父类或上层类
    - 抛出的异常类型数目不能比原有方法多（类型多 非 个数多）

```

#### 自定义异常

```
- 在程序中，可能会遇到任何标准异常类都没有充分描述清楚的问题，这时候可以创建自己的异常类
- 从Exception类或它的子类派生一个子类
- 习惯上，定义的类应该包含2个构造器，一个是默认构造器，一个是带有详细信息的构造器

建议：
    - 避免使用异常机制代替错误处理
    - 处理异常不能代替简单测试
    - 不要进行**小力度**的异常处理（如一行一个try catch）
    - 异常往往在**高层**处理

```