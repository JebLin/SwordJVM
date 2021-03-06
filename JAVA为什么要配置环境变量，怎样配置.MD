# JAVA为什么要配置环境变量，怎样配置
<hr />

#### 安装JDK后要配置环境变量，主要有三个：

 - 1 JAVA_HOME ->为JDK的安装目录，如：F:\JAVA\jdk1.6.0_04
 - 2 CLASSPATH ->到哪里找需要执行的.class文件，如: .;%JAVA_HOME%\lib;
前面是一个点[.]再加一个分号[;]再以分号结束[;]k
为什么要加点[.]在编译java文件时，如javac xxx.java 即会在当前目录下生成xxx.class文件,
然后再用java xxx执行，系统会在当前目录下找xxx.class文件，所以要加点，代表是在当前目 录下找class文件来执行。
 - 3 path ->javac.exe的执行程序路径，如：%JAVA_HOME%\bin;
开始菜单->运行->cmd，在控制台里输入regedit,回车后看到注册表出来了！其过程是当输入regedit时候，系统会去系统环境变量里找这个程序的可执行文件路径。打开系统环境变量的设置地方，会看到系统变量windir的值为%SystemRoot%(其实也就是c:\WINDOWS若系统装在c盘的话),我们再打开c:\WINDOWS，可以找到一个叫regedit.exe的程序！


#### 设置系统变量的目的：为了使系统可以找到程序的运行路径！
> 在设置环境变量path时候，之所以要选择C:\Java\jdk1.6.0\bin（JDK的安装目录）的原因呢？打开该目录，发现里面全是一些可执行文件，其中有我们最熟悉的javac.exe,java.exe等等。是啊，之所以要选择bin是因为当我们在编译程序的时候，在控制台输入javac Hello.java是，系统要去系统环境变量下面找javac.exe的执行程序路径。

首先得介绍下面几个参数的作用：
##### current directory(当前目录)：
当前在用的目录就是当前目录。比如说当你打开NOTEPAD，并处于运行状态时候，当前目录就是c:/windows；如果你用cmd命令打开命令行窗口，当前目录就是c:\windows\system32; 如果你在用java这条指令，当前目录就是JAVA下的BIN目录所在的路径，因为java.exe在bin里面。
在java开发配置环境变量时，系统默认(我们对classpath不做任何设定时)的路径也是当前目录。

##### JAVA_HOME:
它是指jdk的安装目录，像D:\j2sdk1.4.2_16。
可以新建一个JAVA_HOME变量，它的取值为D:\j2sdk1.4.2_16。
为什么要设置它呢，不设定可不可以呢？不设定也是可以滴，但是最好还是设置一下。
我们现在就当它是一个变量代换 JAVA_HOME = D:\j2sdk1.4.2_16，就是为了避免多写字，它还有一个好处就是当我们需要改变某个jdk时，只需要改JAVA_HOME的值就可以了。等在后面看了Tomcat的启动分析时你就明白了。
当在环境变量中引用它的时候要用%JAVA_HOME%来表示D:\j2sdk1.4.2_16。
##### Path：
系统变量Path告诉操作系统可执行文件(*.exe、*.bat等)所在的路径，当OS(操作系统)发现某个*.exe时，windows默认从当前目录开始查找这个命令，若查不到，OS就会到Path所设定的路径中去寻找该命令，然后执行。
```
系统默认的系统变量为：Path = %SystemRoot%; %SystemRoot%\system32; %SystemRoot%\System32\Wbem
就是说处于上面3个目录(多个变量用分号隔开)中的*.exe文件，可以在任意地方被执行(在 运行窗口能直接执行的命令，像cmd、notepad等，基本都在上面的3个目录里面)，所以他们可以直接运行。
上面的%SystemRoot%是什么意思呢？%SystemRoot%就是安装操作系统的时候，系统默认的安装路径
若你的windows xp装在C:/WINDOWS
则你的%systemRoot%路径就是c:/windows
%systemRoot%只是一个符号,代表你的系统安装目录
下面是常见系统默认安装路径:
98----c:/windows
2000--c:/winnt
2003--c:/windows
xp----c:/windows
```
当我们要进行java开发时，OS经常需要用到java.exe、javac.exe等，（若jdk安装在D:\j2sdk1.4.2_16）因此应该将D:\j2sdk1.4.2_16\bin（%JAVA_HOME%\bin）加入到系统的path中去。
注意：如果你加入的位置不是在最后，那还需要在bin后面加上英文状态下的分号：%JAVA_HOME%\bin；多个变量之间要用分号隔开，如果它前面没有，你就加一个。
明确一下：%JAVA_HOME%\jre\bin 这个路径是不需要加入Path的。参考：http://java.sun.com/javase/6/docs/technotes/tools/windows/jdkfiles.html

