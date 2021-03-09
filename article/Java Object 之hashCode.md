#Java Object 之hashCode
# 官方参考：

int hashCode ()  Returns a hash code value for the object. This method is supported for the benefit of hash tables such as those provided by HashMap.

The general contract of hashCode is:

Whenever it is invoked on the same object more than once during an execution of a Java application, the hashCode method must consistently return the same integer, provided no information used in equals comparisons on the object is modified. This integer need not remain consistent from one execution of an application to another execution of the same application.

If two objects are equal according to the equals(Object) method, then calling the hashCode method on each of the two objects must produce the same integer result.

It is not required that if two objects are unequal according to the equals(java.lang.Object) method, then calling the hashCode method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.

As much as is reasonably practical, the hashCode method defined by class Object does return distinct integers for distinct objects. (This is typically implemented by converting the internal address of the object into an integer, but this implementation technique is not required by the Java™ programming language.)

# 笔者翻译：

返回一个对象的哈希编码值。这个方法支撑了哈希表的优势。（就像那些被HashMap所提供的一样）

hashCode大致的条约如下：  1、倘若该对象与equals 的对照中是没有信息被修改的，一个相同的对象调无论在Java应用的可执行程序中的哪儿调用调用，hashCode这个方法都必须返回同一个整数。这个整数不需要在两个可执行应用之间一直保持。  2、如果两个对象在equals(Object)这个方法中是相等的，那么这两个对象调用hashCode方法必然也产生相同的整数结果。  3、如果两个对象调用equals(Object)这个方法中的值是不相等的，他们调用hashCode方法返回的值有可能相等。但是，开发者必须明白，为不同的对象创造不同的整数结果是有益于哈希表的。

hashCode方法被Object类定义来对不同的对象返回不同的整数值是很实用的。（这个经常会被用来把一个内部地址转化为一个整数，但是这个应用技术在Java语言中不是很需要）
