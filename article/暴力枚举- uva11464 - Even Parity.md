#暴力枚举- uva11464 - Even Parity
# 题目：

给你一个n×n的01矩阵（每个元素非0即1），你的任务是把尽量少的0变成1，使得每个元素的上、下、左、右的元素（如果存在的话）之和均为偶数。比如，如（a）所示的矩阵至少要把3个0变成1，最终如图（b）所示，才能保证其为偶数矩阵。 输入的第一行为数据组数T（T≤30）。每组数据的第一行为正整数n（1≤n≤15）；接下来的n行每行包含n个非0即1的整数，相邻整数间用一个空格隔开。 对于每组数据，输出被改变的元素的最小个数。如果无解，应输出-1。

# 分析：

1、显然枚举每一位是不现实的，注意到n只有15，那么可以枚举第一行，接下来根据第一行就可以计算出第二行，根据第二行又能计算出第三行，以此类推。

## 代码：

```
#include &lt;cstdio&gt;
#define M 20
int n, Min, a[M][M], b[M][M];
int check(int x, int y)//将其上左右三面的值相加
{
    int sum = 0;
    if(x-1&gt;=0) sum += b[x-1][y];
    if(y-1&gt;=0) sum += b[x][y-1];
    if(y+1&lt;n) sum += b[x][y+1];

    return sum%2;//如果是偶数就返回0，奇数就返回1
}

void dfs(int cur)
{
    //利用深度优先遍历枚举第一行
    if(cur!=n)
    {
        b[0][cur] = 1;
        dfs(cur+1);
        b[0][cur] = 0;
        dfs(cur+1);
    }
    else//枚举完之后开始递推下面每一行的情况
    {
        for(int i = 1; i &lt; n; i++)
            for(int j = 0; j &lt; n; j++)
                b[i][j] = check(i-1,j);
        int cou = 0;
        for(int i = 0; i &lt; n; i++)
            for(int j = 0; j &lt; n; j++)
                if(a[i][j]==1&amp;&amp;b[i][j]==0)
                    return;//题目只能把0变1，不能把1变0，所以直接结束。
                else if(a[i][j]==0&amp;&amp;b[i][j]==1)
                    cou++;//只有当出现原来为0，枚举出的结果中为1的情况，cou才+1
        if(Min&gt;cou)
            Min = cou;
        return;
    }


}
int main ()
{
    int cas, t = 0;
    scanf("%d",&amp;cas);
    while(t++&lt;cas)
    {
        scanf("%d",&amp;n);
        for(int i = 0; i &lt; n; i++)
            for(int j = 0; j &lt; n; j++)
                scanf("%d",&amp;a[i][j]);

        Min = 1e9;
        dfs(0);//开始枚举；
        printf("Case %d: ",t);
        if(Min==1e9)
            printf("-1\n");
        else
            printf("%d\n",Min);
    }
    return 0;
}


```