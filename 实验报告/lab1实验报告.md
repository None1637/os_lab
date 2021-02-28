# lab1实验报告

## 小组成员

- 姓名：张智强 学号：1611357
- 姓名：孟昊纯 学号：1611311

## 环境配置

- 使用在虚拟机中运行的 ubuntu14 作为环境。通过下载已经配置好的 ubuntu14 虚拟机文件，完成了系统环境配置

* 安装 QEMU 时配置失败

  运行`configure`时出现`ERROR: glib-2.48 gthread-2.0 is required to compile QEMU` 

  尝试通过`sudo apt-get install libglib2.0-dev`安装，出现`libglib2.0-dev is already the newest version.`

  再次运行`configure`时仍然出现`ERROR: glib-2.48 gthread-2.0 is required to compile QEMU` 

  解决方案：

  ​	手动下载安装 glib-2.48 ( 下载、解压、使用cd命令进入解压后的目录，然后依次执行 `./configure` 、 `make` 、 `sudo make install`)

  ​	安装 glib 时发现缺少 libffi， 继续手动下载安装 ( `wget ftp://sourceware.org/pub/libffi/libffi-3.3.tar.gz`)

* 发现自己看的是 rcore 而不是 ucore，重新配置，直接成功

* 从 https://github.com/kiukotsu/ucore 获取实验源代码

## 练习1：理解通过make生成执行文件的过程。

> 列出本实验各练习中对应的OS原理的知识点，并说明本实验中的实现部分如何对应和体现了原理中的基本概念和关键知识点。
>
> 在此练习中，大家需要通过静态分析代码来了解：
>
> 1. 操作系统镜像文件 `ucore.img` 是如何一步一步生成的？ ( 需要比较详细地解释 Makefile 中每一条相关命令和命令参数的含义，以及说明命令导致的结果 )
> 2. 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？
>
> 补充材料：
>
> 如何调试 Makefile
>
> 当执行 `make` 时，一般只会显示输出，不会显示 `make` 到底执行了哪些命令。
>
> 如想了解 `make` 执行了哪些命令，可以执行：
>
> ```shell
> $ make "V="
> ```
>
> 要获取更多有关 `make` 的信息，可上网查询，并请执行
>
> ```shell
> $ man make
> ```
>
 ### 练习1.1


- 操作系统镜像文件 `ucore.img` 是如何一步一步生成的？ ( 需要比较详细地解释 Makefile 中每一条相关命令和命令参数的含义，以及说明命令导致的结果 ) 

执行 `make "V="` ，得到以下输出：

```shell
bbrl@ubuntu:~/lab1$ make "V="

+ cc kern/init/init.c
  gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o
  kern/init/init.c:95:1: warning: ‘lab1_switch_test’ defined but not used [-Wunused-function]
   lab1_switch_test(void) {
   ^
+ cc kern/libs/readline.c
  gcc -Ikern/libs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/libs/readline.c -o obj/kern/libs/readline.o
+ cc kern/libs/stdio.c
  gcc -Ikern/libs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/libs/stdio.c -o obj/kern/libs/stdio.o
+ cc kern/debug/kdebug.c
  gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/kdebug.c -o obj/kern/debug/kdebug.o
+ cc kern/debug/kmonitor.c
  gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/kmonitor.c -o obj/kern/debug/kmonitor.o
+ cc kern/debug/panic.c
  gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/panic.c -o obj/kern/debug/panic.o
+ cc kern/driver/clock.c
  gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/clock.c -o obj/kern/driver/clock.o
+ cc kern/driver/console.c
  gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/console.c -o obj/kern/driver/console.o
+ cc kern/driver/intr.c
  gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/intr.c -o obj/kern/driver/intr.o
+ cc kern/driver/picirq.c
  gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/picirq.c -o obj/kern/driver/picirq.o
+ cc kern/trap/trap.c
  gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/trap.c -o obj/kern/trap/trap.o
+ cc kern/trap/trapentry.S
  gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/trapentry.S -o obj/kern/trap/trapentry.o
+ cc kern/trap/vectors.S
  gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/vectors.S -o obj/kern/trap/vectors.o
+ cc kern/mm/pmm.c
  gcc -Ikern/mm/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/mm/pmm.c -o obj/kern/mm/pmm.o
+ cc libs/printfmt.c
  gcc -Ilibs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/  -c libs/printfmt.c -o obj/libs/printfmt.o
+ cc libs/string.c
  gcc -Ilibs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/  -c libs/string.c -o obj/libs/string.o
+ ld bin/kernel
  ld -m    elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel  obj/kern/init/init.o obj/kern/libs/readline.o obj/kern/libs/stdio.o obj/kern/debug/kdebug.o obj/kern/debug/kmonitor.o obj/kern/debug/panic.o obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/intr.o obj/kern/driver/picirq.o obj/kern/trap/trap.o obj/kern/trap/trapentry.o obj/kern/trap/vectors.o obj/kern/mm/pmm.o  obj/libs/printfmt.o obj/libs/string.o
+ cc boot/bootasm.S
  gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootasm.S -o obj/boot/bootasm.o
+ cc boot/bootmain.c
  gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootmain.c -o obj/boot/bootmain.o
+ cc tools/sign.c
  gcc -Itools/ -g -Wall -O2 -c tools/sign.c -o obj/sign/tools/sign.o
  gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign
+ ld bin/bootblock
  ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
  'obj/bootblock.out' size: 472 bytes
  build 512 bytes boot sector: 'bin/bootblock' success!
  dd if=/dev/zero of=bin/ucore.img count=10000
  10000+0 records in
  10000+0 records out
  5120000 bytes (5.1 MB) copied, 0.0606869 s, 84.4 MB/s
  dd if=bin/bootblock of=bin/ucore.img conv=notrunc
  1+0 records in
  1+0 records out
  512 bytes (512 B) copied, 0.000272501 s, 1.9 MB/s
  dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc
  146+1 records in
  146+1 records out
  74879 bytes (75 kB) copied, 0.000571479 s, 131 MB/s
```

根据以上输出可以得知操作系统镜像文件 `ucore.img` 是如何一步一步生成的：

1. 编译 `init.c` `readline.c` `stdio.c` 等生成 `bin/kernel` 需要的文件
2. 链接这些文件生成 `bin/kernel`
3. 编译 `bootasm.S` `bootmain.c`  并链接它们，生成 `bin/bootblock`
4. 根据编译的 `sign.c` 将 472字节的 `bin/bootblock` 构建成 512字节的 boot 引导扇区
5. 生成大小为10000个块的 `ucore.img` ，每个块的大小为512字节
6. 将 boot 引导扇区复制到 `ucore.img` 中的第一个块
7. 将 `bin/kernel` 复制到 `ucore.img` 中以第二个块为开始的位置

以上输出中各个相关命令及命令参数的含义如下：

-  `gcc` 命令：GNU 的 c & c++ 编译器 gcc / g++ 执行编译工作使用的命令，在这里用于编译生成目标代码 ( 机器码 ) 文件

以

```shell
gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o
```

为例，各个参数的含义如下：

`-fno-builtin` 不使用 C 语言的内建函数

`-Wall` 编译后显示所有警告信息

`-ggdb` 在目标代码中加入供调试程序 gdb 使用的附加信息

`-m32` 生成32位机器的汇编代码

`-gstabs` 以 stabs 格式声称调试信息，但是不包括 gdb 调试信息

`-nostdinc` 编译的时候不在标准系统目录中找头文件

`-fno-stack-protector` 禁用 gcc 栈保护机制 stack-protector

`-Ikern/init/` `-Ilibs/` `-Ikern/debug/` `-Ikern/driver/` `-Ikern/trap/` `-Ikern/mm/` 在命令行上指定的库路径， `-I` 参数后的部分是库名

`-c kern/init/init.c` 编译 `init.c` 文件

`-o obj/kern/init/init.o` 生成 `init.o` (目标代码)文件

- `ld` 命令：链接各个文件

以

