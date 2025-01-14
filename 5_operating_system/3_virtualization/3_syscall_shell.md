
# UNIX Shell 的设计和实现

前面的内容：整个计算机系统世界的 “创建” 从 CPU Reset 开始，Firmware 加载操作系统内核代码，操作系统成为状态机的管理者，初始化第一个进程，从而成为一个中断处理程序和系统调用执行者。《操作系统》课程的很重要部分就是操作系统中的对象和操纵这些对象的 API。——我们已经学习了 fork, execve, exit, 和 mmap (munmap/mprotect)。如何用这些 API，以及更多的 API，实现丰富的应用程序？

我们看到的软件的东西，但是这些软件是运行在硬件上的，机器的电路级实现上是个状态机。

这部分的内容
- 更多的操作系统的 api 的补充
- unix shell 的实现

前面有了硬件到固件，固件加载操作系统，操作系统加载了 init 进程，init进程启动了所有的东西。init 进程为了创建更多进程，仅仅靠 mov ret ... 这些指令是做不到的，必须要请求操作系统，操作系统此时也变成了一个中断服务的应用。

## 1. 更多的操作系统 API

什么是系统调用？还是要来看操作系统

每个进程是一个状态机，操作系统里管理着许多状态机，此外还有一些对象，进程可以访问操作系统里的对象。如 pid 下的 maps 和 mem 文件，就是有名字的对象，都是字节序列。想要访问对象，通过文件描述符，这就是一个指向操作系统对象的指针。

everything is a file，这里的 thing 指的就是操作系统的对象。

对象的访问需要指针，指向操作系统对象的指针就是文件描述符。定义一个指针，还需要得到他的值，因此也操作系统也设计了通过名字得到指针的 api。
- open close read write ，lseek

题外话，windows 里的也应该有这样的机制，叫 handle ，打开任务管理器，在性能一栏里，有一个句柄。更像指针。


有一类特殊的对象，IPC Endpoints，管道。管道是一个特殊的文件，可以理解成一个缓冲区。管道有两个口，一个读口，一个写口。传统的文件打开 得到一个fd。管道也是文件，是一个流，不能寻址，不可以做指针运算。

匿名管道。编程创建管道以后，会在某个地方新得到一个文件，一个文件以只读的方式打开，就指向了读口；另一个程序可以以只写的形式打开一个管道。

可以用任何一个命令行工具重定向到命名管道文件，然后就能看到了。

看到管道一般都会看到 IPC 这个词，进程间通信的方式，同步，读的先到那就等。写的先到那就等有人读。

UNIX 管道 (pipe) 是一种典型的进程间通信机制，允许数据在不同的进程之间单向流动。管道可以被视为一种特殊的文件，其中一个进程将数据写入管道的一端，而另一个进程从另一端读取数据。

在bash命令里 `ls | wc --lines` 块的会等慢的。

管道一个更重要的应用是 匿名官管道。这个系统调用要传一个数组进去。

```c
int pipe(int pipefd[2]);
```

这个系统调用返回两个文件描述符 fd[0] 指向了读口，fd[1] 指向了写口。也就是说，同一个进程即有读口又有写口。

当 fork 的时候，这就有用了。

调用以后，返回了 3 和 4，读口为 3，写口为 4，。此后， fork 一下，完整的复制。父进程和子进程同时持有了一个管道，

此时就可以通过进程号判断，子进程关掉写口，父进程关掉读口，然后父进程就可以往里面写东西了，子进程就能读到父进程来的东西了。

## 2. unix shell

在只有系统调用的基础上，构建整个应用生态。


### (日用)操作系统和《操作系统》

我们是操作系统的用户，每天都在用，但是平时用操作系统的体验和学的这个操作系统的差别怎么这么大呢？操作系统提供的 API 并不是 “我们” 作为人类用户能直接使用的。那 “我们” 到底怎么用操作系统？我们用 windows 桌面，ubuntu 图形化界面。我们每天使用操作系统
- 启动应用程序
  - 即时通信
  - 影音娱乐
  - ~~生产学术垃圾~~
- 没了

我们认为的操作系统就是应用程序的启动器，开机，进入桌面，然后启动各种程序。

事实上，人是看不到操作系统的，非计算机专业的人是意识不到**操作系统=对象+api**，即我们刚接触计算机，也会认为这个东西是操作系统

<figure>
  <img src="https://jyywiki.cn/pages/OS/img/win11.jpg" width=400>
</figure>

事实上，上面这个东西是个应用程序，叫 explorer.exe，我们可以直接把这个杀掉，然后桌面就没有了，包括图标菜单这些东西。

在这么多东西里，有一个是特殊的，启动应用的应用。


### shell

当人要用操作系统的时候，人不能直接用操作系统的 API，没法每次都用 c 语言，open，返回一个 fd，然后 write 这个 fd。显然这不是人用操作系统的方式，所以人和操作系统之间其实隔了一个应用程序的距离，这个程序就是 shell。

shell 会把最基本的 IO 管理起来，IO 设备连接了人。shell 是把用户指令翻译成系统调用的工具，shell 也是一个编程语言，用shell已经就是在写程序了。

这个词是怎么来的，**操作系统 = api + 对象**，这个东西是**内核**(第一次明确指出)，人和内核交互，这是不实际的，所以在内核外面包一层 shell。

shell 就是用户看到的，可以把 api 和对象座一层封装，方便用户管理整个操作系统里的对象的应用程序。这个概念是 unix 时候的概念。

所以现在的图形的这个东西，图形的shell。

### unix shell

