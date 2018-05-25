---
layout: post
title: "数据结构笔记（一）：线性表"
date: 2017-08-12
comments: true
tags: 
    - 学习笔记
    - c++
    - 数据结构

---

# **写在前面的话**

开始数据结构的学习，将学习过程中的思维过程和代码记录下来。
教材主要使用清华大学邓俊辉教授的《数据结构(c++语言版)》
辅以清华大学以及浙江大学的mooc。

<!-- more -->

# **概述**

## 线性表 List

线性表的定义：由同类型**数据元素**构成**有序序列**的线性结构

```
- 表中元素的个数称为线性表的**长度**
- 线性表中没有元素时称为**空表**
- 表起始位置称为**表头**，表结束为止称为**表尾**

```

线性表的操作类型：

1. 静态操作：仅读取，内容和组成一般不变，如get、search
2. 动态操作：需写入，局部或整体将改变，如insert、remove

基本操作：

```
1. List MakeList()
> 初始化一个空的线性表
2. Element FindIndex(int index)
> 根据位序k返回相应元素
3. int Find(Element x)
> 查找x第一次出现的位置
4. void Insert(Element x , int index)
> 在位序index前插入一个新元素x
5. void Delete(int index)
> 删除指定位序的元素

```

实现方式：

```
1. 顺序存储实现：数组

```

```
typedef struct ListNode * List
struct ListNode
{
	Element data[MAX_SIZE];
	int last;
}
```

```
2. 链式存储实现：链表
> 不要求逻辑上相邻的两个元素物理上也相邻，通过“链”建立起数据元素之间的逻辑关系

```

以下以链表为主介绍并实现线性表

## **链表特点：**

动态存储：

1. 各元素动态地分配和回收空间
2. 逻辑上相邻的元素记录彼此的物理地址
3. 动态操作高效，静态操作费时，循秩访问低效

各节点通过指针或引用连接，在逻辑上形成线性序列

**链表实现过程中，代码的执行顺序非常重要！要注意的细节也很多。**

以下均用c++实现。

# **分类**

1.单向链表

> 仅有succ指针

2.双向链表

> 有pred和succ指针

3.循环单链表

> 单向链表 + 末节点和首节点相连

4.循环双链表

> 双向链表 + 末节点和首节点相连

5.静态链表

> 在没有指针的语言中使用，借用数组模拟链表

## **建议**

使用**哨兵**节点

单向链表增加**首**哨兵

双向链表增加**首末**哨兵

哨兵不对外公开，在一开始创建，最后销毁

**优点：简化边界条件，增加鲁棒性**

# **单链表**

**基本定义**

```
template < typename T >
struct node
{
    T data;
    node<T> *succ;
};

template < typename T >
struct list
{
    node<T> *header;
    int _size;
}
```

**带接口完整定义**

```
template < typename T >
struct node
{
    T data;
    node<T> *succ;
    node( T const & e , node<T> * suc=NULL):data(e),succ(suc) {}
    node() {succ=NULL;}
    node<T> * insertAft( T const & e );
};

template < typename T >
struct list
{
private:
    node<T> *header;
    int _size;
public:
    void init ();
    int clear ();
    void copyNodes ( node<T> *p , int n );     
    //从p开始复制n个节点到链表末尾
    void merge ( node<T> *& p1, int n1, node<T> *&p2, int n2 );
    //[p1,p1+n) 本串p1开始n1个节点与 L串p2开始n2个节点归并排序
    void mergeSort ( node<T> *&p, int n );      
    // 对从p开始连续的n个节点归并排序

    list() { init(); }
    list( node<T> *p , int n );     // 从p开始复制n个节点
    ~list();

    // 只读接口
    bool empty() { return _size==0; }
    int size() { return _size; }
    T& operator[] ( int r ) const;    // 实现寻秩访问
    node<T> * find( T const&e ) const;
    node<T> * find( T const& e,  node<T>* p, int n ) const;
    // (无序)从节点p向后n个节点（不含p）内找e
    node<T> * search( T const & e , node<T> * p , int n ) const;
    // (有序) 不大于e的最后一个
    void show();

    // 可写接口
    node<T> * insertBefore( node<T> *p, T const & e );
    node<T> * insertAfter( node<T> *p, T const & e );
    node<T> * insertLast( T const & e );
    node<T> * insertFirst( T const & e );
    void createListT( T const * e , int n ); // 尾插法创建链表
    void createListH( T const * e , int n ); // 头插法创建链表
    T remove( node<T> *p );
    int deduplicate (); // 无序列表去重
    int uniquify ();    // 有序列表去重
    void sort(); // 排序
};
```

