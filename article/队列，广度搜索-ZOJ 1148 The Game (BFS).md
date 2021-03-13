#队列，广度搜索-ZOJ 1148 The Game (BFS)
# 大致题意：

有一个BOARD上有很多很多相同大小的卡片。有的位置是空的，有的位置放了卡片。问卡片a和卡片b能不能用横纵直线段连起来，如果能求连线最少由多少段组成，连线可以暂时离开BOARD。

# 理解

1、题目是想要得到连线最少由多少段组成，我们可以理解为连线的转弯个数+1。此处千万不要与路径最短等同！ 2、此题有点像寻找类似“连连看”的路径，但是并不等同，连连看只允许转2个弯，也就是只能有3段。但是此处是无论转多少次都可以。

# 思路

##1、数据结构采用**二维数组**。 由于连线可以暂时离开BOARD，表示BOARD之外也可以走。直接给二维数组的行和列各自+2就行了，最上面、下面、左面、右面都放空格‘ ’，表示可以走。也就是说，实际我们使用的数组的大小为(m+2)*(n+2) ##2、关于路径的搜索，我们采用**广度搜索**。 也就是从初始点开始（上下左右四个方向搜索），我们先搜索完只有1个线段的，然后再搜索只有2个线段的，再搜索只有3个线段的，以此类推。这样就能够保证，一旦搜索到了，返回的数据一定是线段数最少的。

搜索1个线段的时候比较容易思考，就是只要遍历完初始点的四个方向。 **那么搜索2个线段如何实现呢？** 当我们在搜索1个线段的时候，我们**把搜索的每一个点都存起来**，然后我们按照顺序对其中的点执行一样的操作——搜索那个点的上下左右四个方向（只有一个线段的情况）。这样，相对于初始点而言，就是只有2个线段的情况了。 ##3、那么如何**存储这些点**呢？很容易就可以想到，用**队列**。 ##4、题目中还有容易让人想到的问题就是，如何保证不进入死循环，让判断效率更高。 这点其实在真正实现的时候，是自然而然就解决的问题。

我们可以再使用一个二维数组来存储每个点的走过的情况（在笔者demo中直接是存放该点属于第几层的线段），如果一个点已经走过了，那么在判断的时候就直接过滤。

读者可能疑问，**直接过滤不会导致漏掉一些路径吗**？ 让我们再次回顾一下我们遍历的过程，我们是先判断只有1个线段的路径是否可行，再判断2个线段，3个线段，以此类推。 假设，让我在判断2个线段中的时候，发现有一个点已经走过了，那么必然是1个线段的时候，就已经放入队列了。那么，倘若我再次放入队列，我最终所输出的线段个数，必然1个线段的时候放入队列的那个值大1，那么也就没有必要进行判断了。

## 代码：

```
#include&lt;cstdio&gt;
#include&lt;iostream&gt;
#include&lt;queue&gt;
#include&lt;cstring&gt;
using namespace std;

//node用来记录遍历的x和y，放入队列用
struct node
{
    int x,y;
};

//fx与fy方便朝四个方向遍历
int fx[]= {-1,1,0,0};
int fy[]= {0,0,-1,1};

char a[80][80];    //a用来存放用户输入的X和空格
int step[80][80];  //step是重点，表示记录的该点属于第几段（转弯个数+1），为-1的时候表示没有走过。
int n,m;    //n为行，m为列

int bfs(int sx,int sy,int ex,int ey)
{
    //创建队列，并把第一个点的位置放入队列
    queue&lt;node&gt; q;
    node s;
    s.x = sx;
    s.y = sy;
    q.push(s);

    step[sx][sy] = 0;
    a[sx][sy] = ' ';
    a[ex][ey] = ' ';

    //cur表示队列中当前位置，t主要用来将新的node放入队列
    node Front,cur,t;
    //只要队列不为空，就会一直遍历
    //有两种可能，一是到达了终点，跳出while。另一种是队列遍历结束，无法到达终点。
    while(!q.empty())
    {
        Front = q.front();//从第一个点开始查看，先记录第一个点的位置

        if(Front.x == ex &amp;&amp; Front.y == ey)
            return step[ex][ey];

        //查看这个点的四个方向
        for(int i=0; i&lt;4; i++)
        {
            cur = Front;
            t.x = cur.x+fx[i];
            t.y = cur.y+fy[i];

            //这个while表示会朝同一个方向一致查看，直到这个方向不能走了
            //x与y不能越界，该位置上不能有卡片，并且该位置没有走过
            while(t.x&gt;=0 &amp;&amp; t.x&lt;=n+1 &amp;&amp; t.y&gt;=0 &amp;&amp; t.y&lt;=m+1 &amp;&amp; a[t.x][t.y]==' ' &amp;&amp; step[t.x][t.y]==-1)
            {
                //在Front的步数基础上+1,，表示这个
                step[t.x][t.y] = step[Front.x][Front.y]+1;
                q.push(t);

                cur = t;
                t.x = cur.x+fx[i];
                t.y = cur.y+fy[i];
            }
        }
        //由于这个点已经遍历完了，就把它弹出队列
        q.pop();
    }
    return -1;
}

int main()
{
    int count1=1;
    while(scanf("%d%d",&amp;m,&amp;n)!=EOF)  //n行m列
    {
        if(m==0 &amp;&amp; n==0)
            break;
        else
            printf("Board #%d:\n",count1++);;
        getchar();

        //用来放卡片的位置实际是1~m列和1~n行。
        //0与m列，0与n行由于在题目中都时可用走的，所以都用来放空格‘ ’。
        for(int i=1; i&lt;=n; i++)
            gets(a[i]+1);
        for(int i=0; i&lt;=m+1; i++)
        {
            a[0][i]=' ';
            a[n+1][i]=' ';
            a[i][0]=' ';
            a[i][m+1]=' ';
        }


        int count2=1;
        int sx,sy,ex,ey;//sx,sy为起始点位置，ex，ey为终点位置
        while(scanf("%d%d%d%d",&amp;sy,&amp;sx,&amp;ey,&amp;ex)!=EOF)
        {
            if(sx==0 &amp;&amp; sy==0 &amp;&amp; ex==0 &amp;&amp; ey==0)
                break;
            else
                printf("Pair %d: ",count2++);

            //step表示记录的该点的转弯的个数,先设置为-1表示没有走过。
            memset(step,-1,sizeof(step));

            int ans = bfs(sx,sy,ex,ey);

            if(ans == -1)
                printf("impossible.\n");
            else
                printf("%d segments.\n",ans);

            //如果之前已经走通，那么起始点与
            a[sx][sy] = a[ex][ey] ='X';
        }
        printf("\n");
    }
    return 0;
}


```