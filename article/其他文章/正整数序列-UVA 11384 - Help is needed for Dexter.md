#正整数序列-UVA 11384 - Help is needed for Dexter
# 题目：

给定正整数n，你的任务是用最少的操作次数把序列1, 2, …, n中的所有数都变成0。每次操作可从序列中选择一个或多个整数，同时减去一个相同的正整数。比如，1,2,3可以把2和3同时减小2，得到1,0,1。 输入包含多组数据。每组仅一行，为正整数n（n≤109）。输入结束标志为文件结束符（EOF）。 对于每组数据，输出最少操作次数。

# 思路：

找规律发现从中间开始分成两边，把大的那边同时减掉这样是最优的。设最少的步数为 f(n)，则答案为 f(n/2) + 1。

## 代码

```
#include &lt;stdio.h&gt;

int main()
{
    int n;
    while(scanf("%d", &amp;n) != EOF)
    {
        int ans = 0;
        for(; n &gt; 0; n /= 2)
            ans++;
        printf("%d\n", ans);
    }
    return 0;
}


```