根据编译器的不同行为，常量又分为编译时常量和运行时常量
编译时常量肯定就是运行时常量，只是编译时常量在编译的时候就被计算执行计算，并带入到程序中一切可能用到它的计算式中

```
public class Test {
    public static void main(String[] args) {
        System.out.println(FinalTest007.x);
        System.out.println("------------");
        System.out.println(FinalTest007.y);
    }
}

class FinalTest007{

    // 静态常量,编译时常量 -- final，不会再变的了
    public static final int x = 2;

    //静态常量，运行时常量 -- "加载链接初始化"的第三阶段 "初始化" 才赋值的
    public static final int y = "test".length();

    // 静态变量 -- 会变的量
    public static int z = 2;

    static{
        System.out.println("FinalTest007 static block");
    }
}
```
以Java为例，static final int x = 2将是一个编译时常量，编译后的符号表中将找不到x，所有对x的引用都被替换成了2。

（1）a被作为编译期全局常量，并不依赖于类，而b作为运行期的全局常量，其值还是依赖于类的。

（2）编译时常量在编译时就可以确定值，上例中的a可以确定值，但是c在编译器是不可能确定值的。

（3）由于编译时常量不依赖于类，所以对编译时常量的访问不会引发类的初始化。



用样的原因，静态块的执行在运行时常量之前，在编译时常量之后

