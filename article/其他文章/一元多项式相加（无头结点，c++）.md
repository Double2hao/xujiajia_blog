#一元多项式相加（无头结点，c++）
**【实验内容】**

结合书上第41页的例子，采用链式存储结构，将两个线性链表表示的一元多项式相加，并输出。

 

**【参考方法说明】**

根据一元多项式相加的运算规则：对于两个一元多项式中所有指数相同的项，对应系数相加，若其和不为零，则构成“和多项式”中的一项；对于两个一元多项式中所有指数不相同的项，则分别复抄到“和多项式”中去。

 

**题目分析：**

1、题目中并没有说明是如何输入（可能会重复输入同一个指数，或者指数较小的在指数大的之后输入），既然本题只是做个测试，目的只是为了熟悉链表，那么就没有必要太去纠结那些。直接默认是按照指数递增序列输出便是了。

2、使用的思想还是双向链表，便于插入。

笔者为了方便也考虑过使用一个insert的函数，但是由于每次做insert之前都需要做一个position位置的遍历，代码效率是极低的。（这个复杂度是n^2，笔者的代码是n）

3、为了提高代码的可读性，安全性，所以选用c++。

 

效果图：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2500.png" alt=""> 

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2501.png" alt=""> 

 

代码：



```
#include &lt;stdio.h&gt;
#include &lt;iostream&gt;
using namespace std;
typedef struct LNode
{
    int coe,index;
    struct LNode *next;
} LNode;

//建立一个多项式类
class Polynoimal
{
private:
    LNode *head;//定义私有头指针来找到这个多项式
public:
    LNode *p;
    LNode *curr;
    LNode *next;

    Polynoimal();//构造多项式
    LNode* getHead();//获取头指针
    void Plus(Polynoimal h2);//两个多项式的加法
    void Print();//输出多项式

};

Polynoimal::Polynoimal()
{
    LNode *s;
    int n;
    int coe,index;
    cout&lt;&lt;"请输入你要创建的表的大小："&lt;&lt;endl;
    cout&lt;&lt;"n=";
    cin&gt;&gt;n;
    cout&lt;&lt;endl;
    cout&lt;&lt;"请按照指数从小到大的顺序输入，并且不要输入相同指数的项！"&lt;&lt;endl;

	//首先在外面接收第一个结点是为了让头指针指向它
    cout&lt;&lt;"请输入系数与指数的大小："&lt;&lt;endl;
    cout&lt;&lt;"coefficient=";
    cin&gt;&gt;coe;
    cout&lt;&lt;"index=";
    cin&gt;&gt;index;
    s=new LNode;
    s-&gt;coe=coe;
    s-&gt;index=index;
    head=s;
    curr=s;

    for(int i=0; i&lt;n-1; i++)
    {
        s=new LNode;
        cout&lt;&lt;"请输入系数与指数的大小："&lt;&lt;endl;
        cout&lt;&lt;"coefficient=";
        cin&gt;&gt;coe;
        cout&lt;&lt;"index=";
        cin&gt;&gt;index;
        s-&gt;coe=coe;
        s-&gt;index=index;
        curr-&gt;next=s;
        curr=s;
    }
    curr-&gt;next=NULL;//最后不要忘了让最后一个结点指向空
}

LNode* Polynoimal::getHead()
{
    return head;
}

void Polynoimal::Plus(Polynoimal h2)
{
    p=NULL;
    curr=head;
    next=curr-&gt;next;
    LNode *curr2=h2.getHead();

//如果h2多项式已遍历完了那么就不执行
    while(curr2!=NULL)
    {

        LNode *s;
        //要判断当前多项式是否已经遍历完，如果已经遍历完了，直接让h2中剩下的结点插入尾部就可以
        if(curr!=NULL)
        {
            //如果系数相等
            if(curr-&gt;index==curr2-&gt;index)
            {
                curr-&gt;coe+=curr2-&gt;coe;
                //如果系数等于0，那么删除这个结点，并且用前一个结点将后面的连起来，不然就断了
                //如果不等于0，直接光标后移就可以了
                if(curr-&gt;coe==0)
                {
                    LNode *q=curr;
                    curr=curr-&gt;next;
                    p-&gt;next=curr;
                    if(curr!=NULL)
                        next=curr-&gt;next;
                    delete q;
                    curr2=curr2-&gt;next;
                }
                else
                {
                    p=curr;
                    curr=curr-&gt;next;
                    if(curr!=NULL)
                        next=curr-&gt;next;
                    curr2=curr2-&gt;next;
                }
            }
            //如果当前光标的指数小于h2中的光标的指数
            else if(curr-&gt;index&lt;curr2-&gt;index)
            {
                //倘若已经是最后一个结点，那么直接让当前多项式遍历完，让h2中剩下结点都插入尾部
                if(next==NULL)
                {
                    p=curr;
                    curr=NULL;
                    continue;
                }
                //如果当前光标的下一个结点的指数小于等于h2中光标当前的指数
                //那么就让当前光标后移
                if(next-&gt;index&lt;=curr2-&gt;index)
                {
                    p=curr;
                    curr=curr-&gt;next;
                    if(curr!=NULL)
                        next=curr-&gt;next;
                }
                //如果当前光标的下一个结点的指数大于h2中光标当前的指数
                //那么直接插入就可以了
                else
                {
                    s=new LNode;
                    s-&gt;coe=curr2-&gt;coe;
                    s-&gt;index=curr2-&gt;index;
                    curr-&gt;next=s;
                    s-&gt;next=next;

                    p=curr;
                    curr=s;
                    if(curr!=NULL)
                        next=curr-&gt;next;
                    curr2=curr2-&gt;next;
                }
            }
            //如果如果当前光标的指数大于h2中的光标的指数
            //根据整体思路可知，这种情况只会出现在第一个结点处
            //解释一下：
            //如果h2中有多个结点的指数小于当前多项式的第一个结点的指数，那么h2的第一个结点便会插入当前多项式的第一个结点的位置
            //然后由于两个多项式的指数是按照递增排列的
            //当h2中指数最小的结点插入当前多项式之后,将当前多项式光标定位第一个结点，
            //那么h2中定然不存在比第一个结点指数更小的的结点了
            //于是，这种情况便再也不会发生了
            else if(curr-&gt;index&gt;curr2-&gt;index)
            {
                s=new LNode;
                s-&gt;coe=curr2-&gt;coe;
                s-&gt;index=curr2-&gt;index;
                s-&gt;next=curr;

                head=s;
                curr=s;
                next=curr-&gt;next;
                curr2=curr2-&gt;next;
            }
        }
        //如果当前多项式已经遍历完，那么直接让h2中剩下的结点插入当前多项式表尾
        else if(curr==NULL)
        {
            s=new LNode;
            s-&gt;coe=curr2-&gt;coe;
            s-&gt;index=curr2-&gt;index;
            s-&gt;next=NULL;

            p-&gt;next=s;
            p=s;
            curr2=curr2-&gt;next;
        }
    }
}

void Polynoimal::Print()
{
    LNode *pa= head;
    cout&lt;&lt;"多项式各项的值依次为："&lt;&lt;endl;
    while(pa!=NULL)
    {
        cout&lt;&lt;"系数为：";
        cout&lt;&lt;pa-&gt;coe;
        cout&lt;&lt;"    指数为：";
        cout&lt;&lt;pa-&gt;index&lt;&lt;endl;
        pa=pa-&gt;next;
    }
    cout&lt;&lt;endl;
}


int main()
{
    Polynoimal h1;
	printf("head1的");
    h1.Print();
    Polynoimal h2;
	printf("head2的");
    h2.Print();
    h1.Plus(h2);
	printf("相加后head1的");
    h1.Print();
	printf("相加后head2的");
    h2.Print();
    return 0;
}

```