##### CLASSPATH：
 - CLASSPATH是专门针对java的，它相当于windows的path；path是针对整个windows的。
 - CLASSPATH告诉java虚拟机(jvm)要使用或执行的*.class文件放在什么地方。

> 所谓的JVM就好像是在微软OS上面再激活另外一个OS，对JVM来说CLASSPATH就好像是对微软OS来说的PATH，所以要用jvm开运行程序就需要设定classpath，然而jvm像windows一样它也有个默认的查找class文件的路径，对刚开始学习java的我们来说，默认的已经够我们用了，那就是当前路径，因此不设置classpath也可以。
在windows中 classpath 大小写没有关系，其他的环境变量名称也一样。
```
参照：http://java.sun.com/javase/6/docs/technotes/tools/windows/classpath.html
    如果你在classpath中用到 other resource files，那说明你已经熟悉classpath了，你可以自己随意设置。

    当我们不设定classpath时，系统默认的classpath是当前目录，如果你个人想设置classpath的话，那么务必在classpath中加入" ." ，这个英文状态下的点就表示当前目录。
    至于classpath中要不要加入其他的路径(包括文件目录、包的根目录等)，这要看开发的需要，一般我们初学者是用不到的。
```
##### path的设置倒没有什么疑惑，但是classpath就有疑惑了，到底里面该不该加rt.jar ,dt.jar ,tool.jar呢？
可以肯定的是rt.jar是不需要的，jdk会自动加载。
你可以用 java -verbose 命令来查看。
```
    我们可以来研究一下%JAVA_HOME%\lib底下的 tools.jar 和 dt.jar 是甚么咚咚：
http://java.sun.com/javase/6/docs/technotes/tools/windows/jdkfiles.html 或者 http://java.sun.com/j2se/1.4.2/docs/tooldocs/windows/jdkfiles.html
    那tools.jar,dt.jar到底需不需要加进classpath呢？我查过一些资料，在Sun JDK 1.2及后续版本中，包含了一组可在程序运行时刻编译和执行Java代码的API。这些API被包含在tools.jar类库中。这个功能允许Java程序在运行时动态编译、执行小的代码块，在有些情况下这个功能会让Java应用程序的架构更加灵活、开放。本来还有个例子我看得不是太懂，就不贴出来了。
    如果您现在还未明白什么是动态编译或者是否需要“动态编译java程序”，那就当作不需要；系统自己默认设定，阁下您也默认好了 ^_^
至于dt.jar包，在jdk1.4.0之后不用导入它，运行时会自动导入--这个是我在搜集了n个资料后选择的一个较安全放心的答案
```

CATALINA_HOME:
```
    长话短说了，tomcat启动就不分析了，告诉你怎么设就可以了，在Tomcat 4以前，用的是TOMCAT_HOME来表示Tomcat的安装目录，中间还用过TOMCAT_BASE，在Tomcat 4以后，采用了新的Servlet容器Catalina，所以环境变量的名字也改为了CATALINA_HOME，所以设置的时候不用把这3个变量都设了，要CATALINA_HOME就够了。如果你安装的是解压缩版的也可以不用设置CATALINA_HOME，直接双击bin里面的startup.bat就可以了，关闭的时候有shutdown.bat。
    综上：（在winXP中）以jdk安装目录为-D:\j2sdk1.4.2_16，tomcat安装目录为-D:\ApacheTomcat\apache-tomcat-6.0.14设置环境变量如下：你也可以参考http://java.sun.com/javase/6/docs/technotes/tools/windows/jdkfiles.html 自己决定那些是需要设置的。
JAVA_HOME = D:\j2sdk1.4.2_16

    Path 环境变量中在最前面加入(若系统原来没有就新建) %JAVA_HOME
```

