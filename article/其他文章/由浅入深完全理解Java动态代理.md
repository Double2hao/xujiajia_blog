#由浅入深完全理解Java动态代理
# 前言

看Retrofit源码的时候涉及到了Java动态代理，这个知识点之前在学习Java反射的时候就碰到过，不过也仅仅是停留在理论的学习。终于在Retrofit源码的时候看到了实际的使用，也是格外兴奋，正好以此为契机，把Java动态代理这一块整理一下。 本文也是尽量由浅入深的讲，过多理论知识也不多加阐述。

>  
 本文参考  


# 场景

我们有一个字体提供类，有多种实现（从磁盘，从网络，从系统）。 最简单的情况下，我们会这么写。

```
public interface FontProvider {
	Font getFont(String name);
}
public class View {
	Font font;
	View(){
		this.font=new FontProviderFromDisk().getFont("something");
	}
}

```

但是，这么写会导致fontProvider只要改变，我们就需要更改View的代码。 因此耦合度太高了，我们可以使用代理模式去掉耦合。 ##静态代理模式

```
public interface FontProvider {
	Font getFont(String name);
}
public class ResourceProvider {
	public static FontProvider getFontProvider() {
        return new FontProviderFromDisk();
    }
}

public class View {
	Font font;
	View(){
		this.font=ResourceProvider.getFontProvider().getFont("something");
	}
}

```

可以看到，我们把FontProvider的创建工作交给了ResourceProvider，这样如果FontProvider的创建有更改，我们也不需要去修改View的代码。 ##动态代理模式 现在需求继续增多，我们不仅仅要文字提供，还需要图片提供，音乐提供等等，如果使用静态代理，代码如下。

```
public class ResourceProvider {
    public static FontProvider getFontProvider() {...}
    public static ImageProvider getImageProvider() {...}
    public static MusicProvider getMusicProvider() {...}
    ......
}

```

在静态代理的情况下，我们需要再去新建两个类。 三者代码代码逻辑几乎一模一样，就是获取到Provider，然后根据Provider获取到内容。 现在仅仅只有三个，如果有十个种类提供呢？难道我们也重新新建十个的类吗？ 既然逻辑都一样，那我们能不能将其统一。 这里就可以使用动态代理实现。

```
public class ProviderHandler implements InvocationHandler{
	private Object target;

	public ProviderHandler(Object target){
		this.target=target;
	}
	
	public Object invoke(Object proxy, Method method, Object[] args)
			throws Throwable {
		return method.invoke(target, args);
	}
}

public class ResourceProvider {
	
	public static FontProvider getFontProvider() {
		Class&lt;FontProvider&gt; targetClass = FontProvider.class;
		FontProvider fonProvider=(FontProvider)Proxy.newProxyInstance(
				targetClass.getClassLoader(),
	            new Class[] { targetClass },
	            new ProviderHandler(new FontProviderFromDisk()));
        return fonProvider;
    }
}

public class View {
	Font font;
	View(){
		this.font=ResourceProvider.getFontProvider().getFont("something");
	}
}

```

可以看到View的代码是不变的，不管是什么代理模式都能实现解耦的作用。 ProviderHandler： 通过这个类可以通过invoke创建某个类。invoke的三个参数分别为：用来代理实例，要创建的类的类类型，参数（这里指构造方法的参数）。 ResourceProvider： 通过使用ProviderHandler创建fontProvider。

单单动态代理的概念，看完上面这些基本就可以理解了，接下来我们在提供字体的基础上加上缓存，看一看要如何操作。 这边主要是结合Java反射的使用。

# 添加缓存

## 静态代理

```
public interface FontProvider {
	Font getFont(String name);
}
public class CachedFontProvider implements FontProvider {
    private FontProvider fontProvider;
    private Map&lt;String, Font&gt; cached;

    public CachedFontProvider(FontProvider fontProvider) {
        this.fontProvider = fontProvider;
    }

    public Font getFont(String name) {
        Font font = cached.get(name);
        if (font == null) {
            font = fontProvider.getFont(name);
            cached.put(name, font);
        }
        return font;
    }
}
public class ResourceProvider {
	
	public static FontProvider getFontProvider() {
		 return new CachedFontProvider(new FontProviderFromDisk());
    }
}

public class View {
	Font font;
	View(){
		this.font=ResourceProvider.getFontProvider().getFont("something");
	}
}

```

这里主要添加了一个CachedFontProvider，这个Provider构造的时候会传入其他文字的Provider，可以是硬盘的，内存的等。就是简单的通过HashMap来实现了缓存。 ResourceProvider 中使用CachedFontProvider就能让文字拥有缓存的功能了。 当然也可以看到，View的代码还是不会变的，因为这块逻辑已经和View本身解耦了。

## 动态代理

```
public interface FontProvider {
	Font getFont(String name);
}
public class CachedFontProvider implements FontProvider {
    private FontProvider fontProvider;
    private Map&lt;String, Font&gt; cached;

    public CachedFontProvider(FontProvider fontProvider) {
        this.fontProvider = fontProvider;
    }

    public Font getFont(String name) {
        Font font = cached.get(name);
        if (font == null) {
            font = fontProvider.getFont(name);
            cached.put(name, font);
        }
        return font;
    }
}
public class ProviderHandler implements InvocationHandler{
	private Map&lt;String, Object&gt; cached = new HashMap&lt;&gt;();
	    private Object target;

	    public ProviderHandler(Object target) {
	        this.target = target;
	    }

	    public Object invoke(Object proxy, Method method, Object[] args)
	        throws Throwable {
	    	//这里method其实就是构造函数
	    	//获取函数的参数类型
	        Type[] types = method.getParameterTypes();
	      //这里限定参数只有一个，并且类型是String
	        if (method.getName().matches("get.+") &amp;&amp; (types.length == 1) &amp;&amp;
	                (types[0] == String.class)) {
	            String key = (String) args[0];
	            Object value = cached.get(key);
	            if (value == null) {
	                value = method.invoke(target, args);
	                cached.put(key, value);
	            }
	            return value;
	        }
	        return method.invoke(target, args);
	    }
}
public class ResourceProvider {
	
	public static FontProvider getFontProvider() {
		Class&lt;FontProvider&gt; targetClass = FontProvider.class;
		FontProvider fontProvider=(FontProvider)Proxy.newProxyInstance(
				targetClass.getClassLoader(),
	            new Class[] { targetClass },
	            new ProviderHandler(new FontProviderFromDisk()));
        return fontProvider; 
    }
}
public class View {
	Font font;
	View(){
		this.font=ResourceProvider.getFontProvider().getFont("something");
	}
}

```

这里动态代理其实就是解决如何动态控制缓存的问题。 与静态代理相比，其实就是添加了ProviderHandler，然后ResourceProvider由本来的静态创建变成了使用ProviderHandler动态创建。 ProviderHandler： 使用反射，根据函数类型进行筛选。然后根据对应的构造函数创建对象，并把这个对象缓存到cache中。