#动态规划-最大子矩阵和（ZOJ 1074 TO THE MAX ）
# 题目：

有一个包含正数和负数的二维数组。一个子矩阵是指在该二维数组里，任意相邻的下标是1×1或更大的子数组。一个子矩阵的和是指该子矩阵中所有元素的和。 本题中，把具有最大和的子矩阵称为最大子矩阵。例如，如下数组的最大子矩阵位于左下角，其和为15。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040189360.png" alt="这里写图片描述">

## 输入

是N×N个整数的数组。第一行是一个正整数N，表示二维方阵的大小。接下来是N2个整数（由空格和换行隔开）。该数组的N2个整数，是以行序给出的。先是第一行的数，由左到右；然后是第二的数，由左到右，等等。N可能达到100，数组元素的范围是[-127，127]。 ##输出： 输出是最大子矩阵的和。

# 思路：

1、此问题是“最大字段和”问题的推广。 2、代码具体过程如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040190481.png" alt="这里写图片描述">

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040192762.png" alt="这里写图片描述">

## 代码：

```
#include&lt;cstdio&gt;
#include&lt;cstring&gt;

int n;
int a[110][110];
int b[110];

int main()
{
    while(scanf("%d",&amp;n)!=EOF)
    {
        for(int i=0; i&lt;n; i++)
            for(int j=0; j&lt;n; j++)
                scanf("%d",&amp;a[i][j]);

        int Max = -32767;
        for(int i=0; i&lt;n; i++)
        {
            //数组b表示i~j行，对应列元素的和
            //将二维动态规划问题转化为一维动态规划问题
            memset(b, 0, sizeof(b));
            for(int j=i; j&lt;n; j++)
            {
                //下面是针对数组b求最大子段和的动态规划算法
                int sum=0;
                for(int k=0; k&lt;n; k++)
                {
                    b[k] += a[j][k];
                    sum += b[k];
                    if(sum&lt;0) sum = b[k];
                    if(sum&gt;Max) Max = sum;
                }
            }
        }
        printf("%d\n",Max);
    }

    return 0;
}


```