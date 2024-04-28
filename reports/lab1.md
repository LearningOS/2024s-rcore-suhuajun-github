简答作业
1. sbi: RustSBI-QEMU Version 0.2.0-alpha.3
bad_address: 触发Store/AMO access fault
bad_instructions: 触发Illegal instruction
bad_register: 触发Illegal instruction

2.
L40：刚进入 __restore 时，a0 表示指向TrapContext结构体的指针
__restore可以从__alltraps进入，调用完trap_handler，然后返回到__restore
__restore也可以直接通过__switch函数进入，因为goto_restore里面把ra变量设置成了__restore，__switch会把ra变量存储到ra寄存器，执行ret指令后cpu就从__restore开始执行

L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。

ld t0, 32*8(sp)     读取TrapContext结构体中的sstatus到t0
ld t1, 33*8(sp)     读取TrapContext结构体中的sepc到t1
ld t2, 2*8(sp)      读取TrapContext结构体中的sp到t2
csrw sstatus, t0    恢复status
csrw sepc, t1       恢复sepc
csrw sscratch, t2   把sp的值存储到sscratch，这可不是内核栈指针，是用户栈指针，之后还有一个csrrw会用内核栈指针和sscratch交换

L50-L56：为何跳过了 x2 和 x4？
x2是sp，是用户栈指针，之前的csrrw已经给用户栈指针交换到sscratch了，之后再读取出来保存，现在先跳过
x4注释说了么，用户程序不会使用x4，x4是Thread pointer，用户程序不使用线程指针吗？不知道，注释说不使用，不保存

L60：该指令之后，sp 和 sscratch 中的值分别有什么意义？
就是之前说的，把用户栈指针和内核栈指针交换

__restore：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？
sret
cpu就是这样设计的这条指令

L13：该指令之后，sp 和 sscratch 中的值分别有什么意义？
进入__alltraps之前，sscratch中保存的是内核栈指针
csrrw把sp和sscratch交换，执行完这条指令后sp中保存的是内核栈指针，sscratch中保存的是用户栈指针

从 U 态进入 S 态是哪一条指令发生的？
有可能是syscall函数里面的ecall指令，也有可能是syscall6函数里面的ecall指令


简单总结你实现的功能（200字以内，不要贴代码）。
实现用户态计时和统计系统调用执行的次数


荣誉准则

1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：
基本上是看不懂rust代码，先用ucore打草稿，然后把ucore的结果硬搬过来
记性不好，之前的基本上已经不记得了，最近的一次记得，群友nomodeset指正了我的设计，我之前使用的是计算用户态的执行时间，现在是计算 “用户态的执行时间”

2. 此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：
ucore: https://learningos.cn/uCore-Tutorial-Guide-2024S/chapter3/5exercise.html
另一个ucore: https://nankai.gitbook.io/ucore-os-on-risc-v64/lab1/shi-zhong-zhong-duan
riscv官方文档: file:///home/suhuajun/Downloads/riscv-privileged-20211203.pdf
               file:///home/suhuajun/Downloads/riscv-spec-20191213.pdf

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。
