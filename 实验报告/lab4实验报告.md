# lab4实验报告

## 小组成员

- 姓名：张智强 学号：1611357 分工占比：50%
- 姓名：孟昊纯 学号：1611311 分工占比：50%

## 练习0：填写已有实验

> 本实验依赖实验1/2/3。请把你做的实验1/2/3的代码填入本实验中代码中有“LAB1”,“LAB2”,“LAB3”的注释相应部分。

涉及到的文件为 `kern/debug/kdebug.c` 、 `kern/trap/trap.c`、 `kern/mm/pmm.c`、 `kern/mm/default_pmm.c`、`kern/mm/swap_fifo.c`、`kern/mm/vmm.c`，复制 lab1 / lab2 / lab3 中的代码即可。

## 练习1：分配并初始化一个进程控制块（需要编码）

> alloc_proc函数（位于kern/process/proc.c中）负责分配并返回一个新的struct proc_struct结构，用于存储新建立的内核线程的管理信息。ucore需要对这个结构进行最基本的初始化，你需要完成这个初始化过程。
>
> > 【提示】在alloc_proc函数的实现中，需要初始化的proc_struct结构中的成员变量至少包括：state/pid/runs/kstack/need_resched/parent/mm/context/tf/cr3/flags/name。
>
> 请在实验报告中简要说明你的设计实现过程。请回答如下问题：
>
> - 请说明proc_struct中`struct context context`和`struct trapframe *tf`成员变量含义和在本实验中的作用是啥？（提示通过看代码和编程调试可以判断出来）

实现 `alloc_proc` 函数需要完成对一个新的 `proc_struct` 结构的初步初始化，该数据结构是 ucore 中的进程控制块 PCB，其定义如下：

```c
struct proc_struct {
    enum proc_state state;                      // Process state
    int pid;                                    // Process ID
    int runs;                                   // the running times of Proces
    uintptr_t kstack;                           // Process kernel stack
    volatile bool need_resched;                 // bool value: need to be rescheduled to release CPU?
    struct proc_struct *parent;                 // the parent process
    struct mm_struct *mm;                       // Process's memory management field
    struct context context;                     // Switch here to run process
    struct trapframe *tf;                       // Trap frame for current interrupt
    uintptr_t cr3;                              // CR3 register: the base addr of Page Directroy Table(PDT)
    uint32_t flags;                             // Process flag
    char name[PROC_NAME_LEN + 1];               // Process name
    list_entry_t list_link;                     // Process link list 
    list_entry_t hash_link;                     // Process hash list
};
```

根据实验指导书中的提示以及中的注释，`alloc_proc` 函数中需要初始化的 `proc_struct` 结构的成员变量至少包括：`state` / `pid` / `runs` / `kstack` / `need_resched` / `parent` / `mm` / `context` / `tf` / `cr3` / `flags` / `name`。其中大多数变量只需要清零即可，但有些成员变量需要设置特殊的值。

`state` 用于表示程序运行的状态。由 `kern/process/proc.c` 中开头部分的注释以及 `proc.h` 中枚举类型 `proc_state` 的定义，我们可以知道这里应该将`state` 赋值为 `PROC_UNINIT`。

`pid` 是进程的 id，由 `kern/process/proc.c` 中为程序分配 `pid` 的函数 `get_pid` 可知，分配后的 `pid` 都是正数。况且根据操作系统知识我们知道有 0 号进程和 1 号进程的存在，它们的 `pid` 分别为 0 和 1。所以这里可以将 `pid` 赋值为 -1，以作区分。

 `cr3` 保存页表的物理地址，当一个线程是内核线程时，`cr3` 等于 `boot_cr3` 。而 `boot_cr3` 指向 ucore 启动时建立好的内核虚拟空间的页目录表首地址。因为该函数初始化的都是内核线程，所以这里可以将`cr3` 赋值为 `boot_cr3` 。

以上三个变量的赋值在实验指导书中 "创建第 0 个内核线程 idleproc" 一节内有代码描述。

其他的成员变量都只需要赋值为 0，就可以完成初步的初始化。其中指针需要赋值为 `NULL` ，数组则使用 `memset` 函数将每一项赋值为 0，`context` 是由 8 个 `uint32_t` 变量组成的结构，因此也可以当做数组来处理。

