#Java内存模型和Volatile
# Java内存模型

<img src="https://img-blog.csdnimg.cn/20190629171938923.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> 为了提高代码的执行速度，Java内存模型中有两个优化方案：

### 1、每个Java线程会有一个工作内存，工作内存中的内容会在一定条件下同步到主内存。

>  
 为什么工作内存可以提高代码的执行速度？ 从硬件上来看CPU的速度&gt;内存的速度。每次CPU去内存中读写数据的时候，CPU一般都是阻塞的，需要等内存执行完毕后CPU才会继续执行。 Java工作内存可以认为是CPU高速缓存，那么就能避免“去内存中读写数据”而导致的CPU阻塞。 


### 2、默认支持指令重排

>  
 现在CPU一般都是多核。 假设有两个内核，把代码分成两份，每个内核执行一份，肯定能比一个内核执行一整份代码快。而且也是提高的了CPU内核的利用率，如果一个内核执行一整份代码段话，另一个内核是没有事情做的。 不过，把代码分成多份让多个内核执行，就没法保证执行的代码的顺序了。（当然真正的指令重排肯定不是“把代码分成几份”这么简单） 


# Volatile

volatile主要能解决两个问题：

### 1、解决引入“工作内存”导致的一致性问题。

简单来说，线程A写了一个data数据到自己的工作内存，然后线程B去自己的工作内存读取data数据，结果是线程B读取到的data数据并不是线程A写入的那个。

一个变量如果使用了volatile，可以认为就是不使用工作内存了，每次读写都会直接操作主内存。 这时候再看上面的问题，线程A直接写一个data数据到主内存，线程B去主内存读取data数据，结果就是线程B读到了线程A写入的数据data。

### 2、针对一些特殊场景禁止指令重排。

假设有两个线程A和B。 线程A代码如下：

```
volatile boolean initialized=false;

readConfigFromFile();
initialized=true;

```

线程B代码如下：

```
while(!initialized){
sleep();
}
doSomeWithConfig();

```

在这种情况下，如果代码没有顺序执行的话，有可能会直接导致线程B获取不到config信息而崩溃。

# 一个注意点

volatile只能够保证该对象每次读写能直接操作主内存。而不代表对这个变量的所有操作都是能多线程之间同步的。

假设如下代码：

```
public class testMain {

    public static volatile int a=0;
    
    public static void main(String[] args) {
        Thread[] threads=new Thread[10];
        for(int i=0;i&lt;10;i++){
            threads[i]=new Thread(new Runnable() {
                @Override
                public void run() {
                    for(int i=0;i&lt;1000;i++){
                        a++;
                    }
                }
            });
        }
        for(int i=0;i&lt;10;i++){
            threads[i].start();
        }
        try {
            Thread.sleep(1000);
            System.out.println(a);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

最终输出的结果肯定会是一个小于10 * 1000的数。为什么不是正好10 * 1000呢？ 其实结果很容易解释：由于多个线程同时去读写a，并对其进行‘++’操作，很可能前面一个线程的‘++’操作还没执行完，后面一个线程已经又去读a的值了，这就会导致同时会有两个线程对同一个值进行了同样的一次’++'操作。 但是这个“线程间不同步”和volatile没什么关系，这是运算操作本身的问题。

如果要解决这个问题也很简单，给a++这个操作加上synchronize标签，让这个操作变成线程安全就可以，如下：

```
//如果使用了synchronized也不需要用volatile了
public static int a=0;

//由于是在静态方法中，所以要用类对象
synchronized (testMain.class){
    a++;
}

```