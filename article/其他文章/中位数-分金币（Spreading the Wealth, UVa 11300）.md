#中位数-分金币（Spreading the Wealth, UVa 11300）
# 分金币（Spreading the Wealth, UVa 11300）

圆桌旁坐着n个人，每人有一定数量的金币，金币总数能被n整除。每个人可以给他左右相邻的人一些金币，最终使得每个人的金币数目相等。你的任务是求出被转手的金币数量的最小值。比如，n=4，且4个人的金币数量分别为1,2,5,4时，只需转移4枚金币（第3个人给第2个人两枚金币，第2个人和第4个人分别给第1个人1枚金币）即可实现每人手中的金币数目相等。

## 【输入格式】

输入包含多组数据。每组数据第一行为整数n（n≤1 000 000），以下n行每行为一个整数，按逆时针顺序给出每个人拥有的金币数。输入结束标志为文件结束符（EOF）。

## 【输出格式】

对于每组数据，输出被转手金币数量的最小值。输入保证这个值在64位无符号整数范围内。

## 【样例输入】

3 100 100 100 4 1 2 5 4

## 【样例输出】

0 4

# 思路

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910966970.png " alt="这里写图片描述"> <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910967871.png " alt="这里写图片描述">

# 程序理解

个人认为程序中比较需要理解的就是c数组： 由上方思路中我们可知： M=A1-x2+x1 M=A2-x3+x2 M=A3-x4+x3

而c数组，此处就是代表交换的金币的个数,正数表示其他人给i金币，负数表示i给其他人金币，如下： x2=x1-C1; x3=x1-C2; x4=X1-C3;

最后由上方思路中，我们可以推得： C0 = 0 C1 = A1 - M = C0 + A1 - M C2 = A1 - M + A2 - M = C1 + A2 - M … Cn = An-1 - M + An - M = Cn-1 + An - M

规律：Cn = Cn-1 + An - M

代码如下

```
#include&lt;cstdio&gt;
#include&lt;algorithm&gt;
using namespace std;

const long maxN=1000000;
//a[i]表示i处初始金币的个数
//c[i]表示i处交换的金币的个数,正数表示其他人给i金币，负数表示i给其他人金币
long a[maxN],c[maxN];

int main()
{
    int n;
    while(scanf("%ld",&amp;n)!=EOF)
    {
        long sum=0,arv;//arv表示所有金币的平均数

        for(int i=0; i&lt;n; i++)
        {
            scanf("%ld",&amp;a[i]);
            sum+=a[i];
        }
        arv=sum/n;

        c[0]=0;
        for(int i=1; i&lt;n; i++)
            c[i]=c[i-1]+a[i]-arv;
        sort(c,c+n);

        long x1=c[n/2];//x1表示每个人交换的金币的中位数
        long ans=0;
        for(int i=0; i&lt;n; i++)
            ans+=abs(x1-c[i]);
        printf("%ld\n",ans);
    }
    return 0;
}


```