#Java   super关键字（android中常见）
super在android中比较常见，没有java基础也并不理解，所以空出时间学习了一下。

 

在Java类中使用super来引用基类的成分，比较简单，示例如下：



```
class FatherClass{
	public int value;
	public void f(){
		value=100;
		System.out.println
		("FatherClass.value:"+value);
	}
}


class ChildClass extends FatherClass{
	public int value;
	public void f(){
		super.f();
		value=200;
		System.out.println
		("ChildClass.value:"+value);
		System.out.println(value);
		System.out.println(super.value);
	}
}


public class test1 {
	public static void main(String[] args){
		ChildClass cc=new ChildClass();
		cc.f();
	}
}
```



FatherClass.value:100 ChildClass.value:200 200 100 

 

 

另外继承中的构造也是用到了super，具体规则如下：

 



 1、子类的构造过程中必须调用其基类的构造方法。

 2、子类可以在自己的构造方法中使用super(argument_list）调用基类的构造方法。

 3、如果子类的构造方法中没有显示的调用基类的构造方法，则系统默认调用基类的无参数构造方法。

 4、如果子类构造方法中既没有显示调用基类构造方法，而基类又没有无参数的构造方法，则编译出错。

示例如下：（此处最好可以自己试验一下）



```
class SuperClass{
	private int n;
	
	SuperClass(){
		System.out.println("调用SuperClass()");
	}
	SuperClass(int n){
			System.out.println("调用SuperClass("+n+")");
		}
}

class SubClass extends SuperClass{
	private int n;
	
	SubClass(int n){
		
		//当子类的构造方法中没有写super的时候，系统默认的调用父类的没有参数的构造方法
		//相当于此处写了如下:
		//super();
		
		System.out.println("调用SuberClass("+n+")");
		this.n=n;
	}
	
	SubClass(){
		super(300);
		//在子类构造过程当中必须调用父类构造方法,并且super必须写在第一句（先有爸爸再有儿子）
		
		System.out.println("调用SubClass()");
	}
}
public class test2 {
	public static void main(String[] args){
		SubClass sc1=new SubClass();
		
		SubClass sc2=new SubClass(400);
		
	}
}

```



调用SuperClass(300) 调用SubClass() 调用SuperClass() 调用SuberClass(400) 
