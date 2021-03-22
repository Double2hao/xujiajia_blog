#和为s的两个数字
>  
 参考《剑指Offer》 


# 题目

输入一个递增排序的数组和一个数字s，在数组中查找两个数，是的它们的和正好是s。如果有多对数字的和等于s，输出任意一对即可。

# 解析

分别记录数组的最小的和最大的两个数，由于数组本身是递增的，即数组第一个数和最后一个数。  如果两个数的和大于sum，那么后面一个数就向前挪，如果两个数的和小于sum，那么前面一个数就向后挪。如果前后两个数挪到了同一个位置，就表示不存在这样一个和。

# 代码：

```
#include&lt;iostream&gt;
using namespace std;

int FindNumbersWithSum(int data[],int length,int sum)
{
    int i1=0;
    int i2=length-1;
    while(i1&lt;i2){
        long long s=data[i1]+data[i2];
        if(s==sum){
            cout&lt;&lt;data[i1]&lt;&lt;'+'&lt;&lt;data[i2]&lt;&lt;'='&lt;&lt;sum&lt;&lt;endl;
            return 1;
        }
        else if(s&lt;sum)
            i1++;
        else
            i2--;
    }
    return 0;
}

int main()
{
    int length=6;
    int data[length]={<!-- -->1,2,4,7,11,16};
    int sum=15;
    int isFound=FindNumbersWithSum(data,length,sum);
    if(isFound==0)
        cout&lt;&lt;"找不到能组成该sum的两个数"&lt;&lt;endl;

    return 0;
}

```