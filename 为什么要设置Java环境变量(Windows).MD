#为什么要设置Java环境变量(Windows)
## 一、基本原理
```
    环境变量（environment variables）一般是指在操作系统中用来指定操作系统运行环境的一些参数，如：临时文件夹位置和系统文件夹位置等。
    环境变量是在操作系统中一个具有特定名字的对象，它包含了一个或者多个应用程序所将使用到的信息。例如Windows和DOS操作系统中的path 环境变量，当要求系统运行一个程序而没有告诉它程序所在的完整路径时，系统除了在当前目录下面寻找此程序外，还应到path中指定的路径去找。用户通过设置环境变量，来更好的运行进程。
    JAVA的初学者，必须要学会配置PATH环境变量，否则无法编译、运行JAVA程序。那么，为什么要配置PATH环境变量呢？PATH环境变量是做什么的呢？我们从一个常见的问题入手。
    当我们安装好JAVA后，若没有正确配置PATH环境变量，则在命令行窗口（开始->运行->输入cmd，回车）下运行命令java -version（显示JAVA的版本）命令时，会出现如下的错误提示信息：
```
```
C:\Users\jianbin_lin>java -version
'java' 不是内部或外部命令，也不是可运行的程序
或批处理文件。
```

    这条错误信息的意思是，你要求运行的“java”这条命令我不认识，不知道是个什么玩意儿。那么究竟这个“java”命令是何方神圣呢？在JDK的安装目录下（我的是D:\SOFTWARE\CODE\JDK1.8\INSTALL\bin），有一个bin目录。打开此目录，会发现大量的可执行文件，后缀名为exe的文件。其中有一个可执行文件即为java.exe。现在在命令行窗口下把当前目录切换到这个bin目录下（Shift+右键）。

```
D:\SOFTWARE\CODE\JDK1.8\INSTALL\bin>java -version
java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
```
```
    实际上我们执行“java”命令，就是要运行“java.exe”这个可执行文件（我们运行其他命令，本质上也是要执行某个程序文件）。
    与java命令类似，javac命令目前也能在bin目录下执行，但到其他目录下就只能写出其完整路径了。在学习JAVA的过程中，我们经常需要使用javac和java这两条命令来编译、运行我们的JAVA程序。难道我们每次运行这两条命令都要先切换到bin目录下吗或写出一长串的绝对路径吗？能不能有一个办法，让我们能够在任何目录下都能自如地运行这些命令呢？
明白了这些，我们就把bin目录的完整路径添加到PATH中。在命令行窗口下，可使用set命令完成此类的任务。直接运行set，会显示系统当前所有环境变量的值，运行set /?，会显示关于此命令的帮助信息。
　　使用set命令设置环境变量值的格式为：set 环境变量名=环境变量值。我们可以使用命令set path=D:\SOFTWARE\CODE\JDK1.8\INSTALL\bin将java等程序文件所在的目录添加到PATH环境变量中（Windows下环境变量名不区分大小写，这与UNIX不同）。但是这样会使PATH的值只有“D:\SOFTWARE\CODE\JDK1.8\INSTALL\bin”，它预先设定供其他程序使用的值就都被覆盖了。因此，我们应该把值“D:\Java\jdk1.6.0_24\bin”追加到PATH中。为此，我们可以使用如下命令：
```
```
C:\Users\Administrator>set path=%path%;D:\SOFTWARE\CODE\JDK1.8\INSTALL\bin
```

    把PATH放在两个百分号之间，指把PATH原有的值取出。其后的分号表示分隔不同的路径值，之后才是我们要添加的值。注意，请在英文输入法状态下使用此命令。现在我们转到任意一个目录下执行java -version命令试试。结果如下图所示：

```
C:\Users\Administrator>java -version
java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
```
        可见运行完全成功。但是这种使用set命令的方式设置的环境变量只对当前命令行窗口有效。一旦关闭此窗口，再次运行另一个命令行窗口时，PATH环境变量还是原来的值。因此，我们必须在Windows下修改PATH环境变量。步骤如下：
        右击桌面“我的电脑”图标->属性->高级->环境变量，就打开了了设置环境变量的对话框。上面部分是为某个用户设置。Windows设计为可多用户使用一台机器，每人一个账户。为某个账户设置的变量只对该用户有效。下面部分是设置系统变量，对系统中的每个用户有效。现在一般电脑都只有一个用户，即Administrator。因此只需要为此账户设置即可。找到PATH环境变量，点“编辑”。在“变量值”框内现有值的后面输入英文输入法的分号，然后添加你想添加的值（在这里是D:\SOFTWARE\CODE\JDK1.8\INSTALL\bin）。一路确定，OK！