**基本接口**：

\1. 创建链表

> 分为：创建空链表、根据已有数据进行头、尾插入、复制已有链表

\2. 插入单节点

> 分为在某节点前、后插入，在表头、表尾插入

\3. 查找数据

> 无序单链表：查找是否存在 有序单链表：返回不大于目标数据的最后一个元素

\4. 循秩访问第n个节点

> 可重载[]运算符，也可单独使用一个函数

\5. 删除单个节点

\6. 清空链表

> 思路：循环调用删除单个节点的remove函数删除header的直接后继，直至header的直接后继为NULL

\7. 打印链表

\8. 检查链表是否为空

\9. 返回链表长度

\10. 排序（归并、插入、选择等）

\11. 去重（无序、有序版）

\12. 反转

## 具体实现

### 1. 创建链表

1-1 创建空链表

```
template < typename T >
void list<T>::init()
{
    header = new node<T>; //构造函数已将后继默认置位NULL
    _size=0;
}
```

1-2 尾插法

```
template < typename T >
void list<T>::createListT( T const *& e , int n )
{
    node<T> * last=header; // last作为链表最后一个节点
    node<T> * tem;
    _size+=n;
    for( int i=0; i<n; i++ )
    {
        tem = new node<T>( e[i] );
        last->succ=tem;  // 顺序不可反
        last=tem;
    }
    return;
}
```

1-3 头插法

```
template < typename T >
void list<T>::createListH( T const *& e , int n )
{
    node<T> * tem;
    _size+=n;
    for( int i=0; i<n; i++ )
    {
        tem = new node<T>( e[i], header->succ );
        header->succ=tem;
    }
    return;
}
```

1-4 复制链表

```
template < typename T >
list<T>::list( node<T> * p, int n )
{
    init();
    copyNodes( p , n );
}

template < typename T >
void list<T>::copyNodes( node<T> * p, int n )
{
    node<T> *tem=header;
    while( tem->succ ) // 先探好路再迈步
        tem=tem->succ;  // tem指向存在的最后一个元素
    node<T> *create;
    _size+=n;
    while(n--)
    {
        create=new node<T> ( p->data );
        tem->succ=create;
        p=p->succ;
        tem=tem->succ;  // 实时更新链表尾
    }
    return;
}
```

### 2. 插入单节点

2-0. 准备：node类的插入函数

```
template < typename T >
node<T> * node<T>::insertAft( T const & e )
{
    node<T> *tem=new node<T>( e , succ );
    succ=tem;
    return tem;
}
```

2-1. 在某节点前插入

```
template < typename T >
node<T> * insertBefore( node<T> *p, T const & e )
{
    _size++;
    node<T> *tem=header;
    while( (tem=tem->succ)!=p );
    return tem->insertAft(e);
}
```

2-2. 在某节点后插入

```
template < typename T >
node<T> * list<T>::insertAfter( node<T> *p, T const & e)
{
    _size++;
    return p->insertAft(e);
}
```

2-3. 插入表头

```
template < typename T >
node<T> * list<T>::insertFirst( T const & e )
{
    _size++;
    return header->insertAft(e);
}
```

2-4. 插入表尾

```
template < typename T >
node<T> * list<T>::insertLast( T const & e )
{
    _size++;
    node<T> *tem=header;
    while( tem->succ )
        tem=tem->succ;
    return tem->insertAft(e);
}
```

### 3. 查找元素

3-1. 无序链表查找

3-1-0. 整链表查找

```
template < typename T >
node<T> * list<T>::find( T const& e ) const
{
    return find( e, header, ,_size );
}
```

3-1-1. 部分链表查找

```
template <typename T>
node<T> * list<T>::find( T const& e, node<T>* p , int n ) const
{  ( p , p+n ]
    while(n--)
        if( e==( p=p->succ )->data )
            return p;
    return NULL;
}
```

3-2. 有序链表查找

返回不大于e的最后一个元素，方便后续插入操作

