#通过考试（概率dp）
# 题目描述

小明同学要参加一场考试，考试一共有n道题目，小明必须做对至少60%的题目才能通过考试。考试结束后，小明估算出每题做对的概率，p1,p2,…,pn。你能帮他算出他通过考试的概率吗？

# 输入

输入第一行一个数n（1&lt;=n&lt;=100），表示题目的个数。第二行n个整数，p1,p2,…,pn。表示小明有pi%的概率做对第i题。（0&lt;=pi&lt;=100）

# 输出

小明通过考试的概率，最后结果四舍五入，保留小数点后五位。

# 样例输入

```
4
50 50 50 50
```

# 样例输出

```
0.31250
```

# 时间限制

C/C++语言：1000MS其它语言：3000MS

# 内存限制

C/C++语言：65536KB其它语言：589824KB

# 解析

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039239160.png" alt="这里写图片描述" title="">

# 代码

```
#include &lt;bits/stdc++.h&gt;
#define maxn 109
using namespace std;
int n,a[maxn];
double dp[maxn][maxn];
int main(){
    scanf("%d",&amp;n);
    for(int i=1;i&lt;=n;i++)
        scanf("%d",&amp;a[i]);
    dp[0][0]=1;
    for(int i=1;i&lt;=n;i++){
        dp[i][0]=dp[i-1][0]*(100.0-a[i])/100;
        for(int j=1;j&lt;=i;j++)
            dp[i][j]=dp[i-1][j]*(100.0-a[i])/100+dp[i-1][j-1]*1.0*a[i]/100;
    }
    int low=(3*n+4)/5;
    double ans=0;
    for(int i=low;i&lt;=n;i++)
        ans+=dp[n][i];
    printf("%.5f\n",ans);
    //system("pause");
    return 0;
}
```