```shell
ld -m    elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel  obj/kern/init/init.o obj/kern/libs/readline.o obj/kern/libs/stdio.o obj/kern/debug/kdebug.o obj/kern/debug/kmonitor.o obj/kern/debug/panic.o obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/intr.o obj/kern/driver/picirq.o obj/kern/trap/trap.o obj/kern/trap/trapentry.o obj/kern/trap/vectors.o obj/kern/mm/pmm.o  obj/libs/printfmt.o obj/libs/string.o
```

为例，各个参数的含义如下：

`-m elf_i386` 输出兼容 i386 架构的 elf 文件

`-nostdlib` 仅搜索那些在命令行上显式指定的库路径，在连接脚本中 ( 包含在命令行上指定的连接脚本 ) 指定的库路径都被忽略

`-T tools/kernel.ld` 指定链接脚本为 `kernel.ld`

`-o bin/kernel` 输出 `kernel` 文件

剩下的参数都是需要链接的文件

- `dd`命令：用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换。

以

```shell
dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc
```

为例

`if=bin/kernel` 指定输入文件名

`of=bin/ucore.img` 指定输出文件名

`seek=1` 从输出文件开头跳过 1 个块后再开始复制

`conv=notrunc` 用指定的参数转换文件， `notrunc` 参数为不截短输出文件

### 练习1.2

- 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？

根据练习1.1可知，从 `sign.c` 中可以找到符合规范的硬盘主引导扇区的特征，由 `sign.c` 中的以下一段代码可知：一个被系统认为是符合规范的硬盘主引导扇区的特征是以 0x55AA 结尾，大小为 512 字节。

```c
    char buf[512];
    memset(buf, 0, sizeof(buf));
    FILE *ifp = fopen(argv[1], "rb");
    int size = fread(buf, 1, st.st_size, ifp);
    if (size != st.st_size) {
        fprintf(stderr, "read '%s' error, size is %d.\n", argv[1], size);
        return -1;
    }
    fclose(ifp);
    buf[510] = 0x55;
    buf[511] = 0xAA;
    FILE *ofp = fopen(argv[2], "wb+");
    size = fwrite(buf, 1, 512, ofp);
    if (size != 512) {
        fprintf(stderr, "write '%s' error, size is %d.\n", argv[2], size);
        return -1;
    }
    fclose(ofp);
    printf("build 512 bytes boot sector: '%s' success!\n", argv[2]);
    return 0;
```

## 练习2：使用qemu执行并调试lab1中的软件。

> 为了熟悉使用 qemu 和 gdb 进行的调试工作，我们进行如下的小练习：
>
> 1. 从 CPU 加电后执行的第一条指令开始，单步跟踪 BIOS 的执行。
> 2. 在初始化位置 0x7c00 设置实地址断点,测试断点正常。
> 3. 从 0x7c00 开始跟踪代码运行,将单步跟踪反汇编得到的代码与 `bootasm.S` 和 `bootblock.asm`进行比较。
> 4. 自己找一个 bootloader 或内核中的代码位置，设置断点并进行测试。
>
> > 提示：参考附录“启动后第一条执行的指令”，可了解更详细的解释，以及如何单步调试和查看 BIOS 代码。
> >
> > 提示：查看 `labcodes_answer/lab1_result/tools/lab1init` 文件，用如下命令试试如何调试 bootloader 第一条指令：
> >
> > ```shell
> >  $ cd labcodes_answer/lab1_result/
> >  $ make lab1-mon
> > ```
>
### 练习2.1


- 从 CPU 加电后执行的第一条指令开始，单步跟踪 BIOS 的执行。

进入 lab1 文件目录后，输入 `make debug` 打开 QEMU 进行模拟，发现弹出了另一个命令行窗口，输出了以下内容：

```asm
0x0000fff0 in ?? ()
The target architecture is assumed to be i8086
Breakpoint 1 at 0x7c00

Breakpoint 1, 0x00007c00 in ?? ()
=> 0x7c00:      cli    
   0x7c01:      cld    
(gdb) 
```

此时运行到了位于 0x7c00 处的断点，像是练习2.2的内容。

查阅 ppt 发现这里不是 CPU 加电后执行的第一条指令应该在的内存地址，执行的也不是 BIOS 。

打开 lab1 里的 Makefile 文件，查找 `make debug` 对应部分的代码

```makefile
debug: $(UCOREIMG)
	$(V)$(QEMU) -S -s -parallel stdio -hda $< -serial null &
	$(V)sleep 2
	$(V)$(TERMINAL) -e "gdb -q -tui -x tools/gdbinit"
```

得知使用 `make debug` 时执行了上面 3 行指令，其中第 1 行是打开 QEMU 加载 `ucore.img` 进行模拟，第 3 行是打开一个新的命令行窗口执行`gdb -q -tui -x tools/gdbinit`

查阅 ucore 实验指导书，发现 `tool/gdbinit` 是 gdb 配置文件，用于让 gbd 在启动时自动载入一些命令

而 `tools/gdbinit` 中的内容如下：

```shell
file bin/kernel
target remote :1234
set architecture i8086
b *0x7c00
continue
x /2i $pc
# break kern_init
# continue
```

可以发现这段代码是装载内核的可执行文件进行调试，并且在 0x7c00 处设置了断点

为了让 gdb 从 CPU 加电后执行的第一条指令开始调试，修改 tools/gdbinit 的内容为：

```shell
file bin/bootblock
target remote :1234
set architecture i8086
x /2i $pc
```

结果报错了：

```
tools/gdbinit:1: Error in sourced command file:
"/home/bbrl/ucore-master/labcodes/lab1/bin/bootblock": not in executable format:
 File format not recognized
```

因为 `bootblock` 不是可执行文件，无法装载进 gdb 进行调试

所以修改 `tools/gdbinit` 的内容为：

```shell
target remote :1234
set architecture i8086
x /2i $pc
```

此时 gdb 的输出如下：

```asm
0x0000fff0 in ?? ()
The target architecture is assumed to be i8086
=> 0xfff0:      add    %al,(%eax)
   0xfff2:      add    %al,(%eax)
(gdb) 
```

根据 ppt 里实模式下的内存布局， CPU 加电后执行的第一条指令所处的地址应该是 0xffff0 而不是 0xfff0 。但是，实模式下的寻址方式与保护模式不同，开机时 CPU 的 CS:IP 寄存器被强制初始化为 0xf000:0xfff0 ，如果 gdb 输出的是 IP 寄存器的值，这里应该就是 CPU 加电后执行的第一条指令。

接着输入`si`(`stepi`命令的缩写)，单步跟踪 BIOS 的执行：

```shell
(gdb) si
0x0000e05b in ?? ()
```

0xffff0 处的内容是跳转指令 `jmp f000:e05b` ，因此执行下一条指令时 IP 寄存器的值为 0xe05b

接着继续单步跟踪BIOS的执行，发现全是 `in ?? ()`。使用 `x /2i $pc` 反汇编当前的指令，得到以下输出：

```asm
(gdb) x /2i $pc
=> 0xe066:      add    %al,(%eax)
   0xe068:      add    %al,(%eax)
   0xe06a:      add    %al,(%eax)
   0xe06c:      add    %al,(%eax)
   0xe06e:      add    %al,(%eax)
   0xe070:      add    %al,(%eax)
(gdb) 
```

无法反编译出 BIOS 执行的内容，使用 `quit` 退出。

### 练习2.2

- 在初始化位置 0x7c00 设置实地址断点，测试断点正常。

为了设置断点，修改 `tools/gdbinit` 中的内容，如下所示：

```shell
file bin/kernel
target remote :1234
set architecture i8086
b *0x7c00
continue
x /2i $pc
# break kern_init
# continue
```

如实验2.1中所述的那样，得到以下输出：

```asm
0x0000fff0 in ?? ()
The target architecture is assumed to be i8086
Breakpoint 1 at 0x7c00

Breakpoint 1, 0x00007c00 in ?? ()
=> 0x7c00:      cli    
   0x7c01:      cld    
(gdb) 
```

由此可知，断点正常。

### 练习2.3

