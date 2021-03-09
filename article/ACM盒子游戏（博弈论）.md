#ACM盒子游戏（博弈论）
##  盒子游戏

```
#include&lt;stdio.h&gt;
int main()
{
    int n;
    while(scanf("%d",&amp;n)!=EOF&amp;&amp;n)
    {
        n++;
        while(n%2==0)
            n/=2;
        if(n==1)
            printf("Bob\n");
        else
            printf("Alice\n");
    }
    return 0;
}
```


