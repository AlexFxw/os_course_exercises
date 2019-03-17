# lec6 SPOC思考题


NOTICE
- 有"w3l2"标记的题是助教要提交到学堂在线上的。
- 有"w3l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。

## 与视频相关思考题

### 6.1	非连续内存分配的需求背景
  1. 为什么要设计非连续内存分配机制？
       1. 增加内存分配的灵活性，避免内外碎片
       2. 提高内存使用效率，可以支持内存共享这类的功能


 1. 非连续内存分配中内存分块大小有哪些可能的选择？大小与大小是否可变?

    可变，大块方便管理，小块灵活性高。


 1. 为什么在大块时要设计大小可变，而在小块时要设计成固定大小？小块时的固定大小可以提供多种选择吗？

    大块本身管理较为容易，即使大小可变内存管理难度也不会太大。小块的话则需要固定对齐来降低内存管理难度。

### 6.2	段式存储管理
  1. 什么是段、段基址和段内偏移？
       1. 段：一个程序（进程）使用到的内存空间
       2. 段基址：段的起始地址。
       3. 段内偏移：段内的索引。


  1. 段式存储管理机制的地址转换流程是什么？为什么在段式存储管理中，各段的存储位置可以不连续？这种做法有什么好处和麻烦？

       1. 段基址+段内偏移可以得到实际地址
       2. 可以不连续是因为程序中不同的段往往不需要跨段访问，这是源于程序的逻辑结构的。
       3. 好处是方便管理而且容易实现进程保护，坏处的地址转换比较麻烦。


### 6.3	页式存储管理
 1. 什么是页（page）、帧（frame）、页表（page table）、存储管理单元（MMU）、快表（TLB, Translation Lookaside Buffer）和高速缓存（cache）？

     1. 页：一段固定大小的连续逻辑地址空间。
     1. 帧：一段固定大小的连续物理地址空间。
     1. 页表：负责页表号到实际地址映射关系的硬件。
     1. 存储管理单元：CPU中负责映射地址的部件。
     1. 快表：CPU里面一段完成“逻辑页号-物理页号”映射的Buffer
     1. Cache：根据某种算法把页的内容预加载的设计

 1. 页式存储管理机制的地址转换流程是什么？为什么在页式存储管理中，各页的存储位置可以不连续？这种做法有什么好处和麻烦？

     1. 根据逻辑页号从页表中查找相应的物理页号。
     1. 根据物理页号加上页内偏移得到实际的物理地址。

    好处是可以不连续，方便内存管理中的存储分配和回收。但地址转换比较复杂（页表项访问开销和页表存储开销），并且频繁进行增加开销（每次存储访问会变成两次或更多）。


### 6.4	页表概述
 1. 每个页表项有些什么内容？有哪些标志位？它们起什么作用？
     1. 存在位：是否存在一个物理帧和逻辑页号相对应
     1. 修改位：页面的内容是否被修改了
     1. 引用位：过去一段时间里是否有对这个页的引用。
 1. 页表大小受哪些因素影响？
    1. 页大小
    2. 地址空间大小
    3. 进程数目


### 6.5	快表和多级页表
 1. 快表（TLB）与高速缓存（cache）有什么不同？

    快表是虚拟页号到物理页号的映射，得到的是一个地址。cache缓存的则是页的内容。

 1. 为什么快表中查找物理地址的速度非常快？它是如何实现的？为什么它的的容量很小？

    因为是在CPU里面直接使用电路实现的。

 1. 什么是多级页表？多级页表中的地址转换流程是什么？多级页表有什么好处和麻烦？

    页号分成多段，从第一层查到第二层的索引，以此类推最后得到物理地址。好处是可以降低页表的大小，坏处是需要多访问几层多花事件。


### 6.6	反置页表
 1. 页寄存器机制的地址转换流程是什么？

    根据页寄存器以及逻辑地址进行Hash得到物理地址。

 1. 反置页表机制的地址转换流程是什么？

    逻辑地址、进程号进行哈希，以此索引物理页号。

 1. 反置页表项有些什么内容？

    PID、逻辑页号、标志位。