- 从 0x7c00 开始跟踪代码运行，将单步跟踪反汇编得到的代码与 `bootasm.S` 和 `bootblock.asm` 进行比较。

根据实验指导书的提示，在 `tools/gdbinit` 中加入以下代码，使 gdb 在每次 gdb 命令行前强制反汇编当前的指令：

```
define hook-stop 
x/i $pc 
end 
```

接着输入 `make debug` ，在得出练习2.2的输出后，单步跟踪，得到以下反汇编代码：

```asm
0x7c00:      cli    
0x7c01:      cld    
0x7c02:      xor    %eax,%eax
0x7c04:      mov    %eax,%ds
0x7c06:      mov    %eax,%es
0x7c08:      mov    %eax,%ss
0x7c0a:      in     $0x64,%al
0x7c0c:      test   $0x2,%al
0x7c0e:      jne    0x7c0a
0x7c10:      mov    $0xd1,%al
0x7c12:      out    %al,$0x64
0x7c14:      in     $0x64,%al
0x7c16:      test   $0x2,%al
0x7c18:      jne    0x7c14
0x7c1a:      mov    $0xdf,%al
0x7c1c:      out    %al,$0x60
0x7c1e:      lgdtl  (%esi)
0x7c23:      mov    %cr0,%eax
0x7c26:      or     $0x1,%ax
0x7c2a:      mov    %eax,%cr0
0x7c2d:      ljmp   $0xb866,$0x87c32
0x7c32:      mov    %0x10,%ax
0x7c36:      mov    %eax,%ds
0x7c38:      mov    %eax,%es
0x7c3a:      mov    %eax,%fs
0x7c3c:      mov    %eax,%gs
0x7c3e:      mov    %eax,%ss
0x7c40:      mov    $0x0,%ebp
0x7c45:      mov    $0x7c00,%esp
0x7c4a:      call   0x7cd1

0x7cd1:      push   %ebp
0x7cd2:      mov    %esp,%ebp
0x7cd4:      push   %edi
0x7cd5:      push   %esi
0x7cd6:      push   %ebx
0x7cd7:      mov    $0x1,%ebx
0x7cdc:      sub    $0x1c,%esp
0x7cdf:      lea    0x7f(%ebx),%eax
0x7ce2:      mov    %ebx,%edx
0x7ce4:      shl    $0x9,%eax
0x7ce7:      inc    %ebx
0x7ce8:      call   0x7c72

0x7c72:      push   %ebp
0x7c73:      mov    %edx,%ecx
0x7c75:      mov    %esp,%ebp
0x7c77:      mov    $0x1f7,%edx
0x7c7c:      push   %edi
0x7c7d:      mov    %eax,%edi
0x7c7f:      in     (%dx),%al
0x7c80:      and    $0xffffffc0,%eax
0x7c83:      cmp    $0x40,%al
0x7c85:      jne    0x7c7f
0x7c87:      mov    $0x1f2,%edx
0x7c8c:      mov    $0x1,%al
0x7c8e:      out    %al,(%dx)
0x7c8f:      movzbl %cl,%eax
0x7c92:      mov    $0xf3,%dl
0x7c94:      out    %al,(%dx)
0x7c95:      movzbl %ch,%eax
0x7c98:      mov    $0xf4,%dl
0x7c9a:      out    %al,(%dx)
0x7c9b:      mov    %ecx,%eax
0x7c9d:      mov    $0xf5,%dl
0x7c9f:      shr    $0x10,%eax
0x7ca2:      movzbl %al,%eax
0x7ca5:      out    %al,(%dx)
0x7ca6:      shr    $0x18,%ecx
...
```

反汇编的代码与 `bootasm.S` 比较，有以下差异：

1. 反汇编代码中的指令不带指示长度的后缀，而 `bootasm.S` 的指令有。比如，反汇编代码中的 `xor %eax,%eax` 在 `bootasm.S` 中是 `xorw %ax, %ax` 。
2. 反汇编代码中的通用寄存器是 32 位 ( 带有 e 前缀 ) ，而 `bootasm.S` 的代码中的通用寄存器是 16 位 ( 不带 e 前缀 )。
3. 反汇编代码中将标识符转换成了物理地址。比如，反汇编代码中的 `jne 0x7c0a` 在 `bootasm.S` 中是 `jnz seta20.1` (  `jne` 和 `jnz` 对应于完全相同的机器码)。
4. 反汇编代码中不止包含 `bootasm.S` 的代码。反汇编代码的 `0x7c4a: call 0x7cd1` 对应 `bootasm.S` 中的 `call bootmain` ，之后的反汇编代码对应的是 `bootmain.c` 中的内容。

与 `bootblock.asm` 比较，有以下差异：

1. 对于较长的指令，在 `bootblock.asm` 中似乎被分为多个指令来储存，但在二进制文件中是一样的。例如，反汇编代码中的 `0x7c1e: lgdtl (%esi)` 和  `0x7c23: mov %cr0,%eax` 在 `bootblock.asm` 中对应 `lgdtl  (%esi)` 、 `insb   (%dx),%es:(%edi)` 、  `jl     7c33` 、  `and    %al,%al` 四条指令。
2. `bootblock.asm` 中除了代码以外还有注释、部分 C 语言源代码以及每条汇编指令对应的机器码，而反汇编代码只有汇编代码。

### 练习2.4

- 自己找一个 bootloader 或内核中的代码位置，设置断点并进行测试。

修改 `tools/gdbinit` 中的内容，将断点位置设置在内核代码的 `kern_init()` 处：

```shell
file bin/kernel
target remote :1234
break kern_init
continue
```

执行`make debug` ，得到以下输出：

```
   ┌──kern/init/init.c─────────────────────────────────────────────────────────┐
   │12      int kern_init(void) __attribute__((noreturn));                     │
   │13      void grade_backtrace(void);                                        │
   │14      static void lab1_switch_test(void);                                │
   │15                                                                         │
   │16      int                                                                │
B+>│17      kern_init(void) {                                                  │
   │18          extern char edata[], end[];                                    │
   │19          memset(edata, 0, end - edata);                                 │
   │20                                                                         │
   │21          cons_init();                // init the console                │
   │22                                                                         │
   │23          const char *message = "(THU.CST) os is loading ...";           │
   │24          cprintf("%s\n\n", message);                                    │
   └───────────────────────────────────────────────────────────────────────────┘
remote Thread 1 In: kern_init                           Line: 17   PC: 0x100000 
0x0000fff0 in ?? ()
Breakpoint 1 at 0x100000: file kern/init/init.c, line 17.

Breakpoint 1, kern_init () at kern/init/init.c:17
(gdb) 
```

可以看到程序成功地在 `kern_init()` 函数的入口处停了下来，断点设置成功。

## 练习3：分析bootloader进入保护模式的过程。

> BIOS 将通过读取硬盘主引导扇区到内存，并转跳到对应内存中的位置执行 bootloader 。请分析 bootloader 是如何完成从实模式进入保护模式的。
>
> 提示：需要阅读**小节“保护模式和分段机制”**和 `lab1/boot/bootasm.S` 源码，了解如何从实模式切换到保护模式，需要了解：
>
> - 为何开启 A20，以及如何开启 A20
> - 如何初始化 GDT 表
> - 如何使能和进入保护模式
>

### 练习3.1

- 为何开启 A20 ，以及如何开启 A20 

为何开启 A20？首先从 A20 是什么说起。

8086CPU 有 20 位地址线，可寻址空间范围为 0 ~ 2^20 ( 00000h ~ fffffh ) 的 1MB 内存空间。但 8086 寻址使用的寄存器都是 16 位的，无法直接寻址 1MB 内存空间，所以 8086 提供了段地址加偏移地址的地址转换机制。 PC 机的寻址结构是 segment:offset ， segment 和 offset 都是 16 位寄存器，最大值是为0ffffh 。而换算成物理地址的计算方法是将 segment 左移 4 位，再加上 offset 。因此， segment:offset 所能表达的寻址空间最大为 0ffff0h + 0ffffh = 10ffefh ，可表示的寻址空间比 8086 的物理寻址空间更大。

