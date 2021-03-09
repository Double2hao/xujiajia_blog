#JAVA 串的处理（简单的string与stringbuffer的使用）
 串的处理

在实际的开发工作中，对字符串的处理是最常见的编程任务。

本题目即是要求程序对用户输入的串进行处理。具体的规则如下：

1、  把每个单词的首字母变为大写。

2、  把数字与字母之间用下划线字符（_）分开，使得更清晰

3、  把单词中间有多个空格的调整为1个空格。

 

例如：

用户输入：you and   me whatcpp2005program

则程序输出：You And Me What Cpp_2005_program

用户输入：this is  a   99cat

则程序输出：This Is A 99_cat

我们假设：用户输入的串只有小写字母，空格和数字，不含其它的字母或字符。

每个单词间由1个或多个空格分隔。

假设用户输入的串的长度不超过200个字符。

 



```
import java.util.Scanner;

public class three {
	public static void main(String[] args){
		Scanner input =new Scanner(System.in);
		String str1=input.nextLine();
		String str2=input.nextLine();
		StringBuffer a=new StringBuffer(str1);
		StringBuffer b=new StringBuffer(str2);
		//"you and  me what cpp2005program"
		//"this is  a   99cat"
		change(a);
		change(b);
	}


	private static void change(StringBuffer c) {
		int num=c.length();
		for(int i = 0;i&lt;num-1;i++){
			if(i==0){
				c.setCharAt(0, (char) (c.charAt(i)+'A'-'a'));
			}
			else if((c.charAt(i)==' ')&amp;&amp;(Character.isLetter(c.charAt(i+1))))
				c.setCharAt(i+1, (char) (c.charAt(i+1)+'A'-'a'));
			else if((c.charAt(i)==' ')&amp;&amp;(c.charAt(i+1)==' ')){
				c.deleteCharAt(i);
				i--;
				num--;
			}	
			else if((Character.isLetter(c.charAt(i)))&amp;&amp;(Character.isDigit(c.charAt(i+1)))){
				c.insert(i+1,'_');
			}
			else if((Character.isLetter(c.charAt(i+1)))&amp;&amp;(Character.isDigit(c.charAt(i)))){
				c.insert(i+1,'_');
			}
		}
		
		System.out.println(c);
	}
	
}

```



<img src="" alt=""> 

 

 