因此，完成后的代码如下：

```c
// alloc_proc - alloc a proc_struct and init all fields of proc_struct
static struct proc_struct *
alloc_proc(void) {
    struct proc_struct *proc = kmalloc(sizeof(struct proc_struct));
    if (proc != NULL) {
        proc->state = PROC_UNINIT;
        proc->pid = -1;
        proc->runs = 0;
        proc->kstack = 0;
        proc->need_resched = 0;
        proc->parent = NULL;
        proc->mm = NULL;
        memset(&(proc->context), 0, sizeof(struct context));
        proc->tf = NULL;
        proc->cr3 = boot_cr3;
        proc->flags = 0;
        memset(&(proc->name), 0, PROC_NAME_LEN);
    }
    return proc;
}
```

- 请说明proc_struct中`struct context context`和`struct trapframe *tf`成员变量含义和在本实验中的作用是啥？（提示通过看代码和编程调试可以判断出来）

`context`：进程的上下文，用于进程切换。在 ucore 中，所有的进程在内核中也是相对独立的（例如独立的内核堆栈以及上下文等）。使用 `context` 保存寄存器的目的就在于在内核态中能够进行上下文之间的切换，为进程调度做准备。实际利用 `context` 进行上下文切换的函数是在 `kern/process/switch.S` 中定义的 `switch_to`。定义 `context` 结构的代码如下：

```c
// Saved registers for kernel context switches.
// Don't need to save all the %fs etc. segment registers,
// because they are constant across kernel contexts.
// Save all the regular registers so we don't need to care
// which are caller save, but not the return register %eax.
// (Not saving %eax just simplifies the switching code.)
// The layout of context must match code in switch.S.
struct context {
    uint32_t eip;
    uint32_t esp;
    uint32_t ebx;
    uint32_t ecx;
    uint32_t edx;
    uint32_t esi;
    uint32_t edi;
    uint32_t ebp;
};
```

根据注释，它用于在内核上下文切换时保存各寄存器的值。

`tf`：中断帧的指针，总是指向内核栈的某个位置。当进程从用户空间跳到内核空间时，中断帧记录了进程在被中断前的状态。当内核需要跳回用户空间时，需要调整中断帧以恢复让进程继续执行的各寄存器值。除此之外，ucore 内核允许嵌套中断。因此为了保证嵌套中断发生时 `tf` 总是能够指向当前的 trapframe，ucore 在内核栈上维护了 `tf` 的链。定义 `trapframe` 结构的代码如下：

```c
/* registers as pushed by pushal */
struct pushregs {
    uint32_t reg_edi;
    uint32_t reg_esi;
    uint32_t reg_ebp;
    uint32_t reg_oesp;          /* Useless */
    uint32_t reg_ebx;
    uint32_t reg_edx;
    uint32_t reg_ecx;
    uint32_t reg_eax;
};

struct trapframe {
    struct pushregs tf_regs;
    uint16_t tf_gs;
    uint16_t tf_padding0;
    uint16_t tf_fs;
    uint16_t tf_padding1;
    uint16_t tf_es;
    uint16_t tf_padding2;
    uint16_t tf_ds;
    uint16_t tf_padding3;
    uint32_t tf_trapno;
    /* below here defined by x86 hardware */
    uint32_t tf_err;
    uintptr_t tf_eip;
    uint16_t tf_cs;
    uint16_t tf_padding4;
    uint32_t tf_eflags;
    /* below here only when crossing rings, such as from user to kernel */
    uintptr_t tf_esp;
    uint16_t tf_ss;
    uint16_t tf_padding5;
} __attribute__((packed));
```

从 `trapframe` 的成员变量可以看出，`tf` 保存了各寄存器的值以及与中断帧有关的一些变量。

这两个变量用于保存和恢复程序运行的上下文环境，`context`用于内核进程切换，`tf`用于中断。

## 练习2：为新创建的内核线程分配资源（需要编码）

