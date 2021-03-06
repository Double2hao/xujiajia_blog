#二分法-长城守卫（Beijing Guards, CERC 2004, LA 3177）
# 题目：

有n个人围成一个圈，其中第i个人想要ri个不同的礼物。相邻的两个人可以聊天，炫耀自己的礼物。如果两个相邻的人拥有同一种礼物，则双方都会很不高兴。问：一共需要多少种礼物才能满足所有人的需要？假设每种礼物有无穷多个，不相邻的两个人不会一起聊天，所以即使拿到相同的礼物也没关系。 比如，一共有5个人，每个人都要一个礼物，则至少要3种礼物。如果把这3种礼物编号为1, 2, 3，则5个人拿到的礼物应分别是：1,2,1,2,3。如果每个人要两个礼物，则至少要5种礼物，且5个人拿到的礼物集合应该是：{1,2},{3,4},{1,5},{2,3},{4,5}。

## 输 入 格 式

输入包含多组数据 。 每组数据的第一行为一个整数 n（1≤n≤100 000）；以下n行按照圈上的顺序描述每个人的需求，其中每行为一个整数ri（1≤ri≤100 000），表示第i个人想要ri个不同的礼物。输入结束标志为n=0。

## 输 出 格 式

对于每组数据，输出所需礼物的种类数。

## Sample Input

3 4 2 2 5 2 2 2 2 2 5 1 1 1 1 1 0

## Sample Output

8 5 3

# 分析

1、首先先测试几组数据，我们不难发现，当n为偶数时，问题其实很简单，如下 s1=r1+r2; s2=r2+r3; …… s(n-1)=r(n-1)+r(n); s(n)=r(n)+r1; 在n个s中找到最大的一个即为最终结果。

2、在此基础上，我们可以把问题分解成**奇数和偶数两种情况**。那么**奇数**的时候又如何求呢？

首先打个比方： 此时有5个礼物1、2、3、4、5，有5个人的r[i]都为2。 **第一个人：**用想，直接选1、2两个礼物。 **第五个人：**我们希望的情况当然是是4、5两个礼物，这样才尽量不会与第一个人取相同的礼物。 **第二个人：**我们会选择3、4两个礼物，为什么呢？因为我们不仅要考虑与1不同，我们还要保证礼物更少。

由此我们可以知道，在n为奇数的情况下，第i个人的取礼物的规则为： **i为偶数时：尽量向左取；** **i为奇数时：尽量向右取；**

看到这里如果读者有疑问，那么你就是找到关键点了：向右取需要在礼物个数已知的情况下才能执行，而在真正的问题中，**有多少个礼物不是定死的，是需要我们求的**。 针对这个问题，我们就可以采取**二分枚举**的办法了，每一次都把礼物定死，求一下试试看：**如果可行**，那么就查看一下礼物是否能取得更少；**如果不可行**，那么就增加礼物个数。

## 代码：

```
#include &lt;cstdio&gt;
#include &lt;cstring&gt;
#include&lt;algorithm&gt;
using namespace std;

int r[100010];
int L[100010],R[100010];
int n;

int Try(int s)
{
    memset(R,0,sizeof R);
    memset(L,0,sizeof L);
    L[1]=r[1];
    for(int i=2; i&lt;=n; i++)
    {
        if(i%2==0)//为偶数时，尽量向左取
        {
            L[i]=min(r[i],r[1]-L[i-1]);
            R[i]=r[i]-L[i];
        }
        else//为奇数时，尽量向右取
        {
            R[i]=min(r[i],(s-r[1])-R[i-1]);
            L[i]=r[i]-R[i];
        }
    }
    return (L[n]==0);
}

int main()
{
    while(scanf("%d",&amp;n)!=EOF,n)
    {

        int t=0,s=0;
        for(int i=1; i&lt;=n; i++)
        {
            scanf("%d",&amp;r[i]);
            s=max(r[i]+r[i-1],s);
            t+=r[i];
        }
        s=max(s,r[n]+r[1]);
        if(n%2==0)
        {
            printf("%d\n",s);
            continue;
        }

        int ans=t;
        while(s&lt;=t)//s=max(r[i]+r[i-1],s),t+=r[i];让s和t分别作为二分的头和尾
        {
            int mid=(s+t)/2;
            int flag=Try(mid);//如果可行，就返回1
            if(flag)//即使可行，也可能选的礼物过多
            {
                if(ans&gt;mid)
                    ans=mid;
                if(t!=mid)
                    t=mid;
                else
                    break;
            }
            else//如果不行，必然是选的礼物过少不够分
            {
                if(s!=mid+1)
                    s=mid+1;
                else
                    break;
            }
        }
        printf("%d\n",ans);
    }
    return 0;
}


```