# 深入JAVA虚拟机

<hr>

## JAVA虚拟机与程序的生命周期
在如下几种情况下，JAVA 虚拟机将结束生命周期
- 执行了System.exit()方法
- 程序正常执行结束
- 程序在执行过程中遇到了异常或错误而异常终止
- 由于操作系统出现错误而导致 JAVA 虚拟机进程终止

## 类的加载、连接、初始化

加载： 查找并加载类的二进制数据 （把 .class文件加载到内存）

连接：
- 验证：确保被加载的类的正确性（这句话是防止有些人直接把文件后缀改成 .class文件就想欺骗 JVM）
- 准备：为类的静态变量分配内存，并讲起初始化为默认值（注意这里只是静态变量，实例变量是必须等到对象生成的时候才初始化的）
- 解析：把类中的符号引用转换为直接引用


初始化： 为类的静态变量赋予正确的初始值



 
## 类的使用方式
### 主动使用（六种）
- 创建类的实例 （new Test());
- 访问某个类或接口的静态变量，或者对该静态变量赋值 (int b = Test.a; 或者 Test.a = b) ( 注意！！！静态变量，不是常量！！！编译时 final 常量！！）
- 调用类的静态方法（Test.doStaticMethod() )
- 反射 ( Class.forName("indi.sword.Test")
- 初始化一个类的子类 ( class Parent{} ,class Child extends Parent{ static int a; } , class Test{ main{ Clild.a = 1} }
- Java虚拟机启动时被标明为启动类的类（java Test，方法的入口类）

### 被动使用
除了以上六种情况，其他使用JAVA类的方式都被看作是对类的被动使用，都不会导致类的初始化。

```
public class Test02 {
    public static void main(String[] args) {
        System.out.println(FinalTest.x); // 引用静态的常量，不属于类的主动使用 
        System.out.println("---------------");
        System.out.println(FinalTest.y);
    }
}

class FinalTest{
    public static final int x = 2; // 静态常量 -- final，不会再变的了
    public static int y = 2; // 静态变量

    static{
        System.out.println("FinalTest static block");
    }
}
```
```
public class Test03 {
    public static void main(String[] args) {
        System.out.println(FinalTest02.x);
    }
}
class FinalTest02{
    //这个属于运行时常量。加载链接初始化 的第三阶段 初始化才赋值的
    public static final int x = new Random().nextInt(100);

    static{
        System.out.println("FinalTest2 static block");
    }
}

```

<hr >
### 一、类的加载

> 类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构。

#### 加载.class文件的方式
- 从本地系统中直接加载
- 通过网络下载.class文件
- 从zip，jar等归档文件中加载.class文件
- 从专有数据库中提取.class文件
- 将Java源文件动态编译为.class文件

    
 
> 类的加载的最终产品是位于堆区中的Class对象
Class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口

#### 类加载器
JAVA虚拟机自带的类加载器
 - 根类加载器（Bootstrap）（使用C++编写，程序员无法再JAVA代码中获得该类）
 - 扩展类加载器（Extension）（使用JAVA代码实现）
 - 系统类加载器（System）（也叫应用加载器，使用JAVA代码实现）

用户自定义的类加载器
 - java.lang.ClassLoader的子类
 - 用户可以定制类的加载方式


注意：
 - 类加载器并不需要等到某个类被“首次主动使用”时再加载它
 - JVM规范允许类加载器在预料到某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了 .class 文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误（LinkageError 错误）
 - 如果这个类一致没有被程序主动使用，那么类加载器就不会报告错误。

```
public class Test01 {
    public static void main(String[] args) throws Exception{
        Class clazz = Class.forName("java.lang.String");
        System.out.println(clazz.getClassLoader()); // null 

        Class clazz2 = Class.forName("indi.sword.util.jvm.Test00");
        System.out.println(clazz2.getClassLoader()); // sun.misc.Launcher$AppClassLoader@14dad5dc
    }
}

```

###  二、类的连接 （验证 --> 准备 --> 解析）
> 类被加载后，就进入连接阶段。连接就是将已经读入到内存的类的二进制数据合并到虚拟机的运行时环境中去。

####  1、类的验证

类的验证的内容
> 类文件的结构检查、语义检查、字节码检查、二进制兼容性的验证

类的验证详解：

 - 类文件的结构检查：确保 .class 类文件遵从 JAVA类文件的固定格式。 （你随便搞个txt换个 .class后缀肯定过不去）
 - 语义检查 ： 确保类本身符合 Java 语言的语法规定，比如验证 final 类型的类有没子类，以及 final类型的方法没有被覆盖。（虽然你可能会问，类似操作的话，编译器就过不了了，不可能有这种情况，但是有些人恶意修改 .class文件也不是不可能）
 - 字节码检查：确保字节码流可以被 Java 虚拟机安全地执行。字节码流代表 Java 方法（包括静态方法和实例方法），它是由被称作操作码的单字节指令组成的序列，每一个操作码后都跟着一个或多个操作数。字节码验证步骤会检查每个操作码是否合法，即是否有着合法的操作数。）
 - 二进制兼容性的验证 ：确保相互引用的类之间协调一致。例如在 worker 类的 gotowork() 方法中调用了 Car类的 run() 方法。 Java 虚拟机在验证 Worker类时，会检查在方法区内是否存在 Car 类的 run() 方法，假如不存在 （ 当 worker 类和 Car 类 jdk编译版本不兼容，比如woker.class是在jdk5上面编译的， car.class是在jdk6上面编译的 就会出现问题），就会抛出 NoSuchMethodError 错误。

####  2、类的准备

在准备阶段，Java 虚拟机为累的静态变量分配内存，并设置默认的初始值，例如对于以下Sample类，在准备阶段，将为 int 类型的静态变量 a 分配 4 个字节的内存空间，并且赋予默认值 0，为 long 类型的静态变量 b 分配 8 个字节的内存空间，并且赋予默认值 0 。
```
public class Sample{
    private static int a = 1;
    public static long b;

    static{
        b = 2;
    }
}

```
####  3、类的解析
在解析阶段， Java 虚拟机会把类的二进制数据中的符号引用替代为直接引用。例如在worker类的 gotoWork() 方法中会引用 Car类的run()方法。
```
public void gotoWork(){
    car.run(); // 这段代码在 Worker 类的二进制数据中表示为符号引用
}
```
在 Worker 类的二进制数据中，包含了一个对 car类的 run() 方法的符号引用，它由 run() 方法的全名和相关描述符组成。在解析阶段， Java 虚拟机会把这个符号引用替换成一个指针，该指针指向 car 类的 run() 方法在方法区内的内存位置，这个指针就是直接引用。

###  三、类的初始化
在初始化阶段， Java 虚拟机执行了类的初始化语句，为类的静态变量赋予了初始值。在程序中，静态变量的初始化有两种途径：
 - 1、在静态变量的声明处进行初始化。
 - 2、在静态代码块中进行初始化。

例如在以下代码中，静态变量 a 和 b 都显示初始化，而静态变量 c 没有被显示初始化，它将保持默认值 0 .( 对比上述 2.2的类的准备）
```
public class Sample{
    private static int a = 1;  // 在静态变量的声明处进行初始化
    public static long b; 
    public static long c;

    static{
        b = 2; // 在静态代码块中进行初始化
    }
}
```
静态变量的声明语句，以及静态代码块都被看做是类的初始化语句，Java 虚拟机会按照初始化语句在类文件中的先后顺序来依次执行它们。
以下例子， 结果 a = 4； （记住：上述 2.2的类的准备会把 0 赋给 a )
```
class Sample{
    private static int a = 1;  // 在静态变量的声明处进行初始化
    static { a = 2;}
    static { a = 4;}
}

```

#### 类的初始化步骤
 - 1、假如这个类还没有被加载和连接，那就先进行加载和连接。
 - 2、假如类存在直接的父类，并且这个父类还没有被初始化，那就先初始化直接的父类。（但是！！！ 在初始化一个类时，不会初始化它所实现的接口。在初始化一个接口时，并不会先初始化它的父接口。因此，一个父接口并不会因为它的子接口或者实现类的初始化而初始化。只有当程序首次使用特定接口的静态变量时，才会导致该接口的初始化。）
 - 3、假如类中存在初始化语句，那就依次执行这些初始化语句。

注意：
- 只有当程序访问的静态变量或静态方法确实在当前类或当前接口中定义时，才可以认为是对类或接口的主动使用。
- 调用 ClassLoader 类的 loadClass 方法加载一个类，并不是对类的主动使用，不会导致类的初始化。

```
public class Test04 {
    static{
        System.out.println("Test04 static block");
    }
    public static void main(String[] args) {
        // 只有当程序访问的静态变量或静态方法确实在当前类或当前接口中定义时，才可以认为是对类或接口的主动使用。
        System.out.println(child.a); // 不会初始化 Child类
    }
}
class Parent{
    static int a = 3;
    static {
        System.out.println("Parent static block");
    }
}
class child extends Parent{
    static{
        System.out.println("Child static block");
    }
}
```
```
public class Test05 {
    static{
        System.out.println("Test05 static block");
    }

    public static void main(String[] args) {
        Parent2 parent2;
        System.out.println("----------------------");
        parent2 = new Parent2();

        System.out.println(Parent2.a);
        System.out.println(Child2.b);
    }
}

class Parent2{
    static int a = 3;
    static {
        System.out.println("Parent2 static block");
    }
}
class Child2 extends Parent2{
    static int b =4;
    static {
        System.out.println("Child2 static block");
    }
}

运行结果：
Test05 static block
----------------------
Parent2 static block
3
Child2 static block
4
```

```
public class Test06 {
    public static void main(String[] args) {
        System.out.println(Child3.a);
        System.out.println("-------------------");
        // 只有当程序访问的静态变量或静态方法确实在当前类或当前接口中定义时，才可以认为是对类或接口的主动使用。
        Child3.doSomething();
    }
}
class Parent3{
    static int a = 3;
    static {
        System.out.println("Parent3 static block");
    }
    static void doSomething(){
        System.out.println("do something");
    }
}
class Child3 extends Parent3{
    static {
        System.out.println("Child3 static block");
    }
}

运行结果：
Parent3 static block
3
-------------------
do something
```
```
public class Test07 {
        public static void main(String[] args) throws Exception{

        // 获得系统类加载器
        ClassLoader loader = ClassLoader.getSystemClassLoader();

        // 调用 ClassLoader 类的 loadClass 方法加载一个类，并不是对类的主动使用，不会导致类的初始化。
        // 加载 连接 初始化。 只是加载
        Class<?> clazz = loader.loadClass("indi.sword.util.jvm.CL");
        System.out.println("--------------------");
        clazz = Class.forName("indi.sword.util.jvm.CL");
    }
}

class CL{
    static{
        System.out.println("class CL");
    }
}

运行结果：
--------------------
class CL
```


## 类加载器
  类加载器用来把类加载到 Java 虚拟机中。从 JDK 1.2版本开始，类的加载过程采用了父类委托机制，这种机制能更好地保证 Java 平台的安全。在此委托机制中，除了 Java 虚拟机自带的根类加载器（Bootstrap） 外，其余的类加载器都有且只有一个父加载器。当 Java 程序请求加载器 loader1 加载 Sample类时，loader1 首先委托自己的父加载器去加载 Sample 类，若父加载器能加载，则由父加载器完成加载任务，否则才由加载器 loader1 本身加载 Sample 类。

 
1、JAVA虚拟机自带的类加载器
- 根（Bootstrap）类加载器： 该加载器没有父加载器。它负责加载虚拟机的核心类库，如 java.lang.* 等。例如 java.lang.String 就是由根类加载器加载的。根类加载器从系统属性 sun.boot.class.path 所指定的目录中加载类库。根类加载器的实现依赖于底层操作系统，属于虚拟机的实现的一部分，它并没有继承 java.lang.ClassLoader 类。
- 扩展类加载器（Extension）: 它的父加载器为根类加载器。它从 java.ext.dirs 系统属性所指定的目录中加载类库，或者从 JDK 的安装目录的 jre\lib\ext 子目录（扩展目录）下在家类库，如果把用户创建的 JAR 文件放在这个目录下，也会自动由扩展类加载器加载。扩展类加载器是纯 Java 类，是 Java.lang.ClassLoader 类的子类。
- 系统类加载器（System）：也叫应用加载器，它的父加载器为扩展类加载器。它从环境变量 classpath 或者 系统属性 java.class.path 所指定的目录中加载类，它是用户自定义的类加载器的默认父加载器。系统类加载器是纯 Java 类，是java.lang.ClassLoader 类的子类。

2、用户自定义类类加载器
 >除了以上虚拟机自带的加载器外，用户还可以定制自己的类加载器（User-defined Class Loader）。Java 提供了抽象类 java.lang.ClassLoader，所有用户自定义的类加载器应该继承 ClassLoader 类，然后覆盖它的 findClass(String name)方法即可，该方法依据参数指定的类的名字，返回对应的Class对象的应用。

    根类加载器（bootstrap）   <--  扩展类加载器（Extend） <-- 系统类加载器（System）  <-- 用户自定义类加载器


 
## 类加载的父委托机制
 - 在父亲委托机制中，各个加载器按照父子关系形成了树形结构，除了根类加载器以外，其余的类加载器都有且只有一个父加载器（可以理解为直接继承）。

 
直接看下面例子是怎么 loadClass的 ↓↓↓

```
Class clazz1 = loader.loadClass("Sample");
```
### 用户自定义类类加载器
代码例子：
MyClassLoader.java
```
package indi.sword.util.jvm.UserClassLoader;

import java.io.*;

public class MyClassLoader extends ClassLoader {

    private String name; //类加载器的名字
    private String path = "d:\\"; // 加载类的路径
    private final String fileType = ".class"; // class 文件的扩展名

    public MyClassLoader(){
        super(); //让系统类加载器成为该类加载器的父加载器
    }

    public MyClassLoader(String name){
        super(); //让系统类加载器成为该类加载器的父加载器
        this.name = name;
    }

    public MyClassLoader(ClassLoader parent,String name){
        super(parent); // 显式指定该类加载器的父加载器
        this.name = name;
    }

    public static void main(String[] args) throws Exception{

        // 根加载器null --> 扩展类加载器Extends --> 应用类加载器 --> loader1
        MyClassLoader loader1 = new MyClassLoader("loader1");
        loader1.setPath("d:\\myapp\\serverlib\\");

        // 根加载器null --> 扩展类加载器Extends --> 应用类加载器 --> loader1 --> loader2
        // 参数 loader1 将作为 loader2 的父加载器（loader2 包装了 loader1 ，具体看构造方法）
        MyClassLoader loader2 = new MyClassLoader(loader1,"loader2");
        loader2.setPath("d:\\myapp\\clientlib\\");

        // 根加载器null --> loader3
        MyClassLoader loader3 = new MyClassLoader(null,"loader3");
        loader3.setPath("d:\\myapp\\otherlib\\");

        test(loader2);
        System.out.println("---------------");
        test(loader3);
        System.out.println("---------------");
    }

    public static void test(ClassLoader loader) throws Exception{
        /*
            注意：第三层的“应用类加载器”会自动加载环境变量classPath指定的文件路径。 .; 表示当前目录。

            当前编译完目录下肯定没有 Sample.class 文件，但是有 indi.sword.util.jvm.UserClassLoader.Sample.class 文件，请认清两者的区别
            每次加载都会从最上层加载器往下方加载，那么test(loader2);
            加载顺序就是  BootStrap根加载器 --> 扩展类加载器 --> 应用类加载器，前俩个加载的是系统的 Class 文件，所以从第三个看起就好了。
            很明显，“应用类加载器”找不到 Sample.class 文件，
            那么就会去找父加载器 loader1 指定的目录即"d:\\myapp\\serverlib\\" 下有没有 Sample.class 有的话就加载并逐层返回（子类就负责返回就行了，不用再去加载 Class 了），
            没有的话，就加载 loader2 指定的目录即"d:\\myapp\\clientlib\\" 下的Sample.class。找到的话，加载成功并且逐层返回。
            没有的话就报“ClassNotFoundException”，依此类推。

            那么如果是loader3的两层结果情况，那么根加载器bootstrap显然无法加载 Sample.class，
            那么就直接取找 loader3目录下的"d:\\myapp\\otherlib\\" Sample.class文件，
            找到的话，加载成功并且返回。没有的话就报“ClassNotFoundException”。
         */
        Class clazz1 = loader.loadClass("Sample");
        Object object1 = clazz1.newInstance(); // 小tip，newInstance()是对类的主动使用，会加载类


        /*
            下面这个类，可以被“应用类加载器”在当前目录下加载并逐层返回。
            但是 loader3 会报“ClassNotFoundException”
         */
//        Class clazz2 = loader.loadClass("indi.sword.util.jvm.UserClassLoader.Sample");
//        Object object2 = clazz2.newInstance();
    }
    /*
        复写这个方法，因为 ClassLoader.loadClass(String,boolean) 方法会用到
        idea ： Crrl + Alt + B 这个方法就可以找到引用
     */
    @Override
    public Class<?> findClass(String name) throws ClassNotFoundException {
        Class<?> clazz = null;
        try {
            byte[] b = loadClassData(name);
            // Converts an array of bytes into an instance of class
            if(b.length > 0){
                clazz = defineClass(name,b,0,b.length);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return clazz;
    }

    /*
        加载二进制文件，把数据加载到内存中来
     */
    private byte[] loadClassData(String name){
        byte[] data = new byte[0];
        try {
            InputStream is = null;
            data = null;
            ByteArrayOutputStream baos = null;

            this.name = this.name.replace(".","\\");
            String fileName = path + name + fileType;
            is = new FileInputStream(new File(fileName));
            baos = new ByteArrayOutputStream();
            int ch = 0;
            while( -1 != (ch = is.read())){
                baos.write(ch);
            }
            data = baos.toByteArray();

            baos.close();
            is.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return data;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPath() {
        return path;
    }

    public void setPath(String path) {
        this.path = path;
    }

    public String getFileType() {
        return fileType;
    }

    @Override
    public String toString(){
        return this.name + " - " +  this.getClass().getClassLoader().getClass().getName();
    }
}


```
Dog.java
```
package indi.sword.util.jvm.UserClassLoader;

public class Dog{
    public Dog(){
        System.out.println("Dog is loaded by : " + this.getClass().getClassLoader());
    }
}

```
Sample.java
```
package indi.sword.util.jvm.UserClassLoader;

public class Sample{
    public int v1 = 1;
    public Sample(){
        System.out.println("Sample is loaded by : " + this.getClass().getClassLoader().toString());
        new Dog();
    }
}

```
 - 若有一个类加载器能成功加载Sample类，那么这个类加载器被称为Sample类的“定义类加载器”，所有能成功返回 Class 对象引用的类加载器称为“初始类加载器”（包括了“定义类加载器”）
```
假设 loader1 实际加载了 Sample类（不是他爸爸加载的），则 loader1 为 Sample 类的“定义类加载器”，loader2 和 loader1 为 Sample 类的 “初始类加载器”。
```

####  需要重要指出的是，加载器之间的父子关系实际上指的是加载器对象之间的包装关系，而不是类之间的继承关系。一对父子加载器可能是同一个加载器类的两个实例，也可能不是。在子加载器对象中包装了一个父加载器对象。上面例子的 loader1 和 loader2 都是自定义类的 MyClassLoader 的实例，并且 loader2 包装了 loader1 ，loader1 是loader2 的父加载器。
```
// 根加载器null --> 扩展类加载器Extends --> 应用类加载器 --> loader1
MyClassLoader loader1 = new MyClassLoader("loader1");
loader1.setPath("d:\\myapp\\serverlib\\");

// 根加载器null --> 扩展类加载器Extends --> 应用类加载器 --> loader1 --> loader2
// 参数 loader1 将作为 loader2 的父加载器（loader2 包装了 loader1 ，具体看构造方法）
MyClassLoader loader2 = new MyClassLoader(loader1,"loader2");
loader2.setPath("d:\\myapp\\clientlib\\");
```
```
public class Test08 {
    public static void main(String[] args) {
        MyClassLoader loader1 = new MyClassLoader("loader1");
        MyClassLoader loader2 = new MyClassLoader(loader1,"loader2");
        System.out.println(loader2);
        System.out.println("---------------------------");
        ClassLoader classLoader = loader2;
        while( null != classLoader){
            classLoader = classLoader.getParent();
            System.out.println(classLoader);
        }
    }
}

运行结果：
loader2  //   用户自定义的类加载器 loader2
---------------------------
loader1   // 用户自定义的类加载器 loader1
sun.misc.Launcher$AppClassLoader@14dad5dc // 系统类System加载器
sun.misc.Launcher$ExtClassLoader@312b1dae  // 扩展类Extend加载器
null // bootstrap根类加载器

```


#### 父委托机制的优点
> 能够提高软件系统的安全性。因为在此机制下，用户自定义的类加载器不可能加载应该由父加载器加载的可靠类，从而防止不可靠甚至恶意的代码代替由父加载器加载的可靠代码。
```
例如，java.lang.Object 类总是由根类加载器加载，其他任何用户自定义的类加载器都不可能加载含有恶意代码的 java.lang.Object 类。
```
    
#### 类的命名空间（类加载器、所有父加载器所加载的类）
```
    每个类加载器都有自己的命名空间，命名空间由该加载器及所有父加载器所加载的类组成。
    在同一个命名空间中，不会出现类的完整名字（包括类的报名）相同的两个类；在不同的命名空间中，有可能会出现类的完整名字（包括类的报名）相同的两个类。（比如你也来新建一个 java.lang.Object 类，这个呢本应该是由“根加载器”加载的，可是用户自己定义的都是由“应用类加载器”加载的） 
```
```
    同一个命名空间内的类是相互可见的。
    子加载器的命名空间包含所以父加载器的命名空间。因此子加载器加载的类能看见父加载器加载的类。例如系统类加载器加载的类能看见根类加载器加载的类（如 java.lang.* )。
    由父加载器加载的类不能看见子加载器加载的类。（“根类加载器”看不到“应用加载器”加载的类，loader1加载的类也看不到 loader2 加载的类。
    如果两个加载器之间没有直接或间接的父子关系，那么它们各自加载的类相互不可见。
```

下面的例子正面 命名空间之间是隔离的。
例子1 ，启动类与Sample类不属于同一个命名空间。
```
/*
    启动类是 系统类加载器加载的，属于爸爸。 Sample类是 loader1加载的，属于儿子。
    下面的类在 cmd下跑会报错。java.lang.NoClassDefFoundError: Sample。因为爸爸看不到儿子。

    记住了，这个类是要连同 Sample MyClassLoader一起丢在 default包，
    然后编译后的 class 文件拿去 D:\myapp\syslib 目录下，打开 cmd跑的，
    (可能你在default包下可以运行成功，因为都是同一个命名空间，当然没问题）
    然后记得把 syslib 上不要放Sample类，不然不会用 loader1加载，切忌。
    不然你还真测试不出来不同命名空间的差异出来。
 */
public class TestNameZone {
    public static void main(String[] args) throws Exception{
        System.out.println("本启动类的类加载器 --> " + new TestNameZone().getClass().getClassLoader()); // 系统类加载器：sun.misc.Launcher$AppClassLoader@14dad5dc

        MyClassLoader loader1 = new MyClassLoader("loader1");
        loader1.setPath("d:\\myapp\\otherlib\\");
        Class clazz = loader1.loadClass("Sample"); // 创建一个 Sample 类对象。 loader1加载的。
//        Class clazz = loader1.getParent().loadClass("Sample"); // 如果让他爸爸加载，那就是在同一个命名空间，只不过你需要把Sample 和Dog都加目录去。
        Object object = clazz.newInstance();
        Sample sample = (Sample)object; // 这一句报错，为什么呢。 因为object的类命名空间在 loader1里面，启动类爸爸根本就看不到。
        System.out.println(sample.v1);
    }
}

运行结果：
 D:\myapp\syslib>java TestNameZone
        本启动类的类加载器 --> sun.misc.Launcher$AppClassLoader@4e0e2f2a
        Sample is loaded by : loader1 - sun.misc.Launcher$AppClassLoader
        Dog is loaded by : loader1 - sun.misc.Launcher$AppClassLoader
        Exception in thread "main" java.lang.NoClassDefFoundError: Sample
                at TestNameZone.main(TestNameZone.java:10)
        Caused by: java.lang.ClassNotFoundException: Sample
                at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
                at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
                at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:331)
                at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
                ... 1 more
      
```
#### 运行时包
```
    在同一类加载器加载的属于相同包的类组成了运行时包。
    决定两个类是不是属于同一个运行时包，不仅仅要看他们的包名是否相同，还要看类加载器是否相同。只有属于同一运行时包的类才能相互访问包课件（即defautl级别）的类和类成员。这样的限制能避免用户自定义的类冒充核心类库的类，去访问核心类库的包的可见成员。假设用户自己定义了一个类 java.lang.spy，并且由用户自定义的类加载器加载，由于 java.lang.Spy 和核心类库 java.lang.* 由不同的加载器加载，他们属于不同的运行时包，所以 java.lang.spy 不能访问核心类库 java.lang包中的包可见成员。
```



 
 

 

 
 

 
## 类的卸载
```
    当 Sample 类被加载、连接、初始化后，它的生命周期就开始了。当代表 Sample 类的 Class 对象不再被引用，即不可触及时，Class 对象就会结束生命周期， Sample 类在方法区内的数据也会被卸载，从而结束 Sample 类的生命周期。由此可见，一个类合适结束生命周期，取决于代表它的 Class 对象何时结束生命周期。
    由 Java 虚拟机自带的类加载器所加载的类，在虚拟机的生命周期中，始终不会被卸载。
    那么Java虚拟机自带的类包括，bootstrap根类加载器、extend扩展类加载器、System系统类加载器。Java虚拟机本身会始终引用这些类加载器，而这些类加载器则会始终引用它们所加载的类的 Class 对象，因此这些 Class 对象始终是可触及的。
```


#####  由用户自定义的类加载器所加载的类是可以被卸载的！！！
例子证明：
```

/*
    当 Sample 类被加载、连接、初始化后，它的生命周期就开始了。当代表 Sample 类的 Class 对象不再被引用，即不可触及时，
    Class 对象就会结束生命周期， Sample 类在方法区内的数据也会被卸载，从而结束 Sample 类的生命周期。
    由此可见，一个类合适结束生命周期，取决于代表它的 Class 对象何时结束生命周期。
    由 Java 虚拟机自带的类加载器所加载的类，在虚拟机的生命周期中，始终不会被卸载。
    那么Java虚拟机自带的类包括，bootstrap根类加载器、extend扩展类加载器、System系统类加载器。Java虚拟机本身会始终引用这些类加载器，而这些类加载器则会始终引用它们所加载的类的 Class 对象，因此这些 Class 对象始终是可触及的。
 */
//  由用户自定义的类加载器所加载的类是可以被卸载的！！！
public class TestUnloadClassLoader {
    public static void main(String[] args) throws Exception{

        /*
            为什么直接从根加载器加载呢？
            因为我不受 应用类加载器的影响，如果你直接 new MyClassLoader("loader1"); 的话，
            它会从应用类加载器加载，那么由于应用类加载器不会被卸载，所以待会 Sample 类都是由应用类（系统类）加载器加载的，
            那么不会二次加载 Sample。
            由于我们是要测试卸载自定义类，所以需要直接继承根类加载器，或者！你把这个编译好的类拉到 d:\\myapp\\syslib 下面去跑。
            （因为那个目录下没有 Sample类，那么就是应用类加载器在当前目录 .; 没法加载，会让子类 loader1 去对应目录下加载。
         */
//        MyClassLoader loader1 = new MyClassLoader("loader1");
        MyClassLoader loader1 = new MyClassLoader(null,"loader1");
        loader1.setPath("D:\\myapp\\otherlib\\"); // 记得在这个目录下丢一个 Sample 类

        Class clazz = loader1.loadClass("Sample");
        System.out.println(clazz.hashCode());
        Object obj = clazz.newInstance();

        obj = null;
        clazz = null;
        loader1 = null;
        /*
            loader1.getParent() = null; // 这个是编译会报错的。
            因为 ： public final ClassLoader getParent(); final 方法，
            保护了应用类加载器以及头上的 ext加载器 与 bootstrap加载器不会被修改。
         */
//        loader1 = new MyClassLoader("loader1");
        loader1 = new MyClassLoader(null,"loader1");

        loader1.setPath("D:\\myapp\\otherlib\\");
        clazz = loader1.loadClass("Sample");
        System.out.println(clazz.hashCode()); // hashCode 不一样说明就是不同的对象。
    }
}
-----------------------
运行结果：
666641942
Sample is loaded by : loader1
Dog is loaded by : loader1
1338668845
```


 