8086 只有 20 根地址线，使用这种寻址方式并没有任何影响，访问 100000h ~ 10ffefh 的地址时会发生“回卷”，忽视地址的第 21 位，实际访问的是 00000h ~ 0ffefh 的地址。但下一代的 80286CPU 有 24 根地址线，还提供了保护模式，可以访问到 1MB 以上的内存空间，真的可以访问到100000h ~ 10ffefh 的地址。为了保持向下兼容，就有了 A20 Gate。

A20 是第 21 条地址线，开机时 A20 地址线控制被屏蔽，使得 A20 地址线的值总是为 0 ，这样就避免了在实模式下访问到 100000h ~ 10ffefh 地址，实现了向下兼容。

具体的实现是将 A20 地址线控制和键盘控制器 8042 的一个输出进行 AND 操作，来控制 A20 地址线的使能和屏蔽，一开始时 A20 地址线控制是被屏蔽的，直到系统软件通过一定的 IO 操作去打开它。

那么为什么要开启 A20 呢？

在保护模式下，由于使用了 32 位地址线，如果 A20 恒等于 0 ，那么系统只能访问奇数兆的内存，即只能访问 0 - 1M 、2 - 3M 、 4 - 5M 等。这样无法有效访问所有可用内存。所以在保护模式下，必须开启 A20 。

如何开启 A20 ？

只需要操作 8042 芯片输出端口 ( 64h ) 的 bit 1 ，就可以控制 A20  Gate 。但是，当准备向 8042 的输入缓冲区里写数据时，缓冲区中可能还有其它数据没有处理。所以，要首先禁止键盘操作，同时等待数据缓冲区中没有数据以后，才能真正地去操作 8042 打开或者关闭 A20 Gate 。打开 A20 Gate 的具体步骤大致如下：

1. 等待 8042 Input buffer 为空；
2. 发送 Write 8042 Output Port 命令到 8042 Input buffer；
3. 等待 8042 Input buffer 为空；
4. 将从 8042 Output Port 中得到字节的第 2 位置为 1 ，然后将其写入 8042 Input buffer；

`bootasm.S` 中的代码也是如此，只是简略了一部分 ( 似乎没有禁止键盘操作 ) ：

```asm
# Enable A20:
#  For backwards compatibility with the earliest PCs, physical
#  address line 20 is tied low, so that addresses higher than
#  1MB wrap around to zero by default. This code undoes this.

seta20.1:
    inb $0x64, %al                                  # 等待8042 Input buffer为空
    testb $0x2, %al                                 # 如果 %al 第低2位为1，则ZF = 0, 则跳转
    jnz seta20.1                                    # 如果 %al 第低2位为0，则ZF = 1, 则不跳转
    movb $0xd1, %al                                 # 0xd1 -> port 0x64
    outb %al, $0x64                                 # 0xd1 : 向 8042 Output Port 写数据

seta20.2:
    inb $0x64, %al                                  # 等待8042 Input buffer为空
    testb $0x2, %al
    jnz seta20.2
    movb $0xdf, %al                                 # 0xdf -> port 0x60, 0xdf = 11011111
    outb %al, $0x60                                 # 将 8042 Output Port 的 A20 bit(bit 1)置1
```

### 练习3.2

- 如何初始化 GDT 表？

查看 `bootasm.S` 中的代码，发现使用了 `lgdt gdtdesc` 指令来载入 GDT 表，该指令载入的是 `geddesc` 标识符的值。

 `lgdt` 指令将 GDT 的入口地址 ( 在这里也就是 `gdtdesc` 标识符对应的地址 ) 放入 CPU 的 GDTR 寄存器，此后 CPU 就会根据此寄存器中的内容作为 GDT 的入口来访问 GDT 表。而 GDTR 是一个 48 位寄存器，其中低 16 位储存的是 GDT 表长度，高 32 位储存的是 GDT 表在内存中的起始地址。

 `gdtdesc` 标识符后描述的全局描述符表由 `bootasm.S` 中的以下代码描述，可以发现该 GDT 表的长度是 0x17 + 1 = 0x18 ( 即 24 字节 ) ，起始地址是标识符 `gdt` 的值。

```asm
# Bootstrap GDT
.p2align 2                                          # force 4 byte alignment
gdt:
    SEG_NULLASM                                     # null seg
    SEG_ASM(STA_X|STA_R, 0x0, 0xffffffff)           # code seg for bootloader and kernel
    SEG_ASM(STA_W, 0x0, 0xffffffff)                 # data seg for bootloader and kernel

gdtdesc:
    .word 0x17                                      # sizeof(gdt) - 1
    .long gdt                                       # address gdt
```

标识符 `gdt` 后就是 GDT 表的具体内容，包含 3 项，每项的大小为 8 字节。第一项是空白项，后面两项段描述符分别对应代码段和数据段。

查阅 `asm.h` ，找到段描述符初始化函数 `SEG_ASM` 的定义，如下：

```asm
#define SEG_ASM(type,base,lim)                 \
  .word (((lim) >> 12) & 0xffff), ((base) & 0xffff);     \
  .byte (((base) >> 16) & 0xff), (0x90 | (type)),       \
    (0xC0 | (((lim) >> 28) & 0xf)), (((base) >> 24) & 0xff)
```

该函数有三个参数， type 是这个段的访问权限， base 是这个段的起始地址， lim 是这个段的大小上限。

### 练习3.3

- 如何使能和进入保护模式？

将 `cr0` 寄存器置 1 就能使能和进入保护模式，`bootasm.S` 中的这段代码完成了这一功能：

```asm
movl %cr0, %eax
orl $CR0_PE_ON, %eax
movl %eax, %cr0
```
### 练习3

- bootloader 是如何完成从实模式进入保护模式的？

`bootasm.S` 中的以下代码完成了这一过程：

```asm
start:
.code16                                             # Assemble for 16-bit mode
    # 关中断
    cli                                             # Disable interrupts
    # 操作方向标志位DF，使内存串操作指令寻址向地址增加的方向执行
    cld                                             # String operations increment
    # 设置重要数据段寄存器 (DS, ES, SS).
    xorw %ax, %ax                                   # Segment number zero
    movw %ax, %ds                                   # -> Data Segment
    movw %ax, %es                                   # -> Extra Segment
    movw %ax, %ss                                   # -> Stack Segment
    
    # 开启 A20:
seta20.1:
    inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
    testb $0x2, %al                                 # 如果 %al 第低2位为1，则ZF = 0, 则跳转
    jnz seta20.1                                    # 如果 %al 第低2位为0，则ZF = 1, 则不跳转

    movb $0xd1, %al                                 # 0xd1 -> port 0x64
    outb %al, $0x64                                 # 0xd1 means: write data to 8042's P2 port

seta20.2:
    inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
    testb $0x2, %al
    jnz seta20.2

    movb $0xdf, %al                                 # 0xdf -> port 0x60
    outb %al, $0x60                                 # 0xdf = 11011111, means set P2's A20 bit(the 1 bit) to 1
    
    # 载入全局描述符表GDT
    lgdt gdtdesc
    # 将cr0寄存器置1，使能和进入保护模式
    movl %cr0, %eax
    orl $CR0_PE_ON, %eax
    movl %eax, %cr0
    
    # 通过长跳转更新cs的基地址
    ljmp $PROT_MODE_CSEG, $protcseg
    
.code32                                             # Assemble for 32-bit mode
protcseg:
    # 设置保护模式数据段寄存器
    movw $PROT_MODE_DSEG, %ax                       # Our data segment selector
    movw %ax, %ds                                   # -> DS: Data Segment
    movw %ax, %es                                   # -> ES: Extra Segment
    movw %ax, %fs                                   # -> FS
    movw %ax, %gs                                   # -> GS
    movw %ax, %ss                                   # -> SS: Stack Segment

    # 设置堆栈指针
    movl $0x0, %ebp
    movl $start, %esp
    # 完成进入保护模式，调用bootmain函数
    call bootmain
```

