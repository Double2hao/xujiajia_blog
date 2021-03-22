#工作安排（dfs深度优先搜索）
# 题目

>  
 现在有n位工程师和6项工作(编号为0至5)，现在给出每个人能够胜任的工作序号表(用一个字符串表示，比如：045，表示某位工程师能够胜任0号，4号，5号工作)。现在需要进行工作安排，每位工程师只能被安排到自己能够胜任的工作当中去，两位工程师不能安排到同一项工作当中去。如果两种工作安排中有一个人被安排在的工作序号不一样就被视为不同的工作安排，现在需要计算出有多少种不同工作安排计划。 


## 输入描述:

>  
 输入数据有n+1行：  第一行为工程师人数n(1 ≤ n ≤ 6)  接下来的n行，每行一个字符串表示第i(1 ≤ i ≤ n)个人能够胜任的工作(字符串不一定等长的) 


## 输出描述:

>  
 输出一个整数，表示有多少种不同的工作安排方案 


## 输入例子:

>  
 6  012345  012345  012345  012345  012345  012345 


## 输出例子:

>  
 720 


# 分析

1、题目要求是要计算出有多少种不同的工作安排，说白了就是遍历。  2、介于题目中有特别说明n与工作类型的数目都很小，所以可以使用深度优先搜索。  3、一般做算法题，除非像这道题特别说明搜索的范围很小，不然不会使用深度优先搜索。原因是遍历每一种情况时间复杂度和空间复杂度都很高。比如01背包问题其实也可以用DFS，但是由于复杂度过高，所以我们一般采用动态规划。

# 代码：

```

import java.util.Scanner;

public class Main {

    private static int n;
    private static int count=0;
    private static boolean[] f=new boolean[6];
    private static void dfs(int[][] p,int[] length,int i) {
        if(i==n){
            count++;
            return;
        }
        for(int j=0;j&lt;length[i];j++){
            if(f[p[i][j]]==false){
                f[p[i][j]]=true;
                dfs(p,length,i+1);
                f[p[i][j]]=false;
            }
        }   

    }

    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);

        n=scan.nextInt();
        int[][] p=new int[n][6];
        int[] length=new int[6];
        String s;

        for(int i=0;i&lt;n;i++){
            s=scan.next();
            length[i]=s.length();
            for(int j=0;j&lt;length[i];j++)
                p[i][j]=Integer.parseInt(s.charAt(j)+"");
        }


        dfs(p,length,0);

        System.out.println(count);
    }

}
```