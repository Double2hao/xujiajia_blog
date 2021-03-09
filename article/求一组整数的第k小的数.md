#求一组整数的第k小的数
题目：

求一组整数的第k小的数 

输入：

5 2

10 60 4 2 7

输出：

4

 

解题思路：

1、使用快速排序的思路解题。

2、与快排不同的是，快排会在排序结束才会结束。此题也是从小到大排序，但是，只要找到第k小的数，就跳出递归。

3、此题有一些小地方可以优化容易忽略，比如判断k是否会大于n，判断用i和j扫描时是否会越界，交换时是否会自己与自己交换等等。

 

快排基本思想：



 1．先从数列中取出一个数作为基准数。

 2．分区过程，将比这个数大的数全放到它的右边，小于或等于它的数全放到它的左边。

 3．再对左右区间重复第二步，直到各区间只有一个数。

 了解快排可看此篇文章：

  

 代码：

 

```
#include&lt;iostream&gt;

using namespace std;

//返回数组中的第k个最小元素的启动函数，注意会破坏原数组
int findMin(int *x, int x_size, int k);
//实现查找数组中第K个最小元的功能函数
int kMin(int *x, int left, int right, int k);
void Swap(int *x,int i,int j);

int main()
{
    int n,k;
    cin&gt;&gt;n;
    cin&gt;&gt;k;

    int x[n];
    for(int i=0; i&lt;n; i++)
    {
        cin&gt;&gt;x[i];
    }
    cout&lt;&lt;findMin(x,n,k);

}

int findMin(int *x, int n, int k)
{
    //判断k的值是否过大，即超过数组的大小
    //若是则返回第0个元素，主要是为了防止无效的递归
    if(n &lt; k)
    {
        cout&lt;&lt;"k cannot &gt; n!"&lt;&lt;endl;
        return 0;
    }
    return kMin(x, 0, n-1, k);
}

int kMin(int *x, int left, int right, int k)
{
    //取数组第一个元素为参照
    int centre = x[left];
    int i = left+1;
    int j = right;
    while(true)
    {
        //从前向后扫描，找到第一个小于参考值的值
        //此时要检查下标，防止i&gt;right

        //从后向前扫描，防止数组越界
        //此时不用检查下标，因为第一个数为参照值
        while(x[i] &lt; centre &amp;&amp; i&lt;=right)
            ++i;
        while(x[j] &gt; centre)
            --j;

        //如果没有完成，就交换
        if(i &lt; j)
            Swap(x,i,j );
        else
            break;
    }
    //把参考数值放在正确的位置
    Swap(x,left,i);

    //由于我们是从小到大排序的
    //如果此时centre的位置刚好为k，则centre为第k个最小的数
    //如果此时centre的位置比k前,递归地在其右边寻找
     //如果此时centre的位置比k后,递归地在其左边寻找
    if(i+1 == k)
        return x[i];
    else if(i+1 &lt; k)
        kMin(x, i+1, right, k);
    else
        kMin(x, left, i-1, k);
}

void Swap(int *x,int i,int j)
{
    //如果i和j相等的话就不进行交换
    if(i==j)
        return;

    int temp=x[i];
    x[i]=x[j];
    x[j]=temp;
}

```


