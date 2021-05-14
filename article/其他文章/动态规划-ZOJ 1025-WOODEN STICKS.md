#动态规划-ZOJ 1025-WOODEN STICKS
# 题目：

现有n根木棒，已知它们的长度和重量。要用一部木工机一根一根地加工这些木棒。该机器在加工过程中需要一定的准备时间，是用于清洗机器，调整工具和模版的。 木工机需要的准备时间如下： （1）第一根木棒需要1min的准备时间； （2）在加工了一根长为l ，重为w的木棒之后，接着加工一根长为l ‘(l ≤ l’ )，重为 w’ ( w≤w’)的木棒是不需要任何准备时间的，否则需要一分钟的准备时间。

给定n根木棒，找到最少的准备时间。例如现在有长和重分别为（4，9），（5，2），（2，1），（3，5）和（1，4）的五根木棒，那么所需准备时间最少为2min，顺序为（1，4），（3，5），（4，9），（2，1），（5，2）。

## 输入

输入有多组测试例。输入数据的第一行是测试例的个数T。 每个测试例两行： 第一行是一个整数n（1≤n≤5000），表示有多少根木棒；第二行包括n×2个整数，表示l1，w1，l2，w2，l3，w3，…，ln，wn，其中li和wi表示第i根木棒的长度和重量。 数据由一个或多个空格分隔。

## 输出

输出是以分钟为单位的最少准备时间，一行一个。

## 输入样例

3 5 4 9 5 2 2 1 3 5 1 4 3 2 2 1 1 2 2 3 1 3 2 2 3 1

## 输出样例

2 1 3

# 分析：

1、本题关键之一是如何**把一个二维的问题转化成一个一维的问题**。采用的方法是：先根据木棍长度把木棍进行升序排序，然后直接对重量进行处理就可以了。 2、第二个关键就是如何在排好序的木棍中通过重量找出能分成几组。思路如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911416020.png " alt="这里写图片描述">

## 代码：

```
#include &lt;stdio.h&gt;
#include &lt;algorithm&gt;
#include &lt;string.h&gt;
using namespace std;

struct node
{
    int l;
    int w;
};

int cmp(struct node x,struct node y)
{
    if(x.l==y.l)
        return x.w&lt;y.w;
    else
        return x.l&lt;y.l;
}


int Find(struct node t[],int n)
{
    //数组b表示木棒分组的序号
    int b[5003];
    memset(b, 0, sizeof(b));
    b[0]=1;

    int k;
    for (int i=1; i&lt;n; i++)
    {
        //计算第i个木棒的的分组序号
        k=0;
        for (int j=0; j&lt;i; j++)
            if (t[i].w&lt;t[j].w &amp;&amp; k&lt;b[j])
                k=b[j];    //找出尽量大的k
        b[i]=k+1;
    }

    //查找最大的分组序号（数组b中的最大值）
    int Max=0;
    for (int i=0; i&lt;n; i++)
        if (b[i]&gt;Max) Max=b[i];
        
    return Max;
}

int main(void)
{
    int ncase;
    struct node t[5005];

    scanf("%d",&amp;ncase);
    while(ncase--)
    {
        int n;
        scanf("%d",&amp;n);
        for (int i=0; i&lt;n; i++)
            scanf("%d %d",&amp;t[i].l,&amp;t[i].w);

        sort(t,t+n,cmp);
        printf("%d\n",Find(t,n));
    }
    return 0;
}


```