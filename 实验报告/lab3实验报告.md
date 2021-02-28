# lab3实验报告

## 小组成员

- 姓名：张智强 学号：1611357 分工占比：50%
- 姓名：孟昊纯 学号：1611311 分工占比：50%

## 练习0：填写已有实验

> 本实验依赖实验1/2。请把你做的实验1/2的代码填入本实验中代码中有“LAB1”,“LAB2”的注释相应部分。

涉及到的文件为 `kern/debug/kdebug.c` 、 `kern/trap/trap.c`、 `kern/mm/pmm.c`、 `kern/mm/default_pmm.c`，复制 lab1 / lab2 中的代码即可。

## 练习1：给未被映射的地址映射上物理页（需要编程）

> 完成do_pgfault（mm/vmm.c）函数，给未被映射的地址映射上物理页。设置访问权限 的时候需要参考页面所在 VMA 的权限，同时需要注意映射物理页时需要操作内存控制 结构所指定的页表，而不是内核的页表。注意：在LAB3 EXERCISE 1处填写代码。执行
>
> ```
> make　qemu
> ```
>
> 后，如果通过check_pgfault函数的测试后，会有“check_pgfault() succeeded!”的输出，表示练习1基本正确。
>
> 请在实验报告中简要说明你的设计实现过程。请回答如下问题：
>
> - 请描述页目录项（Page Directory Entry）和页表项（Page Table Entry）中组成部分对ucore实现页替换算法的潜在用处。
> - 如果ucore的缺页服务例程在执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

- 请描述页目录项（Page Directory Entry）和页表项（Page Table Entry）中组成部分对 ucore 实现页替换算法的潜在用处。

根据 lab2，页表项/页目录项中 `PTE_AVAIL` 对应的几位没有被硬件使用，作为保留位给操作系统使用。ucore 可以利用这些位来完成一些页替换算法中对数据结构的要求，它们可以在物理页中储存用于实现页替换算法的标识。比如 LRU 算法的近似实现 Not Recently Used Algorithm，需要为每个映射的物理页引入一个“使用”标识 (use bit) ，这个标识就可以储存在 `PTE_AVAIL` 中。

- 如果 ucore 的缺页服务例程在执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

ucore 通过建立 `mm_struct` 和 `vma_struct` 数据结构，描述了 ucore 模拟应用程序运行所需的合法内存空间。当访问内存产生 `page faul`  异常时，可获得访问的内存的方式（读或写）以及具体的虚拟内存地址，这样 ucore 就可以查询此地址，看是否属于 `vma_struct` 数据结构中描述的合法地址范围中，如果在，则可根据具体情况进行请求调页/页换入换出处理（练习2涉及的部分）；如果不在，则报错。

`mm_struct` 和 `vma_struct` 数据结构用来描述不在物理内存中但又被应用程序所使用的“合法”虚拟页。当 ucore 访问这些“合法”虚拟页时，会由于没有虚实地址映射而产生页访问异常。这时，如果我们正确实现了练习1，则 `do_pgfault` 函数会申请一个空闲物理页，并建立好虚实映射关系，从而使得这样的“合法”虚拟页有实际的物理页帧对应。

查阅实验指导书及 ucore 源码中的注释，以及 lab1 中的中断处理过程和 lab2 中的相似问题，我们可以知道， ucore 在出现页访问异常以后，硬件需要完成的事如下：

> 1. 将发生错误的线性地址保存在 `cr2` 寄存器中
> 2. 向栈中依次压入 `EFLAGS` ， `CS` ， `EIP` ，以及页访问异常码 `error code`，如果 `page fault` 发生在用户态，则还需要先压入 `ss` 和 `esp` ，并且切换到内核栈
> 3. 根据中断描述符表查询到对应 `page fault` 的 `ISR`，跳转到对应的 `ISR` 处执行，接下来将由软件进行 `page fault` 处理

在硬件完成了这些事以后，中断服务例程会调用页访问异常处理函数 `do_pgfault` 进行具体处理。根据 `vmm.c` 中的注释，我们了解到 `do_pgfault` 函数的参数分别为：

