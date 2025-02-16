
# 总结


对于计算机系统的一个总结：从逻辑门到计算机


## 数字电路

操作系统的故事是从数字电路开始的，数字电路的课程回答一个问题：所有的计算机系统的一切都是从逻辑门构造出来的。

第一个有趣的例子，数字电路模拟器，用一段非常简单的代码，实现一个数码管。不仅仅去模拟一个电路，把模拟电路的结果和模拟外设的程序连接起来，实现非常有趣的效果。

数码管的例子，展示了 unix 哲学和“编程”的力量。如果实现一个模拟器，是一个非常有意思的事情。

数字电路这门课，就是去定义一个状态机，组合逻辑电路，每个时钟来的时候，切换下一个状态。

一切计算机系统的故事从数字系统开始。

## 编程语言和算法

在学习计算机系统的时候，并不是学习电路如何一层一层搭建到整个计算机，这太复杂了。

学习计算机的路线，是从编程开始的，即学习怎么和计算机打交道，把我们想的东西告诉计算机。

操作系统这门课给了编程的不一样的视角，。

编程想的是函数、分支循环、把物理世界的某个东西投射到计算机。

程序就是个状态机，C 程序的状态，gdb 调试的每一小步就是一个状态的迁移。

非递归的汉诺塔给的启示：程序是有状态的，我们可以用程序模拟出状态。

这个东西为什么要在操作系统课讲呢？操作系统连接了软件和硬件。电路级的东西太底层了，往上抽象一层，体系结构，ISA，屏蔽了相当多的电路细节。

ISA 抽象给了内存和寄存器的抽象，给了指令集，ISA 是可以用电路来实现的。有了 ISA 的抽象，就可以把程序和指令联系起来了，这里就有了编译器。编译器也是一个程序，把高级语言描述的状态机翻译成低级语言描述的状态机。

如果学习体系结构的话，可以尝试一下 RISC-V，既容易用电路实现，有足够支撑状态机运行。

理解好状态机模型，从高级语言状态机到低级语言状态机。

## 

然后，操作系统就是状态机的管理者。

操作系统自己会有一些状态，然后他会管理其他状态机。


