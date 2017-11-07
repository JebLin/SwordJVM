# Java堆，方法区，Java栈和本地方法栈


> 运行时数据区包括：虚拟机栈区，堆区，方法区，本地方法栈，程序计数器
```
虚拟机栈区 ：也就是我们常说的栈区，线程私有，存放基本类型，对象的引用和 returnAddress ，在编译期间完成分配。
堆区 : JAVA 堆，也称 GC 堆，所有线程共享，存放对象的实例和数组， JAVA 堆是垃圾收集器管理的主要区域。
方法区 ：所有线程共享，存储已被虚拟机加载的类信息，常量，静态变量，即时编译器编译后的代码等数据。这个区域的内存回收目标主要是针对常量池的对象的回收和对类型的卸载。
程序计数器 （Program Counter ）：线程私有，每个线程都有自己独立的程序计数器，用来指示下一条指令的地址。
```



最近在看《深入理解Java虚拟机》，书中给了几个例子，比较好的说明了几种OOM（OutOfMemory）产生的过程，大部分的程序员在写程序时不会太关注Java运行时数据区域的结构：

感觉有必要通过几个实在的例子来加深对这几个区域的了解
### 1）Java堆
所有对象的实例分配都在Java堆上分配内存，堆大小由-Xmx和-Xms来调节，sample如下所示：
```
public class HeapOOM {  
      
    static class OOMObject{}  
  
    /** 
    * @param args 
    */  
    public static void main(String[] args) {  
        List<OOMObject> list = new ArrayList<OOMObject>();  
          
        while(true){  
            list.add(new OOMObject());  
        }  
    }  
  
} 
```
加上JVM参数-verbose:gc -Xms10M -Xmx10M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:+HeapDumpOnOutOfMemoryError，就能很快报出OOM：
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
并且能自动生成Dump。
### 2）方法区
方法区是存放虚拟机加载类的相关信息，如类的结构、静态变量和常量，大小由-XX:PermSize和-XX:MaxPermSize来调节，类太多有可能撑爆永久带：
```
public class MethodAreaOOM {  
      
    static class OOMOjbect{}  
  
    /** 
    * @param args 
    */  
    public static void main(String[] args) {  
        // TODO Auto-generated method stub  
        while(true){  
            Enhancer eh = new Enhancer();  
            eh.setSuperclass(OOMOjbect.class);  
            eh.setUseCache(false);  
            eh.setCallback(new MethodInterceptor(){  
  
                @Override  
                public Object intercept(Object arg0, Method arg1,  
                        Object[] arg2, MethodProxy arg3) throws Throwable {  
                    // TODO Auto-generated method stub  
                    return arg3.invokeSuper(arg0, arg2);  
                }  
                  
            });  
            eh.create();  
        }  
    }  
  
}  

```
加上永久带的JVM参数：-XX:PermSize=10M -XX:MaxPermSize=10M，运行后会报如下异常：
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space
静态变量或常量也会有可能撑爆方法区：
```
public class ConstantOOM {  
  
    /** 
    * @param args 
    */  
    public static void main(String[] args) {  
        // TODO Auto-generated method stub  
        List<String> list = new ArrayList<String>();  
        int i=0;  
        while(true){  
            list.add(String.valueOf(i++).intern());  
        }  
    }  
  
}  

```

同样加上JVM参数：-XX:PermSize=10M -XX:MaxPermSize=10M，运行后报如下异常：
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space
### 3）Java栈和本地方法栈
栈是存放线程调用方法时存储局部变量表，操作，方法出口等与方法执行相关的信息，栈大小由Xss来调节，方法调用层次太多会撑爆这个区域，samples如下所示：
```
package com.cutesource;  
  
public class StackOOM {  
  
    /** 
    * @param args 
    */  
      
    private int stackLength = 1;  
      
    public void stackLeak(){  
        stackLength++;  
        stackLeak();  
    }  
      
    public static void main(String[] args) throws Throwable{  
        // TODO Auto-generated method stub  
        StackOOM oom = new StackOOM();  
        try{  
            oom.stackLeak();  
        }catch(Throwable err){  
            System.out.println("Stack length:" + oom.stackLength);  
            throw err;  
        }  
          
    }  
  
}  


```
设置JVM参数：-Xss128k，报出异常：
Exception in thread "main" java.lang.StackOverflowError
打印出Stack length:1007，这里可以看出，在我的机器上128k的栈容量能承载深度为1007的方法调用。当然报这样的错很少见，一般只会出现无限循环的递归中，另外，线程太多也会占满栈区域：
```
package com.cutesource;  
  
public class StackOOM {  
  
    /** 
    * @param args 
    */  
      
    private int stackLength = 1;  
      
    private void dontStop(){  
        while(true){  
            try{Thread.sleep(1000);}catch(Exception err){}  
        }  
    }  
      
    public void stackLeakByThread(){  
        while(true){  
            Thread t = new Thread(new Runnable(){  
  
                @Override  
                public void run() {  
                    // TODO Auto-generated method stub  
                    dontStop();  
                }  
                  
            });  
            t.start();  
            stackLength++;  
        }  
    }  
      
    public static void main(String[] args) throws Throwable{  
        // TODO Auto-generated method stub  
        StackOOM oom = new StackOOM();  
        try{  
            oom.stackLeakByThread();  
        }catch(Throwable err){  
            System.out.println("Stack length:" + oom.stackLength);  
            throw err;  
        }  
          
    }  
  
}  


```

报出异常：Exception in thread "main" java.lang.OutOfMemoryError:unable to create new native thread
不过在windows上运行这个例子要小心，会出现系统假死的情况，有可能需要重启机器才行。
以上几个例子虽然比较简单，但能很好帮助普通的程序员更加直观的了解Java堆，方法区，Java栈和本地方法栈。

## java的内存机制

Java 把内存划分成两种：一种是栈内存，另一种是堆内存。

      在函数中定义的一些基本类型的变量和对象的引用变量都是在函数的栈内存中分配，当在一段代码块定义一个变量时，Java 就在栈中为这个变量分配内存空间，当超过变量的作用域后，Java 会自动释放掉为该变量分配的内存空间，该内存空间可以立即被另作它用。

　　  堆内存用来存放由 new 创建的对象和数组，在堆中分配的内存，由 Java 虚拟机的自动垃圾回收器来管理。在堆中产生了一个数组或者对象之后，还可以在栈中定义一个特殊的变量，让栈中的这个变量的取值等于数组或对象在堆内存中的首地址，栈中的这个变量就成了数组或对象的引用变量，以后就可以在程序中使用栈中的引用变量来访问堆中的数组或者对象，引用变量就相当于是为数组或者对象起的一个名称。引用变量是普通的变量，定义时在栈中分配，引用变量在程序运行到其作用域之外后被释放。而数组和对象本身在堆中分配，即使程序运行到使用 new 产生数组或者对象的语句所在的代码块之外，数组和对象本身占据的内存不会被释放，数组和对象在没有引用变量指向它的时候，才变为垃圾，不能在被使用，但仍然占据内存空间不放，在随后的一个不确定的时间被垃圾回收器收走（释放掉）。

> 这也是 Java 比较占内存的原因，实际上，栈中的变量指向堆内存中的变量，这就是 Java 中的指针！