总结起来可以分为以下几步：

1. 关中断，调整串操作方向为自增
2. 设置重要数据段寄存器 ( `DS` , `ES` , `SS` )，将它们的值清0
3. 开启 A20
4. 载入全局描述符表 GDT
5. 将 `cr0` 寄存器置 1 ，使能和进入保护模式
6. 通过长跳转更新 `CS` 的基地址
7. 设置保护模式数据段寄存器
8. 设置堆栈指针
9. 调用 `bootmain` 函数

## 练习4：分析bootloader加载ELF格式的OS的过程。

> 通过阅读 bootmain.c，了解 bootloader 如何加载 ELF 文件。通过分析源代码和通过 qemu  来运行并调试bootloader&OS，
>
> - bootloader 如何读取硬盘扇区的？
> - bootloader 是如何加载 ELF 格式的  OS？
>
> 提示：可阅读“硬盘访问概述”，“ELF执行文件格式概述”这两小节。
>

### 练习4.1

- bootloader 是如何读取硬盘扇区的？

阅读 `bootmain.c` ，找到了读取硬盘扇区的函数 `readseg()` ，它实现了对读取单个扇区的函数 `readsect()` 的封装：

```c
/* *
 * readseg - read @count bytes at @offset from kernel into virtual address @va,
 * might copy more than asked.
 * */
static void
readseg(uintptr_t va, uint32_t count, uint32_t offset) {
    uintptr_t end_va = va + count;

    // round down to sector boundary
    va -= offset % SECTSIZE;

    // translate from bytes to sectors; kernel starts at sector 1
    uint32_t secno = (offset / SECTSIZE) + 1;

    // If this is too slow, we could read lots of sectors at a time.
    // We'd write more to memory than asked, but it doesn't matter --
    // we load in increasing order.
    for (; va < end_va; va += SECTSIZE, secno ++) {
        readsect((void *)va, secno);
    }
}
```

该函数将以字节地址转换为扇区地址，然后根据需要读取的扇区的数量依次调用读取单个扇区的函数。

读取单个硬盘扇区的函数 `readsect()` 的代码如下：

```c
/* readsect - 从 @secno 的位置读取一个扇区的内容放入 @dst 对应的内存地址*/
static void
readsect(void *dst, uint32_t secno) {
  // 等待磁盘准备
  waitdisk();
  // 写入参数，只读取1个扇区
  outb(0x1F2, 1);
  // 写入参数，将需要读取的扇区的相关信息写入IO地址
  outb(0x1F3, secno & 0xFF);
  outb(0x1F4, (secno >> 8) & 0xFF);
  outb(0x1F5, (secno >> 16) & 0xFF);
  outb(0x1F6, ((secno >> 24) & 0xF) | 0xE0);
  // 发出读取磁盘的命令
  outb(0x1F7, 0x20);           // cmd 0x20 - read sectors
  // 等待磁盘准备
  waitdisk();
  // 读取一个扇区的内容
  insl(0x1F0, dst, SECTSIZE / 4);
}
```

由上可知读一个扇区的流程大致如下：

1. 等待磁盘准备好
2. 发出读取扇区的命令
3. 等待磁盘准备好
4. 把磁盘扇区数据读到指定内存

这四步操作都需要使用 `in` / `out` 指令操作 IO 端口。 `bootmain.c` 中使用的 `outb()` 、 `inb()` 等函数在 `x86.h` 中描述，它们是被封装的用于操作 IO 端口的与函数名同名的汇编指令 `outb` 、 `inb` 等。

用于等待磁盘准备好的函数 `waitdisk()` 代码如下：

```c
/* waitdisk - wait for disk ready */
static void
waitdisk(void) {
  while ((inb(0x1F7) & 0xC0) != 0x40)
    /* do nothing */;
}
```

`inb(0x1F7)` 是读取磁盘的状态和命令寄存器，仅当这个 IO  地址的第 1 位为 1 ，第 2 位为 0 时跳出循环，此时磁盘已准备好。

### 练习4.2

- bootloader 是如何加载 ELF 格式的 OS ？

阅读相关资料及 `bootmain.c` ，在 `bootmain.c` 中， `bootmain()` 函数实现了加载 ELF 格式的 OS ，代码如下：

```c
/* bootmain - the entry of bootloader */
void
bootmain(void) {
    // 从硬盘中读取第一页(4KB)的内容，也即是 ELF 的 ELF header
    readseg((uintptr_t)ELFHDR, SECTSIZE * 8, 0);

    // 判断这是否是一个有效的 ELF 文件
    if (ELFHDR->e_magic != ELF_MAGIC) {
        goto bad;
    }

    struct proghdr *ph, *eph;

    // 根据 ELF header 找到 program header 表，加载每个 program header 描述的程序段至内存(无视 ph 标签)
    ph = (struct proghdr *)((uintptr_t)ELFHDR + ELFHDR->e_phoff);
    eph = ph + ELFHDR->e_phnum;
    for (; ph < eph; ph ++) {
        readseg(ph->p_va & 0xFFFFFF, ph->p_memsz, ph->p_offset);
    }

    // 从 ELF header 找到内核的入口
    // note: does not return
    ((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))();

bad:
    outw(0x8A00, 0x8A00);
    outw(0x8A00, 0x8E00);

    /* do nothing */
    while (1);
}

```

其过程是这样的：

1. 调用 `readseg()` 函数读取硬盘中的前 8 个扇区，获取 ELF header
2. 校验 ELF Header 的 `e_magic` 字段，确保这是一个ELF文件
3. 根据 ELF Header 的 `e_phoff` 字段得到 program header 表的位置偏移，根据 `e_phnum` 字段得到 program header 表中的入口数目。这样就可以得到每个 program header 的指针
4. 遍历 program header 表中的每一项，由 `p_offset` 字段得到段在硬盘中相对文件头的偏移位置，由 `memsz` 字段得到段在内存中应占用的字节数，由 va 字段得到段的第一个字节将被放到内存中的虚拟地址。随后调用 `readseg()` 函数，读取扇区，将硬盘中对应位置的段加载进内存对应的地址。
5. 加载好内核程序的每个段之后，根据 ELF Header 的 `e_entry` 字段找到内核程序的入口，启动内核

## 练习5：实现函数调用堆栈跟踪函数

> 我们需要在 lab1 中完成 `kdebug.c` 中函数 `print_stackframe` 的实现，可以通过函数 `print_stackframe` 来跟踪函数调用堆栈中记录的返回地址。在如果能够正确实现此函数，可在 lab1 中执行 “`make qemu`”后，在 qemu 模拟器中得到类似如下的输出：
>
> ```
> ……
> ebp:0x00007b28 eip:0x00100992 args:0x00010094 0x00010094 0x00007b58 0x00100096
>     kern/debug/kdebug.c:305: print_stackframe+22
> ebp:0x00007b38 eip:0x00100c79 args:0x00000000 0x00000000 0x00000000 0x00007ba8
>     kern/debug/kmonitor.c:125: mon_backtrace+10
> ebp:0x00007b58 eip:0x00100096 args:0x00000000 0x00007b80 0xffff0000 0x00007b84
>     kern/init/init.c:48: grade_backtrace2+33
> ebp:0x00007b78 eip:0x001000bf args:0x00000000 0xffff0000 0x00007ba4 0x00000029
>     kern/init/init.c:53: grade_backtrace1+38
> ebp:0x00007b98 eip:0x001000dd args:0x00000000 0x00100000 0xffff0000 0x0000001d
>     kern/init/init.c:58: grade_backtrace0+23
> ebp:0x00007bb8 eip:0x00100102 args:0x0010353c 0x00103520 0x00001308 0x00000000
>     kern/init/init.c:63: grade_backtrace+34
> ebp:0x00007be8 eip:0x00100059 args:0x00000000 0x00000000 0x00000000 0x00007c53
>     kern/init/init.c:28: kern_init+88
> ebp:0x00007bf8 eip:0x00007d73 args:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8
> <unknow>: -- 0x00007d72 –
> ……
> ```
>
> 请完成实验，看看输出是否与上述显示大致一致，并解释最后一行各个数值的含义。
>
> 提示：可阅读小节“函数堆栈”，了解编译器如何建立函数调用关系的。在完成 lab1 编译后，查看`lab1/obj/bootblock.asm`，了解 bootloader 源码与机器码的语句和地址等的对应关系；查看`lab1/obj/kernel.asm`，了解 ucore OS 源码与机器码的语句和地址等的对应关系。
>
> 要求完成函数 `kern/debug/kdebug.c::print_stackframe` 的实现，提交改进后源代码包（可以编译执行），并在实验报告中简要说明实现过程，并写出对上述问题的回答。
>