现在再打开一个命令行窗口，输入echo %path%命令查看path的值（或者直接使用path命令亦可）。
```
C:\Users\Administrator>path
PATH=D:\SOFTWARE\CODE\Spring-boot-cli\spring-boot-cli-1.3.0.RELEASE-bin\spring-1.3.0.RELEASE\bin;D:\SOFTWARE\CODE\MYSQL\mysql-5.6.26-winx64\bin;D:\SOFTWARE\CODE\MAVEN\apache-maven-3.3.9\bin;D:\SOFTWARE\CODE\JDK1.8\INSTALL\bin;D:\SOFTWARE\CODE\JDK1.8\INSTALL\jre\bin;C:\ProgramData\Oracle\Java\javapath;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0
```
      其实我们执行其他命令都是这个原理。举例来说，当你显示PATH的值时，你会发现第一个值是C:\WINDOWS\system32。在Windows下打开这个目录，你会看到很多可执行文件，它们的名字与我们常用的命令的名字是一致的。也就是说，我们执行这些命令，实际上系统就是根据PATH的这个值找到了相应的可执行文件，再运行这些文件的结果。
      如cmd上输入notepad，会弹出一个记事本，因为系统会去path里面各个目录下找notepad.exe文件，于是呢，在C:\WINDOWS\system32找到了匹配
```
C:\Users\Administrator>notepad
```

## 二、内部命令、外部命令和批处理文件
```
    现在我们通过命令set path=（等号右边什么都没有，或随便写点东西）来清除PATH的值
    我们现在启动一个命令行窗口，按上述的方法把PATH的值全部清除，然后再运行dir、cd等命令。出乎我们意料的是，它们仍然能够正确执行。这，又是什么原因呢？
```
```
    我们来看看刚才那些报告错误的信息，它们都提到了“内部命令”和“外部命令”的概念。那么什么是内部命令和外部命令呢？
内部命令和外部命令是DOS（disk operating system，微软早期基于命令行的操作系统）时代的概念，百度百科的解释是：内部命令是随每次启动的COMMAND_COM装入并常驻内存，而外部命令是一条单独的可执行文件。粗略地讲，所谓内部命令就是最核心、使用最多的命令。为了提高响应速度，系统一启动，这些命令就被加载到内存，因此可以迅速、直接地执行；而外部命令由于使用相对较少，就不预先加载到内存，当用户使用时，再到硬盘上（c:\windows\system32）找相应的可执行文件，然后加载到内存执行。像dir、cd等都是内部命令，而诸如attrib、format等都是外部命令。尽管DOS的时代早已成为了历史，但某些操作却必须在命令行模式下完成，对专业人士来说更是如此。因此，Windows产品一直保留着命令行模式这个工具。
    另外一个概念就是批处理文件（后缀名为.bat，来源于批量的英语单词batch），它是另外一种可以执行的文件。简单地说，批处理文件包含了很多DOS命令。文件执行时，就一条一条地执行这些命令。不一定顺序执行，像通用的程序设计语言一样，它也有自己的流程控制。批处理文件创建很简单：用任何一个文本编辑器（如Windows的记事本）创建一个文本文件，然后把后缀名改为.bat即可。创建好的批处理文件，你也可以用文本编辑器打开，查看它的“源代码”。

```
## 三、关于CATALINA_HOME环境变量的设置（适用于tomcat6/7）

```
    这句话的意思就是没有正确设置CATALINA_HOME环境变量，而此变量是运行Tomcat所必须的。而当你不理会这个信息，在命令行窗口把当前目录切换到tomcat安装目录下的bin目录下，再执行批处理文件startup.bat来启动tomcat时，发现也可以成功。难道现在不需要CATALINA_HOME环境变量了吗？

    实际上，在startup.bat中，首先判断CATALINA_HOME是否为空。若为空，就把当前目录设为CATALINA_HOME的值。然后查找CATALINA_HOME所指示的目录下是否有个bin目录，此bin目录下是否有个叫做catalina.bat的批处理文件（此文件负责启动tomcat）。若没有，则把当前目录的父目录设为CATALINA_HOME的值，再重复上述的查找。若仍失败，就报图十所示的错误。因此，当你在tomcat安装目录下的bin目录下运行startup.bat文件时，它就会按照上面所说的第二种方式自动正确设置CATALINA_HOME的值，因为catalina.bat文件就在tomcat安装目录下的bin目录里。

    虽然在此种情况下能正确运行，但还是建议你正确设置CATALINA_HOME环境变量。与PATH不同的是，PATH系统本身就由，而CATALINA_HOME需要你自己“新建”，在“环境变量”窗口点“新建”即可。

    为了能够在任何目录下都能运行tomcat安装目录下bin目录里的批处理文件，如startup.bat，可把此目录添加到PATH环境变量中（我的是d:\tomcat7\bin）。为了提高灵活性，可在PATH环境变量的最后添加上%catalina_home%\bin（与前一个值之间不要忘了用英文的分号隔开）。把catalina_home放置在两个百分号之间，表示引用这个环境变量的值。这样，当你把tomcat的安装目录换了以后，就只需要修改catalina_home的值，而不需要修改PATH的值了，因为它可以自动计算出新的正确值。


```

