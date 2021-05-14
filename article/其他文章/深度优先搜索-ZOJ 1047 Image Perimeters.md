#深度优先搜索-ZOJ 1047 Image Perimeters
# 大致题意

给你一张图，图上有‘X’和‘.’。‘X’可以八个方向连通，这些连起来的‘X’成为一个块。问：输入一个坐标，这个坐标所在的块的周长是多少。 如图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910799850.png " alt="这里写图片描述">

# 分析

1、使用上下左右和对角线方向的深度优先搜索。 2、尝试几组数据，不难发现，在块中的’X’。‘X’靠近边界，或者’X’的周围是’.’,那么周长就会+1。 如此，其实我们也可以简化问题，只需要在输入的数据上下左右都加上一行（列）’.’。这样我们就只需判断’X’是否周围是’.’，如果是，那么周长就+1。 3、问题就可以分成两种了，**一是查找块的大小**，也就是将可以遍历到的’X’都放入块，**二是计算块的周长**，就是遍历块中X，查看它们周围是否是’.’，如果是，周长就+1。

## 代码

```
#include &lt;cstdio&gt;
#include &lt;cstring&gt;

int dir[4][2] = {<!-- -->{-1,0},{1,0},{0,-1},{0,1}};//用作上下左右遍历
int diagonal[4][2] = {<!-- -->{-1,-1},{-1,1},{1,-1},{1,1}};//用作对角线方向的遍历


int m,n;
char a[30][30];//存储输入的数
int perimeter;  //周长
int visit[30][30];//记录某个点是否遍历过

void dfs(int x,int y)
{
    int i,newx,newy;
    visit[x][y] = 1;   //标记此处已经遍历过

    for(i=0; i&lt;4; i++)//上下左右搜
    {
        newx = x + dir[i][0];
        newy = y + dir[i][1];
        if(a[newx][newy] == 'X' &amp;&amp; visit[newx][newy] == 0)
            dfs(newx,newy);  //如果遍历到的是X，并且没有访问过，那么久继续递归
        else if(a[newx][newy] == '.')
            perimeter++;       //如果遍历到的是'.'，那么周长就+1
    }

    for(i=0; i&lt;4; i++)//对角线搜
    {
        newx = x + diagonal[i][0];
        newy = y + diagonal[i][1];
        if(a[newx][newy] == 'X' &amp;&amp; visit[newx][newy] == 0)
            dfs(newx,newy);
        //对角线就不需要碰到'.'让周长+1了
    }
}

int main()
{
    int i,j;
    int xx,yy;
    while(scanf("%d%d%d%d",&amp;m,&amp;n,&amp;xx,&amp;yy),(m||n||xx||yy))
    {
        perimeter = 0;
        memset(a,'.',sizeof(a));//在外面围一圈.便于判断
        memset(visit,0,sizeof(visit));
        for(i=1; i&lt;=m; i++)
        {
            getchar();//把回车吃掉。。。
            for(j=1; j&lt;=n; j++)
                scanf("%c",&amp;a[i][j]);
        }

        dfs(xx,yy);
        printf("%d\n",perimeter);
    }
    return 0;
}


```