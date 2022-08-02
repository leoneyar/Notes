# golang面试题

## 1.slice切片底层原理

`slice`实际上是一个结构体类型，包含3个字段，分别是

- array: 是指针，指向一个数组，切片的数据实际都存储在这个数组里。
- len: 切片的长度。
- cap: 切片的容量，表示切片当前最多可以存储多少个元素，如果超过了现有容量会自动扩容。

### slice扩容机制

- 如果新申请长度大于 2 倍的旧容量，那么最终容量就是新的切片长度 。

- 如果旧容量小于 256，那么最终容量就是旧容量的两倍 。
- 如果旧容量大于 256，最终容量不一定是多少，因为涉及到移位。（newcap = oldcap+(oldcap+3*256)/4，大概是之前容量的 1.25 倍）
- 如果最终容量计算值溢出，则最终容量就是新的切片长度。

## 2.进程通信

进程同步与进程通信很容易混淆，它们的区别在于：

- 进程同步：控制多个进程按一定顺序执行；
- 进程通信：进程间传输信息。

进程通信是一种手段，而进程同步是一种目的。也可以说，为了能够达到进程同步的目的，需要让进程进行通信，传输一些进程同步所需要的信息。

### 1. 管道

它具有以下限制：

- 只支持半双工通信（单向交替传输）；
- 只能在父子进程或者兄弟进程中使用。

### 2. FIFO

也称为命名管道，去除了管道只能在父子进程中使用的限制。

### 3. 消息队列

相比于 FIFO，消息队列具有以下优点：

- 消息队列可以独立于读写进程存在，从而避免了 FIFO 中同步管道的打开和关闭时可能产生的困难；
- 避免了 FIFO 的同步阻塞问题，不需要进程自己提供同步方法；
- 读进程可以根据消息类型有选择地接收消息，而不像 FIFO 那样只能默认地接收。

### 4. 信号量

它是一个计数器，用于为多个进程提供对共享数据对象的访问。

### 5. 共享存储

允许多个进程共享一个给定的存储区。因为数据不需要在进程之间复制，所以这是最快的一种 IPC。

需要使用信号量用来同步对共享存储的访问。

多个进程可以将同一个文件映射到它们的地址空间从而实现共享内存。另外 XSI 共享内存不是使用文件，而是使用内存的匿名段。

### 6. 套接字

与其它通信机制不同的是，它可用于不同机器间的进程通信。

## 3. TCP和UDP的区别

- UDP是无连接的，TCP是面向连接的；
- UDP是不可靠传输，不使用流量控制和拥塞控制，TCP是可靠传输，使用流量控制和拥塞控制；
- UDP是面向报文传输，TCP是面向字节流传输；
- UDP支持一对一，一对多，多对一和多对多交互通信，TCP只能一对一通信；
- UDP首部开销小，仅8字节，TCP首部最小20字节，最大60字节；
- UDP适用于实时应用（IP电话、视频会议、直播等），TCP适用于要求可靠传输的应用，例如文件传输。

## 4 .什么是协程（Goroutine）

协程是**用户态轻量级线程**，它是**线程调度的基本单位**。通常在函数前加上go关键字就能实现并发。一个Goroutine会以一个很小的栈启动2KB或4KB，当遇到栈空间不足时，栈会**自动伸缩**， 因此可以轻易实现成千上万个goroutine同时启动。

## 5. 请你讲一下Go面向对象是如何实现的？

Go实现面向对象的两个关键是struct和interface。

封装：对于同一个包，对象对包内的文件可见；对不同的包，需要将对象以大写开头才是可见的。

继承：继承是编译时特征，在struct内加入所需要继承的类即可：

```text
type A struct{}
type B struct{
A
}
```

多态：多态是运行时特征，Go多态通过interface来实现。类型和接口是松耦合的，某个类型的实例可以赋给它所实现的任意接口类型的变量。

Go支持多重继承，就是在类型中嵌入所有必要的父类型

## 6 .空 struct{} 的用途

