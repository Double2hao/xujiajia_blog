#二分法-派（Pie, NWERC 2006, LA 3635）
# 题目：

有F+1个人来分N个圆形派，每个人得到的必须是一整块派，而不是几块拼在一起，且面积要相同。求每个人最多能得到多大面积的派（不必是圆形）。

输入的第一行为数据组数T。每组数据的第一行为两个整数N和F （ 1 ≤ N ， F ≤ 10 000 ） ；第二行为 N 个整数 ri（1≤ri≤10 000），即各个派的半径。

对于每组数据，输出每人得到的派的面积的最大值，精确到10-3。

# 思路

1、题目涉及精确到10-3，可以朝二分法的算法上想。 2、此题关键是二分的条件。假设取得值小了，可以多分出的饼的个数大于等于F+1，那么此时就需要把饼设置大一些来尝试。假的取的值大了，导致可以分出的饼的个数小于&lt;F+1，那么就要把饼设置地小一些来尝试。 3、二分的结束条件就是当精确值达到10-3。

## 代码：

```
#include &lt;cstdio&gt;
#include &lt;cmath&gt;
double PI=acos(-1.0);

int n,f;
double R[10005];

double Find(double l,double r)
{
    if (r - l &lt;= 1e-4)
        return r;

    double mid = (l + r) / 2;

    int sum = 0;
    for (int i = 0; i &lt; n; i++)
        sum += R[i] / mid;

    //如果sum&gt;=f，说明饼太小了。
    //如果sum&lt;f，说明饼太大了，不够分。
    if (sum &gt;= f)
        return Find(mid,r);
    else
        return Find(l,mid);
}

int main()
{
    int t;
    scanf("%d",&amp;t);
    while (t--)
    {
        scanf("%d%d",&amp;n,&amp;f);
        f = f + 1;
        double sum = 0;
        for (int i = 0; i &lt; n; i++)
        {
            scanf("%lf",&amp;R[i]);
            R[i] = PI * R[i] * R[i];
            sum += R[i];
        }

        double ans = Find(0,sum/f);
        printf("%.4lf\n",ans);
    }
    return 0;
}


```