> 创建一个内核线程需要分配和设置好很多资源。kernel_thread函数通过调用**do_fork**函数完成具体内核线程的创建工作。do_kernel函数会调用alloc_proc函数来分配并初始化一个进程控制块，但alloc_proc只是找到了一小块内存用以记录进程的必要信息，并没有实际分配这些资源。ucore一般通过do_fork实际创建新的内核线程。do_fork的作用是，创建当前内核线程的一个副本，它们的执行上下文、代码、数据都一样，但是存储位置不同。在这个过程中，需要给新内核线程分配资源，并且复制原进程的状态。你需要完成在kern/process/proc.c中的do_fork函数中的处理过程。它的大致执行步骤包括：
>
> - 调用alloc_proc，首先获得一块用户信息块。
> - 为进程分配一个内核栈。
> - 复制原进程的内存管理信息到新进程（但内核线程不必做此事）
> - 复制原进程上下文到新进程
> - 将新进程添加到进程列表
> - 唤醒新进程
> - 返回新进程号
>
> 请在实验报告中简要说明你的设计实现过程。请回答如下问题：
>
> - 请说明ucore是否做到给每个新fork的线程一个唯一的id？请说明你的分析和理由。

`alloc_proc` 只完成了对进程控制块的初始化，并没有对进程分配资源。ucore 一般通过 `do_fork` 实际创建新的内核线程。而 `do_fork` 的功能是创建当前内核线程的一个副本，它们的执行上下文、代码、数据都一样，但是存储位置不同。在这个过程中，需要给新内核线程分配资源，并且复制原进程的状态。

根据注释，完成的代码如下：

```c
    proc = alloc_proc();//分配并初始化进程控制块
    if (proc == NULL) {
        goto fork_out;
    }
    proc->parent = current;//设置父进程为当前进程
    if (setup_kstack(proc) != 0) {//分配并初始化内核栈
        goto bad_fork_cleanup_proc;
    }
    if (copy_mm(clone_flags, proc) != 0) {//根据clone_flag标志复制或共享进程内存管理结构
        goto bad_fork_cleanup_kstack;
    }
    copy_thread(proc, stack, tf);//设置进程在内核正常运行和调度所需的中断帧和执行上下文
    bool intr_flag;
    local_intr_save(intr_flag);//屏蔽中断
    {
        proc->pid = get_pid();//为进程分配pid
        hash_proc(proc);//把设置好的进程控制块放入hash_list
        nr_process++;
        list_add(&proc_list, &(proc->list_link));//把设置好的进程控制块放入proc_list
    }
    local_intr_restore(intr_flag);//恢复中断
    wakeup_proc(proc);//把进程状态设置为“就绪”态；
    ret = proc->pid;//设置返回码为子进程的id号
fork_out:
    return ret;
bad_fork_cleanup_kstack:
    put_kstack(proc);
bad_fork_cleanup_proc:
    kfree(proc);
    goto fork_out;
```

这里需要注意的是，如果 `alloc_proc()` 、`setup_kstack()` 、`copy_mm()` 执行没有成功，则需要做对应的出错处理，把相关已经占有的内存释放掉。

- 请说明ucore是否做到给每个新fork的线程一个唯一的id？请说明你的分析和理由。

```c
// get_pid - alloc a unique pid for process
static int
get_pid(void) {
    static_assert(MAX_PID > MAX_PROCESS);
    struct proc_struct *proc;
    list_entry_t *list = &proc_list, *le;
    static int next_safe = MAX_PID, last_pid = MAX_PID;
    if (++ last_pid >= MAX_PID) {
        last_pid = 1;
        goto inside;
    }
    if (last_pid >= next_safe) {
    inside:
        next_safe = MAX_PID;
    repeat:
        le = list;
        while ((le = list_next(le)) != list) {
            proc = le2proc(le, list_link);
            if (proc->pid == last_pid) {
                if (++ last_pid >= next_safe) {
                    if (last_pid >= MAX_PID) {
                        last_pid = 1;
                    }
                    next_safe = MAX_PID;
                    goto repeat;
                }
            }
            else if (proc->pid > last_pid && next_safe > proc->pid) {
                next_safe = proc->pid;
            }
        }
    }
    return last_pid;
}
```

