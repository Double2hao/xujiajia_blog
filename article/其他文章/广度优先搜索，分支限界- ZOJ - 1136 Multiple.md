#广度优先搜索，分支限界- ZOJ - 1136 Multiple
# Description

a program that, given a natural number N between 0 and 4999 (inclusively), and M distinct decimal digits X1,X2…XM (at least one), finds the smallest strictly positive multiple of N that has no other digits besides X1,X2…XM (if such a multiple exists).

The input file has several data sets separated by an empty line, each data set having the following format:

On the first line - the number N On the second line - the number M On the following M lines - the digits X1,X2…XM.

For each data set, the program should write to standard output on a single line the multiple, if such a multiple exists, and 0 otherwise.

An example of input and output:

## Input

22 3 7 0 1

2 1 1

## Output

110 0

# 思路

1、题目一看就是搜索：把数的组合从小到大一个个尝试，如果可以是N的倍数，那么就输出。

2、要把组合的数从小到大输出就需要先把数升序排序，然后从小到大依次尝试，很容易就想到要用**广度优先搜索（队列）**。

3、如果仅仅是暴力，那么此题也就仅仅是一个bfs的demo了，但事实不是。此题还需要考虑**整数越界**的问题，如果N=4999，组合成的数是N的上几万倍这时候可能就会越界。（据说使用 long long也可以，但是笔者还是考虑了下）所以要使用string来存储组合成的整数。那么如何能够把问题缩小化呢？很简单就是存储余数。 **打个比方：** N=3，t1=10，t2=4。 (t1**10+t2)%N=2 //这是我们一般的处理方式 t1%N=1，(1**10+t2)%N=2 //这是能够缩小数量级的处理方式

4、**num[5010]<strong>的设置绝对是代码中最巧妙的一点，主要作用是**对余数的判断</strong>。num[i]=1时表示余数为i的情况已经出现过，num[i]=0时表示余数为i的情况还没有出现过。在代码中主要有两个作用： 一、**减少重复遍历**。如果出现同样的余数，代表在此位置之前已经出现过了这样的余数，在此位置之前的定然是更小的数。此题最终结果是找出最小的倍数，所以就没有必要帮相同的情况放进队列了。（如果之前更小的数不能得出结果，余数相同，更大的数也必然得不出结果） 二、**避免死循环**。如果N=2，M=2，X1=3，X2=5。很容易可以明白，这种情况是怎么都得不到结果的。如果有没有对重复的余数的判断，那么会一直有数入站，队列永远不会空，while也永远不会结束。但是如果有了余数这个限制条件，当所有余数都已经出现过，就不会再有数入队列，因此当队列遍历完的时候，就可以return “0”，表示找不到这个数。

## 代码:

```
#include&lt;cstdio&gt;
#include&lt;iostream&gt;
#include&lt;cstring&gt;
#include&lt;string&gt;
#include&lt;queue&gt;
#include&lt;algorithm&gt;
using namespace std;

int n,m;//n为要除以的数，m表示有m个数字组合
int p[12];//p用来存储m个数
int num[5010];//num[i]=1时表示余数为i的情况已经出现过，num[i]=0时表示余数为i的情况还没有出现过

struct node
{
    string s;//表示当前数字的string格式
    int res;//当然数字除以n的余数
};

string bfs()
{
    queue&lt;node&gt; q;
    for(int i=0; i&lt;m; i++)//先把把每一个数都放进队列,都只有个位数
    {
        node ss;
        ss.s+=char('0'+p[i]);//把int转化为数字
        ss.res=p[i];

        if(ss.s=="0")//数字为0的情况没意义，直接continue
            continue;
        q.push(ss);
    }

    while(!q.empty())
    {
        node s=q.front();
        q.pop();

        for(int i=0; i&lt;m; i++)
        {
            node t;
            t.s=s.s+char('0'+p[i]);
            t.res=(10*s.res+p[i])%n;

            if(num[t.res]==1)//如果此余数已经出现过,就不放入队列了
                continue;
            else
            {
                num[t.res]=1;//如果上面的情况都不是，表示此数没有出现过相同的余数，则进行处理
                q.push(t);
            }

            if(t.res==0)//如果余数等于0，表示已经找到
                return t.s;

        }
    }
    return "0";
}

int main()
{
    while(scanf("%d%d",&amp;n,&amp;m)!=EOF)
    {
        if(n==0)//除数等于0没有意义，直接输出0
        {
            printf("0\n");
            continue;
        }

        memset(num,0,sizeof(num));
        for(int i=0; i&lt;m; i++)
            scanf("%d",p+i);

        sort(p,p+m);
        cout&lt;&lt;bfs()&lt;&lt;endl;
    }
    return 0;
}


```