#贪心-突击战（Commando War, UVa 11729）
# 题目：

你有n个部下，每个部下需要完成一项任务。第i个部下需要你花Bi分钟交待任务，然后他会立刻独立地、无间断地执行Ji分钟后完成任务。你需要选择交待任务的顺序，使得所有任务尽早执行完毕（即最后一个执行完的任务应尽早结束）。注意，不能同时给两个部下交待任务，但部下们可以同时执行他们各自的任务。

输入包含多组数据，每组数据的第一行为部下的个数N（ 1 ≤ N ≤ 1 000 ） ；以下 N 行每行两个正整数 B 和 J（1≤B≤10 000，1≤J≤10 000），即交待任务的时间和执行任务的时间。输入结束标志为N=0。

对于每组数据，输出所有任务完成的最短时间。

## 样例输入

3 2 5 3 2 2 1 3 3 3 4 4 5 5 0 ##样例输出 Case 1: 8 Case 2: 15

# 分析

1、题目比较简单，使用贪心算法。此题中，我们想要的情况定然是，希望有尽可能多的人在同时工作，希望一个人工作的同时能够在给另一个人布置任务。所以只要按照工作的时间从大到小排序即可。 2、定义结构体，存放各个部下布置任务的时间和完成任务的时间。

## 代码：

```
#include &lt;stdio.h&gt;
#include &lt;algorithm&gt;
using namespace std;

struct NODE
{
    int give;
    int work;
} war[1005];

int cmp(NODE x,NODE y)
{
    if(x.work &gt; y.work)
        return 1;
    return 0;
}

int main()
{
    int n,cas = 1;
    while(scanf("%d",&amp;n),n)
    {
        for(int i = 0; i&lt;n; i++)
            scanf("%d%d",&amp;war[i].give,&amp;war[i].work);
        sort(war,war+n,cmp);//按照工作的时间排序

        int ans = 0,sum = 0;
        for(int i = 0; i&lt;n; i++)
        {
            sum+=war[i].give;
            if(sum+war[i].work&gt;ans)//可能会  先布置任务的人还没结束，后布置任务的人已经结束
                ans=sum+war[i].work;
        }
        printf("Case %d: %d\n",cas++,ans);
    }

    return 0;
}


```