由 `get_pid` 函数可知，使用 fork 或 clone 系统调用产生的进程均会由内核分配一个新的唯一的 `pid`，避免与现有的进程 `pid` 重复。而且分配时设置了锁，暂时屏蔽了中断，使分配 `pid` 时不会发生其他的系统调用，这样就做到了给每个新 fork 的线程一个唯一的 id。 

## 练习3：阅读代码，理解 proc_run 函数和它调用的函数如何完成进程切换的。（无编码工作）

> 请在实验报告中简要说明你对proc_run函数的分析。并回答如下问题：
>
> - 在本实验的执行过程中，创建且运行了几个内核线程？
> - 语句`local_intr_save(intr_flag);....local_intr_restore(intr_flag);`在这里有何作用?请说明理由

`proc_run` 函数如下所示：

```c
// proc_run - make process "proc" running on cpu
// NOTE: before call switch_to, should load  base addr of "proc"'s new PDT
void
proc_run(struct proc_struct *proc) {
    if (proc != current) {
        bool intr_flag;
        struct proc_struct *prev = current, *next = proc;
        local_intr_save(intr_flag);//屏蔽中断
        {
            current = proc;//将当前进程设为目标进程
            load_esp0(next->kstack + KSTACKSIZE);//修改esp0
            lcr3(next->cr3);//修改页表项
            switch_to(&(prev->context), &(next->context));//上下文切换
        }
        local_intr_restore(intr_flag);//恢复中断
    }
}
```

该函数的功能是让 `proc` 进程运行在 CPU 上。

首先我们查阅资料得知 `local_intr_save(intr_flag)` 和 `local_intr_restore(intr_flag)` 分别是屏蔽中断和恢复中断，这里屏蔽中断时为了避免在进程切换过程中发生中断。

在屏蔽中断以后执行了 4 条语句，其中`load_esp0` 函数的定义如下，它修改了 `esp0` ，用于调整栈帧。根据实验指导书，该函数在 `proc_run` 中的作用是设置任务状态段 ts 中特权态 0 下的栈顶指针 `esp0` 为下一个内核线程的内核栈的栈顶。

```c
/* *
 * load_esp0 - change the ESP0 in default task state segment,
 * so that we can use different kernel stack when we trap frame
 * user to kernel.
 * */
void
load_esp0(uintptr_t esp0) {
    ts.ts_esp0 = esp0;
}
```

`lcr3` 函数的定义如下，它调用了汇编语句来修改 `cr3` 寄存器的值，用于修改当前的页表项，完成页表切换。

```c
static inline void
lcr3(uintptr_t cr3) {
    asm volatile ("mov %0, %%cr3" :: "r" (cr3) : "memory");
}
```

`switch_to` 函数的定义如下，可以看出该函数保存了当前进程各寄存器的值，并将下一个进程储存的各寄存器的值写入寄存器，即切换进程的上下文环境。

```assembly
.text
.globl switch_to
switch_to:                      # switch_to(from, to)

    # save from's registers
    movl 4(%esp), %eax          # eax points to from
    popl 0(%eax)                # save eip !popl
    movl %esp, 4(%eax)
    movl %ebx, 8(%eax)
    movl %ecx, 12(%eax)
    movl %edx, 16(%eax)
    movl %esi, 20(%eax)
    movl %edi, 24(%eax)
    movl %ebp, 28(%eax)

    # restore to's registers
    movl 4(%esp), %eax          # not 8(%esp): popped return address already
                                # eax now points to to
    movl 28(%eax), %ebp
    movl 24(%eax), %edi
    movl 20(%eax), %esi
    movl 16(%eax), %edx
    movl 12(%eax), %ecx
    movl 8(%eax), %ebx
    movl 4(%eax), %esp

    pushl 0(%eax)               # push eip

    ret
```

- 在本实验的执行过程中，创建且运行了几个内核线程？

由实验指导书可知创建且运行了第 0 个内核线程 `idleproc` 和第 1 个内核线程 `initproc`。

- 语句local_intr_save(intr_flag);....local_intr_restore(intr_flag);在这里有何作用?请说明理由

 `local_intr_save(intr_flag)` 和 `local_intr_restore(intr_flag)` 分别是屏蔽中断和恢复中断，这里屏蔽中断是为了避免在进程切换过程中发生中断。