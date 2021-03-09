#蚂蚁-UVA 10881 - Piotr's Ants
# 题目：

一根长度为L厘米的木棍上有n只蚂蚁，每只蚂蚁要么朝左爬，要么朝右爬，速度为1厘米/秒。当两只蚂蚁相撞时，二者同时掉头（掉头时间忽略不计）。给出每只蚂蚁的初始位置和朝向，计算T秒之后每只蚂蚁的位置。 输入的第一行为数据组数。每组数据的第一行为3个正整数L, T,n（0≤n≤10 000）；以下n行每行描述一只蚂蚁的初始位置，其中，整数x为蚂蚁距离木棍左端的距离（单位：厘米），字母表示初始朝向（L表示朝左，R表示朝右）。 对于每组数据，输出n行，按输入顺序输出每只蚂蚁的位置和朝向（Turning表示正在碰撞）。在第T秒之前已经掉下木棍的蚂蚁（正好爬到木棍边缘的不算）输出Fell off。

# 思路

1、由于蚂蚁之间不会互相穿过，所以蚂蚁的**相对顺序**是不变的。 2、计算时，为了让问题简单化，就假设蚂蚁会互相穿过。 这点必须要思考明白：比如此刻有A、B两只蚂蚁，A坐标为2向右行走，B坐标为4向左行走，2秒后A还是在2，B还是在4。我们完全可以认为**是B代替了A向右行走的动作**。 3、然后就得到了两组很重要的数据，一是**最初所有蚂蚁的相对位置顺序**，二是**最终所有蚂蚁的位置和方向**。两者都根据坐标排序，然后一一对应就可以知晓每个蚂蚁最终的情况了。

## 代码

```
#include &lt;cstdio&gt;
#include &lt;algorithm&gt;
using namespace std;

const int maxN = 1e5+10;
struct ant
{
    int id;//表示该蚂蚁从左至右是第几只蚂蚁
    int pos;//记录该蚂蚁当前坐标位置
    int dir;//记录该蚂蚁的方向，left表示-1，right表示1，turning表示0,。
};
ant before[maxN], after[maxN];
int index[maxN];

bool cmp(const ant &amp;a, const ant &amp;b)
{
    return a.pos &lt; b.pos;
}

int main()
{
    int icase, l, t, n, w = 1;
    scanf("%d", &amp;icase);
    while(icase--)
    {
        scanf("%d%d%d", &amp;l, &amp;t, &amp;n);//l为杆子长度，t表示输出t秒后的状态，n表示数据个数
        for(int i = 0; i &lt; n; ++i)
        {
            int p, d;
            char c;
            scanf("%d %c", &amp;p, &amp;c);
            d = ('L'==c ? -1 : 1);
            before[i] = (ant){i, p, d};
            after[i]  = (ant){0, p+t*d, d};
        }

        sort(before, before+n, cmp);
        for(int i = 0; i &lt; n; ++i)
            index[before[i].id] = i;
        sort(after, after+n, cmp);
        for(int i = 0; i &lt; n-1; ++i)
        {
            if(after[i].pos == after[i+1].pos)
                after[i].dir = after[i+1].dir = 0;
        }

        printf("Case #%d:\n", w++);
        for(int i = 0; i &lt; n; ++i)
        {
            if(after[index[i]].pos&gt;l ||after[index[i]].pos&lt;0)
                printf("Fell off\n");

            else if(0 == after[index[i]].dir)
                printf("%d Turning\n", after[index[i]].pos);

            else if(1 == after[index[i]].dir)
                printf("%d R\n", after[index[i]].pos);

            else if(-1 == after[index[i]].dir)
                printf("%d L\n", after[index[i]].pos);
        }
        printf("\n");
    }
    return 0;
}


```