-  用map模拟一个set，那么就要把值置为struct{}，struct{}本身不占任何空间，可以避免任何多余的内存分配。
- 有时候给通道发送一个空结构体,channel<-struct{}{}，也是节省了空间。
- 仅有方法的结构体。

## 7 .new和make的区别？

- new只用于分配内存，返回一个指向地址的**指针**。它为每个新类型分配一片内存，初始化为0且返回类型*T的内存地址，它相当于&T{}
- make只可用于**slice,map,channel**的初始化,返回的是**引用**。

## 8.golang的内存管理的原理清楚吗？简述go内存管理机制 ([链接](https://cloud.tencent.com/developer/article/1422392))

golang内存管理基本是参考tcmalloc来进行的。go内存管理本质上是一个内存池，只不过内部做了很多优化：自动伸缩内存池大小，合理的切割内存块。

一些基本概念：
页Page：一块8K大小的内存空间。Go向操作系统申请和释放内存都是以页为单位的。
span : 内存块，一个或多个连续的 page 组成一个 span 。如果把 page 比喻成工人， span 可看成是小队，工人被分成若干个队伍，不同的队伍干不同的活。
sizeclass : 空间规格，每个 span 都带有一个 sizeclass ，标记着该 span 中的 page 应该如何使用。使用上面的比喻，就是 sizeclass 标志着 span 是一个什么样的队伍。
object : 对象，用来存储一个变量数据内存空间，一个 span 在初始化时，会被切割成一堆等大的 object 。假设 object 的大小是 16B ， span 大小是 8K ，那么就会把 span 中的 page 就会被初始化 8K / 16B = 512 个 object 。所谓内存分配，就是分配一个 object 出去。

1.mheap

一开始go从操作系统索取一大块内存作为内存池，并放在一个叫mheap的内存池进行管理，mheap将一整块内存切割为不同的区域，并将一部分内存切割为合适的大小。

![](D:\GithubRepository\Notes\面试\1657958289618.png)

mheap.spans ：用来存储 page 和 span 信息，比如一个 span 的起始地址是多少，有几个 page，已使用了多大等等。

mheap.bitmap 存储着各个 span 中对象的标记信息，比如对象是否可回收等等。

mheap.arena_start : 将要分配给应用程序使用的空间。

2.mcentral

用途相同的span会以链表的形式组织在一起存放在mcentral中。这里用途用**sizeclass**来表示，就是该span存储哪种大小的对象。

找到合适的 span 后，会从中取一个 object 返回给上层使用。

3.mcache

为了提高内存并发申请效率，加入缓存层mcache。每一个mcache和处理器P对应。Go申请内存首先从P的mcache中分配，如果没有可用的span再从mcentral中获取。

