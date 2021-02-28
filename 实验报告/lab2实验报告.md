# lab2实验报告

## 小组成员

- 姓名：张智强 学号：1611357 分工占比：50%
- 姓名：孟昊纯 学号：1611311 分工占比：50%

## 练习0：填写已有实验

> 本实验依赖实验1。请把你做的实验1的代码填入本实验中代码中有“LAB1”的注释相应部分。

涉及到的文件为 `kern/debug/kdebug.c` 和 `kern/trap/trap.c`，复制 lab1 中的代码即可。

## 练习1：实现 first-fit 连续物理内存分配算法（需要编程）

> 在实现 first fit 内存分配算法的回收函数时，要考虑地址连续的空闲块之间的合并操作。提示:在建立空闲页块链表时，需要按照空闲页块起始地址来排序，形成一个有序的链表。可能会修改 default_pmm.c 中的 default_init， default_init_memmap， default_alloc_pages， default_free_pages 等相关函数。请仔细查看和理解default_pmm.c中的注释。
>
> 请在实验报告中简要说明你的设计实现过程。请回答如下问题：
>
> - 你的 first fit 算法是否有进一步的改进空间

在 first-fit 连续物理内存分配算法中，分配器储存一个表示所有空闲内存块 ( 以页为最小单位的连续地址空间 ) 的链表，并将这些内存块按地址从低到高排列。收到内存请求时，沿链表查找第一个足以满足请求的块。如果找到的块比需要的大，则将其拆分，剩余的部分作为一个新的空闲内存块加入到空闲内存块链表中。

该算法的执行可以分为三个部分：初始化、分配内存、释放内存。而它的具体实现可以分为数据结构和函数两部分。

数据结构方面，可以使用 `libs/list.h` 中定义的可挂接任意元素的通用双向链表结构 `list_entry`作为储存查找有序空闲块的数据结构。`default_pmm.c` 中定义的 `free_area` 变量用于完成对空闲块的管理，而该变量的结构在 `mm/memlayout.h` 中定义，如下：

```c
/* free_area_t - maintains a doubly linked list to record free (unused) pages */
typedef struct {
    list_entry_t free_list;         // the list header
    unsigned int nr_free;           // # of free pages in this free list
} free_area_t;
```

其中 `free_list` 是空闲块双向链表的头， `nr_free` 以页为单位表示空闲块的总数。

函数方面，`kern/mm/pmm.h` 中定义了一个通用的分配算法的函数列表，用结构 `pmm_manager` 来表示，定义如下：

```c
// pmm_manager is a physical memory management class. A special pmm manager - XXX_pmm_manager
// only needs to implement the methods in pmm_manager class, then XXX_pmm_manager can be used
// by ucore to manage the total physical memory space.
struct pmm_manager {
    const char *name;                                 // XXX_pmm_manager的名字
    void (*init)(void);                               // 初始化内部描述、管理XXX_pmm_manager的数据结构
                                                      // (空闲块列表、空闲块总数等)
    void (*init_memmap)(struct Page *base, size_t n); // 根据初始空闲物理内存空间设置描述、管理数据结构
    struct Page *(*alloc_pages)(size_t n);            // 分配 >=n 页，取决于分配算法
    void (*free_pages)(struct Page *base, size_t n);  // 释放 >=n 页，释放指定的页
    size_t (*nr_free_pages)(void);                    // 返回空闲页数
    void (*check)(void);                              // 检查XXX_pmm_manager的正确性 
};
```

通过其中的注释可以了解到，`pmm_manager` 描述了一个物理内存管理器，每种具体的物理内存页分配算法只需实现该类中的函数即可。

查看 `default_pmm.c` ，可以发现 ucore 中使用的默认物理内存管理器的各个函数指针指向了该文件中的各个函数。根据实验指导书，在本实验中，我们实现 first-fit 连续物理内存分配算法只需要修改这些函数。

