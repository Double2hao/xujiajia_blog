#贪婪-zoj 1029--MOVING TABLES
# 题目：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1430.png" alt="这里写图片描述">

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1431.png" alt="这里写图片描述">

## 输入

输入数据有T组测试例，在第一行给出测试例个数T。 每个测试例的第一行是一个整数N（1≤N≤200)，表示要搬运办公桌的次数。接下来N行，每行两个正整数s和t，表示一张桌子，是从房间号码s移到到房间号码t。

## 输出

每组输入都有一行输出数据，为一整数T，表示完成任务所花费的 最小时间。

## 输入样例

3 4 10 20 30 40 50 60 70 80 2 1 3 2 200 3 10 100 20 80 30 50

## 输出样例

10 20 30

# 分析:

1、使用贪婪法，首先将二维问题转化成一维问题。我们不难发现：1与2其实表示同一个位置，399与400其实表示同一个位置。创建一个room对象，存放开始搬桌子的位置s和结束搬桌子的位置e，并且强行设置s小于e，如果s大于e，则两者交换位置。

2、根据e对所有的room对象升序排序。

3、只需要判断前一个的e与后一个的s的大小就可以知道是否能同时进行。

## 代码：

```
#include &lt;iostream&gt;
#include &lt;algorithm&gt;
#include &lt;string.h&gt;
#include &lt;cstdio&gt;
using namespace std;

struct room
{
    int s;
    int e;
} r[410];

int change( int num )//1与2其实表示同一个位置，399与400其实表示同一个位置
{
    if( num%2==0 )
        return num/2;
    else
        return num/2+1;
}

int cmp( room a,room b )
{
    return a.s&lt;b.s;
}

int main()
{
    int T;
    scanf( "%d",&amp;T );
    while( T-- )
    {
        int n;
        scanf( "%d",&amp;n );
        for(int  i=0; i&lt;n; i++ )//将二维问题转化成一维问题，并让所有的s&lt;e，方便贪婪算法
        {
            scanf( "%d%d",&amp;r[i].s,&amp;r[i].e );
            r[i].s=change( r[i].s );
            r[i].e=change( r[i].e );
            if( r[i].s&gt;r[i].e )
            {
                int temp=r[i].s;
                r[i].s=r[i].e;
                r[i].e=temp;
            }
        }

        sort( r,r+n,cmp );

        int num=0;
        int flag[410];//用来存储各种情况是否计算过
        memset( flag,0,sizeof(flag) );

        while( 1 )
        {
            int isEnd=1;//只有在flag[1~n]中的值都为1时，isEnd才会不等于0；
            int last=-1;//用来存储前一个的结束时间
            for( int i=0; i&lt;n; i++ )
            {
                if( flag[i]==0)//只会处理没有遍历过的情况
                {
                    isEnd=0;//如果flag[i]=0，就表示还没有遍历结束
                    if(r[i].s&gt;last)//这种情况表示可以一起通过
                    {
                        flag[i]=1;
                        last=r[i].e;
                    }
                }
            }
            if(isEnd)
                break;
            num++;
        }

        printf( "%d\n",num*10 );
    }
    return 0;
}


```