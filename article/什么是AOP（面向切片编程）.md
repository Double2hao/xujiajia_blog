#什么是AOP（面向切片编程）
>  
 本文参考  


# AOP wiki 解释

>  
 In computing, aspect-oriented programming (AOP) is a programming paradigm that aims to increase modularity by allowing the separation of cross-cutting concerns. It does so by adding additional behavior to existing code (an advice) without modifying the code itself, instead separately specifying which code is modified via a “pointcut” specification, such as “log all function calls when the function’s name begins with ‘set’”. This allows behaviors that are not central to the business logic (such as logging) to be added to a program without cluttering the code, core to the functionality. AOP forms a basis for aspect-oriented software development. 


大概翻译如下： 在计算中，面向方面编程（AOP）是一种编程范例，旨在通过分离横切关注点来增加模块性。它通过在不修改代码本身的情况下向现有代码添加其他行为来实现，它通过单独指定“ 切入点 ”规范修改哪些代码，例如“当函数名称以’set’开头时记录所有函数调用”。这允许行为不是业务逻辑的核心 （例如日志记录）要添加到程序中而不会使核心功能的代码混乱。AOP构成了面向方面的软件开发的基础。

# 简单例子理解什么是AOP

单单看wiki的解释基本理解不了，会觉得AOP是个非常高大上的概念。但是其实AOP是一种我们可以使用到平时业务开发中的一种编程方式。 下面通过一个简单的例子来说一下AOP主要为了解决什么问题：

>  
 现在有一个简单的管理系统，它有查询和添加两个功能。现在有个新的需求，期望在查询和添加的时候能够记录log，便于排查问题。 


管理系统和Log模块代码如下：

```
public class ManagerSystem {
    
    public void search(){
    }

    public void insert(){
    }
}

```

```
public class LogManager {
    public static void log(){
    }
}

```

根据上面的需求，我们要给ManagerSystem的行为中记录log，一般情况下，代码会这么写：

```
public class ManagerSystem {

    public void search(){
        LogManager.log();
    }

    public void insert(){
        LogManager.log();
    }
}

```

这样写代码的确是可以实现需求的，但是ManagerSystem本身只是个管理系统，在其中添加了两行打日志的代码就表明其也需要承担一部分管理日志打印的责任，这显然是不合理的。理想情况下，各个模块更应该只专注于自己的需求，而不应该承担一部分和自己需求无关的任务。

举例说下在实际开发中的问题：

### 从打印日志模块的角度来看

如果项目较大，不仅仅有查询和添加这两个地方有打日志的需要，有可能有上百个模块需要打日志。而日志模块与其他模块根本不是同一个人负责的，甚至不是同一个组负责的。这就会有两个问题：
1. 一旦LogManager的逻辑有变动就可能需要去改动其他所有调用方的代码。比如现在只有log()方法来打日志，后面LogManager模块拓展为了debug()，warn()，error()。（这有可能涉及了上百个模块和上千个调用的地方）1. LogManager虽然是日志管理模块，但是却根本无法统一管理自己的调用方，甚至连谁调用了自己也无法知道。（开发者可用通过全局搜索知道谁调用了这个代码，但是这个模块是没有存储调用方的信息的）
### 从管理系统模块的角度来看

管理系统模块真正使用中，定然不会只使用了日志打印这个一个其他模块、有可能还包括性能分析模块、用户类型分析模块等模块。 如果仅仅是按照正常方式开发，那么管理系统模块就相当于也承担了诸多其他模块的责任。而理想情况下，管理系统只负责管理数据可以了，至于其他的应该由那么模块自己负责。

# 如何实现AOP

AOP目前主要是运用在java Spring中，此处就相当于讲一下Spring AOP原理。 主要实现就是使用代理模式，对代理模式不是很了解的可以看下笔者之前的文章：

现在有一个Person类，我们在其doSomething()的方法中加入两行log，在不改变Person这个类的情况下如何实现？

```
public interface IPerson {
    public void doSomething();
}

public class Person implements IPerson {
    public void doSomething(){
        System.out.println("I want wo sell this house");
    }
}

```

实现步骤如下：
1. 首先实现一个Person的代理类，在代理类中加入log。1. 在运行期间用代理类的所有方法覆盖被代理的类的方法，在此处也就是在运行期间用PersonProxy的doSomething()覆盖Person的doSomething()。这样在真正调用的时候使用的就是加了log的代码了。
```
public class PersonProxy {
    private IPerson iPerson;
    private final static Logger logger = LoggerFactory.getLogger(PersonProxy.class);

    public PersonProxy(IPerson iPerson) {
        this.iPerson = iPerson;
    }
    public void doSomething() {
        logger.info("Before Proxy");
        iPerson.doSomething();
        logger.info("After Proxy");
    }
}

```

```
public class PersonProxy implements MethodInterceptor {
    private Object delegate;
    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    public Object intercept(Object proxy, Method method, Object[] args,  MethodProxy methodProxy) throws Throwable {
        logger.info("Before Proxy");
        Object result = methodProxy.invokeSuper(method, args);
        logger.info("After Proxy");
        return result;
    }

    public static Person getProxyInstance() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Person.class);

        enhancer.setCallback(new PersonProxy());
        return (Person) enhancer.create();
    }
}

```