```c
const struct pmm_manager default_pmm_manager = {
    .name = "default_pmm_manager",
    .init = default_init,
    .init_memmap = default_init_memmap,
    .alloc_pages = default_alloc_pages,
    .free_pages = default_free_pages,
    .nr_free_pages = default_nr_free_pages,
    .check = default_check,
};
```

下面对每个相关函数作出说明：

- `default_init`

初始化空闲内存块链表 `free_list` ，将空闲页数 `nr_free` 变量设为 0 。

```c
static void
default_init(void) {
    list_init(&free_list);
    nr_free = 0;
}
```

根据 `default_pmm.c` 中的注释，发现可以直接使用默认的函数实现，无需改动。

- `default_init_memmap`

初始化最初内存块中每一页对应的 `Page` 结构，将 `nr_free` 设为当前空闲内存块的总数。

```c
static void
default_init_memmap(struct Page *base, size_t n) {
    assert(n > 0);
    struct Page *p = base;
    for (; p != base + n; p ++) {
        assert(PageReserved(p));
        p->flags = p->property = 0;
        SetPageProperty(p);
        set_page_ref(p, 0);
        list_add_before(&free_list, &(p->page_link));
    }
    base->property = n;
    nr_free += n;
}
```

根据注释，对每个 Page 需要进行如下的初始化操作：

1. `p->flags` 的 `PG_property` 位需要置 1
2. 如果该页是空闲的而且不是空闲块中的第一页，则 `p->property` 需要被设为 0
3. 如果该页是空闲的并且是空闲块中的第一页，则将 `p->property` 设为空闲块的总页数
4. `p->ref` 需要设置为 0 
5. 使用 `p->page_link` 将该页加入空闲块链表 `free_list`

遍历完所有非保留页之后，将空闲页的总数赋值给 `nr_free` 。

因为最初的内存中除内核以外只有一整块未被占用的物理内存空间，所以只有一个空闲块，只需要将第一个页的 `p->property` 设为总页数。

- `default_alloc_pages`

在 `free_list` 中遍历所有空闲块，查找第一个足以满足请求的空闲内存块，返回该块。如果找不到，返回 NULL 。

```c
static struct Page *
default_alloc_pages(size_t n) {
    assert(n > 0);
    if (n > nr_free) {
        return NULL;
    }
    struct Page *page = NULL;
    list_entry_t *le = &free_list;
    //查找满足请求的空闲内存块
    while ((le = list_next(le)) != &free_list) {
        struct Page *p = le2page(le, page_link);
        if (p->property >= n){
            page = p;
            break;
        }
    }
    //分配页，如果找到的空闲块比请求的大，将其分割
    if (page != NULL) {
        struct Page *p = page;
        for (; p != page + n; p ++) {
            SetPageReserved(p);
            ClearPageProperty(p);
            list_del(&(p->page_link));
        }
        p = page + n;
        if (page->property > n) {
            p->property = page->property - n;
        }
        nr_free -= n;
    }
    return page;
}
```

找到满足需求的空闲块后，需要将分配出去的内存块标记为非空闲，然后把这些块从 `free_list` 中删掉，并更新当前空闲内存块的总数  `nr_free`。如果找到的空闲块大小比需求的更大，则分割页块，将剩余部分第一页的 `p->property` 设置为原空闲块的页数减去分配出去的页数。

- `default_free_pages`

将页重新链接到空闲列表 `free_list` 中，可能会将小的空闲块合并为大的空闲块。

```c
static void
default_free_pages(struct Page *base, size_t n) {
    assert(n > 0);
    assert(PageReserved(base));
    list_entry_t *le = &free_list;
    struct Page *p;
    //查找合适的位置插入释放的页
    while ((le = list_next(le)) != &free_list) {
        p = le2page(le, page_link);
        if (p > base){
            break;
        }
    }
    p = base;
    //将释放的页插入空闲链表并将这些页初始化
    for (; p != base + n; p ++) {
        p->flags = p->property = 0;
        SetPageProperty(p);
        set_page_ref(p, 0);
        list_add_before(le, &(p->page_link));
    }
    //向高位的空闲块合并
    base->property = n;
    p = le2page(le, page_link);
    if (base + n == p) {
        base->property += p->property;
        p->property = 0;
    }
    //向低位的空闲块合并
    le = list_prev(&(base->page_link));
    p = le2page(le, page_link);
    if (p == base - 1 && le != &free_list ){
        while (le != &free_list) {
            if (p + p->property == base) {
                p->property += base->property;
                base->property = 0;
                break;
            }
            le = list_prev(le);
            p = le2page(le, page_link);
        }
    } 
    nr_free += n;
}
```

