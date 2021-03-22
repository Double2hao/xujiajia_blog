#斐波那契数列第n项的高效解法
>  
 参考书籍《剑指Offer》 


## 常见解法

谈及斐波那契数列，我们直接就可以想到f(n)=f(n-1)+f(n-2)。于是做出如下解：

```
long long Fibonacci(unsigned int n)
{
    if(n&lt;=0)
        return 0;
    if(n==1)
        return 1;
    return Fibonacci(n-1)+Fibonacci(n-2);
}

```

## 优化解法：

但是其实此算法是可以优化的，我们可以用树形结构来表示这种依赖关系。 我们可以发现其中f(7)、f(6)是重复计算的。如果n很大的话，这重复计算的量是非常大的。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2270.png" alt="这里写图片描述">

优化代码如下：

```
long long Fibonacci(unsigned int n)
{
    if(n&lt;=0)
        return 0;
    if(n==1)
        return 1;
        
    long long one=1;
    long long two=0;
    long long n=0;
    for(unsigned int i=2;i&lt;=n;++i)
    {
        n=one+two;
        
        two=one;
        one=n;
    }
    return n;
}

```