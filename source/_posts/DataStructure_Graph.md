---
layout: post
title: "数据结构笔记（四）：图"
date: 2017-08-19
comments: true
tags: 
	- 学习笔记
	- c++
	- 数据结构
---


## **概念**

什么是图(Graph)：
表示多对多的关系
包含
一组顶点：通常用V(Vertex)表示顶点集合
一组边：通常用E(Edge)表示边的集合
边是顶点对
无向边 (v,w)$\in$E ， 其中 v,w $\in$ V
有向边 表示从v指向w的边
不考虑重边和自回路

<!-- more -->

此前学过的树、线性序列，均可视作图的**特例**。
在线性序列中，只有直接前驱和后继有邻接关系
在树中，只有父节点和子节点之间定义邻接关系

![序列、树、图](http://ot1c7ttzm.bkt.clouddn.com/image/170819/FhHfljELG8.png?imageslim)

**分类：**
1.**无向图(undigraph)**：所有邻接顶点之间次序无所谓 (u为v好友，则v也必定为u好友)
2.**有向图(digraph)**：所有邻接顶点之间有次序(u为v好友，v不一定为u好友) u->v 即 (u,v) u为尾(tail)，v为头(head)
3.**混合图**：邻接关系中有的与次序有关，有的与次序无关
4.**网络(network)**：边有权重

以有向图为根本学习，因为无向图可以变为邻接顶点之间双向连接的有向图，混合图也可以化成有向图。

![分类图](http://ot1c7ttzm.bkt.clouddn.com/image/170819/20c7c6lK0G.png?imageslim)

概念：

- **路径 **：

  定义：顶点按依次邻接的关系连成的序列
  简单路径：不含重复节点的路径
  一般路径：可能含重复节点
  环路：起点和终点重合

  - **彼此邻接**：
    彼此间存在边的两个顶点

  - **连通**：
    若v到w存在一条路径，则称v和w是连通的
    强连通：

    ```
    有向图中v和w存在双向路径

    ```

    **连通分量**：

    ```
    无向图的**极大**连通子图
        极大顶点数：再加1个顶点就不连通
        极大边数：包含子图中所有顶点相连的所有边
    **强连通分量**：
        有向图的极大强连通子图

    ```

    **连通图**：

    ```
    图中任意两顶点均连通
    **强连通图**：
        有向图中所有顶点都是强连通

    ```

  - **有向无环图**(directed acyclic graph DAG)：
    图为有向图且不包含任何环路

  - **欧拉环路**：
    经过所有边且恰好只经过所有边一次的环路

  - **哈密尔顿环路**：
    经过所有节点且只经过所有点一次

  - **简单图**：
    所有节点均不含自环的图

## 抽象数据类型定义

名称： 图 Graph
数据对象集：G(V,E)由一个非空的有限顶点集合V和一个有限边集合E组成
操作集：

```
- Graph Create();
    建立并返回空图
- Graph InsertVertex(Graph G, Vertex v);
    把顶点V插入G
- Graph InsertEdge(Graph G, Edge e);
    把顶点E插入G    
- void DFS(Graph G , Vertex v);
    从v出发深度优先遍历G
- void BFS(Graph G , Vertex v);
    从v出发广度优先遍历G
- void ShortestPath(Graph G , Vertex v, int Dist[]);
    计算图G中顶点v到任意其它顶点的最短距离
- void MST(Graph G);
    计算图G的最小生成树

```

## 表示

### 邻接矩阵

邻接矩阵：G[N][N]——N个顶点从0到N-1编号

```
- G[i][j]=1 <vi,vj>是边
- G[i][j]=0 <vi,vj>不是边

```

优点

```
- 方便、直观
- 方便检查任意一对顶点间是否有边
- 方便找任一顶点所有邻接点
- 方便计算任一顶点的度
    入度：指向该点的边数
    出度：从该点发出的边数

```

缺点

```
- 浪费空间，存稀疏图（顶点多，边很少的图）时有大量无效元素
- 浪费时间，统计稀疏图中共多少条边

```

基本定义与建图：

```
typedef char Vertex;
typedef int Edge;
#define MaxSize 1000
#define INF 0x7fffffff
enum GraphType{ DG, UG, DN, UN };
/* 有向图，无向图，有向网图，无向网图 */

struct Graph
{
	Vertex vertex[MaxSize];      // 顶点表
	Edge edge[MaxSize][MaxSize]; // 邻接矩阵，边表
	int n , e;                   // 顶点总数和边总数
	GraphType type;
};

void CreateGraph ( Graph * g)
{
	g->type = UN;
	int i,j,w;
	scanf("%d%d",&g->n,&g->e);   // 顶点数和边数
	for( i=0; i<g->n; i++ )
		scanf("%c",&(g->vertex[i]));
	for( i=0; i<g->n ; i++ )     // 初始化
		for( j=0; j<g->n ; j++ )
			g->edge[i][j]=INF;
	for( int k=0; k<g->e ; k++ )
	{
		scanf("%d%d%d",&i,&j,&w);  // 输入e条边的权值
		g->edge[i][j]=w;
		g->edge[j][i]=w;
	}
}
```

### 邻接表

邻接表：G[N]为指针数组，对应矩阵每行一个链表，只存非0元素
优点：

```
- 方便找任一顶点的所有邻接点
- 节约稀疏图的空间
    需要N个头指针，2E个结点（每个节点至少两个域）
    E < n(n-1)/4 时省空间
- 方便计算无向图的度
    有向图只能计算出度，需要构造逆邻接表（存指向自己的边）

```

基本定义与实现

```
typedef char Vertex;
typedef int Edge;
#define MaxSize 1000
#define INF 0x7fffffff
enum GraphType{ DG, UG, DN, UN };
/* 有向图，无向图，有向网图，无向网图 */

struct ENode
{
	int adjv;    // 指向顶点的序号
	ENode * next;
	int weight;
};

struct VNode
{
	Vertex vertex;
	ENode *firstEdge;
};

struct Graph
{
	VNode nodeList[MaxSize];
	int n,e;
	GraphType type;
};

void CreateGraph( Graph *g )
{
	int i,j;
	ENode *edge;
	g->type = DG;
	scanf("%d%d",&(g->n),&(g->e));
	for( i=0 ; i<g->n ; i++)
	{
		scanf("%c",&(g->nodeList[i].vertex));
		g->nodeList[i].firstEdge=NULL;
	}
	for( int k=0; k<g->e ; k++ )
	{
		scanf("%d %d",&i,&j);
		edge = new ENode();
		edge->adjv = j;
		edge->next = g->nodeList[i].firstEdge;
		g->nodeList[i].firstEdge = edge;
	}
}
```

## 遍历

### 深度优先搜索 DFS (Depth First Search)

相当于树的先序遍历

N个顶点，E条边，时间复杂度：
用邻接表存储时，为O(N+E)
用邻接矩阵存储时，为O($N^2$);

伪代码:

```
void DFS( Vertex v )
{
	visited[v]=true;
	for( v的每个邻接点w )
		if(!visited[w])
			DFS(w);
}
```

### 广度优先搜索 BFS (Breadth First Search)

相当于树的层序遍历

用**队列**实现，把起始结点加入队列后
每次取队头节点，把该结点所有未访问的结点加入队尾，重复该操作直至队列为空，即所有结点都被访问过

N个顶点，E条边，时间复杂度和DFS相同：

```
- 用邻接表存储时，为O(N+E)
- 用邻接矩阵存储时，为O($N^2$);

```

伪代码：

```
void BFS( Vertex v )
{
	visited[v]=true;
	queue.enqueue(v);
	while(!q.empty())
	{
		v=q.dequeue();
		for( v 的每个邻接点w )
			if(!visited[w])
			{
				visited[w]=true;
				q.enqueue(w);
			}
	}

}
```

BFS适合找最优解
DFS适合找任意一解

## 最短路径

定义：
在网络中，求两个不同顶点之间所有路径中，边权值之和最小的一条路径

```
- 该路径就是两点之间的最短路径
- 第一个顶点为源点

```

分类：

```
1. 单源最短路：
    从某固定源点出发，求其到所有其他顶点的最短路径
    - 无权图
    - 有权图
2. 多源最短路：
    求任意两顶点之间的最短路径

```

### 无权图的单源最短路算法

按照**递增（非递减）**的顺序找出到各个顶点的最短路

伪代码

```
// dist[w] = s到w的最短距离
// dist[s] = 0
// path[w] = s到w路径的倒数第二个顶点

void　Unweighted( int s )
{
	queue.enqueue(s);
	while(!queue.empty())
	{
		v = queue.dequeue();
		for( v的每个邻接点w )
		{
			if( dist[w]==-1 )
			{
				dist[w]=dist[v]+1;
				path[w]=v;
				queue.enqueue(w);
			}
		}
	}

}
void print(int s , int w)
{
	if(path[w]==-1) return;
	while(path[w]!=s)
	{
		stack.push(path[w]);
		w=path[w];
	}
	while(!stack.empty())
		printf("%d\n",stack.pop());
}
```

时间复杂度： O( v + e )

### 有权图的单源最短路算法

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170919/7H93JDhHe3.JPG)

不考虑**负值圈(negative-cost cycle)**

按照递增顺序找到各个顶点的最短路： Dijkstra算法

#### Dijkstra:

```
- 令 S = { 源点S + 已经确定了最短路径的顶点vi }
- 对任一未收录的顶点v，定义dist[v]为s到v的最短路径长度，但该路径仅经过s中的顶点。即路径{s -> (vi$\in$s) -> v}的最小长度
- 路径按照递增（非递减）的顺序生成
    - 真正的最短路一定只经过s中的顶点
    - 每次从未收录的顶点中选一个dist最小的收录
    - 增加一个v进入s，可能影响w的最小值
        dist[w]=min(dist[w]+<v,w>);
    - 初始化为INF

```

伪代码

```
void Dijkstra( int s )
{
	while(1)
	{
		v = 未收录顶点中dist最小的
		if( v 不存在 )
			break;
		collected[v]=true;
		for( v 的每个顶点w )
			if( collected[w]==false )
				if( dist[v]+E<v,w> < dist[w])
				{
					dist[w]=dist[v]+E<v,w>;
					path[w]=v;
				}
	}
}
```

时间复杂度：取决于如何找到dist最小的节点

方法1： 直接扫描所有未收录节点 O(v)
O($v^2$+e) (稠密图效果好)
方法2： 把dist存入最小堆中 O(logv)
更新dist[w]的值： O(logv)
O(elogv) (稀疏图效果好)

### 多源最短路

方法1： 把单源最短路调用V遍 O( v3+e*v )

方法2： Floyd算法 O( v3 )

#### Floyd算法：

```
- D(k)[i][j] = 路径 { i -> {l<=k} -> j }的最小长度 (从i到j只经过编号<=k的顶点)
- D(0)[i][j]...D(v-1)[i][j]给出i到j的真正最短距离
- 初始化D为带权值的邻接矩阵，对角元为0
- 若i,j之间无直接边，则初始化为INF
- 若D(k-1)已完成，递推D(k):
    - 若k$\notin$最短路径，则 D(K)=D(K-1)
    - 若k$\in$最短路径,则该路径必定由两段最短路径组成：
        D(k)[i][j]= D(k-1)[i][k]+D(k-1)[k][j]

```

伪代码：

```
void Floyd()
{
	for( int i=0; i<n ; i++ )
		for( int j=0; j<n ; j++ )
		{	
			D[i][j]=G[i][j]
			path[i][j]=-1;
		}
	for( int k=0; k<n ; k++ )
		for( int i=0; i<n ; i++ )
			for( int j=0; j<n ; j++ )
				if(D[i][k]+D[k][j]<D[i][j])
				{	
					D[i][j]=D[i][k]+D[k][j];
					path[i][j]=k;
				}
}
void print( int i , int j )
{
	if(i==j) return;
	print(i,path[i][j]);
	printf(path[i][j]);
	print(path[i][j],j);
}
```

## 最小生成树 Minimum Spanning Tree

定义：

```
- 一棵树：
    - 无回路
    - V个顶点一定有V-1条边
- 生成树：
    - 包含所有顶点
    - V-1条边都包含在图里
    - 生成树中任加一条边都一定构成回路
- 最小：
    - 边的权重和最小

```

最小生成树存在 <=> 图连通

方法：**贪心**

约束条件：

```
1. 只能用图里的边
2. 只能正好用掉V-1条边
3. 不能有回路

```

### Prim算法

核心思想：让小树长大

思路：从起始点出发，每次选取到树的距离（权值）最小的节点并加入树中，重复操作直至所有节点都纳入树中

适用场景：稠密图

伪代码：

```
void Prim()
{
	MST = {s}  // 初始化，树中只有初始结点
	while(1)
	{
		v = 未收录结点中到树距离最小的
		if( v不存在 )
			break;
		把v收入MST中： dist[v]=0
		for( v的所有邻接点w )
			if( w 未被收录：dist[w]!=0 && E<v,w> < dist[w] )
			{
				dist[w]=E<v,w>;
				parent[w]=v;
			}
	}
	if( MST里节点数< v 个 )
		Error( 生成树不存在 ) // 说明图本身不连通
}
```

初始化： dist[v]= E (若s和v有直接边) 或 INF (s和v没有直接边)

不需要实际构造一棵树，用parent数组记录节点的父节点序号，初始化为-1

时间复杂度： O($v^2$)

### Kruskal算法

核心思想：森林合并成树

思路：把每个结点都看作单独的一棵树，每次从边中选取权值最小的(不构成回路)，并将边所连接的两棵树合并，重复操作直至收录了v-1条边

适用场景：稀疏图

伪代码：

```
void Kruskal (Graph G)
{
	MST = {}
	while( MST不到V-1条边 && E中还有边 )
	{
		E中选取权重最小的边E<v,w>;   // 用最小堆
		把E<v,w>从e中删除
		if( E<v,w> 不在MST中构成回路 )   // 并查集，判断v和w在不在同一棵树（集合）
			将E<v,w>加入MST；
		else
			无视E<v,w>
	}
	if( MST里节点数< v 个 )
		Error( 生成树不存在 ) // 说明图本身不连通
}
```

时间复杂度： O( E*logE )

## 拓扑排序

AOV （Activity On Vertex）网络

拓扑序：若v到w有一条路径，那么v一定排在w前面，满足该条件的顶点序列

拓扑排序: 获得一个拓扑序的过程为

AOV若有**合理**的拓扑序，则必定是有向无环图DAG(Directed Acyclic Graph) (合理：不存在环)

思路：扫描图中所有节点，选取所有入度为0的结点，记录或输出，并修改与这些节点相连的结点的入度，重复操作直至所有结点都被记录或输出

用途：拓扑排序，还可以用于检测有向图是否为DAG

伪代码：

```
void TopSort()
{
	for( 每个顶点v )
		if( Indegree[v]==0 )
			queue.enqueue(v);
	while(!queue.empty())
	{
		v=queue.dequeue();
		输出或记录v;
		cnt++;
		for( v的每个结点w )
			if( --Indegree[w]==0 )
				queue.enqueue(w);
	}
	if( cnt!=v )
	{
		Error(图中有回路)
		break;
	}
}
```

时间复杂度： O(v+e)

### 关键路径问题

AOE( Activity On Edge )网络

```
- 一般用于安排项目的工序

```

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170920/ed832gdd6G.JPG)

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170920/C4JeG3mb6d.JPG)

EarliestTime[0]=0
EarliestTime[j] = max($\in$E) {EarliestTime[i]+C}

![mark](http://ot1c7ttzm.bkt.clouddn.com/image/170920/K9ba44fdec.JPG)

LastTime[i]=min($\in$E) { LastTime[j]-C}

机动时间： D = LastTime[j]-EarlistTime[i]-C

关键路径：绝对不允许延误的活动组成的路径