```c
// @mm         : the control struct for a set of vma using the same PDT
// @error_code : the error code recorded in trapframe->tf_err which is setted by x86 hardware
// @addr       : the addr which causes a memory access exception, (the contents of the CR2 register)
```

其中 `mm` 是用来控制 `vma` 的数据结构 `mm_struct` ，`error_code` 是由硬件得到的页访问异常码， `addr` 是发生错误的线性地址。

接着查看 `vmm.c` 中的注释，发现 `do_pgfault` 函数会执行以下几件事：

1. 从 `mm` 中找到包含地址 `addr` 的 `vma` ，如果找不到，退出。
2. 检查页访问异常码 `error_code` 的 flag
3. 尝试找到地址 `addr` 对应的 `pte`，如果该 `pte` 对应的页表不存在，为该页表分配页
4. 如果物理地址 `addr` 不存在，则为其分配页，并将新分配页的物理地址与虚拟地址映射
5. 如果物理地址 `addr` 已存在，则我们需要将该地址对应页的数据从磁盘换入内存，并将物理地址映射到虚拟地址，触发 swap manager 来记录这个页的访问情况

第 3、4 步是练习 1  的内容，第 5 步则是练习 2 的前一部分内容。

根据注释完成练习 1 的代码，如下所示：

```c
    /*LAB3 EXERCISE 1: YOUR CODE*/
    //(1) try to find a pte, if pte's PT(Page Table) isn't existed, then create a PT.
    ptep = get_pte(mm->pgdir, addr, 1);
    if (ptep == NULL) {
        cprintf("get_pte() failed\n");
        goto failed;
    }              
    if (*ptep == 0) {
    //(2) if the phy addr isn't exist, then alloc a page & map the phy addr with logical addr
        if (pgdir_alloc_page(mm->pgdir, addr, perm) == NULL){
            cprintf("pgdir_alloc_page() failed\n");
            goto failed;
        }
    }
    else {
```

### 练习2：补充完成基于FIFO的页面替换算法（需要编程）

> 完成vmm.c中的do_pgfault函数，并且在实现FIFO算法的swap_fifo.c中完成map_swappable和swap_out_victim函数。通过对swap的测试。注意：在LAB3 EXERCISE 2处填写代码。执行
>
> ```
> make　qemu
> ```
>
> 后，如果通过check_swap函数的测试后，会有“check_swap() succeeded!”的输出，表示练习2基本正确。
>
> 请在实验报告中简要说明你的设计实现过程。
>
> 请在实验报告中回答如下问题：
>
> - 如果要在ucore上实现"extended clock页替换算法"请给你的设计方案，现有的swap_manager框架是否足以支持在ucore中实现此算法？如果是，请给你的设计方案。如果不是，请给出你的新的扩展和基此扩展的设计方案。并需要回答如下问题
>   - 需要被换出的页的特征是什么？
>   - 在ucore中如何判断具有这样特征的页？
>   - 何时进行换入和换出操作？

根据 `vmm.c` 中的注释完成 `do_pgfault` 函数，即练习 1 中所述的第 5 步内容，将页面换入，代码如下：

```c
    else {
        if (swap_init_ok) {
            struct Page *page = NULL;
            if (swap_in(mm, addr, &page) != 0){			//将物理页从磁盘加载到内存中
                cprintf("swap_in() failed\n");
                goto failed;
            }
            page_insert(mm->pgdir, page, addr, perm);	//将物理页与虚拟页建立映射
            swap_map_swappable(mm, addr, page, 1);		//将该物理页设为可交换的
            page->pra_vaddr = addr;					   //建立虚实地址映射，维护page中的虚拟地址属性
        }
        else {
            cprintf("no swap_init_ok but ptep is %x, failed\n",*ptep);
            goto failed;
        }
   }
   ret = 0;
```

随后在实现 FIFO 算法的 `swap_fifo.c` 中完成 `map_swappable` 和 `swap_out_victim`函数。

