#今年暑假不AC详细解析（贪心）
>  
 今年暑假不AC 
   
 Problem Description 
 “今年暑假不AC？” 
 “是的。” 
 “那你干什么呢？” 
 “看世界杯呀，笨蛋！” 
 “@#$%^&amp;*%...”
  
 确实如此，世界杯来了，球迷的节日也来了，估计很多ACMer也会抛开电脑，奔向电视了。 
 作为球迷，一定想看尽量多的完整的比赛，当然，作为新时代的好青年，你一定还会看一些其它的节目，比如新闻联播（永远不要忘记关心国家大事）、非常6+7、超级女生，以及王小丫的《开心辞典》等等，假设你已经知道了所有你喜欢看的电视节目的转播时间表，你会合理安排吗？（目标是能看尽量多的完整节目） 
  
  
 Input 
 输入数据包含多个测试实例，每个测试实例的第一行只有一个整数n(n&lt;=100)，表示你喜欢看的节目的总数，然后是n行数据，每行包括两个数据Ti_s,Ti_e (1&lt;=i&lt;=n)，分别表示第i个节目的开始和结束时间，为了简化问题，每个时间都用一个正整数表示。n=0表示输入结束，不做处理。 
  
  
 Output 
 对于每个测试实例，输出能完整看到的电视节目的个数，每个测试实例的输出占一行。 
  
  
 Sample Input 
 12 1 3 3 4 0 7 3 8 15 19 15 20 10 15 8 18 6 12 5 10 4 14 2 9 0
  
  
  
 Sample Output 
 5 
  
   
  
  
   
  
  
  主要思路：（贪心法）
  
  
  首先是用结束的时间对各个数据排序，然后从第一个节目开始，用每个节目的结束时间比较后面一个节目的开始时间，如果后一个的开始时间大于等于前一个的结束时间，那么可以看的节目就+1。（不太清楚的读者可以自己用个数据带进去将整个算法经历一遍）
  
  
   
  
  
  
  <pre><code class="language-cpp">#include &lt;stdio.h&gt;
#include &lt;algorithm&gt;
#include &lt;iostream&gt;
using namespace std;

struct pro
{
    int b,e;
}s[111];

bool cmp(pro a,pro b)
{
    return a.e&lt;b.e;
}

int main()
{
    int i,j,n;
    while(scanf("%d",&amp;n)&amp;&amp;n)
    {
        for(i=0;i&lt;n;i++)
            scanf("%d %d",&amp;s[i].b,&amp;s[i].e);
        sort(s,s+n,cmp);//用结束时间给所有数据排序
        //对sort排序还是不太理解的读者可以百度一下
        //sort比较简单，但是很方便

        int x=s[0].e;//第一个节目的结束时间
        int cnt=1;//可以看的节目数
        for(i=1;i&lt;n;i++)
        {
            if(s[i].b&gt;=x)//此处用后一个节目的开始时间比较后一个节目的结束时间
            {
                x=s[i].e;
                cnt++;
            }
        }
        printf("%d\n",cnt);
    }
    return 0;
}</code></pre>
   
   
  
  
   
  


> 

> 

> 

> 

> 
