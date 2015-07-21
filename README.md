# Soot

------
> * **Soot**提供了四种不同的中间体表示法（IRS）作为分析使用，不同的IR有不同的抽象水平。四种分别是Baf，Grimp,Jimple,Shimple。

#### Soot数据结构 ：

> * SootClass 代表了一个单独的类加载到Soot。
> * 封装了常用的CURD，配合mybatis-generator 自动生成dao、model、mapper层，减少重复劳动，提高生产力，实现快速、平稳的开发
> * SootMethod 表示了一个类的一个方法
> * SootField 表示一个类的成员字段
> * Body 表示了一个方法体，并且对于不同的IRs有不同的表示形式。

------

## 基本Soot构造
> * 1、我们可以用body访问各种不同的信息。可以检索一个集合，可以访问局部变量，获得构成body的语句，异常处理。
* 2、声明：由单元接口所体现
* 3、价值：一个简单的基准表示一个值，可以是局部变量，常量，表达式等等。
* 4、引用：在Soot中引用被称为框（即Boxes），可分为两类ValueBox和UnitBox
* 5、ValueBox：我们需要指针指向值的概念，每一个单元有一个使用单元的值。
* 6、UnitBox：一些单元需要引用指定到其他单元，例如 goto语句。
## 中间表示
> ### Soot一共有四种中间表示即Baf,Jimple,Shimple,Grimp。每一种有各自的代码抽象。其中Baf就是字节码表示类似Java的字节码。Jimple是三地址表示形式。
> ### Baf 是一种基于堆栈的字节码表示形式，Baf中的指令说明对应于Soot中的单元（Unit），其中的所有指令都实现了Inst接口。并且Baf常用于字节码的分析、优化和转换。
> ###  Jimple 是基于类型化的三地址中间表示形式，在Soot中Jimple可以被直接创建。
 * 1).从字节码到Jimple的转换是先从字节码到非类型化的Jimple的转换。
 * 2).转换的一个重要的步骤就是表达式的线性化，由于是三地址的形式，每条语句至多仅仅指代3个本地变量或者常量。
 *  3).Jimple的语句对应于Soot单元。其中核心的语句有：NopStmt、IdentityStmt、AssignStmt。过程内的控制流语句：IfStmt、GotoStmt、TableSwitchStmt、LookupSwitchStmt。过程间的控制流语句：InvokeStmt、ReturnStmt、ReturnVoidStmt。监视语句：EnterMonitorStmt、ExitMonitorStmt。
 *  4).Jimple转换后的字节码是Java源代码语Java字节码之间的混合代码，其中局部变量有$符号表示，在源程序中的不是局部变量的没有$符号。
 * 5).线性化过程中把语句分析为三地址形式的，下面为例：
int x = (f.bar(2) + a) * b -->转换后--> $i4 = $i3 +i0;
i2 = $i4 *i1;
> ### Shimple:
* 它的表示形式可以理解成是Jimple表示形式的单个静态分配表，需要SSA形式的支持。Shimple和Jimple几乎上完全相同，但是有目的单一静态点有两处不同（即phi-node）。Shimple和Jimple分析后的Java代码只有两处不同，一个就是phi-node指令，另一个就是io变量值被分割成了好几个。
优点：由于Shimple进行的是显示的编码控制流，这样我们的可以很容易的进行分析Shimple代码。
> ### Grimp:Grimp比Jimple更加接近Java源代码，因此方便阅读，有三处不同：
* 1).表达式目录树不被线性化。
* 2).对象实例化和构造函数的调用已经被形成新的运算符。
* 3).新的局部变量没有$符号创建。
* Grimp中间表示可以从soot.grimp获取。
## Soot工具
> ### (1).Soot可以从命令行调用
java [javaoptions] soot.main(soot.jar) [sootoptions] classname
其中classname就是要分析的java类。
> ### (2).在Soot中有三种类型的类：
* 1).参数类：是指定到soot类中的类，使用命令行可以明确列出来的。
* 2).应用类：被Soot转化或分析的类。
* 3).库类：应用类所调用的却不是应用类的类，他们在分析或转换中被使用，但是本身不会参与转换。
> ### (3).影响类分类的两种模型：
* 1).应用模型：由参数类所引用的类就是应用类。
* 2).非应用模型：参数类所引用的类将是库类。
> ### (4).Soot的执行阶段
* 1).执行阶段被分成了几个包，第一步就是产生的Jimple代码被投向各种包，通过jbp（Jimple的body包）。
* 2).过程内分析：每个类都去处理得到信息，但是各个类之间并不会调用其他类所产生的信息。
* 3).过程间的分析：需要把Soot放在整个程序模型中。并且这些包产生的信息对其他Soot的其他路径上的节点是可获得的。
> ### (5)内部自带的分析
* 1).无指针分析
java soot.Main -f J -p jap jap.npcolorer on classname
* 2).数组边界分析

## 数据流程框架
> ### （1）.一般分为四个步
* 1）.分析的本质：提供了三种不同的实现，结果是形成两个maps。
* 2）.分析的近似水平：一次分析可分为可能分析和必须分析。
* 3）.执行流：分析工作真正执行的位置，大致分为两个部分：
 ** a）.将信息从中集合移动到外集合。
 ** b）.添加信息到输出集合上。
* 4）.初始状态：决定单元格的初始化内容为分析的切入点。
> ### 2.注释代码
* 目的是优化java程序，想与相关的代码标记信息，程序就可以使用这些信息来执行优化操作。
> ### 3.调用图形构造器
* 过程间的分析是一种重要的实体，在Soot中图形构造器可以调用所有已知的方法。
> ### 4.指向分析
* 目的就是计算一个有参方法，返回可能的目标的集合。
> ### 5.提取抽象的控制流图
* 利用Soot可以提取抽象的控制流图的中间表示形式提供给自己的分析用。

## Soot作为命令行工具的使用
> ### (1).soot.truck.jar包含了soot以及所有的库文件
http://gitlab.buptnsrc.com/flowdroid/soot
> ### (2).处理输入文件分为三类：
* 1).java源代码
* 2).java字节码（class文件）
* 3).Jimple源文件
> ### (3).在执行命令行的过程中需要注意的是Soot需要自己的classpath，根据这条路径上的jar文件或目录加载相应的库文件，默认情况该路径为空，所以执行时候要指定路径。如下命令（例如有Main.java源程序）：
* java -cp soot-truck.jar soot.Main -cp . -pp Main
  -pp就是指定了classpath

> ### (4).处理一个目录的文件需要使用 -process-dir选项：
java -cp soot-trunk.jar soot.Main -cp . -pp -process-dir /javaproject/
该命令就是处理javaproject目录下的所有文件。
> ### (5).处理某一个类型的文件需要-src-prec选项，一共有四种类型
* 1).c：指class文件。
* 2).only-class：只是使用class文件。
* 3).J :指jimple类型的文件。
* 4).java：指java源文件。

java -cp soot-trunk.jar soot.Main -cp . -pp -src-prec java
该命令就是去对本目录下的所有java文件进行处理。


