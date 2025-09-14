##GDB简单学习
###1. gbd的一些常见用法：

|命令|解释|示例|
| :----: | :----: | :----: |
| file <文件名> | 加载被调试的可执行文件名<br>因为一般都在被调试程序所在目录下执行GDB，因而文本名不需要带路径 | (gdb) file gdb-sample |
| r | Run的简写，运行被调试的程序。<br> 如果此前没有下过断点，则执行完整个程序；如果有断点，则程序暂停在第一个可用断点处。 | (gdb) r |
| c | Continue的简写，继续执行被调试程序，直至下一个断点或程序结束。 | (gdb) c |
| b <行号> <br> b<文件名：行号> <br> b <函数名称> <br> b *<函数名称> <br> b *<代码地址> <br> d [编号] | b: Breakpoint的简写，设置断点。两可以使用“行号”“函数名称”“执行地址”等方式指定断点位置。<br> 其中在函数名称前面加“*”符号表示将断点设置在“由编译器生成的prolog代码处”。如果不了解汇编，可以不予理会此用法。<br> d : Delete breakpoint的简写，删除指定编号的某个断点，或删除所有断点。断点编号从1开始递增。比如 d n，删除第n个断点。<br> clear n：表示清除第n行的断点。注意和d n的区别。 | (gdb) b 8 <br> (gdb) b main.cpp:12 <br> (gdb) b main <br> (gdb) b * main <br> (gdb) b *0x804835c <br> (gdb) d <br> (gdb) d 1 |
| s , n | s: 执行一行源程序代码，如果此行代码中有函数调用，则进入该函数；<br> n: 执行一行源程序代码，此行代码中的函数调用也一并执行。<br> s 相当于其它调试器中的“Step Into (单步跟踪进入)”；<br> n 相当于其它调试器中的“Step Over (单步跟踪)”。<br> <font color=#98549>这两个命令必须在有源代码调试信息的情况下才可以使用（GCC编译时使用“-g”参数）。</font>| (gdb) s <br> (gdb) n |
|finish| 结束内部函数的调用 | (gdb) finish |
| si , ni | 	si命令类似于s命令，ni命令类似于n命令。所不同的是，这两个命令（si/ni）所针对的是汇编指令，而s/n针对的是源代码。 | (gdb) si <br> (gdb) ni |
| start | 开始执行程序,在main函数的第一条语句前面停下来 |  |
| watch | 监视变量值的变化， 当设置的监视变量值发生变化时会提示 | watch j <br> watch i |
| p | Print的简写，显示指定变量（临时变量或全局变量）的值。 | (gdb) p variable <br> (gdb) p nGlobalVar |
| set var name = value | 在程序运行过程中动态改变变量的值 | set var i = 8 |
| display ... <br> undisplay <编号> | display，设置程序中断后欲显示的数据及其格式。<br> 例如，如果希望每次程序中断后可以看到即将被执行的下一条汇编指令，可以使用命令“display /i $pc” <br> <font color=#354986>其中 $pc 代表当前汇编指令，/i 表示以十六进制显示。</font>当需要关心汇编代码时，此命令相当有用。<br> undispaly，取消先前的display设置，编号从1开始递增 | (gdb) display /i $pc <br> (gdb) undisplay 1 |
| i | Info的简写，用于显示各类信息，详情请查阅“help i”。 | (gdb) i r |
| l | list的缩写，表示列举代码，默认列举10行，并且每次从上一个列举的尾部继续向下 | l 6 (表示以第六行为中心显示一共10行代码)<br> l 1,13(显示1到13行的代码) <br> list function(显示以函数为中心的10行代码)|
| bt | 即backtrace的缩写，产看函数调用信息(堆栈)。一般在程序设置断点或者bug处进入堆栈查看 | bt |
| f | frame的缩写，查看栈帧。<br> 一般在bt进入堆栈后，会i显示栈帧的信息，这时`f n`即表示进入哪个栈帧。 | f 0 <br> f 1 |
| q | Quit的简写，退出GDB调试环境。 | (gdb) q |
| help [命令名称] | GDB帮助命令，提供对GDB名种命令的解释说明。<br> 如果指定了“命令名称”参数，则显示该命令的详细说明；如果没有指定参数，则分类显示所有GDB命令，供用户进一步浏览和查询。 | (gdb) help display |

x命令 查看内存地址中的值


###2. 示例：
1). 代码：

