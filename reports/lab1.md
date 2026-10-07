# lab1 实验报告

## 1. 实验目标

本实验构建一个最小的 RISC-V 可执行内核。内核由 OpenSBI 交接后从
`0x80200000` 开始执行，初始化内核栈，进入 C 语言初始化函数，通过 SBI
提供的字符输出接口打印启动信息，最后进入死循环。

## 2. 代码执行流程

1. QEMU 模拟 RISC-V 机器从复位地址 `0x1000` 开始执行。
2. OpenSBI 完成机器初始化，并把内核镜像加载到物理地址 `0x80200000`。
3. 链接脚本将 `kern_entry` 设置为内核入口，并将代码段放在
   `0x80200000`。
4. `kern/init/entry.S` 设置内核栈顶，然后跳转到 `kern_init`。
5. `kern_init` 清零 `.bss` 区域，调用 `cprintf` 输出启动信息，然后进入死循环。
6. 字符输出经过 `cprintf`、`cons_putc`、`sbi_console_putchar`，
   最终通过 `ecall` 请求 OpenSBI 输出字符。

## 3. 主要模块

### 3.1 启动入口

`kern/init/entry.S` 定义了全局符号 `kern_entry`。启动时：

- `la sp, bootstacktop` 将栈指针设置为预留内核栈的高地址端。
- 栈在 RISC-V 调用约定下向低地址增长，因此此时可以安全支持 C 函数调用。
- `tail kern_init` 使用尾调用跳转到 `kern_init`，不再保留当前入口的返回地址。
  这与 `kern_init` 不返回的设计相匹配。

### 3.2 内存布局

`tools/kernel.ld`：

- 指定输出架构为 RISC-V；
- 指定入口点为 `kern_entry`；
- 将内核基地址设置为 `0x80200000`；
- 依次组织 `.text`、`.rodata`、`.data`、`.sdata` 和 `.bss`；
- 导出 `edata` 和 `end`，供 C 代码清零 `.bss`。

### 3.3 SBI 与控制台输出

`libs/sbi.c` 使用 RISC-V 内联汇编把 SBI 功能号放入 `x17`，
把参数放入 `x10` 至 `x12`，执行 `ecall`，再从 `x10` 读取返回值。
`kern/driver/console.c` 对 SBI 字符输出进行封装，`kern/libs/stdio.c`
再向上提供 `cprintf`。同时，`sbi_console_getchar` 对 SBI 的控制台读字符
服务进行封装，供 `cons_getc` 和后续的输入功能使用。

## 4. 练习一：理解内核启动入口

### 4.1 `la sp, bootstacktop`

该指令将符号 `bootstacktop` 的地址加载到栈指针寄存器 `sp`。
`entry.S` 中的 `bootstack` 到 `bootstacktop` 之间预留了
`KSTACKSIZE` 字节空间，栈顶位于高地址端。设置 `sp` 后，内核才具备
执行 C 语言函数调用、保存返回地址和局部变量的栈空间。

### 4.2 `tail kern_init`

这是一个尾调用伪指令，作用是跳转到 `kern_init`，但不为当前入口建立新的
返回地址。由于 `kern_init` 被声明为不返回并且最后进入死循环，尾调用可以
避免无意义的返回路径和额外的栈帧。

## 5. 练习二：GDB 启动流程

本节需要在已经配置好 RISC-V GDB 和 QEMU 的 Linux/WSL 环境中完成。

### 5.1 计划使用的命令

```gdb
target remote localhost:1234
x/i $pc
watch *0x80200000
continue
break *0x80200000
continue
break kern_entry
continue
info registers
```

### 5.2 待实测记录

- QEMU 复位后初始 PC：指导书给出的地址是 `0x1000`。
- OpenSBI 的起始地址：指导书给出的地址是 `0x80000000`。
- 内核入口地址：应为 `0x80200000`，需要用 GDB 和符号表实测确认。
- `kern_entry` 处的寄存器状态：待运行 `info registers` 后记录。
- `la sp, bootstacktop` 执行前后的 `sp`：待单步执行后记录。

## 6. 构建与运行验证

当前 Windows 主机尚未检测到 GNU Make、RISC-V 交叉编译器、RISC-V GDB
和 `qemu-system-riscv64`；WSL 查询目前返回权限错误。因此以下结果需要在
工具链可用的 Linux/WSL 环境中补充：

```text
make:
make qemu:
make debug / make gdb:
```

## 7. 实验中的关键知识点

| 实验知识点 | 对应 OS 原理 |
| --- | --- |
| OpenSBI 与内核交接 | 固件、引导程序和操作系统的启动链 |
| 链接脚本 | 程序装载地址、代码/数据段布局 |
| 内核栈 | 函数调用、局部变量和执行上下文 |
| SBI `ecall` | 特权级切换与受控服务调用 |
| `.bss` 清零 | 全局未初始化数据的运行时初始化 |
| ELF 到 binary | 可执行文件布局与裸机装载 |

## 8. 本实验尚未覆盖的 OS 知识

本实验还没有实现页表和虚拟内存、异常与中断处理、进程/线程、调度、
同步互斥、文件系统和用户态程序等机制。这些内容将在后续 lab 中逐步加入。