unix shell 是“终端”时代的伟大设计。command-line interface 的巅峰。

Shell 是一门 “把用户指令翻译成系统调用” 的编程语言，所以我们在用终端的时候其实是在编程。

如果让今天能做的所有事情在终端上都能做，应该做什么样的设计呢？这不容易的，系统级别的设计。

先来分析需求：
- 今天我们可以用 gui 管理多个程序，如最小化，关闭。1970年由同样的需求，比如 vim 里编辑代码，然后编译，要能切换能管理
- 把多个程序协同起来，完成一件事情。Windows 的设计是完全违反 unix 哲学，当我们用unix的时候，有些东西是只有程序员会这么想。window 诞生的时候，计算机已经足够强大，可以支撑GUI，从用户友好的角度出发，用户不应该直到这么多细节。用户问系统要什么，系统就给什么才对。


2000 年以前，磁盘非常小，内存非常小，就有了一门 “把用户指令翻译成系统调用” 的编程语言，这就是 shell。

人把需求、命令告诉 shell，shell 把命令翻译成系统调用。所有软件的本质都是把人的需求在计算机里实现。受限于终端的能力，人只能和shell 发命令，后来发现更好的方法，用键盘鼠标，graphical shell。

shell 设计时受当时的性能限制，做了许多妥协，甚至大多是缩写。从今天看有些古怪，甚至有点难用。只有过了一些门槛，可能才能领会的当时设计的巧妙。

不过换个角度，如果把 shell 理解成编程语言，不好用好像也没啥毛病。没有哪个语言时好用的。因为编程语言时严格的数学对象，错一点点都无法通过编译。

shell里有相当多的历史约定，从多年前留下来。当第一次看到终端的时候，还是很绝望地，他不会告诉用户任何事情，用户说什么他就做什么，但问题是我不知道应该说什么。

“Unix is user-friendly; it's just choosy about who its friends are.”

平时使用 shell 的时候，常用的语法

一旦正式的开始用shell，编程，就已经走上码农的不归路了，命令行里写的每一段话都是程序。比如 `ls -l | wc --lines` 甚至可以写出表达式。此外还有一些常用的语法，或者说特性，学习这个语言的思路：**基于文本替换的快读工作流搭建**
- 重定向: cmd > file < file 2> /dev/null
- 顺序结构: cmd1; cmd2, cmd1 && cmd2, cmd1 || cmd2
- 管道: cmd1 | cmd2
- 预处理: $(), <()
- 变量/环境变量、控制流…
甚至可以像今天多任务控制一样，最小化后台运行
- fg，bg，wait
- 今天的 GUI 并没有比 CLI 多做太多的事情

$() 完整的替换，有些命令必须要一个文件，用 <()


```note
人工智能时代，我们为什么还要读手册？今天的人工智能还是被动的。给他一个明确的问题，他会回答。

如果给一个宽泛的问题，有时候甚至还一本正经的胡言乱语，说车轱辘话。

手册就不一样的，这是一个完备的，列出所有的功能。我们可以自己找到我们需要的，而 AI 只能给出通用的。毕竟我们想要一个什么样的，只有自己知道，还得靠自己。
```

我们去看 bash 的手册，还会发现更多东西。没事就去 `man sh` 一下吧。

## 3.复刻经典

如何实现一个 Zero-dependency UNIX Shell。无任何依赖，即在操作系统的 api 上直接构建 shell。这个shell 的一些功能：
- 也是一个状态机，同 minimal.s
- 完全基于系统调用 API
- 零库函数依赖，`-ffreestanding` 编译，`ld` 链接
- 支持重定向，管道，如 `ls>a.txt`，`ls | wc -l`
- 支持后台执行 `ls&`
- 支持命令组合 `(echo a; echo b) | wc -l` 可以解析括号

相当完善，还是直接读代码看思想吧，有 400 行。

拿到一个项目，如何阅读代码呢？如果不了解一个应用程序的运行逻辑，可以看看运行时发生的效果。

对于这个小项目，把命令翻译成系统调用，所以可以用 strace 看看，会发现很多细节。

- strace
  - 适当的分屏和过滤
  - AI 
- gdb
  - 单进程单线程，
  - 单进程多线程也行
  - 多进程怎么办呢？总是有人能帮忙的，在 AI 时代更是如此

看看一个shell如何实现的。


## 4. 更多关于 shell 的讨论

shell 并不是完美的，shell 在 1970 年，在自然语言、机器语言、当时的算力之间达到平衡。

平衡意味着不完美。如
- 操作的优先级
  - `ls > a.txt | cat` 
  - bash 的行为是收不到的，zsh 是既重定向又管道
- 管道里，所有东西都是文本
  - 有空格就很麻烦

比如

```bash
$echo hello > /etc/a/txt
bash: /etc/a.txt: Permission denied

$ sudo echo hello > /etc/a.txt
bash: /etc/a.txt: Permission denied
```

这个有意思的地方是为什么？因为 shell 先重定向打开文件，准备好文件描述符，再 execve，打开文件时还是普通用户，就拒绝了。



## 关于 pipe 的讨论

pipe 系统调用本质希望的是实现一个生产者和消费者的同步。如 ls | grep 命令，管道连接。我们希望的是无论生产者或消费者，谁跑的更快谁就等等。
- pipe read 在没有数据时会等待
- pipe write 在有读者打开时，会写入操作系统的缓冲区并返回

一个特性，如果write的内容不太多，一对 write read 是原子的。如果 write 超过了 PIPE_BUF ，可能会被拆成多份。