在完成这两个函数前，我们先简单地了解了一下FIFO 算法。 FIFO 算法是根据页面换入顺序，优先换出最早换入的页面的算法，即“先进先出”。而在本次实验中，通过维护一个先进先出的链表 `mm` 使得在换出页面时可以换出最早换入的页面。

`kern/mm/swap.h` 中定义了 ucore 中通用的页面替换算法的函数列表，用结构  `swap_manager` 表示，其定义如下：

```c
struct swap_manager
{
     const char *name;
     /* Global initialization for the swap manager */
     int (*init)            (void);
     /* Initialize the priv data inside mm_struct */
     int (*init_mm)         (struct mm_struct *mm);
     /* Called when tick interrupt occured */
     int (*tick_event)      (struct mm_struct *mm);
     /* Called when map a swappable page into the mm_struct */
     int (*map_swappable)   (struct mm_struct *mm, uintptr_t addr, struct Page *page, int swap_in);
     /* When a page is marked as shared, this routine is called to
      * delete the addr entry from the swap manager */
     int (*set_unswappable) (struct mm_struct *mm, uintptr_t addr);
     /* Try to swap out a page, return then victim */
     int (*swap_out_victim) (struct mm_struct *mm, struct Page **ptr_page, int in_tick);
     /* check the page relpacement algorithm */
     int (*check_swap)(void);     
};
```

实现 FIFO 页面替换算法只需要在 `mm/swap_fifo.c` 中实现 `swap_manager` 结构里对应的几个函数，FIFO 算法很简单，只用到了 7 个函数中的 4 个，其他 3 个函数都是 `{return 0;}` 。

在 4 个需要实现的函数中，对应 `check_swap` 的 `_fifo_check_swap` 函数是检查该页面替换算法是否正确实现的函数，对应 `init_mm` 的 `_fifo_init_mm` 是初始化 FIFO 算法中 `mm_struct` 数据结构的函数，这两个函数已经在 `mm/swap_fifo.c` 中实现。

需要我们完成的是 `_fifo_map_swappable` 和 `_fifo_swap_out_victim` 函数。

 `_fifo_map_swappable`  函数用于将指定的物理页设为可被换出。将指定的物理页插入到 FIFO 算法中维护的用于储存可被交换物理页的链表 `mm` 的尾部。这样就能保证 FIFO 算法知道哪个页是最早换入的。根据注释，我们可以完成该函数，代码如下所示：

```c
static int
_fifo_map_swappable(struct mm_struct *mm, uintptr_t addr, struct Page *page, int swap_in)
{
    list_entry_t *head=(list_entry_t*) mm->sm_priv;
    list_entry_t *entry=&(page->pra_page_link);
    assert(entry != NULL && head != NULL);
    //record the page access situlation
    /*LAB3 EXERCISE 2: YOUR CODE*/ 
    //(1)link the most recent arrival page at the back of the pra_list_head qeueue.
    list_add(head, entry);
    return 0;
}
```

 `_fifo_swap_out_victim` 函数用于选择即将被换出的物理页，该函数在将页换出内存的函数 `swap_out` 中被调用，决定了换出哪一个页。而根据 FIFO 算法的定义，我们只需要取出 `mm` 中链表头的物理页即可。因为该页是最早被插入到链表中的，所以应该最先被换出。根据注释，我们可以完成该函数，代码如下：

```c
static int
_fifo_swap_out_victim(struct mm_struct *mm, struct Page ** ptr_page, int in_tick)
{
     list_entry_t *head=(list_entry_t*) mm->sm_priv;
     assert(head != NULL);
     assert(in_tick == 0);
     /* Select the victim */
     //(1)  unlink the  earliest arrival page in front of pra_list_head qeueue
     list_entry_t *le = head->prev;
     assert(le != head);//判断链表非空
     struct Page* page = le2page(le, pra_page_link);
     list_del(le);
     //(2)  set the addr of addr of this page to ptr_page
     *ptr_page = page;
     return 0;
}
```

然而在完成了练习 1 与练习 2 的代码以后，执行 `make qemu` ，仍然没有 `“check_pgfault() succeeded!”`的输出。

