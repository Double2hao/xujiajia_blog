#动态规划-ZOJ 1 163 THE STAIRCASES
# 题目：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/430.png" alt="这里写图片描述">

# 分析：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/431.png" alt="这里写图片描述">

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/432.png" alt="这里写图片描述">

## 代码：

```
#include&lt;stdio.h&gt;
#include&lt;string.h&gt;

int main()
{
    long long f[501][501];//f[i, j]表示i块积木堆成最高一排不超过j块积木的方案数

    memset(f, 0, sizeof(f));
    for(int i=0; i&lt;501; i++)
        f[0][i] = 1;    //f[3, 3]=f[3 ,2]+f[0, 2],f[0,2]就表示由3块砖竖起来的这种情况

    for (int i = 1; i &lt; 501; i++)    //公式递推
        for (int j = 1; j &lt; 501; j++)
            if(i&gt;=j)
                f[i][j] = f[i][j - 1] + f[i - j][j - 1];
            else
                f[i][j] = f[i][j - 1];

    int n;
    while (scanf("%d", &amp;n), n)
        printf("%lld\n", f[n][n] - 1);
    return 0;
}


```