```
[[include]]<iostream>
using namespace std;
int nGlobalVar = 0;
int TempFunction(int a, int b) {
    cout<<"a is :"<<a<<" and b is : "<<b<<endl;
    return a + b;
}
int main() {
    int n = 1;
    n ++;
    n --;

    nGlobalVar += 100;
    nGlobalVar -= 12;


    cout<<"n is : "<<n<<" and nGlobalVar is : "<<nGlobalVar<<endl;

    n = TempFunction(3, nGlobalVar);
    cout<<"n is : "<<n<<endl;
    return 0;
}

```
2). 进行编译：`g++ -o gdb_test1 gdb_test1.cpp -g`

在这里使用参数 -g 表示将源代码信息编译到可执行文件中。如果不使用参数 -g，会给后面的GDB调试造成不便。

3). 输出gdb，进入gdb调试：

首先使用file命令加载要调试的可执行文件： `(gdb) file gdb_test1`
可以看到：Reading symbols from gdb_test1... ，此时已经加载成功，使用`r`命令运行加载进来的可执行文件。
```
(gdb) r
Starting program: /home/zhangli/code/Cpp/GDB_test/gdb_test1
n is : 1 and nGlobalVar is : 88
a is :3 and b is : 88
n is : 91
[Inferior 1 (process 12301) exited normally]
```
接着，使用`b`命令在main函数开头设置一个断点： `b main`，会有如下的提示：
```
(gdb) b main
Breakpoint 1 at 0x5555555551ec: file gdb_test1.cpp, line 10.
```
注意到断点设在了第10行，是本程序的第一个断点（序号为1）；断点处的代码地址为 0x5555555551ec（此值可能仅在本次调试过程中有效）。回过头去看源代码，第10行中的代码为“n = 1”，恰好是 main 函数中的第一个可执行语句（<font color=red>前面的“int n;”为变量定义语句，并非可执行语句</font>）。

再次使用“r”命令执行（Run）被调试程序：
```
(gdb) r
Starting program: /home/zhangli/code/Cpp/GDB_test/gdb_test1

Breakpoint 1, main () at gdb_test1.cpp:10
10          n = 1;

```
可以看到程序停在了main函数的地一个可执行程序处，上面最后一行信息为：下一条将要执行的源代码为“n = 1;”，它是源代码文件gdb_test1.cpp中的第10行。
下面使用“s”命令（Step）执行下一行代码（即第10行“n = 1;”）：
```
(gdb) s
11          n ++;
```
上面的信息表示已经执行完“n = 1;”，并显示下一条要执行的代码为第11行的“n++;”。
既然已经执行了“n = 1;”，即给变量 n 赋值为 1，那我们用“p”命令（Print）看一下变量 n 的值是不是 1 ：
```
(gdb) p n
$1 = 1
```
果然是 1。（$1大致是表示这是第一次使用“p”命令——再次执行“p n”将显示“$2 = 1”——此信息应该没有什么用处。）下面我们分别在第18行、TempFunction 函数开头各设置一个断点（分别使用命令`b 18`和`b TempFunction`）：
```
(gdb) b 18
Breakpoint 2 at 0x555555555219: file gdb_test1.cpp, line 18.
(gdb) b TempFunction(int, int)
Breakpoint 3 at 0x555555555183: file gdb_test1.cpp, line 5.
```
使用“c”命令继续（Continue）执行被调试程序，程序将中断在第二个断点（18行），此时全局变量 nGlobalVar 的值应该是 88；再一次执行“c”命令，程序将中断于第三个断点（5行，TempFunction 函数开头处），此时tempFunction 函数的两个参数 a、b 的值应分别是 3 和 88：
```
(gdb) c
Continuing.

Breakpoint 2, main () at gdb_test1.cpp:18
18          cout<<"n is : "<<n<<" and nGlobalVar is : "<<nGlobalVar<<endl;
(gdb) c
Continuing.
n is : 1 and nGlobalVar is : 88

Breakpoint 3, TempFunction (a=3, b=88) at gdb_test1.cpp:5
5           cout<<"a is :"<<a<<" and b is : "<<b<<endl;
```
再一次执行“c”命令（Continue），因为后面再也没有其它断点，程序将一直执行到结束：
```
(gdb) c
Continuing.
a is :3 and b is : 88
n is : 91
[Inferior 1 (process 29984) exited normally]
```
4). 使用汇编方式对可执行程序进行调试：
有时候需要看到编译器生成的汇编代码，以进行汇编级的调试或跟踪，又该如何操作呢？这就要用到display命令`display /i $pc`：
```
(gdb) display /i $pc
1: x/i $pc
<error: No registers.>
```
这里的报错不用在意，因为程序未运行的时候并没有使用寄存器。之后当我们再次使用中断的时候就可以显示汇编代码：
```
(gdb) r
Starting program: /home/zhangli/code/Cpp/GDB_test/gdb_test1

Breakpoint 1, main () at gdb_test1.cpp:10
10          n = 1;
1: x/i $pc
=> 0x5555555551ec <main()+8>:   movl   $0x1,-0x4(%rbp)
```
我们看到了汇编代码，“n = 1;”对应的汇编代码是`movl   $0x1,-0x4(%rbp)`，并且以后程序每次中断都将显示下一条汇编指定（“si”命令用于执行一条汇编代码——区别于“s”执行一行C代码）：
```
(gdb) si
11          n ++;
1: x/i $pc
=> 0x5555555551f3 <main()+15>:  addl   $0x1,-0x4(%rbp)
```
接下来我们试一下命令`b *<函数名称>`。为了更简明，有必要先删除目前所有断点（使用“d”命令——Delete breakpoint）