根据实验指导书中编译执行后得到的正确输出，在 `“check_pgfault() succeeded!”` 前应有一句为 `“page fault at 0x00000100: K/W [no page found].”`的输出，表示出现页访问异常。但在测试时并没有出现这句输出，这表明没有出现页访问异常。但我们只对练习中涉及到的部分做了修改，应该不会出现这种情况。为了确认是代码问题还是设备问题，我们尝试对参考答案中的 lab3_result 执行 `make qemu` ，结果仍然没有 `“check_pgfault() succeeded!”`，并且输出与之前的无异。然后我们又在网上找了一份已完成的代码，发现结果一样。这时我们基本排除了代码上存在的问题，推测可能是虚拟机或电脑上存在某些问题。因此我们换了一台电脑，并重新配置了虚拟机环境，这次成功地得到了`“check_pgfault() succeeded!”` 的输出，并在一些简单的 debug 后得到了 `“check_swap() succeeded!”` 的输出。

- 如果要在ucore上实现"extended clock页替换算法"请给你的设计方案，现有的swap_manager框架是否足以支持在ucore中实现此算法？如果是，请给你的设计方案。如果不是，请给出你的新的扩展和基此扩展的设计方案。并需要回答如下问题
  - 需要被换出的页的特征是什么？
  - 在ucore中如何判断具有这样特征的页？
  - 何时进行换入和换出操作？

extended clock 页替换算法是 clock 算法的改进型，它对每个页面设置了两个标志位，分别是访问位 A 和修改位 M ，该算法优先换出最近既未被访问也未被修改的页。由两个标志位可以将页面分为以下4种类型：

1. (A=0，M=0)：该页最近未被访问，也未被修改，因为不需要写磁盘，换出代价小，优先被换出。
2. (A=0，M=1)：该页最近未被访问，但被修改过，换出代价比上面的大，其次被换出。
3. (A=1，M=0)：该页最近已被访问，但未被修改，虽然该页有可能再次被访问，但是如果在换出页面时没有找到 A=0 的页面，该算法会遍历所有的页，将 A 置 0，此时这一类页会被优先换出。
4. (A=1，M=1)：该页最近已被访问，且已被修改。

由此可知，需要被换出的页的特征是访问位 A=0 ，且该算法优先换出修改位M=0 的页。

在页被访问时，将A置1，在页被修改时，将M置1，这样就能在 ucore 中判断具有这样特征的页。

当保存在磁盘中的页需要被访问时，进行换入操作。当储存在物理内存中的页被页面替换算法选择时，进行换出操作。

## 参考答案分析

### 练习1

练习1的思路比较简单，所以我们的答案与参考答案基本一致。差别在于没有将 `ptep` 的赋值运算放在判断条件中，以及在输出上也有差别。这里的错误输出主要是方便调试，而我们答案中的输出相比参考答案省略了一些信息。如果是一个较多人开发的项目，使用更详细的错误输出能提高调试的效率。

### 练习2

 `vmm.c` 中 `do_pgfault` 函数的后一部分里，我们的答案与参考答案相似，但我们没有对返回值 `ret` 作出修改。该函数返回值的用途可能类似错误码，我们的答案相比参考答案可能会导致以后的调试比较困难。

`swap_fifo.c` 中的 `_fifo_map_swappable` 非常简单，只有一行，所以我们的答案与参考答案完全一致。而 `fifo_swap_out_victim` 函数中我们的实现相比参考答案缺少了一句判断语句，没有判断由通用链表节点转换为的页是否非空，这可能导致我们完成的函数在运行过程中出现奇怪的 bug 。

## 实验中涉及的知识点列举

本实验中涉及到的知识点如下：

- 虚拟内存管理的设计与实现
- Page Fault异常处理过程
- 页面置换机制的实现

对应的OS原理中的知识点如下：

- 虚拟内存的概念
- 缺页操作的发生和处理
- FIFO 等页面替换算法

本实验中涉及到的知识点是对应的OS原理中的知识点的具体实现和详细说明。