- 在 lab1 中完成 `kdebug.c` 中函数 `print_stackframe` 的实现，可以通过函数 `print_stackframe` 来跟踪函数调用堆栈中记录的返回地址。

阅读相关资料后，打开 `kdebug.c` ，根据注释内容实现函数 `print_stackframe()` ：

```c
void
print_stackframe(void) {
     /* LAB1 YOUR CODE : STEP 1 */
     /* (1) 调用 read_ebp() 获得 ebp 的值. ebp 的类型为 (uint32_t);
      * (2) 调用 read_eip() 获得 eip 的值. eip 的类型为 (uint32_t);
      * (3) 遍历 0 .. STACKFRAME_DEPTH
      *    (3.1) 输出 ebp, eip 的值
      *    (3.2) (uint32_t)调用参数 [0..4] = 地址 (unit32_t)ebp +2 [0..4] 的内容
      *    (3.3) cprintf("\n");
      *    (3.4) 调用 print_debuginfo(eip-1) 打印C调用函数名和行号等。
      *    (3.5) 弹出一个调用栈帧
      *           注意: 调用函数的返回地址 eip  = ss:[ebp+4]
      *                 调用函数的 ebp = ss:[ebp]
      */
    uint32_t ebp = read_ebp();
    uint32_t eip = read_eip();
    for (int i = 0; i < STACKFRAME_DEPTH; i++){
        uint32_t *args = (uint32_t *)ebp + 2;
        cprintf("ebp:0x%08x eip:0x%08x ", ebp, eip);
        cprintf("args:0x%08x 0x%08x 0x%08x 0x%08x", args[0], args[1], args[2], args[3]);
        cprintf("\n");
        print_debuginfo(eip-1);
        eip = *((uint32_t *)ebp + 4);
        ebp = *((uint32_t *)ebp);
    }
}
```

打印的具体内容根据实验指导书中的输出结果可以推出。而整个函数中其中比较关键的是 (3.5) 弹出调用堆栈帧的这一步，需要通过当前栈底 `ebp` 来获取上个调用函数的栈底 `ebp` 和返回地址 `eip` 。

根据"函数堆栈"小节，在程序执行到一个函数的实际指令前，已经有以下数据顺序入栈：参数、返回地址、`ebp` 寄存器。由此得到类似如下的栈结构 ( 参数入栈顺序跟调用方式有关，这里以 C 语言默认的 CDECL 为例 ) ：

```
+|  栈底方向      | 高位地址
 |    ...        |
 |    ...        |
 |  参数3        |
 |  参数2        |
 |  参数1        |
 |  返回地址      |
 |  上一层[ebp]   | <-------- [ebp]
 |  局部变量       |  低位地址
```

由此可知栈中上个函数的栈底就储存在当前栈底 `ebp` 所指向的内存空间中，而上个函数的返回地址则储存在当前栈底 `ebp` 往上的一个地址。因为实际编程中的操作系统是 32 位的，地址也是 32 位的，而 32 位就是 4 字节，所以上个函数的返回地址储存在 `edp + 4` 指向的内存空间中。

随后执行 `make qemu`， 编译时发生错误：

```shell
+ cc kern/debug/kdebug.c
kern/debug/kdebug.c: In function ‘print_stackframe’:
kern/debug/kdebug.c:307:5: error: ‘for’ loop initial declarations are only allowed in C99 mode
     for (int i = 0; i < STACKFRAME_DEPTH; i++){
     ^
kern/debug/kdebug.c:307:5: note: use option -std=c99 or -std=gnu99 to compile your code
make: *** [obj/kern/debug/kdebug.o] Error 1

```

错误为：'for' 循环初始声明只允许在 C99 模式下使用。查看项目中其他使用 for 循环的语句后，我们将变量 i 的声明放在了for 循环之前，顺利通过编译。

接着执行 `make qemu`，得到以下输出：

```
ebp:0x00007b08 eip:0x001009a7 args:0x00010094 0x00000000 0x00007b38 0x00100092
    kern/debug/kdebug.c:306: print_stackframe+22
ebp:0x00007b18 eip:0x00007b38 args:0x00000000 0x00000000 0x00000000 0x00007b88
    <unknow>: -- 0x00007b37 --
ebp:0x00007b38 eip:0x00000000 args:0x00000000 0x00007b60 0xffff0000 0x00007b64
    <unknow>: -- 0xffffffff --
ebp:0x00007b58 eip:0xffff0000 args:0x00000000 0xffff0000 0x00007b84 0x00000029
    <unknow>: -- 0xfffeffff --
ebp:0x00007b78 eip:0x00007b84 args:0x00000000 0x00100000 0xffff0000 0x0000001d
    <unknow>: -- 0x00007b83 --
ebp:0x00007b98 eip:0xffff0000 args:0x0010349c 0x00103480 0x0000130a 0x00000000
    <unknow>: -- 0xfffeffff --
ebp:0x00007bc8 eip:0x0000130a args:0x00000000 0x00000000 0x00000000 0x00010094
    <unknow>: -- 0x00001309 --
ebp:0x00007bf8 eip:0x00000000 args:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8
    <unknow>: -- 0xffffffff --
ebp:0x00000000 eip:0x64e4d08e args:0xf000e2c3 0xf000ff53 0xf000ff53 0xf000ff54
    <unknow>: -- 0x64e4d08d --
ebp:0xf000ff53 eip:0xf000ff53 args:0x00000000 0x00000000 0x00000000 0x00000000
    <unknow>: -- 0xf000ff52 --
ebp:0x00000000 eip:0x00000000 args:0xf000e2c3 0xf000ff53 0xf000ff53 0xf000ff54
    <unknow>: -- 0xffffffff --
ebp:0xf000ff53 eip:0xf000ff53 args:0x00000000 0x00000000 0x00000000 0x00000000
    <unknow>: -- 0xf000ff52 --
ebp:0x00000000 eip:0x00000000 args:0xf000e2c3 0xf000ff53 0xf000ff53 0xf000ff54
    <unknow>: -- 0xffffffff --
ebp:0xf000ff53 eip:0xf000ff53 args:0x00000000 0x00000000 0x00000000 0x00000000
    <unknow>: -- 0xf000ff52 --
ebp:0x00000000 eip:0x00000000 args:0xf000e2c3 0xf000ff53 0xf000ff53 0xf000ff54
    <unknow>: -- 0xffffffff --
ebp:0xf000ff53 eip:0xf000ff53 args:0x00000000 0x00000000 0x00000000 0x00000000
    <unknow>: -- 0xf000ff52 --
ebp:0x00000000 eip:0x00000000 args:0xf000e2c3 0xf000ff53 0xf000ff53 0xf000ff54
    <unknow>: -- 0xffffffff --
ebp:0xf000ff53 eip:0xf000ff53 args:0x00000000 0x00000000 0x00000000 0x00000000
    <unknow>: -- 0xf000ff52 --
ebp:0x00000000 eip:0x00000000 args:0xf000e2c3 0xf000ff53 0xf000ff53 0xf000ff54
    <unknow>: -- 0xffffffff --
ebp:0xf000ff53 eip:0xf000ff53 args:0x00000000 0x00000000 0x00000000 0x00000000
    <unknow>: -- 0xf000ff52 --
```

对比实验指导书中的输出结果，可以发现存在两个问题： `print_debuginfo(eip-1)` 没能正确执行、最后一个堆栈的值连续输出了多次。