之后使用命令`b *main`在 main 函数的 prolog 代码处设置断点（<font color=#625>prolog、epilog，分别表示编译器在每个函数的开头和结尾自行插入的代码</font>）：
```
(gdb) r
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/zhangli/code/Cpp/GDB_test/gdb_test1

Breakpoint 2, main () at gdb_test1.cpp:8
8       int main() {
1: x/i $pc
=> 0x5555555551e4 <main()>:     push   %rbp
(gdb) si
0x00005555555551e5      8       int main() {
1: x/i $pc
=> 0x5555555551e5 <main()+1>:   mov    %rsp,%rbp
(gdb) si
0x00005555555551e8      8       int main() {
1: x/i $pc
=> 0x5555555551e8 <main()+4>:   sub    $0x10,%rsp
(gdb) si
10          n = 1;
1: x/i $pc
=> 0x5555555551ec <main()+8>:   movl   $0x1,-0x4(%rbp)
(gdb) si
11          n ++;
1: x/i $pc
=> 0x5555555551f3 <main()+15>:  addl   $0x1,-0x4(%rbp)
(gdb) si
12          n --;
1: x/i $pc
=> 0x5555555551f7 <main()+19>:  subl   $0x1,-0x4(%rbp)
(gdb) si
14          nGlobalVar += 100;
1: x/i $pc
=> 0x5555555551fb <main()+23>:  mov    0x2f33(%rip),%eax        # 0x555555558134 <nGlobalVar>
```
这时，我么可以使用`i r`命令查看当前寄存器中的内容：（`i r`即`Infomation Register`）
```
(gdb) i r
rax            0x5555555551e4      93824992236004
rbx            0x0                 0
rcx            0x100               256
rdx            0x7fffffffdab8      140737488345784
rsi            0x7fffffffdaa8      140737488345768
rdi            0x1                 1
rbp            0x7fffffffd9c0      0x7fffffffd9c0
rsp            0x7fffffffd9b0      0x7fffffffd9b0
r8             0x7ffff7d99a40      140737351621184
r9             0x0                 0
r10            0x6                 6
r11            0x7ffff7ee38a0      140737352972448
r12            0x555555555090      93824992235664
r13            0x7fffffffdaa0      140737488345760
r14            0x0                 0
r15            0x0                 0
rip            0x5555555551fb      0x5555555551fb <main()+23>
eflags         0x202               [ IF ]
cs             0x33                51
ss             0x2b                43
ds             0x0                 0
es             0x0                 0
fs             0x0                 0
gs             0x0                 0
```
当然也可以显示某一个特定的寄存器：`i r rax`
```
(gdb) i r rax
rax            0x5555555551e4      93824992236004
```
最后，我们输出`q`即退出。

补充：
`gdb --args bin/cnstream_test --gtest_filter=Inferencer.*`
gtest指定参数,对TEST进行调用

##3.gdb反汇编
**汇编命名习惯的历史由来：**
最先开始,Intel 8086和8088有十四个16位寄存器，比如`AX, BX, CX, DX`等等。然后Intel出了32位处理器，相对于16位处理器是是扩展的`extended`，于是在16位的寄存器基础上加上E前缀，比如`AX`变成了`EAX`，在后来，AMD出了64位处理器，采用的`R`前缀。