总结（[原文](https://cloud.tencent.com/developer/article/1422414)）

- Go在程序启动时，会向操作系统申请一大块内存，之后自行管理。
- Go内存管理的基本单元是mspan，它由若干个页组成，每种mspan可以分配特定大小的object。
- mcache, mcentral, mheap是Go内存管理的三大组件，层层递进。mcache管理线程在本地缓存的mspan；mcentral管理全局的mspan供所有线程使用；mheap管理Go的所有动态分配内存。
- 极小对象会分配在一个object中，以节省资源，使用tiny分配器分配内存；一般小对象通过mspan分配内存；大对象则直接由mheap分配内存。

## 9.go如何进行调度的。GMP中状态流转。

Go里面GMP分别代表：G：goroutine，M：线程（真正在CPU上跑的），P：调度器。

![](D:\GithubRepository\Notes\面试\1657973184981.png)

调度器是M和G之间桥梁。

go进行调度过程：

- 某个线程尝试创建一个新的G，那么这个G就会被安排到这个线程的G本地队列LRQ中，如果LRQ满了，就会分配到全局队列GRQ中；

- 尝试获取当前线程的M，如果无法获取，就会从空闲的M列表中找一个，如果空闲列表也没有，那么就创建一个M，然后绑定G与P运行。

- 进入调度循环：

- - 找到一个合适的G
  - 执行G，完成以后退出

## 10 . map的底层实现 （[链接](https://blog.csdn.net/chenxun_2010/article/details/103768011?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.pc_relevant_aa&spm=1001.2101.3001.4242.1&utm_relevant_index=3)）

go map 采用的是哈希查找表，并且使用链表解决哈希冲突
    

```go
type hmap struct {
    count     int //map元素的个数，调用len()直接返回此值// map标记:
// 1. key和value是否包指针
// 2. 是否正在扩容
// 3. 是否是同样大小的扩容
// 4. 是否正在 `range`方式访问当前的buckets
// 5. 是否有 `range`方式访问旧的bucket
flags     uint8 

B         uint8  // buckets 的对数 log_2
noverflow uint16 // overflow 的 bucket 近似数
hash0     uint32 // hash种子 计算 key 的哈希的时候会传入哈希函数
buckets   unsafe.Pointer // 指向 buckets 数组，大小为 2^B 如果元素个数为0，就为 nil

// 扩容的时候，buckets 长度会是 oldbuckets 的两倍
oldbuckets unsafe.Pointer // bucket slice指针，仅当在扩容的时候不为nil

nevacuate  uintptr // 扩容时已经移到新的map中的bucket数量
extra *mapextra // optional fields
}
```
注意：**B 是buckets 数组的长度的对数，也就是说 buckets 数组的长度就是 2^B。bucket 里面存储了 key 和 value。**

**buckets 是一个指针，最终它指向的是一个结构体：（buckets是bmap类型的数组，数组长度是2^B）**

```go
// A bucket for a Go map.
type bmap struct {
    tophash [bucketCnt]uint8
}
```

**bmap就是我们所说的桶bucket，实际上就是每个bucket固定包含8个key和value(可以查看源码bucketCnt=8).实现上面是一个固定的大小连续内存块，分成四部分：**

1. 每个条目的状态
2. 8个key值
3. 8个value值
4. 指向下个bucket的指针

桶里面会最多装 8 个key，这些key之所以会落入同一个桶，是因为它们经过哈希计算后，哈希结果是“一类”的。在桶内，又会根据 key 计算出来的 hash值的高 8 位来决定 key 到底落入桶内的哪个位置（一个桶内最多有8个位置）。

**每个 bucket 设计成最多只能放 8 个 key-value 对，如果有第 9 个 key-value 落入当前的 bucket，那就需要再构建一个 bucket ，通过 overflow 指针连接起来**

## 11.知道golang的**内存逃逸**吗？什么情况下会发生内存逃逸？

逃逸分析就是确定一个变量要放堆上还是栈上，什么是堆？什么是栈？

- 堆：一般来讲是人为手动进行管理，手动申请、分配、释放。一般所涉及的内存大小并不定，一般会存放较大的对象。另外其分配相对慢，涉及到的指令动作也相对多
- 栈：由编译器进行管理，自动申请、分配、释放。一般不会太大，我们常见的函数参数（不同平台允许存放的数量不同），局部变量等等都会存放在栈上

`golang程序变量`会携带有一组校验数据，用来证明它的整个生命周期是否在运行时完全可知。如果变量通过了这些校验，它就可以在`栈上`分配。否则就说它 `逃逸` 了，必须在`堆上分配`。

能引起变量逃逸到堆上的**典型情况**：

- **在方法内把局部变量指针返回** 局部变量原本应该在栈中分配，在栈中回收。但是由于返回时被外部引用，因此其生命周期大于栈，则溢出。
- **发送指针或带有指针的值到 channel 中。** 在编译时，是没有办法知道哪个 goroutine 会在 channel 上接收数据。所以编译器没法知道变量什么时候才会被释放。
- **在一个切片上存储指针或带指针的值。** 一个典型的例子就是 []*string 。这会导致切片的内容逃逸。尽管其后面的数组可能是在栈上分配的，但其引用的值一定是在堆上。
- **slice 的背后数组被重新分配了，因为 append 时可能会超出其容量( cap )。** slice 初始化的地方在编译时是可以知道的，它最开始会在栈上分配。如果切片背后的存储要基于运行时的数据进行扩充，就会在堆上分配。
- **在 interface 类型上调用方法。** 在 interface 类型上调用方法都是动态调度的 —— 方法的真正实现只能在运行时知道。想像一个 io.Reader 类型的变量 r , 调用 r.Read(b) 会使得 r 的值和切片b 的背后存储都逃逸掉，所以会在堆上分配。

## 12.[简述 Go 语言GC(垃圾回收)的工作原理](https://learnku.com/articles/59021)

垃圾回收机制是Go一大特(nan)色(dian)。Go1.3采用**标记清除法**， Go1.5采用**三色标记法**，Go1.8采用**三色标记法+混合写屏障**。

**标记清除法**

分为两个阶段：标记和清除

标记阶段：从根对象出发寻找并标记所有存活的对象。

清除阶段：遍历堆中的对象，回收未标记的对象，并加入空闲链表。

缺点是需要暂停程序STW。

**三色标记法**

1. 将所有对象标记为白色

2. 从根节点集合出发，将第一次遍历到的节点标记为灰色放入集合列表中

3. 遍历灰色集合，将灰色节点遍历到的白色节点标记为灰色，并把灰色节点标记为黑色

4. 循环这个过程

5. 直到灰色节点集合为空，回收所有的白色节点


这种方法有一个缺陷，如果对象的引用被用户修改了，那么之前的标记就无效了。因此Go采用了**写屏障技术**，当对象新增或者更新会将其着色为灰色。

**Go1.8 三色标记 + 混合写屏障**

基于插入写屏障和删除写屏障在结束时需要 STW 来重新扫描栈，所带来的性能瓶颈，Go 在 1.8 引入了混合写屏障的方式实现了弱三色不变式的设计方式，混合写屏障分下面四步

1. GC开始时，将栈上的全部对象标记为黑色（不需要二次扫描，无需STW）；
2. GC期间，任何栈上创建的新对象均为黑色
3. 被删除引用的对象标记为灰色
4. 被添加引用的对象标记为灰色

## 13. [channel底层实现？](https://learnku.com/articles/58480)

runtime/chan.go

```go
type hchan struct {
    qcount   uint           // 队列中剩余元素
    dataqsiz uint           // 队列长度，eg make(chan int64, 5), dataqsiz为5
    buf      unsafe.Pointer // 数据存储环形数组
    elemsize uint16         // 每个元素的大小
    closed   uint32         // 是否关闭 0 未关闭
    elemtype *_type         // 元素类型
    sendx    uint           // 发送者写入位置
    recvx    uint           // 接受者读数据位置
    recvq    waitq          // 接收者队列，保存正在读取channel的goroutine
    sendq    waitq          // 发送者队列，保存正在发送channel的goroutine
    lock     mutex          // 锁
}
```

waitq 是双向链表，sudog 为 goroutine 的封装

```go
type waitq struct {
    first *sudog
    last  *sudog
}
```

make(chan int, 6)

![img](https://cdn.learnku.com/uploads/images/202106/26/40362/ZJwf0bBqHp.png!large)

上图为一个长度为 6，类型为 int, 两个接收者，三个发送者的 channel，当前接收者准备读数据的位置为 0，发送者发送数据位置为 4

**注意，一般情况下 recvq 和 sendq 至少有一个为空。只有一个例外，那就是同一个 goroutine 使用 select 语句向 channel 一边写数据，一边读数据。**

channel底层实现在`src/runtime/chan.go`中

channel内部是一个循环链表。内部包含buf, sendx, recvx, lock ,recvq, sendq几个部分；

buf是有缓冲的channel所特有的结构，用来存储缓存数据。是个循环链表；

- sendx和recvx用于记录buf这个循环链表中的发送或者接收的index；
- lock是个互斥锁；
- recvq和sendq分别是接收(<-channel)或者发送(channel <- xxx)的goroutine抽象出来的结构体(sudog)的队列。是个双向链表。