此处与顺序表不同，哪怕是在有序链表中查找，和无序相比时间复杂度同为O(n）,原因在于链表的循秩访问问题，详见4. 而在顺序表中，有序表可以借由二分等算法将复杂度降低

```
template <typename T>
node<T> * list<T>::search( T const & e, node<T> * p, int n ) const
{  [ p , p+n )
    node<T> *tem=header;  // tem作为p的直接前驱
    while( tem->succ != p )
        tem=tem->succ;
    while( n-- &&p )
    {
        if( p->data>e )
            break;
        p=p->succ;
        tem=tem->succ;
    }
    return tem;
}
```

### 4. 循秩访问单个节点

方式：重载()运算符

```
template <typename T>
T & list<T>::operator [] (int r) const
{
    node<T> *p=header;
    while( ( p=p->succ ) && r-- );
    return p->data;
}
```

此处为链表和顺序表的区别，哪怕重载了[]运算符，但链表的循秩访问和顺序表的循秩访问**本质不同**。

链表的为披着循秩访问外皮的按位置访问。

而顺序表 V[i] = V + i*s (s为单个元素大小)

时间复杂度为O(n)，哪怕在双向链表中可以通过判断r和_size/2的大小选择从前或后访问来节省一些时间，但复杂度依旧为O(n/2)=O(n)。

### 5. 删除单个节点

```
template <typename T>
T list<T>::remove( node<T> * p )
{
    T tem=p->data;
    node<T> *pre=header;
    while( pre->succ!=p )
        pre=pre->succ;
    pre->succ=p->succ;
    delete p;
    _size--;
    return tem;
}
```

### 6. 清空链表

```
template <typename T>
int list<T>::clear()
{
    int old_size=_size;
    if( header->succ==NULL )
        return 0;
    while( header->succ )
        remove( header->succ );
    // 在remove操作里有_size自减的操作，因此这里不再重新将_size置零
    return old_size;
}
```

### 10. 排序

10-1 归并排序

归并排序思路：将表不断二分再重新合并，每次合并从两个已有序的表头选取较小的加入目标表尾，若其中一个表为空，则将另一表整体接入目标表尾

该算法思路：同一链表内的归并排序：现将p1段和p2段截断，以px3保留非排序区域的头节点。以p1的直接前继做新表头，每次绑定两表头较小的节点，直至p1、p2段其中一段为空，然后连接上另一非空链表剩余部分，最后将非排序区域的尾部接上

```
template <typename T>
void list<T>::merge ( node<T> * &p1, int n1, node<T> *&p2, int n2 ) //使用引用绑定节点防止丢失
{
    node<T> *head;
    node<T> *pree=header;
    while( pree->succ!=p1 )
        pree=pree->succ;
    head=pree;
    node<T> *px1=p1,*px2=p2,*px3;
    for(int i=0; i<n1-1 ; i++)
        px1=px1->succ;
    for(int i=0; i<n2-1 ; i++)
        px2=px2->succ;
    px3=px2->succ;
    px1->succ=px2->succ=NULL;
    px1=p1;
    while( n1&&n2 )
    {
        if( px1->data<=p2->data )
        {
            pree->succ=px1;
            pree=pree->succ;
            px1=px1->succ;
            n1--;
        }
        else
        {
            pree->succ=p2;
            pree=pree->succ;
            p2=p2->succ;
            n2--;
        }
    }
    if( n1 )
        pree->succ=px1;
    if( n2 )
        pree->succ=p2;
    while( pree->succ )
        pree=pree->succ;
    pree->succ=px3;
    p1=head->succ;  // 最后设置好排序后的头结点
    return;
}

template <typename T>
void list<T>::mergeSort( node<T> *&p, int n )
{ // [ p, p+n ]
    if(n<2) return;
    int mid=n/2;
    node<T> *tem=p;
    for(int i=0; i<mid; i++)  tem=tem->succ;
    mergeSort( p, mid );
    mergeSort( tem, n-mid );
    merge( p,mid, tem , n-mid );
    return;
}

template <typename T>
void list<T>::sort( )
{
    mergeSort( header->succ , _size );
}
```

### 11. 去重

11-1. 无序链表

常规思路：将链表分为已去重和未去重区域，每次取未去重区域第一个元素，在已去重区域查找是否有相同数值的节点，若存在，则任意删除其一，若不存在，则将该节点加入已去重区域。 时间复杂度 O(n2)

其它思路：将链表数据以数组保存，在数组中剔除重复数据后再赋值并删除多余节点。时间复杂度O(nlogn)

```
template <typename T>
int list<T>::deduplicate()
{
    if( _size<2 ) // 少于2个节点无需去重
        return;
    node<T> *p=header->succ->succ; //第一个节点必定已去重
    node<T> *pre=header->succ;
    int old_size=_size;  // 保存_size，方便返回删除数目
    int r=1;  // 已去重数目
    while( p )
    {
        if( find( p->data , header , r )!=NULL )
        {    // 借用find接口
            pre->succ=p->succ;
            delete p;
            p=pre->succ;
            continue;
        }
        r++;
        p=p->succ;
        pre=pre->succ;
    }
    return old_size-_size;
}
```

11-2. 有序链表

思路：一指针A从表头开始，每次检测相邻节点，若相同则删除后节点，若不同，A向后移动直至表尾。 时间复杂度O(n)

```
template <typename T>
int list<T>::uniquify()
{
    if(_size<2)
        return 0;
    int old_size=_size;
    node<T> *p=header->succ,*q;
    while( NULL != ( q = p->succ ) ) // q作为p的直接后继，检测是否重复
        (p->data == q->data) ? remove(q) : p=q; // 若重复则删除直接后继，若不重复则p向后一步
    return old_size-_size;
}
```

### 12. 反转

思路：固定目前的首节点，即header->succ，每次将它的直接后继重新绑定至header的直接后继。

```
template <typename T>
void list<T>::reverse()
{
    if(_size<2)
        return;
    node<T> *fir=header->succ;
    node<T> *tem;
    while( fir->succ )
    {
        tem=fir->succ;
        fir->succ=tem->succ;
        tem->succ=header->succ;
        header->succ=tem;
    }
    return;
}
```

# **双向链表**

双链表和单链表类似，但不同在于有**首末**两个哨兵，每个节点有**前驱**和**后缀**指针，可以省去从头访问到某节点的前驱的时间。

代码实现与单向链表相差不大，只是需要额外注意需要多维护pred指针和trailer哨兵。

**基本定义**

```
template <typename T>
struct ListNode
{
    T data;
    ListNode<T> * pred;
    ListNode<T> * succ;
};
template <typename T>
struct List
{
    int _size ;
    ListNode<T> * header,*trailer;
};
```

**带接口定义**

```
template <typename T>
struct ListNode
    //列表节点模板类（以双向链表形式实现）
{
    T data;
    ListNode<T> * pred;
    ListNode<T> * succ;

    ListNode () {}
    ListNode ( T const & e, ListNode<T> * p = NULL, ListNode<T> *a = NULL)
        :data(e),pred(p),succ(a){}

    ListNode<T> * insertAsPred ( T const & e );
    ListNode<T> * insertAsSucc ( T const & e );
};

template <typename T>
struct List
{
private:

    int _size ;
    ListNode<T> * header,*trailer;

protected:
    void init ();
    void merge ( ListNode<T> * p1, int n1 , List<T>&L,  ListNode<T> *p2, int n2); //[p1,p1+n) 本串p1开始n1个节点与 L串p2开始n2个节点归并排序
    void mergeSort ( ListNode<T> *p, int n );      // 对从p开始连续的n个节点归并排序
    void selectionSort ( ListNode<T> *p, int n );   // [ p,p+n ) 选择排序
    void insertionSort ( ListNode<T> *p, int n );   // 对从p开始连续的n个节点插入排序

public:
    List() { init(); }
    List( ListNode<T> *p , int n );     // 从p开始复制n个节点
    List( T *p , int n );
    ~List();

    // 只读接口
    int size() const { return _size; }
    bool empty() { return _size==0; }
    T & operator [] ( int r ) const;    // 实现寻秩访问
    ListNode<T> * first()const
        {return header->succ;}
    ListNode<T> * last() const
        {return trailer->pred;}
    ListNode<T> * find( T const&e ) const
        {return find(e,_size,trailer);}
    ListNode<T> * find( T const& e, int n, ListNode<T>* p ) const; // (无序)从节点p向前n个节点（不含p）内找e
    ListNode<T> * search( T const & e , int n , ListNode<T> * p ) const; // (有序) 不大于e的最后一个
    ListNode<T> * selectMax ( ListNode<T> *p , int n )const ;
    void show() const;

    // 可写接口
    ListNode<T> * insertBefore( ListNode<T> *p, T const & e)
        { _size++; return p->insertAsPred(e); }
    ListNode<T> * insertAfter( ListNode<T> *p, T const & e)
        { _size++; return p->insertAsSucc(e); }
    ListNode<T> * insertLast( T const & e )
        { _size++; return trailer->insertAsPred(e);}
    ListNode<T> * insertFirst( T const & e )
        { _size++; return header->insertAsSucc(e);}
    T remove( ListNode<T> *p );
    int deduplicate (); // 无序列表去重
    int uniquify ();    // 有序列表去重
    int clear ();
    void copyNodes ( ListNode<T> *p , int n );     // 从p开始复制n个节点
    void sort( ListNode<T> *p, int n, int mod );
};
```

## 1. 排序

1-1. 插入排序

思路：将链表分为前后已排序和未排序区域，每次取未排序区域首节点在已排序区域选位置插入。

```
template <typename T>
void List<T>::insertionSort( ListNode<T> * p , int n )
{
    for(int r=0; r<n; r++)
    {
        insertAfter( search( p->data , r , p ) , p->data );
        p=p->succ;
        remove( p->pred );
    }
}

template <typename T>
T List<T>::remove( ListNode<T> * p )
{
    T tem=p->data;
    p->pred->succ=p->succ;
    p->succ->pred=p->pred;
    delete p;
    _size--;
    return tem;
}

// node类的准备工作
template <typename T>
ListNode<T>* ListNode<T>::insertAsSucc( T const & e )
{
    ListNode<T> *a=new ListNode<T>( e, this, succ );
    succ->pred=a;
    succ=a;
    return a;
}
```

1-2. 选择排序

思路：每次从未排序区域中选择最大元素并移入未排序区域最后，即已排序区域之首。

```
template <typename T>
void List<T>::selectionSort( ListNode<T> *p, int n )
{
    if(n<2) return;
    ListNode<T> *head=p->pred , *tail=p;
    for( int i=0; i<n; i++ )
        tail=tail->succ;
    while( 1<n )
    {
        insertBefore( tail, remove( selectMax( head->succ , n ) ) );
        tail=tail->pred;
        n--;
    }
    return;
}
template <typename T>
ListNode<T> * List<T>::selectMax ( ListNode<T> *p , int n )const
{
    ListNode<T> *tem=p;
    while(n--&&p)
    {
        if(p->data>=tem->data)
            tem=p;
        p=p->succ;
    }
    return tem;
}

// node类的准备工作
template <typename T>
ListNode<T>* ListNode<T>::insertAsPred( T const & e )
{
    ListNode<T> *a=new ListNode<T>( e, pred, this );
    pred->succ=a;
    pred=a;
    return a;
}
```

# **常见的链表问题**

最常用方法：

> 1. **快慢指针法**
> 2. **数组过渡法**
> 3. **先断后接、先接后断法**

\1. 单链表反转

> 两个搬运工（指针），一个在header，一个在原链表的第一个节点，每次将第一个节点的直接后继断开接到header直接后继，直至原第一个节点后继为NULL。

\2. 找单链表倒数第n个元素、或中间元素

> 快慢指针法。
>
> 找倒数第n个元素，两指针同时走并相差n步，若前面指针到末尾，则后面指针则是倒数第n个节点。（细节：链表长度是否大于等于n）
>
> 找中间元素，慢指针每走一步 快指针则走两步，快指针到尾时慢指针在中间。
>
> 找中间元素，在允许遍历两遍的情况下也可以先遍历一遍求出链表长度，再走第二遍。

\3. 删除无头单链表的某个节点

> 题意为不知header，但需要删除目前current指针指向的节点。
>
> 思路：删除节点需要找到该节点的前驱，既然无法知道current节点的前驱，那就改为删除current的直接后继。将直接后继的数据复制给current，然后删除current的直接后继。

\4. 在无头单链表某节点前增加节点

> 思路和3类似，先将要创建的节点连接在current节点的直接后继，然后交换两节点的数据。

\5. 判断单链表是否有环(可能是部分环，非循环单链表)

> 快慢指针法
>
> 一个步长为2，一个步长为1，若步长为2的跑到末尾，则没有环，若在跑到末尾之前(可能没有末尾)，两指针相遇，则说明有环。

\6. 判断两单链表是否相交

> 首先明确：两单链表一旦相交，自交点之后的节点将完全相同！
>
> 时间复杂度均为O( len1+len2 )
>
> 1. 法1：数组过渡法，将两个单链表每个节点的地址记录于两个数组，看两个数组是否有相同元素
> 2. 法2：先接后断法，将第一个链表首尾相接，然后用法5对第二个链表进行判断，若有环则相交，若无环则不相交，注意完成判断后要将第一个链表断开。
> 3. 法3：直接法：直接判断两链表末节点是否相同

\7. 已知两单链表相交，求相交点

> 快慢指针法
>
> 先求两链表长度len1,len2，快指针先走abs(len1-len2)步，而后两指针同时前走并判断是否相等，若在某时刻之前不等，而在此时相等，则此时为交点

\8. 求两递增单链表AB差集A-B(元素在A而不在B)

> 归并的思想
>
> 每次取表头元素进行大小判断，若A’<’B则通过，若A==B则剃除，若A>B则将B指针向后移动直至A’<’B或A==B

# **完整实现代码**

[单向链表的完整实现](https://github.com/zedom1/DSA/blob/master/list/single_list.cpp)

[双向链表的完整实现](https://github.com/zedom1/DSA/blob/master/list/double_list.cpp)