###3.1 设计代码并汇编：
**代码：**
```
int g(int x)
{
    return x + 6;
}

int f(int x)
{
    return g(x);
}

int main(void)
{
    return f(2333)+666;
}
```
**汇编:**
在ubuntu平台下，使用`gcc -S -o main.s main.c -m32`将它反汇编成main.s。注意，在AMD64（或者说X86-64）的操作系统为了产生32位的汇编代码，使用`-m32`选项让它生成32位汇编指令。

在代码中有许多以`.`开头的代码行，属于链接时候的辅助信息,在实际中不会执行，把它删除，得到下列的代码就是纯汇编代码了(<font color = #95741>linux默认为AT&T格式，即源寄存器在前，目的寄存器在后</font>)：
```
g:
	pushl	%ebp
	movl	%esp, %ebp  // 实际上是把ebp进栈后的栈顶地址给了esp。
	movl	8(%ebp), %eax
	addl	$6, %eax
	popl	%ebp
	ret
f:
	pushl	%ebp
	movl	%esp, %ebp
	pushl	8(%ebp)
	call	g
	addl	$4, %esp
	leave
	ret
main:
	pushl	%ebp
	movl	%esp, %ebp
	pushl	$2333
	call	f
	addl	$4, %esp
	addl	$666, %eax
	leave
	ret
```
###3.2代码分析：
**常用寄存器：**
`ESP`：栈指针寄存器(extended stack pointer)，其内存放着一个指针，该指针永远指向<font color = red>系统栈**最上面一个栈帧**的栈顶。</font>

`EBP`：基址指针寄存器(extended base pointer)，其内存放着一个指针，该指针永远<font color = red>指向系统栈**最上面一个栈帧**的底部。</font>

注意：esp始终指向栈顶，ebp在堆栈中寻址

**分析：**
每一个函数（在汇编中，函数就是个代码段）的开头都是下面格式
```
函数名:
	pushl	%ebp  // ebp入栈
	movl	%esp, %ebp  // 由于esp是堆栈指针，无法借用，所以需要ebp来存取堆栈
	；// 函数中间过程
	leave（或者popl    %ebp）
	ret
```
注意，`leave`和下面代码等价
```
	movl	%ebp, %esp
	popl	%ebp
```
也有时候，我们把下面代码写成`enter`
```
函数名:
	pushl	%ebp
	movl	%esp, %ebp
```

**函数执行过程：**
我们先分析一下这个函数执行的过程：
1）每次call一个函数，函数总是先把当前的栈底指针压入堆栈(`pushl %ebp`)，然后把栈底指针移动到当前的栈顶，这样子做，相当于在旧的栈上新起了一个栈。然后在新栈上执行函数。
2）结束函数执行的时候，由于我们新引进了一个寄存器，我们可以用`movl %ebp, %esp`来恢复上一个调用函数的堆栈栈顶。当然，如果没有堆栈的变化，我们当然可以优化编译器把这句话去了。
3）这时候，马上就要ret飞回调用它的函数了……别急，我们还需要恢复栈底指针，否则回去的日子就难过了。于是`popl %ebp`(将之前push入栈的上一个函数的栈底值重新返回ebp中)。然后如果可以的话，我们会用leave来代替刚刚的两行代码。

**函数调用**
函数执行一定得是有函数调用了。
```
pushl	$2333
call  f
addl  $4, %esp
```
这是调用`f(2333)`函数的过程，我们可以看到，我们把2333压栈，然后调用了f函数。

由于在之前`pushl %ebp`的时候将ebp的值入栈，且栈顶指针向下增长(小端)，所以此时`movl	%esp, %ebp`获得的值已经-4。 在等到f函数的ret后，返回了现在main函数的call的下一行汇编代码，esp和ebp是同一个值，所以这以后如果压栈的时候，会覆盖了栈底指针，把esp往栈顶上移动1个单位也就是4个字节，这时候就完美解决了调用后的问题，才是真正调用完成了。

那么，怎么取得参数呢？

**函数参数取得：**
这时候，得回头看一下f函数了。这时候，我们发现它用了

    pushl	8(%ebp)
	call	g
	addl    $4, %esp
它把增加了8个字节的地址压栈了，然后调用了g函数。
分析一下为什么是8个字节，我们可以用sizeof关键字来测试得到int占4个字节……所以，它却加了8个字节取值，那么必然是有什么怪东西又入栈了。pushl %ebp是每次函数执行的时候使用的，哈哈，找到了，就是ebp寄存器还占用了4个字节，想想，32位芯片，寄存器$32位=8位/字节\times 4字节$。符合啦。
所以，又发现了ebp寄存器的一个好处，能够让我们方便取得函数的参数……否则后面再去参数，栈位置变了好多，就不方便了。
