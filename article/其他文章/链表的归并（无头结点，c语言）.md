#链表的归并（无头结点，c语言）
【实验内容】

设有两个无头结点的单链表，头指针分别为ha，hb，链中有数据域data，链域next，两链表的数据都按递增序存放，现要求将hb表归到ha表中，且归并后ha仍递增序，归并中ha表中已有的数据若hb中也有，则hb中的数据不归并到ha中，hb的链表在算法中不允许破坏。

 

【参考方法说明】



```
#include &lt;stdio.h&gt;  
#include &lt;stdlib.h&gt;
typedef struct LNode  
{  
    int data;  
    struct LNode *next;  
}LNode, *LinkList;     
```



 

 

先看一下实现效果：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2670.png" alt=""> 

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2671.png" alt=""> 

 

 

笔者愚笨，写的c程序也不多（以后会用c++），写这样一个程序差不多花了5个小时的时间，花了大量的时间在bug的修复上，在此也是把代码分享给大家。（**希望有助于大家对链表的理解**） 

 



```
#include &lt;stdio.h&gt;  
#include &lt;stdlib.h&gt;
typedef struct LNode  
{  
    int data;  
    struct LNode *next;  
}LNode, *LinkList;      
 
void Create_LinkList(LinkList &amp;H)  
{  
    int x;  
    LinkList p, s;  
	//此处是为了避免输入空表，造成bug
	printf("请输入插入节点的数值，输入-1停止\nx=");  
    scanf("%d",&amp;x);  
    if(x != -1)  
    {  
        s = (LNode*)malloc(sizeof(LNode));  
		s-&gt;data = x;  
		H=s; 
		p=H; 
	}
	else  
	{
	 printf("请不要输入空表！请重新输入！\n");
	 Create_LinkList(H);
	 return;
	}
	
    while(1)  
    {  
        printf("请输入插入节点的数值，输入-1停止\nx=");  
        scanf("%d",&amp;x);  
        if(x != -1)  
        {  
            s = (LNode*)malloc(sizeof(LNode));  
            s-&gt;data = x;  
            p-&gt;next = s;  
            p = s;   
		}
        else  
            break;             
    }  
    p-&gt;next = NULL;  
}  
  
  
void Print_LinkList(LinkList &amp;H)  
{  
    LNode *p = H;     
    printf("链表中的元素依次为:\n");  
    while(p)  
    {  
        printf("%d\n",p-&gt;data);  
        p = p-&gt;next;  
    }  
}  
  
  

void MergeList_LinkList(LinkList &amp;ha,LinkList &amp;hb)
{
	LNode *pa=ha;//ha为当前结点
	LNode *pb=hb;
	LNode *p=NULL;//p作为pa前一个结点
	LNode *k=pa-&gt;next;//k在此处用作指向pa后一个结点的指针
	
	while(pb!=NULL)
	{
		//为了避免hb被破坏，所以要创建一个结点来复制hb中元素
		LinkList s=(LNode*)malloc(sizeof(LNode));

		if(pa!=NULL&amp;&amp;pb!=NULL)
		{	
			//pa当前结点和pb当前结点主要是等于，小于，大于三种情况
			if(pa-&gt;data==pb-&gt;data)
			{
				free(s);//当两者相等，不进行操作，并且把之前结点给free掉并且光标后移
				p=pa;
				pa=k;
				pb=pb-&gt;next;
				if(pa!=NULL)
					k=pa-&gt;next;
			}

			else if(pa-&gt;data&lt;pb-&gt;data)
			{
				//如果k=NULL，那么k已经没有意义，不用管它，直接让pa的遍历结束进入下一个循环
				if(k==NULL)
				{
					p=pa;
					pa=k;
					continue;
				}
				//如果pa下一个结点中的值大于pb当前结点，那么就把当前值插入pa表中
				if(k-&gt;data&gt;pb-&gt;data)
				{
				s-&gt;data = pb-&gt;data;  
				s-&gt;next=k;
				pa-&gt;next=s;
				
				pa=s-&gt;next;
				pb=pb-&gt;next;
				p=s;
				if(pa!=NULL)
					k=pa-&gt;next;
				}
				//如果pa下一个结点大于或前结者小于pb当点，那么直接pa光标后移就可以
				else
				{
					p=pa;
					pa=k;
					if(pa!=NULL)
					k=pa-&gt;next;
				}
				
			}

			//根据算法可知，pa当前结点大于pb当前结点的情况可能会在第一个结点发生，在插入结点的时候同时也要让ha指向它
			//如果不在第一个结点发生，那么就直接插入
			//示例：468,123456
			else
			{
				if(p==NULL)
				{
				s-&gt;data=pb-&gt;data;
				s-&gt;next=pa;

				ha=s;
				pa=s;
				k=pa-&gt;next;
				pb=pb-&gt;next;
				}
				else
				{
					s-&gt;data=pb-&gt;data;
					s-&gt;next=pa;
					p-&gt;next=s;

					p=s;
					pa=p-&gt;next;
					k=pa-&gt;next;
					pb=pb-&gt;next;
				}
				
				
			}
		}
		//倘若pa表已经结束，那么直接把pb表中剩下元素全部插入到pa尾部就可以了
		else if(pa==NULL&amp;&amp;pb!=NULL)
		{
			s-&gt;data=pb-&gt;data;
			p-&gt;next=s;
			s-&gt;next=NULL;

			p=s;
			pb=pb-&gt;next;
		}

	}
	
}

int main()  
{  
    LinkList head1;  
    Create_LinkList(head1);  
	printf("head1");
    Print_LinkList(head1);  

	LinkList head2;  
    Create_LinkList(head2); 
	printf("head2");
    Print_LinkList(head2);

	MergeList_LinkList(head1,head2);
	printf("合并后head1");
	Print_LinkList(head1);
	printf("合并后head2");
	Print_LinkList(head2);
  
    return 0;  
}  

```



   