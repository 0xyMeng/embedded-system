---
sort: 1
---
# 裸机开发


- Cortex-M 内核
  - STM32F
  - NXP i.mx rt
- Cortex-A9
  - STM32MP
  - NXP i.mx

ARM 内核的芯片，Cortex内核，外设，以及 STM32 整块芯片。

比较底层，关注外设模块的功能以及接口。主要使用 C 语言。
- STM32微控制器引脚，从外面看
- STM32内部整体结构（CPU+总线+外设）
- Cortex内核与指令，
- 总线与存储器
- IO操作与常用外设

以上为使用 IDE 的裸机开发，狭义的理解为不基于 RTOS 实现功能，使用一些芯片厂家的软件包，即封装好的寄存器操作接口。

从计算机的角度来考虑，如果不使用 IDE，如何让一个芯片工作起来呢。这个事情肯定是可以做到的，事实上 cortex a 系列跑 linux 的芯片，uboot 和 kernel 都没有使用 IDE，，那么 cortem m 芯片也一定可以做这个事情。

真·裸机开发，假设拿到了一个全新的 cortex m3 内核的芯片，没有可以直接使用的 IDE 如 keil IAR ，该如何使用起来呢。






