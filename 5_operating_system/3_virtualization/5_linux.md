
# linux 操作系统

我们已经了解操作系统作为 “状态机的管理者”，通过为应用程序提供 fork, execve, exit, mmap, pipe, open, read, write 等系统调用，配合 C/汇编实现的运行库，就为 “一个进程” 逐步构建出完整的应用世界做好了准备。

这部分内容：完整的应用世界到底是如何构建的？在这节课中，我们用一个 “最小” 的 Linux 系统解答这个问题：
- Linux 操作系统
- Linux 系统启动和 initramfs
- Linux systemd

## 1. linux 操作系统


### minix

minix 年轻人的第一个全功能操作系统。


### linux 的成功

站在巨人的肩膀上构建自己的系统。给linux加一个补丁，就能适配自己的产品。linux 2.6 开始，2003年，大爆发。

<figure>
    <img src="https://jyywiki.cn/pages/OS/img/kernel-loc.png" >
</figure>

2003 后面几年，计算机系统的黄金时代。


### linux kernel

要注意前面是理论模型，并不是具体的那个操作系统。我们好奇 linux 到底符不符合前面的理论模型，我们日常看到的linux，比如 Ubuntu。

linux 的两面：
- kernel 
- kernel 系统调用上的发行版和生态。

github 上 linux 仅仅只是个内核，kernel。

kernel 可以理解成一个可执行的二进制文件，里面就是个指令序列，然后硬件厂商的固件和磁盘上的其它代码一起加载到内存上运行。运行起来就是 kernel，狭义的内核就是指一个在计算机启动后会被运行的程序，这个程序会加载第一个应用程序，还会创建一些对象，如 /dev/console，信息打印到这里。启动linux可能会发现，打印日志时，突然有一瞬间字体会变，这就是内核做了一些配置。最后启动完毕，对象创建完成，我们就可以使用系统调用操作这些对象了。

广义的 linux，我们看到的，图形界面，鼠标键盘能用，vscode 写代码，浏览器上网。或者 Android，底层也是 linux。所有我们早期使用的，都是linux kernel 系统调用上的发行版和应用生态
- 系统工具 coreutils, binutils, systemd, ...
- 桌面系统 Gnome, xfce, Android
- 应用程序 file manager, vscode, ...

```note
systemd，pstree 的根，随着我们对操作系统原理的理解越来越好，可以把一个合理的比较抽象的 kernel 概念映射到真实操作系统。后面会讲为什么 systemd 是根，事实上 init 进程并不是 systemd。
```

如果真想理解 linux 的启动到加载第一个进程的过程，能不能构建一个可以启动的最小的 kernel，然后运行我们自己的程序，一个输出 hello world 的程序，打印 hello world，然后退出。

## 2. 启动 Linux

为什么 pstree 的根是 systemd，启动以后我们有个 ramfs，在 ram 里，仅仅供初始化使用，这个时候系统里还没有磁盘。

还得看看之后发生了什么，。

系统的启动分两个阶段，这个阶段完成后，有个应用程序需要调用 

```c
int pivot_root(const char *new_root, const char *put_old);
```

执行 `/usr/sbin/init` 即 Kernel 的 init 选项。这是个快捷方式，链接到了 `/lib/systemd/systemd`。

ramfs 的事情是创建好足够多的对象，。

## 3. 应用程序的世界


## 操作系统启动后到底做了什么？(2022版未整理)

- 操作系统启动后到底做了什么？ 
  - CPU Reset → Firmware → Loader → Kernel _start() → 执行第一个程序 /bin/init → 中断/异常处理程序
  - 一个最小的 Linux 系统的例子
回顾操作系统内核的启动

cpu reset -> fireware -> boot loader -> kernel_start()

lagecay bios 固件完成操作系统加载。uefi里也有类似的东西。

Q1: 操作系统启动后到底做了什么？
Q2: 操作系统如何管理程序 (进程)？

```note
thread-os 在 main 开始后做一些系统的初始化，然后这个小操作系统会创建许多线程。这个有点像嵌入式系统里运行的小操作系统，CPU\MCU 里没有 MMU，没做虚拟内存的能力，只支持几个线程的执行。

等到任务创建完后，操作系统就不做事情了。
```

操作系统启动后，加载“第一个程序”，然后把所有的控制权交给这个程序。所有其他看到的东西( `pstree` )都是由这个程序启动的。

```bash
systemd─┬─YDLive───7*[{YDLive}]
        ├─YDService─┬─sh───9*[{sh}]
        │           └─23*[{YDService}]
        ├─acpid
        ├─2*[agetty]
        ├─barad_agent─┬─barad_agent
        │             └─barad_agent───2*[{barad_agent}]
        ├─cron
        ├─dbus-daemon
        ├─networkd-dispat
        ├─ntpd───{ntpd}
        ├─packagekitd───2*[{packagekitd}]
        ├─polkitd───2*[{polkitd}]
        ├─rsyslogd───3*[{rsyslogd}]
        ├─sgagent───{sgagent}
        ├─sshd─┬─sshd───sshd───fish
        │      ├─sshd───sshd───fish───sftp-server
        │      ├─sshd───sshd───fish─┬─pstree
        │      │                    └─{fish}
        │      └─sshd───sshd───fish─┬─sftp-server
        │                           └─{fish}
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-network
        ├─systemd-resolve
        ├─tat_agent───6*[{tat_agent}]
        ├─tmux: server───5*[fish]
        └─unattended-upgr───{unattended-upgr}
```

[linux内核启动代码](https://elixir.bootlin.com/linux/latest/source/init/main.c#L1555)

linux 试图启动一些东西。`try_to_run_init_process("/sbin/init")` ，试图启动这个程序，确实也是有的，软连接到 `init -> /lib/systemd/systemd` 确实是 systemd。

如果找不到这个文件，还会往下找。如果所有的都找不到，那么直接拒绝启动。可以试试，删掉这些文件，就无法启动系统了。但是还是有办法的，给内核加个选项。

第一个程序加载好，状态机就开始运行了。第一个程序创建其他程序，一级一级创建出来。

如果真的想让对进程或者linux的理解往前推一步的话，应该做什么呢？

是不是可以实现一个最小的Linux内核的镜像，只有一个程序，只要有一个程序，那么整个操作系统应该就能启动起来。

如果第一个进程，返回了，操作系统里就没有进程了，内核就不知道该干什么了。Kernel panic了。所以我们的第一个进程应该是一个不会终止的。

这个最小 linux 几乎啥也不能做。cpu reset 经过 fireware 然后 loader 加载 os，然后 init 启动第一个进程，然后所有的东西都有了。

创建一些目录，创建一些链接，然后体验就像是普通的 Linux 了。从啥也不能用，再到都能用，甚至可以编译程序。

实际上不仅是 linux ，其他的设备也是类似的模式，华为的鸿蒙。

这个东西可以玩玩，还是挺好玩的。

小结：

linux 操作系统的启动流程

cpu reset -> firmware -> loader -> kernel_start() -> 第一个程序 `/bin/init` -> 程序执行 + 系统调用

操作系统为 (所有) 程序提供 API
- 进程 (状态机) 管理
  - fork, execve, exit - 状态机的创建/改变/删除。这一节的主题
- 存储 (地址空间) 管理
  - mmap - 虚拟地址空间管理
- 文件 (数据对象) 管理
  - open, close, read, write - 文件访问管理
  - mkdir, link, unlink - 目录管理