### 6.7	段页式存储管理
 1. 段页式存储管理机制的地址转换流程是什么？这种做法有什么好处和麻烦？

    先根据段号查到段的页表起始地址，再根据页号从页表中查找物理地址。

 1. 如何实现基于段式存储管理的内存共享？

    段表指针指向同一个段表。

 1. 如何实现基于页式存储管理的内存共享？

    页表指针指向同一个页表。

## 个人思考题
（1） (w3l2) 请简要分析64bit CPU体系结构下的分页机制是如何实现的

64位的寻址空间能够寻址16EB 的内存大小，对于目前的硬件来说太大，因此在X64体系结构下只实现了48位的虚拟地址。每级页表寻址长度为9位，由于在x64体系结构中，普通页大小仍为4KB，然而数据却表示64位长，因此一个4KB页在x64体系结构下只包含512项内容。为了保证页对齐和以页为单位的页表内容换入换出，在x64下每级页表寻址部分长度定位9位。

## 小组思考题
（1）(spoc) 某系统使用请求分页存储管理，若页在内存中，满足一个内存请求需要150ns (10^-9s)。若缺页率是10%，为使有效访问时间达到0.5us(10^-6s),求不在内存的页面的平均访问时间。请给出计算步骤。

$$
500=0.9\cdot150+0.1\cdot x
$$
（2）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示映射不存在。
PFN6..0:页帧号
PT6..0:页表的物理基址>>5
```
在[物理内存模拟数据文件](./03-2-spoc-testdata.md)中，给出了4KB物理内存空间的值，请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents。
```
1) Virtual Address 6c74
   Virtual Address 6b22
2) Virtual Address 03df
   Virtual Address 69dc
3) Virtual Address 317a
   Virtual Address 4546
4) Virtual Address 2c03
   Virtual Address 7fd7
5) Virtual Address 390e
   Virtual Address 748b
```

比如答案可以如下表示： (注意：下面的结果是错的，你需要关注的是如何表示)
```
Virtual Address 7570:
  --> pde index:0x1d  pde contents:(valid 1, pfn 0x33)
    --> pte index:0xb  pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)

Virtual Address 21e1:
  --> pde index:0x8  pde contents:(valid 0, pfn 0x7f)
      --> Fault (page directory entry not valid)

Virtual Address 7268:
  --> pde index:0x1c  pde contents:(valid 1, pfn 0x5e)
    --> pte index:0x13  pte contents:(valid 1, pfn 0x65)
      --> Translates to Physical Address 0xca8 --> Value: 16
```

[链接](https://piazza.com/class/i5j09fnsl7k5x0?cid=664)有上面链接的参考答案。请比较你的结果与参考答案是否一致。如果不一致，请说明原因。

Ans:

```
Virtual Address 6c74:
  --> pde index
```

（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python、ruby、C、C++、LISP、JavaScript等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。

[链接](https://piazza.com/class/i5j09fnsl7k5x0?cid=664)有上面链接的参考答案。请比较你的结果与参考答案是否一致。如果不一致，提交你的实现，并说明区别。

（4）假设你有一台支持[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)的机器，请问你如何设计操作系统支持这种类型计算机？请给出设计方案。

 (5)[X86的页面结构](http://os.cs.tsinghua.edu.cn/oscourse/OS2019spring/lecture06)
---

## 扩展思考题

阅读64bit IBM Powerpc CPU架构是如何实现[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)，给出分析报告。


## interactive　understand VM

[Virtual Memory with 256 Bytes of RAM](http://blog.robertelder.org/virtual-memory-with-256-bytes-of-ram/)：这是一个只有256字节内存的一个极小计算机系统。按作者的[特征描述](https://github.com/RobertElderSoftware/recc#what-can-this-project-do)，它具备如下的功能。
 - CPU的实现代码不多于500行；
 - 支持14条指令、进程切换、虚拟存储和中断；
 - 用C实现了一个小的操作系统微内核可以在这个CPU上正常运行；
 - 实现了一个ANSI C89编译器，可生成在该CPU上运行代码；
 - 该编译器支持链接功能；
 - 用C89, Python, Java, Javascript这4种语言实现了该CPU的模拟器；
 - 支持交叉编译；
 - 所有这些只依赖标准C库。

针对op-cpu的特征描述，请同学们通过代码阅读和执行对自己有兴趣的部分进行分析，给出你的分析结果和评价。