检查代码后发现， `print_debuginfo(eip-1)` 没能正确执行是因为参数eip计算错误。 `eip = *((uint32_t *)ebp + 4)` 获取的是 `ebp[4]` 的值，而注释中所述的应该是 `ebp[1]` 。于是将其修改为 `eip = *((uint32_t *)(ebp + 4))`。

翻阅其他部分的代码发现 `STACKFRAME_DEPTH` 是一个宏定义，值为 20 。这意味着如果没遍历够 20 次，即使该函数获取到了最后一个栈时仍然会尝试获取下一个栈。查看 `bootasm.S` 发现此时第一个栈的栈底是 0x0 ，于是将该 for 循环的终止条件修改成 `i < STACKFRAME_DEPTH && ebp != 0` 。

修改后的函数 `print_stackframe()` 如下：

```c
void
print_stackframe(void) {
    uint32_t ebp = read_ebp();
    uint32_t eip = read_eip();
    int i;
    for (i = 0; i < STACKFRAME_DEPTH && ebp != 0; i++){
        uint32_t *args = (uint32_t *)ebp + 2;
        cprintf("ebp:0x%08x eip:0x%08x ", ebp, eip);
        cprintf("args:0x%08x 0x%08x 0x%08x 0x%08x", args[0], args[1], args[2], args[3]);
        cprintf("\n");
        print_debuginfo(eip - 1);
        eip = *((uint32_t *)(ebp + 4));
        ebp = *((uint32_t *)ebp);
    }
}
```

再次执行 `make qemu`，得到以下输出：

```
ebp:0x00007b08 eip:0x001009a7 args:0x00010094 0x00000000 0x00007b38 0x00100092
    kern/debug/kdebug.c:306: print_stackframe+22
ebp:0x00007b18 eip:0x00100c9f args:0x00000000 0x00000000 0x00000000 0x00007b88
    kern/debug/kmonitor.c:125: mon_backtrace+10
ebp:0x00007b38 eip:0x00100092 args:0x00000000 0x00007b60 0xffff0000 0x00007b64
    kern/init/init.c:48: grade_backtrace2+33
ebp:0x00007b58 eip:0x001000bb args:0x00000000 0xffff0000 0x00007b84 0x00000029
    kern/init/init.c:53: grade_backtrace1+38
ebp:0x00007b78 eip:0x001000d9 args:0x00000000 0x00100000 0xffff0000 0x0000001d
    kern/init/init.c:58: grade_backtrace0+23
ebp:0x00007b98 eip:0x001000fe args:0x0010349c 0x00103480 0x0000130a 0x00000000
    kern/init/init.c:63: grade_backtrace+34
ebp:0x00007bc8 eip:0x00100055 args:0x00000000 0x00000000 0x00000000 0x00010094
    kern/init/init.c:28: kern_init+84
ebp:0x00007bf8 eip:0x00007d68 args:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8
    <unknow>: -- 0x00007d67 --
```

输出与实验指导书中的显示大致一致

- 解释最后一行各个数值的含义

1.  `ebp:0x00007bf8` ：由 `kern/init/init.c:28: kern_init+84` 这一行可以猜测这是 `kern_init` 函数对应栈帧的栈底，也即是 `bootmain.c` 中 `bootmain()` 函数的栈顶。使用`make debug` ，设置断点在 `kern_init` 函数的入口，启动后使用 `layout regs` 指令获取当前寄存器内容的显示，发现此时的 `ebp` 确实是 0x7bf8 。

2.  `eip:0x00007d68` ：因为这一行描述的是 `kern_init` 函数对应栈帧的相关信息，因此 `eip` 应该是 `kern_init` 函数的返回地址。在 `bootmain.c` 中的 `bootmain()` 函数中可以发现 `kern_init` 函数后的下一条指令是 `outw(0x8A00, 0x8A00)` ，查看 `bootblock.asm` 可以发现 `outw` 函数的入口地址就是 0x7d68 ，因此 `eip` 确实是 `kern_init` 函数的返回地址。

   ```asm
   static inline void
   outw(uint16_t port, uint16_t data) {
       asm volatile ("outw %0, %1" :: "a" (data), "d" (port));
       7d68:	ba 00 8a ff ff       	mov    $0xffff8a00,%edx
       7d6d:	89 d0                	mov    %edx,%eax
       7d6f:	66 ef                	out    %ax,(%dx)
       7d71:	b8 00 8e ff ff       	mov    $0xffff8e00,%eax
       7d76:	66 ef                	out    %ax,(%dx)
       7d78:	eb fe                	jmp    7d78 <bootmain+0xa7>
   ```

3. `args:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8` ：一般情况下这里应该是调用函数的各个参数，但 `kern_init` 函数没有参数。由函数 `print_stackframe()` 的代码可知，此时 `args` 数组的地址是 `ebp + 2` ，即 0x7bf8 + 2 * 4 = 0x7c00，是 `bootblock.asm` 的入口地址。`args [0..4]` 也就是 `bootblock.asm` 中前 16 字节指令的值。

完成实验后对比参考答案：

```c
void
print_stackframe(void) {
    uint32_t ebp = read_ebp(), eip = read_eip();

    int i, j;
    for (i = 0; ebp != 0 && i < STACKFRAME_DEPTH; i ++) {
        cprintf("ebp:0x%08x eip:0x%08x args:", ebp, eip);
        uint32_t *args = (uint32_t *)ebp + 2;
        for (j = 0; j < 4; j ++) {
            cprintf("0x%08x ", args[j]);
        }
        cprintf("\n");
        print_debuginfo(eip - 1);
        eip = ((uint32_t *)ebp)[1];
        ebp = ((uint32_t *)ebp)[0];
    }
}
```

发现主要存在以下差异：

1. 参考答案中 for 循环的判断条件使用了 `ebp != 0 && i < STACKFRAME_DEPTH` ，而我们的答案中两个子条件的顺序相反，这导致我的答案中每次循环会多出一次计算，性能有所下降。
2. 参考答案中输出 `args` 使用了一个 for 循环来遍历 `args` 数组，方便扩展。而我们的答案中使用了手动遍历，性能更好但扩展性较差。

## 练习6：完善中断初始化和处理 （需要编程）

