#二叉树的遍历（无头结点，C语言）——更新后
【实验内容】

创建一个二叉树，对这棵动态二叉树进行分析，将其用静态二叉链表表示。二叉树的动态二叉链表结构中的每个结点有三个字段：data，lchild，rchild。静态二叉链表是用数组作为存储空间，每个数组元素存储二叉树的一个结点，也有三个字段：data，lchild，rchild。lchild和rchild分别用于存储左右孩子的下标。

 

老师在上课时规定：

1、二叉树的建立，输出都需要用不同的方式（主要为先序、中序、后序三种）。

2、在输出二叉树结点的值的时候同时要输出它的左右孩子。

3、之后老师又增加了对静态二叉树的要求：不能直接将权值存储在数组中，但是最后需要输出权值。在与老师的交流中，我了解到其实就是需要把原来的二叉树的节点给存储下来。于是如下面的代码中，我便是新建了一个struct，用来存储节点，双亲和左右孩子。

 

 

【参考方法说明】

先定义结构体：



```
typedef struct BiTNode
{
    int data;
    struct BiTNode *lchild,*rchild;
} BiTNode,*BiTree;

```



 

 

1、笔者本来想将每个方式的输入输出都写，然后做一个方式选择。但是由于二叉树的代码本身就比较简单，为了提高效率，就每个采用的不同的方式。

2、由于二叉树需要用到递归，用C++过于麻烦。于是选用C语言，直接用函数解决便可。（编译器使用的是CodeBlocks）

 

效果：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911665250.png " alt=""> 

 

  

代码：（不太理解的读者可以将先序输入的二叉树先画下来，其实逻辑上还是比较清晰的）

```
#include&lt;iostream&gt;
#include&lt;stdio.h&gt;

typedef struct BiTNode
{
    char data;
    struct BiTNode *lchild,*rchild;
} BiTNode,*BiTree;

typedef struct static_BiTNode
{
    BiTNode node;
    int parent,lchild,rchild;
} static_BiTNode;

void Front_creat(BiTree &amp;H);//先序创建
void Middle_print(BiTree &amp;H);//中序输出

static int n=0,i=0;//定义n来确认静态数组的大小，i表示在转换时的光标
void static_transform(BiTree &amp;H,static_BiTNode s[],int parent);//静态二叉树链表的转换，这边其实比较灵活，具体看老师的要求
void static_print(static_BiTNode s[]);//静态二叉树的输出

int main()
{
    BiTree head;
    printf("请按照 先序 输入二叉树各结点的值：\n");
    Front_creat(head);

    printf("\n动态按照 中序 输出为：\n");
    Middle_print(head);

    static_BiTNode s[n];
    static_transform(head,s,0);
    printf("\n转换为的 静态二叉树 为:\n");
    static_print(s);
    return 0;
}

void Front_creat(BiTree &amp;H)
{
    char d;
    printf("请输入当前二叉树结点的值:（值为零即为空）");
    scanf("%c",&amp;d);
    getchar();
    if(d=='0')
        H=NULL;
    else
    {
        n++;//增加一个结点用于静态二叉树的创建
        H=new BiTNode;
        H-&gt;data=d;
        Front_creat(H-&gt;lchild);
        Front_creat(H-&gt;rchild);
    }
}


void Middle_print(BiTree &amp;H)
{
    if(H)
    {
        BiTree left=H-&gt;lchild;
        BiTree right=H-&gt;rchild;
        Middle_print(left);

        printf("%4c: ",H-&gt;data);

        //输出当前结点的值并且输出它的左右孩子的值
        if(left)
            printf("左儿子为：%4c;",left-&gt;data);
        else
            printf("左儿子为：  空;");
        if(right)
            printf(" 右儿子为：%4c;",right-&gt;data);
        else
            printf(" 右儿子为：  空;");
        printf("\n");

        Middle_print(right);
    }
}

void static_transform(BiTree &amp;H,static_BiTNode s[],int parent)
{
    //i为定义的静态int，表示在转换过程中的光标
    int curr=i;//此次递归中的数组下标
    s[curr].node=*H;
    s[curr].parent=parent;

    //如果不为空则继续递归
    //如果为空则让它的值为0，并且结束递归
    if(H-&gt;lchild)
    {
        s[curr].lchild=++i;
        static_transform(H-&gt;lchild,s,curr+1);
    }
    else
        s[curr].lchild=0;

    //同上
    if(H-&gt;rchild)
    {
        s[curr].rchild=++i;
        static_transform(H-&gt;rchild,s,curr+1);
    }
    else
        s[curr].rchild=0;

}

void static_print(static_BiTNode s[])
{
    int k=0;
    while(k&lt;n)
    {
        printf("下标:%2d  ",k);
        printf("value:%3c  ",s[k].node.data);
        printf("parent:%3d  ",s[k].parent);
        printf("lchild:%3d  ",s[k].lchild);
        printf("rchild:%3d  \n",s[k].rchild);
        k++;
    }

}

```

  