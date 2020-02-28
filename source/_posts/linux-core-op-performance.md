---
title: An Analysis of PerformanceEvolution of Linux’s Core Operations
date: 2019-11-13 18:25:17
tags:
---


* KPTI: Kernel page-table isolation 内核页表隔离
    * 解决 melt down 漏洞
    * 完全分离用户空间和内核空间页表来解决页表泄露
    * 从内核空间进入用户空间时，删除内核页表项
        * 进/出内核都会swap page table pointer (CR3寄存器), 导致TLB flush, 导致其他的TLB miss
        * 对于支持process-context identifier(PCID)的cpu，linux做了个优化，给内核和用户空间的TLB entry分别设置不同的PCID，各自访问各自的，避免了TLB flush

* Avoid indirect branch speculation
    * eg: jmp [rax], rax需要运行时决定，cpu会根据历史预执行可能的target，可能被exploiter利用
    * Retpoline patch在编译期将indirect branch (`jmp [rax]`)换成以下thunk:
    ```c
    0 # normal code
    1    call load_label        ;push return address to stack, and jmp to 6
    2 capture_ret_spec:
    3    pause ; lfence         ;此两条inst提示cpu此语句在一个spin-loop里，使之减少power consumption
    4    jmp capture_ret_spec
    5 load_label:
    6    mov rax, [rsp]         ;replace return address with rax (原跳转地址)
    7    ret                    ;此ret将跳转到[rax]
    8    
    9 # rax target
    10 ...    
    ```
    这样可以在不使用indirect branch的情况下达到indirect branch的效果
    
    **运行时cpu会认为跳转的地址是3，从而增加了branch misprediction机率，导致性能变差，预测执行的将是3，因此增加pause和循环跳转，提示cpu减少power消耗**
    * 
* SLAB freelist randomization
    * 等大小的连续内存空间，用于存放等大小的内核数据
    * allocator维护一个freelist，将连续的内存区域用链表串起来
    * 导致：连续分配的对象，在内存上也是相邻存放的，容易被exploit
    * fix: 在slab初始化时，给每块内存分配一个随机数，然后链表是根据随机数的大小从小到大建立，这样连续分配内存给内核对象时，就不是物理相邻的了
    * overhead: 生成随机数，降低locality，将顺序访问变成了随机方法
* Hardened usercopy
    * 在内核和用户空间之间拷贝数据时要validate kernel pointers, 检查数据大小，防止kernel->userspace拷贝过多从而泄露内核数据，也防止userspace->kernel拷贝太多造成buffer overflow
    * 检查包括：
        * kernel pointer != null
        * kernel region doesn't overlap text segment
        * the object’s size does not exceed the size limit of its SLAB if it is allocated using the SLAB allocator
* Fault around
    * Pre-establishes mappings for pages surrounding a faulting page, if they are available in the file cache
    * Adds constant overhead for page faults on read-only file-backed pages
* Control group memory controller
    * Accounts and limits memory usage per control group
    * Adds overhead when establishing or destroying page mappings
* Disableing transparent huge pages
* Userspace page fault handling
    * 用户态进程处理某些内存区域的page fault

* Forced context tracking
* TLB layout specification
* Missing CPU power saving states
