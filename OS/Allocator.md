---
title: Allocator
date: 2024-09-22 16:56:42
tags:
- original
- labs
categories:
- Operating System
---

### 技术路线

#### Allocator

操作系统位于硬件和软件之间，为用户程序提供底层硬件资源的抽象。操作系统提供必要的**系统调用（system calls）**。从而，用户程序不必再考虑如何使用硬件，而是通过这些 **system calls** 来调用硬件资源。

![Position of OS in A Computer System](https://ref.xht03.online/202412101902284.png)

当我们编写 c 语言程序时，我们可以直接通过 `malloc()` 和 `free()` 函数，简洁地申请和释放内存空间。但是 `malloc()` 和 `free()` 实际上并没有申请内存的权限，而是请操作系统的**内核（kernel）**帮忙。**kernel** 将`malloc()`所需的内存分配好并给它。所以，真正负责分配内存空间的是内核的**allocator（分配器）**。

**allocator（分配器）**是一种用于管理内存资源的机制。它负责在操作系统与应用程序之间调度和分配内存。内存分配器通常与动态内存管理有关，允许程序在运行时申请和释放内存。

具象的说：**allocator就是负责内存分配的系统调用函数**。

我们希望实现4个系统调用函数：

- `void kinit()`：初始化内存空间
- `void* kalloc_page()`：分配新的页
- `void kfree_page(void* p)`：释放页
- `void* kalloc(unsigned long long size)`：分配指定大小的内存块
- `void kfree(void* ptr)`：释放指定的内存块

> 请注意`kfree()`、`kfree_page()`接口的隐含意义：在释放空间时，用户不需告知库这块空间的大小。因此，在只传入一个指针的情况下，**allocator**必须能够弄清楚这块内存的大小。

---

#### Free-space management

- 如果需要管理的空间被划分为固定大小的单元，就很容易。

  > 就像：内存空间按**页（Page）**划分，内存由若干页组成。一页通常为4KB（4096 bytes）。

- 如果要管理的空闲空间由大小不同的单元构成，管理就变得困难而有趣。

  > 这种情况出现在：用户级的内存分配库，如`malloc()`和`free()`，或者操作系统用分段（segmentation）的方式实现虚拟内存。

对于第二种情况，出现了**外部碎片（external fragmentation）**的问题：空闲空间被分割成不同大小的小块，成为碎片，后续的请求可能失败，因为没有一块足够大的连续空闲空间，即使这时总的空闲空间超出了请求的大小。

例如在下图中，全部可用空闲空间是60字节，但被切成两个20字节大小的碎片，导致一个25字节的分配请求失败。

我们主要关心的是**外部碎片**。但是，分配程序也可能有**内部碎片（internal fragmentation）**的问题。如果分配程序给出的内存块超出请求的大小，在这种块中超出请求的空间（因此而未使用）就被认为是内部碎片（因为浪费发生在已分配单元的内部），这是另一种形式的空间浪费。

但是，简单起见，这里主要考虑外部碎片。

![外部碎片&内部碎片](https://ref.xht03.online/202412101903052.png)

---

#### Mechanism

**allocator**管理的空间由于历史原因被称为**堆（heap）**，在堆上管理空闲空间的数据结构通常称为**空闲列表（free list）**。该结构包含了管理内存区域中所有空闲块的引用（也就是：地址）。当然，该数据结构不一定真的是列表，而只是某种可以追踪空闲空间的数据结构。

我们先**以页为单位**讨论。

`freelist`本质上是一个指针，指向第一个空闲页的开始地址。每个空闲页开头存放一个指针`run`，指向下一个空闲页。最后一个空闲页的`run`指针指向`NULL`。

```c
// 空闲物理页节点
struct run {
    struct run* next;
};

// 空闲物理页的链表（的头节点）
struct {
    SpinLock lock;
    struct run* freelist;
} kmem;
```

当**allocator**初始化时：allocator本应该将所有物理内存分页，存进`freelist`中。但是由于物理地址刚开始处存放着内核代码，所以allocator会**把内核代码结束处以后的所有地址空间分页**。

![free list](https://ref.xht03.online/202412101904902.png)

当用户程序向**allocator**申请page时：**allocator**返回`freelist`，也就是将`freelist`指向的第一个page分配给它。然后`freelist = run1`，也就是使`freelist`指向下一个空闲页。如此，第一个页“page1”就被分配给用户程序了。需要注意的是：allocator不关心已分配的页，已分配的页由各个申请的用户程序管理。且已分配的页不用再维护`run`指针，所以整页都是可以使用的（只有空闲页才需要在页头写上`run`指针）。

![allocate a page](https://ref.xht03.online/202412101904831.png)

当用户程序向**allocator**释放page时：allocator将返还的page添加到当前链表的第一个，也就是：`freelist`=被释放的页的地址。然后：被释放页的`run`指针=原先`freelist`的值。

![free a page](https://ref.xht03.online/202412101905067.png)

---

以上，我们都是以页为单位，讨论内存分配。但是申请的内存可能很小，远远不足一页。

最朴素的想法就是：我们需要把一页的内存，按不同大小需求，划分成不同小块。

- **已分配的内存块**：

  我们需要保证分配出去的内存能被回收。由于`void kfree(void* ptr)`函数参数只告诉了地址指针，并没有告诉大小。所以每个内存块需要存储其大小`size`和一些必要信息`magic`，即每个内存块都需要有一个**头部（header）**。如此，当`kfree()`时：我们读取`ptr`指向处的往前若干字节就能知道需要释放的内存空间。

- **未分配的空间**：

  类似页于与页之间的`freelist`，我们也会在页内维护一个这样的`freelist`。

![内存块结构](https://ref.xht03.online/202412101905941.png)

![一次分配后的页](https://ref.xht03.online/202412101905643.png)

但是，如此分配内存空间后，即便使用过的内存块都被释放，**空闲列表里也是一串串大小不一的内存碎片**。而且它们在链表里的顺序是随机的，并不是按照地址从小到大排列的，由它们被用户程序释放的顺序所决定。

之所以会这样，是因为：我们忘了**合并（coalesce）列表项**，虽然整个内存空间是空闲的，但却被逐渐分成了小块，因此形成了碎片化的内存空间。

---

事已至此，那如何在内存池（`freelist`中的空闲内存块）中找到符合要求的内存块呢？且我们希望能保证：**快速**和**碎片最小化**。

常见的内存分配算法：

1. **首次适配（First-fit）**：在内存池中找到第一个满足需求的空闲块并分配。
2. **最佳适配（Best-fit）**：寻找最接近所需大小的空闲块进行分配，以减少剩余的空闲空间。
3. **最坏适配（Worst-fit）**：选择最大的空闲块分配，保留较大的剩余空间。
4. **快速适配（Quick-fit）**：预先分配若干种常用大小的内存块，减少分配和释放的时间。

---

#### Segregated list

为了解决内存碎片化的问题，我们采用**分离空闲列表（segregated list）**，也就是：**厚块分配程序（slab allocator）**。

基本想法很简单：如果某个应用程序经常申请一种（或几种）大小的内存空间，那就用一个独立的列表，只管理这样大小的对象。其他大小的请求都一给更通用的内存分配程序。

首先我们介绍基本概念：

- **Cache（缓存）**：SLAB 分配器为特定类型的对象创建缓存，缓存包含多个对象，这些对象的大小是固定的。缓存是预分配的内存块的集合，便于频繁创建和销毁相同类型的对象。

- **Slab（厚块）**：Slab 是 cache 中的组成单元，每个 cache 包含多个 Slab。Slab 是一组预分配的内存块，包含了若干个固定大小的对象块（即 **object**），用于保存对象。一个 Slab 可以是空闲的、部分使用的或者完全使用的。Slab 通过内存池预先分配内存，以便快速提供给请求的对象。
- **Object（对象）**：分配器管理的最小内存单元或内存块，被称为 object。每个 object 是一块固定大小的内存，用来存储具体的数据结构或对象。通过 Slab allocator，内存被分配为一个个 Slab，而 Slab 中则由若干个相同大小的 object 组成。

```c
#define NUM_CACHE 10    // 每个cache的对象大小：8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096

// 厚块节点（25个字节）
struct Slab {
    struct Slab* next;
    void* free_list;    // 空闲对象的链表
    usize free_count;     // 空闲对象的数量
    SpinLock lock;
};

// 厚块链表
struct Cache {
    struct Slab* slabs;
    usize slab_count; // slab的数量
    usize obj_size;   // 每个对象的大小（划分成几个字节的块）
} cache[NUM_CACHE];
```

具体来说：

1. 在内核启动时，它为可能频繁请求的内核对象创建一些**缓存**。这些的对象缓存每个分离了特定大小的空闲列表，因此能够很快地响应内存请求和释放。
2. 如果某个缓存中的空闲空间快耗尽时，它就向通用内存分配程序申请一些**内存厚块（slab）**（总量是页大小和对象大小的公倍数）。
3. 相反，如果给定厚块中对象的引用计数变为0，通用的内存分配程序可以从专门的分配程序中回收这些空间，这通常发生在虚拟内存系统需要更多的空间的时候。

---

### 代码实现

*详见[代码仓库](https://github.com/xht03/OS-2024/tree/lab1)。*

![内存结构](https://ref.xht03.online/202412101906439.png)

#### kinit()

初始化时，我们将始于`end`止于`PHYSTOP`的堆地址空间全部分页，并把他们连接到`kmem`结构体中的`freelist`。

注意：`end`是内核占用的地址空间的终止处，但不一定是4KB的倍数。对于`end`所在页没有用完的部分，我们弃之不用。我们从大于`end`的最小的4KB倍数的地址处开始分页，直至地址终点处`PHYSTOP`。（所以上图中`end`和page1之间并不是直接相接的，可能有空余空间。）

```c
void kinit() {
    init_rc(&kalloc_page_cnt);
    init_spinlock(&kmem.lock);						// 初始化自旋锁
    freerange((void*)end, (void*)P2K(PHYSTOP));		// 将堆划分成一页页
    init_caches();									// 初始化cache
}
```

```c
#define PGROUNDUP(sz)  (((sz)+PAGE_SIZE-1) & ~(PAGE_SIZE-1))
#define PGROUNDDOWN(a) (((a)) & ~(PAGE_SIZE-1))

// 释放指定区间的物理页（只有kinit时使用）
void freerange(void* pa_start, void* pa_end) {
    char *p;
    for(p = (char*)PGROUNDUP((usize)pa_start); p + PAGE_SIZE <= (char*)pa_end; p += PAGE_SIZE) {
        kfree_page(p);
    }
}
```

---

#### kalloc_page()

由于计算机可能是多核运行，可能同时多个进程申请空间。为了避免将同一页分配给多个进程，我们要加锁保持一致性。

```c
void* kalloc_page() {
    increment_rc(&kalloc_page_cnt);
    struct run *r;

    acquire_spinlock(&kmem.lock);   // 加锁
    r = kmem.freelist;
    if(r) {
        kmem.freelist = r->next;    // 从空闲链表中取出一个物理页r
    }
    release_spinlock(&kmem.lock);   // 解锁

    if(r){
        return (void*)r;
    } else {
        return NULL;    // 没有空闲物理页
    }
}
```

---

#### kfree_page()

将指针`p`所指的页添加会页的空闲列表。

```c
void kfree_page(void* p) {
    decrement_rc(&kalloc_page_cnt);
    struct run *r;

    if((usize)p % PAGE_SIZE || (usize)p < K2P(end) || (usize)p >= PHYSTOP) {
        printk("kfree_page fail: p = %p\n", p);   
        return;
    }

    r = (struct run*)p;
    acquire_spinlock(&kmem.lock);   // 加锁
    r->next = kmem.freelist;        // 将释放的物理页加入空闲链表
    kmem.freelist = r;
    release_spinlock(&kmem.lock);   // 解锁
    return;
}
```

---

#### slab_alloc()

给定`cache`，也就是给定对象大小，我们在一串串slab中寻找空闲对象，并返回其地址。如果`cache`中的所有slab都没有空闲对象，那么我们调用`kalloc_page()`申请新的页，并将其划分成新的slab，在返回新slab中的第一个对象的地址。

注意：

- 当我们把新的一页划分后得到slab时，由于每个slab的开头需要存放`Slab`结构体（25字节，但是为了对齐需要32字节）在开头，所以剩下4064字节用于划分对象。对象大小并不一定总是能整除剩余空间大小，所以是有可能划分完所有对象后，存在碎片空间。
- 对齐

```c
void* slab_alloc(struct Cache* cache) {
    //-------------------- 从已有的slab中分配对象 --------------------
    struct Slab* slab = cache->slabs;
    while (slab != NULL) {
        acquire_spinlock(&slab->lock);
        if (slab->free_count > 0) {
            void* obj = slab->free_list;
            slab->free_list = *(void**)obj; // 指向下一个空闲对象
            slab->free_count--;
            release_spinlock(&slab->lock);
            return obj;
        }
        release_spinlock(&slab->lock);
        slab = slab->next;
    }
    
    //-------------------- 如果所有slab都用完了，分配新的页 --------------------
    slab = (struct Slab*)kalloc_page();
    if (slab == NULL) {
        printk("slab_alloc: fail to get a new page\n");
        return NULL;
    }
    
    //-------------------- 初始化新的slab --------------------
    init_spinlock(&slab->lock);
    acquire_spinlock(&slab->lock);
    slab->next = cache->slabs;
    cache->slabs = slab;
    slab->free_list = (void*)((char*)slab + 32);   // 留出空间给 struct Slab(25 bytes)
    slab->free_count = (PAGE_SIZE - 32) / cache->obj_size;   // 8是指针的大小

    //-------------------- 初始化空闲对象链表 --------------------
    char* obj = (char*)slab->free_list;
    for(usize i = 1; i < slab->free_count; i++) {
        *(void**)obj = obj + cache->obj_size;   // 每一个对象的开始，存放下一个对象的地址
        obj += cache->obj_size;
    }
    *(void**)obj = NULL;
    cache->slab_count++;

    //-------------------- 返回第一个空闲对象 --------------------
    obj = slab->free_list;
    slab->free_list = *(void**)obj; // 指向下一个空闲对象
    slab->free_count--;
    release_spinlock(&slab->lock);
    return obj;
}
```

---

#### slab_free()

给出对象地址，我们将其添加会对应slab的空闲对象列表中。

注意：由于slab是由页划分得来的，所以slab的其实地址都是4KB的倍数。而且，每个slab的开始处存放着`struct Slab`结构体。所以我们只需要求出：小于`obj`的最大的4KB倍数（这就是该slab的`Slab`结构体地址）。

```c
void slab_free(void* obj) {
    struct Slab *slab = (struct Slab*)PGROUNDDOWN((usize)obj);
    acquire_spinlock(&slab->lock);
    *(void**)obj = slab->free_list;
    slab->free_list = obj;
    slab->free_count++;
    release_spinlock(&slab->lock);
}
```

---

#### kalloc()

分配指定大小的内存块。我们通过调用`slab_alloc()`来分配适当大小的内存块。

```c
void* kalloc(unsigned long long size) {
    // return NULL;

    //printk("CPU%lld kalloc: size = %lld\n", cpuid(), size);

    if (size > PAGE_SIZE) {
        printk("kalloc: size too large\n");
        return NULL; // 不支持大于一页的分配
    }

    struct Cache* cache = get_cache(size);	// 寻找object_size刚刚大于size的cache
    if (cache == NULL) {
        printk("kalloc: no suitable cache for size %lld\n", size);
        return NULL; // 没有合适的 cache
    }

    return slab_alloc(cache);
}


struct Cache* get_cache(usize size) {
    for (int i = 0; i < NUM_CACHE; i++) {
        if (size <= cache[i].obj_size) {
            return &cache[i];
        }
    }
    return NULL; // 如果没有合适的 cache
}
```

---

#### kfree()

释放指定地址的内存块。我们直接通过调用`slab_free()`把内存块添加会空闲对象列表，这样这个内存块就再被其他程序申请使用了。

```c
void kfree(void* ptr) {
    slab_free(ptr);
}
```

---

### 参考资料

[教材OSTEP第17章：空闲空间管理](https://pages.cs.wisc.edu/~remzi/OSTEP/)

[MIT的xv6操作系统](https://github.com/mit-pdos/xv6-public)

[FDU2024操作系统 (H)：Lab1](https://osh.fducslg.com/operating-systems-h-24fall)