在 `free_list` 中查找第一个在需要释放的物理页 `base` 之后高位的空闲页 `p`，随后将需要释放的页一页一页地插入到该空闲页 `p`之前，并将这些插入的页的 `p->flag` 和 `p->ref` 等初始化。然后尝试向高位的空闲块合并，判断 `p` 对应的空闲块是否与要释放的内存块相连，如果相连，则重新计算 `base` 对应空闲块的页数。向低位合并也类似，首先判断低位的空闲块是否与要释放的内存块相连，如果相连，则向前遍历寻找低位空闲块的第一页，然后重新计算 `base` 对应空闲块的页数。

- 回答问题：你的 first-fit 算法是否有进一步的改进空间？

在本实验中实现的 first-fit 算法还有很大的改进空间，主要在时间效率上还有很大的不足。首先， `free_list` 中完全只可以有空闲块的第一页，没有必要插入其他页，这样可以节省一部分查询、插入、删除的开销。其次，链表的查询时间复杂度为 O(n) ，可以改为使用红黑树作为储存空闲块的数据结构，降低查询时间。

## **练习2：实现寻找虚拟地址对应的页表项（需要编程）**

> 通过设置页表和对应的页表项，可建立虚拟内存地址和物理内存地址的对应关系。其中的get_pte函数是设置页表项环节中的一个重要步骤。此函数找到一个虚地址对应的二级页表项的内核虚地址，如果此二级页表项不存在，则分配一个包含此项的二级页表。本练习需要补全get_pte函数 in kern/mm/pmm.c，实现其功能。请仔细查看和理解get_pte函数中的注释。get_pte函数的调用关系图如下所示：
>
> ![img](https://chyyuu.gitbooks.io/ucore_os_docs/content/lab2_figs/image001.png) 图1 get_pte函数的调用关系图
>
> 请在实验报告中简要说明你的设计实现过程。请回答如下问题：
>
> - 请描述页目录项（Page Directory Entry）和页表项（Page Table Entry）中每个组成部分的含义以及对ucore而言的潜在用处。
> - 如果ucore执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

在 ucore 页式管理的二级分页映射机制中，线性地址被划分为三段，分别是页目录索引、页表索引和页内偏移量。

在 `kern/mm/mmu.h` 中也可以看到这样的注释，其中说明了一些实用的宏定义：

```c
// A linear address 'la' has a three-part structure as follows:
//
// +--------10------+-------10-------+---------12----------+
// | Page Directory |   Page Table   | Offset within Page  |
// |      Index     |     Index      |                     |
// +----------------+----------------+---------------------+
//  \--- PDX(la) --/ \--- PTX(la) --/ \---- PGOFF(la) ----/
//  \----------- PPN(la) -----------/
//
// The PDX, PTX, PGOFF, and PPN macros decompose linear addresses as shown.
// To construct a linear address la from PDX(la), PTX(la), and PGOFF(la),
// use PGADDR(PDX(la), PTX(la), PGOFF(la)).
```

在 ucore 页式管理的二级分页映射机制中，将线性地址转化为物理地址需要经过三步：

1. 通过 `cr3` 寄存器找到一级页表的起始物理地址，然后加上页目录索引找到页目录项。
2. 从页目录项中找到二级页表的起始物理地址，然后加上页表索引找到页表项。
3. 从页表项中找到对应的物理页的起始物理地址，然后再加上页内偏移量就是目标的物理地址。

而 `get_pte` 函数返回一个虚地址对应的二级页表项的内核虚地址，对应地址转换的前两步。

由于二级页表是动态加载的，每个页可以动态地创建或回收。所以指定的二级页表项可能不存在，需要为二级页表分配内存页，然后再返回对应的二级页表项的地址。

查阅实验指导书及注释，完成 `get_pte` 函数。

```c
pte_t *
get_pte(pde_t *pgdir, uintptr_t la, bool create) {
    pde_t *pdep = pgdir + PDX(la);//获取页目录项
    if (!(*pdep & PTE_P)) {//检查该页目录项指向的页是否不存在
        struct Page *page; //如果不存在，为其创建页
        if (!create || (page = alloc_page()) == NULL) {
            return NULL;   //如果不需要创建，或者创建失败，返回NULL
        }
        set_page_ref(page, 1);//设置页引用
        uintptr_t pa = page2pa(page);//获取创建页的线性地址
        memset(KADDR(pa), 0, PGSIZE);//清除页内容
        *pdep = pa | PTE_U | PTE_W | PTE_P;//设置页目录项权限
    }
    pte_t *ptep = ((pte_t *) (KADDR(*pdep & ~0XFFF)) + PTX(la));
    return ptep;
}
```

根据将线性地址转化为物理地址的步骤，可以得知二级页表项的内核虚地址就是页目录项中二级页表的起始内核虚地址加上页表索引，这样就完成了该函数。

- 请描述页目录项（Page Directory Entry）和页表项（Page Table Entry）中每个组成部分的含义以及对 ucore 而言的潜在用处。

在 `kern/mm/mmu.h` 中可以看到以下这段定义，说明了页表项/页目录项各个 flag 位的含义：

```c
/* page table/directory entry flags */
#define PTE_P           0x001                   // Present
#define PTE_W           0x002                   // Writeable
#define PTE_U           0x004                   // User
#define PTE_PWT         0x008                   // Write-Through
#define PTE_PCD         0x010                   // Cache-Disable
#define PTE_A           0x020                   // Accessed
#define PTE_D           0x040                   // Dirty
#define PTE_PS          0x080                   // Page Size
#define PTE_MBZ         0x180                   // Bits must be zero
#define PTE_AVAIL       0xE00                   // Available for software use
                                                // The PTE_AVAIL bits aren't used by the kernel or interpreted by the
                                                // hardware, so user processes are allowed to set them arbitrarily.
```

这里的页表项有 32 位，而这些 flag 最多只有 12 位，从 `mmu.h` 中还可以找到下面这段定义，说明前 20 位是地址。对于页目录项是二级页表的地址，对于页表项是物理页的地址。

```c
// address in page table or page directory entry
#define PTE_ADDR(pte)   ((uintptr_t)(pte) & ~0xFFF)
#define PDE_ADDR(pde)   PTE_ADDR(pde)
```

`PTE_AVAIL` 对应的几位没有被硬件使用，作为保留位给操作系统使用，ucore 可以利用这些位来完成一些其他的内存管理相关的算法。

- 如果 ucore 执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

查阅资料得知，当 ucore 执行过程中出现了页访问异常，硬件需要完成的事如下：
>    1. 将发生错误的线性地址保存在 `cr2` 寄存器中
>    2. 向栈中依次压入 `EFLAGS` ， `CS` ， `EIP` ，以及页访问异常码 `error code`，如果 `page fault` 是发生在用户态，则还需要先压入 `ss` 和 `esp` ，并且切换到内核栈
>    3. 根据中断描述符表查询到对应 `page fault` 的 `ISR`，跳转到对应的 `ISR` 处执行，接下来将由软件进行 `page fault` 处理

## 练习3：释放某虚地址所在的页并取消对应二级页表项的映射（需要编程）

> 当释放一个包含某虚地址的物理内存页时，需要让对应此物理内存页的管理数据结构Page做相关的清除处理，使得此物理内存页成为空闲；另外还需把表示虚地址与物理地址对应关系的二级页表项清除。请仔细查看和理解page_remove_pte函数中的注释。为此，需要补全在 kern/mm/pmm.c 中的page_remove_pte函数。page_remove_pte函数的调用关系图如下所示：
>
> ![img](https://chyyuu.gitbooks.io/ucore_os_docs/content/lab2_figs/image002.png)
>
> 图2 page_remove_pte函数的调用关系图
>
> 请在实验报告中简要说明你的设计实现过程。请回答如下问题：
>
> - 数据结构Page的全局变量（其实是一个数组）的每一项与页表中的页目录项和页表项有无对应关系？如果有，其对应关系是啥？
> - 如果希望虚拟地址与物理地址相等，则需要如何修改lab2，完成此事？ **鼓励通过编程来具体完成这个问题** 

根据 `kern/mm/pmm.c` 中的注释及实验指导书中的说明，完成 `page_remove_pte` 函数。

```c
static inline void
page_remove_pte(pde_t *pgdir, uintptr_t la, pte_t *ptep) {
    if (*ptep & PTE_P) {//检查该页表项是否存在，如果存在，做清除处理
        struct Page *page = pte2page(*ptep);//获取页表项对应页
        if (page_ref_dec(page) == 0) {//减少页引用次数
            free_page(page);//如果页引用次数为0，释放该页
        }
        *ptep = 0;//清空该页表项的值
        tlb_invalidate(pgdir, la);//刷新tlb，如果该页表在tlb中，将其删去
    }
}
```

- 数据结构 Page 的全局变量（其实是一个数组）的每一项与页表中的页目录项和页表项有无对应关系？如果有，其对应关系是啥？

有对应关系，页目录项和页表项最终能指向 Page 中的一个物理页。页表项中的物理地址是 Page 中对应物理页的起始地址，因此可以通过页表项获取 Page 中对应的项。将页表项中的物理地址除以页大小就可以获得 Page 中对应项在 Page 数组中的id。

- 如果希望虚拟地址与物理地址相等，则需要如何修改lab2，完成此事？ 鼓励通过编程来具体完成这个问题 

由实验指导书可知， lab2 中虚拟地址到物理地址的映射关系如下：

```c
 virt addr = linear addr = phy addr + 0xC0000000
```

根据段页式管理的机制我们可以知道，虚拟地址通过段式管理转换为线性地址，随后线性地址通过页式管理转换为物理地址。可以通过调整段表 `gdt` 的方式使得 `virt addr = linear addr - 0xC0000000` ，这样虚拟地址就与物理地址相等了。

## 参考答案分析

### 练习1

`default_init` 、 `default_init_memmap` 基本一致，而在 `default_alloc_pages` 中，参考答案将找到需要分配出去的空闲块后的处理代码放在了遍历 `free_list` 寻找可用空闲块的循环中，差别也不大。

`default_free_pages` 中，参考答案仅对释放的第一页做了初始化处理，而我们的答案中因为上个函数 `default_alloc_pages` 里分配出去的页全都做了处理，在`default_free_pages` 中也对所有释放的页做了逆向的处理。

### 练习2

该函数的代码思路较为简单，我们的答案与参考答案基本一致，除了使用指针的方式略有不同以外。

### 练习3

由于该函数的代码实现较为简单，本实验中的实现与参考答案基本没有区别，因此不再赘述。

## 实验中涉及的知识点列举

本实验中涉及到的知识点如下：

- 系统探测物理内存分布和大小的方式
- ucore 中页式管理的机制
- ucore 启动段页式管理的过程
- 物理内存页分配算法的实现
- 链接地址、虚拟地址、线性地址、物理地址、加载地址的关系
- ucore 中通用双向链表的使用方法

对应的OS原理中的知识点如下：

- 内存页管理机制
- 内存分配算法
- 段页式内存管理
- 二级分页机制

本实验中涉及到的知识点是对应的OS原理中的知识点的具体实现和详细说明。