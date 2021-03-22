#图的搜索+回溯-Seeding（zoj 2100 ）
# Seeding

####Time Limit: 2 Seconds Memory Limit: 65536 KB It is spring time and farmers have to plant seeds in the field. Tom has a nice field, which is a rectangle with n * m squares. There are big stones in some of the squares.

Tom has a seeding-machine. At the beginning, the machine lies in the top left corner of the field. After the machine finishes one square, Tom drives it into an adjacent square, and continues seeding. In order to protect the machine, Tom will not drive it into a square that contains stones. It is not allowed to drive the machine into a square that been seeded before, either.

Tom wants to seed all the squares that do not contain stones. Is it possible?

## Input

The first line of each test case contains two integers n and m that denote the size of the field. (1 &lt; n, m &lt; 7) The next n lines give the field, each of which contains m characters. ‘S’ is a square with stones, and ‘.’ is a square without stones.

Input is terminated with two 0’s. This case is not to be processed.

## Output

For each test case, print “YES” if Tom can make it, or “NO” otherwise.

## Sample Input

4 4 .S… .S… … … 4 4 … …S … …S 0 0

## Sample Output

YES NO

# 大致题意

以左上角的位置为起点，n*m大的地，播种的时候不能走已经走过的路。其中’S’为石头，’.'需要播种的地。

# 思路

1、使用搜索+回溯的思想 2、**搜索：**上下左右四个方向进行搜索。为了方便判断，走过的路就设置为’S’，即让它变为石头。（石头和走过的路都不能再走） 3、**回溯：**由于在搜索的时候，走过的路被设置为了’S’（石头）。那么倘若在搜索的路径选择错误了，走不通的时候就需要回过头去，**把走错的路再次变为‘.’**，这样才能够让之后的搜索继续。

## 代码：

```
#include&lt;stdio.h&gt;

//fx与fy便于做四个方向的搜索
static int fx[4]= {1,-1,0,0};
static int fy[4]= {0,0,1,-1};

char s[10][10];
int m,n;//m、n分别为矩阵的行和列的个数
int sum;//sum为石头和走过的路径的个数之和
int ok;//如果路径存在ok就为1

void dfs(int x,int y)
{
    s[x][y]='S';   //走过的路径就不能走了，相当于是石头，如果路径错误，之后要回溯
    sum++;

    if(sum ==m*n )  //如果石头和走过的路径个数之和等于m*n，说明可以走完
    {
        ok=1;
        return;
    }

    for(int i=0; i&lt;4; i++)
    {
        int xx = x + fx[i];
        int yy = y + fy[i];
        if(xx&gt;=0&amp;&amp;xx&lt;m &amp;&amp; yy&gt;=0&amp;&amp;yy&lt;n &amp;&amp;  s[xx][yy]=='.')
        {
            dfs(xx,yy);
            s[xx][yy]='.';  //此处为回溯，即路径不通时，会把走过的路径重新置为'.'
            sum--;
        }
    }
}

int main()
{
    while(scanf("%d%d",&amp;m,&amp;n),m||n)
    {
        for(int i=0; i&lt;m; i++)
            scanf("%s",s[i]);

        //首先找出有多少个石头
        sum = 0;
        for(int i=0; i&lt;m; i++)
            for(int j=0; j&lt;n; j++)
                if(s[i][j] == 'S')
                    sum++;

        //此处执行递归
        ok = 0;
        dfs(0,0);

        if(ok)
            printf("YES\n");
        else
            printf("NO\n");

    }
    return 0;
}


```