> 请完成编码工作和回答如下问题：
>
> 1. 中断描述符表（也可简称为保护模式下的中断向量表）中一个表项占多少字节？其中哪几位代表中断处理代码的入口？
> 2. 请编程完善 `kern/trap/trap.c` 中对中断向量表进行初始化的函数 `idt_init` 。在 `idt_init` 函数中，依次对所有中断入口进行初始化。使用 `mmu.h` 中的 `SETGATE` 宏，填充 `idt` 数组内容。每个中断的入口由 `tools/vectors.c` 生成，使用 `trap.c` 中声明的 `vectors` 数组即可。
> 3. 请编程完善 `trap.c` 中的中断处理函数 `trap` ，在对时钟中断进行处理的部分填写 `trap` 函数中处理时钟中断的部分，使操作系统每遇到 100 次时钟中断后，调用 `print_ticks` 子程序，向屏幕上打印一行文字”100 ticks”。
>
> > 【注意】除了系统调用中断(T_SYSCALL)使用陷阱门描述符且权限为用户态权限以外，其它中断均使用特权级(DPL)为０的中断门描述符，权限为内核态权限；而ucore的应用程序处于特权级３，需要采用｀int  0x80`指令操作（这种方式称为软中断，软件中断，Tra中断，在lab5会碰到）来发出系统调用请求，并要能实现从特权级３到特权级０的转换，所以系统调用中断(T_SYSCALL)所对应的中断门描述符中的特权级（DPL）需要设置为３。
>
> 要求完成问题 2 和问题 3 提出的相关函数实现，提交改进后的源代码包（可以编译执行），并在实验报告中简要说明实现过程，并写出对问题1的回答。完成这问题2和3要求的部分代码后，运行整个系统，可以看到大约每1秒会输出一次”100 ticks”，而按下的键也会在屏幕上显示。
>
> 提示：可阅读小节“中断与异常”。
>

### 练习6.1

- 中断描述符表（也可简称为保护模式下的中断向量表）中一个表项占多少字节？其中哪几位代表中断处理代码的入口？

查看 `mmu.h` ，找到了中断描述符表项结构的定义：

```c
/* Gate descriptors for interrupts and traps */
struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
};
```

由此可知一个表项占 ( 16 + 16 + 5 + 4 + 3 + 1 + 2 + 1 + 16 ) / 8 = 8 字节。其中 `gd_ss` 是段选择子，用于从 GDT 获取中断处理代码对应的段地址，再加上由 `gd_off_15_0` 和 `gd_off_31_16` 构成的偏移地址就能得到中断处理代码的入口地址。

### 练习6.2

- 请编程完善 `kern/trap/trap.c` 中对中断向量表进行初始化的函数 `idt_init` 。在 `idt_init` 函数中，依次对所有中断入口进行初始化。使用 `mmu.h` 中的 `SETGATE` 宏，填充 `idt` 数组内容。每个中断的入口由 `tools/vectors.c` 生成，使用 `trap.c` 中声明的 `vectors` 数组即可。

根据 `trap.c` 中 `idt_init()` 函数的注释，完成该函数需要以下三步：

1. 定义储存每个中断入口地址的外部变量
2. 使用 `SETGATE` 宏定义填充中断描述符表 `IDT` 中的每一项
3. 使用 `lidt` 指令加载中断描述符表

```c
/* idt_init - initialize IDT to each of the entry points in kern/trap/vectors.S */
void
idt_init(void) {
     /* LAB1 YOUR CODE : STEP 2 */
     /* (1) Where are the entry addrs of each Interrupt Service Routine (ISR)?
      *     All ISR's entry addrs are stored in __vectors. where is uintptr_t __vectors[] ?
      *     __vectors[] is in kern/trap/vector.S which is produced by tools/vector.c
      *     (try "make" command in lab1, then you will find vector.S in kern/trap DIR)
      *     You can use  "extern uintptr_t __vectors[];" to define this extern variable which will be used later.
      * (2) Now you should setup the entries of ISR in Interrupt Description Table (IDT).
      *     Can you see idt[256] in this file? Yes, it's IDT! you can use SETGATE macro to setup each item of IDT
      * (3) After setup the contents of IDT, you will let CPU know where is the IDT by using 'lidt' instruction.
      *     You don't know the meaning of this instruction? just google it! and check the libs/x86.h to know more.
      *     Notice: the argument of lidt is idt_pd. try to find it!
      */
}
```

其中比较关键的是第二步中调用 `SETGATE` 宏定义的参数。

在 `mmu.h` 中找到 `SETGATE` 宏定义，发现有 `gate, istrap, sel, off, dpl` 五个参数。关于五个参数的含义及在 `idt_init()` 函数中该如何填写，如下所示：

1.  `gate` ：需要设置的 IDT 表项。在这里是 `idt[]` 数组的内容。
2.  `istrap` ：如果使用中断门描述符则该值为 0 ，如果使用陷阱门描述符则该值为 1 。由题可知，除了系统调用中断 ( `T_SYSCALL` ) 使用陷阱门描述符以外，其它中断均使用中断门描述符。
3.  `sel` ：中断处理代码的段选择子。中断处理代码在 `vectors.S` 中，由该文件的第二行 `.text` 可知这部分代码在 `.text` 段。而 `.text` 段的段选择子为 `GD_KTEXT`。
4.  `off` ：中断处理代码在代码段中的偏移量。在这里是 `__vectors[]` 数组的内容。
5.  `dpl` ：描述符特权级。由题可知，除了系统调用中断 ( `T_SYSCALL` ) 权限为用户态权限 ( `DPL_USER` ) 以外，其它中断权限均为内核态权限 ( `DPL_KERNEL` ) 。

 完成的该函数的代码如下所示：

```c
void
idt_init(void) {
    extern uintptr_t __vectors[];
    int i;
    for (i = 0; i < 256; i++) {
        SETGATE(idt[i], 0, GD_KTEXT, __vectors[i], DPL_KERNEL);
    }
    SETGATE(idt[T_SYSCALL], 1, GD_KTEXT, __vectors[T_SYSCALL], DPL_USER);
    lidt(&idt_pd);
}
```

完成实验后对比参考答案：

```c
void
idt_init(void) {
    extern uintptr_t __vectors[];
    int i;
    for (i = 0; i < sizeof(idt) / sizeof(struct gatedesc); i ++) {
        SETGATE(idt[i], 0, GD_KTEXT, __vectors[i], DPL_KERNEL);
    }
	// set for switch from user to kernel
    SETGATE(idt[T_SWITCH_TOK], 0, GD_KTEXT, __vectors[T_SWITCH_TOK], DPL_USER);
	// load the IDT
    lidt(&idt_pd);
}
```

发现主要存在以下差异：

1. 参考答案中 for 循环的判断条件使用了 `sizeof(idt) / sizeof(struct gatedesc)`，与我们的答案相比具有更好的扩展性，改变 `idt` 数组长度时不需要修改此处代码。
2. 参考答案中选择将 `T_SWITCH_TOK` 对应中断的权限设为 `DPL_USER` ， 而不是 `T_SYSCALL` ，并且参考答案使用的是中断门描述符。 `T_SWITCH_TOK` 对应扩展练习 Challenge 1 中由用户态切换到内核态的中断，权限为 `DPL_USER` 。而系统调用中断 `T_SYSCALL` 本身也是在用户态调用，并且能够从用户态切换到内核态，和 `T_SWITCH_TOK` 比较相似。具体为什么参考答案中使用的是 `T_SWITCH_TOK` 而不是 `T_SYSCALL` ，可能是因为扩展练习中的内容改变了这部分代码。

### 练习6.3

- 请编程完善 `trap.c` 中的中断处理函数 `trap` ，在对时钟中断进行处理的部分填写 `trap` 函数中处理时钟中断的部分，使操作系统每遇到 100 次时钟中断后，调用 `print_ticks` 子程序，向屏幕上打印一行文字 ”100 ticks” 。

根据注释可知，每次遇到时钟中断时让 `ticks` 变量自增，每 `TICK_NUM` 次时钟中断调用一次 `print_ticks` 子程序即可。代码如下：

```c
case IRQ_OFFSET + IRQ_TIMER:
        /* LAB1 YOUR CODE : STEP 3 */
        /* handle the timer interrupt */
        /* (1) After a timer interrupt, you should record this event using a global variable (increase it), such as ticks in kern/driver/clock.c
         * (2) Every TICK_NUM cycle, you can print some info using a funciton, such as print_ticks().
         * (3) Too Simple? Yes, I think so!
         */
        ticks++;
    	if (ticks % TICK_NUM == 0) {
    		print_ticks();
    	}
        break;
```

完成实验后对比参考答案，发现这确实很简单，我们的答案与参考答案基本一致。

## 列出本实验中的知识点与其在OS原理中对应的知识点：

1. 操作系统镜像文件是如何通过 make 指令生成的 ---- 操作系统文件在磁盘中的储存形式

2. 计算机启动后最早运行了那些程序 ( BIOS、bootloader以及操作系统内核 )  ---- 计算机启动过程

3. bootloader 做了什么 ( 寄存器初始化、进入保护模式、初始化全局描述符表、初始化堆栈、加载内核) ---- 全局描述符表：基于分段机制的内存管理

4. 保护模式是什么，如何进入保护模式 ( 使能 A20 ) ---- CPU的寻址方式

5. ELF 格式是什么，加载 ELF 格式的过程 ( 内核加载过程 ) ---- 可执行文件的加载过程

6. 函数调用的具体过程 ( 参数压栈、返回地址压栈、ebp 压栈以及切换栈帧、地址跳转等 ) ---- 可执行文件的加载过程

7. 中断是什么，CPU 是如何处理中断的 ---- CPU的中断机制、中断描述符表的结构