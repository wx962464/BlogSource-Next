title: JAVA垃圾回收机制
date: 2016-04-25 20:35:35
tags: Java
---

在java中垃圾回收是系统自动完成的，了解它对优化应用程序有很大的帮助。那么我们就从下面几个方面来了解垃圾回收机制：
1.  哪些对象需要回收?
2.  什么时候回收？
3.  怎么去回收？

## 判断对象可以回收的方法：
### 引用计数算法
![引用计数](/img/java-gc/java-gc-jishu.png) 

给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加1；当引用失效时，计数器值就减1；任何时刻计数器为0的对象就是不可能再被使用的。
> 
优点：简单，高效，现在的objective-c用的就是这种算法。
缺点：很难处理循环引用，相互引用的两个对象则无法释放。

### 可达性分析算法（根搜索算法）
![可达性分析](/img/java-gc/java-gc-root_searchble.png)
    从GC Roots（每种具体实现对GC Roots有不同的定义）作为起点，向下搜索它们引用的对象，可以生成一棵引用树，树的节点视为可达对象，反之视为不可达。
在Java语言中，可以作为GC Roots的对象包括下面几种：
- 虚拟机栈（栈帧中的本地变量表）中的引用对象。
* 方法区中的类静态属性引用的对象。
* 方法区中的常量引用的对象。
* 本地方法栈中JNI（Native方法）的引用对象

> 
真正标记以为对象为可回收状态至少要标记两次。

## 常见概念介绍
### 引用类型分类
引用类型有四种类型分别是：强引用（Strong Reference）、软引用(soft Reference)、弱引用（Weak Reference）、虚引用（Phantom Reference），其中强引用是我们常用到的。下面来说明下他们什么时候会被垃圾回收机制回收。
**强引用：**  这种最顽强，只要有一个引用存在，永远都不会被回收掉。
**软引用：** 一般是指还有用，但是非必须的对象。在内存空间不足的情况下，会回收掉此部分内存，如果还不够则会抛出内存溢出异常。
**弱引用：** 一般指非必须的对象，比软引用还要弱，它只能生存到下一次垃圾回收前，如果一旦发生垃圾回收，它将会被回收掉。
**虚引用：** 最弱的引用关系，无法通过虚引用来取得一个对象的实例。为一个对象设置虚引用的唯一目的就是能够在这个对象被回收的时候收到一个系统通知。

<!--more-->

### 并发与并行
* **并行（Parallel）：** 指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。

* **并发（Concurrent）：**指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），用户程序在继续运行，而垃圾收集程序运行于另一个CPU上。

### Minor GC 和 Full GC
* **新生代GC（Minor GC）：**指发生在新生代的垃圾收集动作，因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。

* **老年代GC（Major GC / Full GC）：**指发生在老年代的GC，出现了Major GC，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）。Major GC的速度一般会比Minor GC慢10倍以上。

### 吞吐量
吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量 = 运行用户代码时间 /（运行用户代码时间 + 垃圾收集时间）。
虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。

### Stop the world 概念
因为垃圾回收的时候，需要整个的引用状态保持不变，否则判定是判定垃圾，等我稍后回收的时候它又被引用了，这就全乱套了。所以，GC的时候，其他所有的程序执行处于暂停状态，卡住了。
幸运的是，这个卡顿是非常短（尤其是新生代），对程序的影响微乎其微 （关于其他GC比如并发GC之类的，在此不讨论）。
所以GC的卡顿问题由此而来，也是情有可原，暂时无可避免。

> 
**finalize需要注意地方**
同一个对象的finalize 方法只会执行一次。
在给对象A赋值为空后进行手动调用gc回收内存，会执行复写了finalize方法，在finalize中再将A赋值一个非空的值，再次进行赋值为空，调用gc，之后就不会再执行finalize方法了。

## 几种垃圾回收算法
1. 标记清除算法
2. 复制算法算法
3. 标记整理算法
4. 分带回收算法

### 标记清除算法

这个按名字就很容易理解，就是标记哪些是可回收的，然后进行清除回收处理。标记-清除算法分为两个阶段：标记阶段和清除阶段,标记阶段主要为标记那些对象是可以回收的，清除就是回收标记过的那部分内存空间。
![mark_clear](/img/java-gc/java-gc-mark-clear.png)

> 
**优点：** 简单，易实现
**缺点：** 容易产生内存碎片，对于后面分配大空间时，找不到足够的空间，而主动会触发一次内存回收，增加内存回收的次数。

### 复制算法
此方法将内存按容量分为两块，例如A、B两块，每次只使用其中的一块，当要进行回收操作时，将A中还存活的对象复制到B块中（假设上次使用A），然后对A中所有对象清空就又构成一个完整的内存块。这种方法就避免了标记清除的内存碎片问题。
![copying](/img/java-gc/java-gc-copying.png)
> 
**优点：** 快速高效，不会产生内存碎片。
**缺点：** 可用内存会减少一半，因为是按照均分的。

**注意：** 效率也与存活对象的多少有关，如果存活对象多，复制就多，效率就低了。

### 标记整理法
标记整理法就是在标记清除方法上进行的优化，主要是在标记完成后将这些存活的对象向一端移动，然后将末尾边界后的所有内存空间清除。

![java-gc-compact](/img/java-gc/java-gc-mark-compact.png)

> 
**优点：** 适合存活对象多的，不产生内存碎片

### 分代回收法
分代回收算法实际上是把复制算法和标记整理法的结合，并不是真正一个新的算法，一般分为：老年代（Old Generation）和新生代（Young Generation），老年代就是很少垃圾需要进行回收的，新生代就是有很多的内存空间需要回收，所以不同代就采用不同的回收算法，以此来达到高效的回收算法。
新生代：由于新生代产生很多临时对象，大量对象需要进行回收，所以采用复制算法是最高效的。
老年代：回收的对象很少，都是经过几次标记后都不是可回收的状态转移到老年代的，所以仅有少量对象需要回收，故采用标记清除或者标记整理算法。
所以以上整个过程就达到了最高效的回收办法。

> 
注意：新生代与老年代分配的比例为 2：1

#### 分代回收法详解
分代算法主要核心思想是上面两种算法的结合，但是它里面还有一些特殊的地方，所以这里进行特别的说明。
上面用到的复制算法，按照前面讲的，按照空间1：1分配的，但是分代算法中不是这样的，而是按照8：2 （4：1）的比例进行分配的，其中后面一块又等分成了两块，也就是8:1:1的比例，8为新生代（Eden区），1分别为两块等大小的Survivor区。
所有新创建的对象都是放在Eden区的，所以有很多的临时变量，故每次大部分回收的垃圾都是在Eden区的，所以Eden区会分配那么多的空间，那为什么要分两块Survivor区呢？ 是因为，在进行一次coping算法回收时将Eden区中存活的对象复制到Survivor区，然后在进行一次回收时，Survivor区的存活对象没地方存放了，因为Eden区每次都有新创建的对象存在。故在新建一块Survivor B区，这时将Eden和Survivor A区存活的对象放在B区，然后清空Eden和Survivor A区。这样就相当于A和B每次回收后都有一个是全新的也就是空的，就是为了循环这种操作。

#### 分代回收，内存是如何分配的
1.  新创建一个对象，默认是分在Eden区的，当Eden区内存不够时，触发一次minor gc （新生代回收），Eden区存活对象放入Survivor A区，然后新对象放入Eden区。
2.  再新建一个对象，放入Eden区发现又满了，在进行minor gc，这时将Eden区和Survivor A区存活的对象移动到Survivor B区，然后清空Eden和Survivor A。
3.  如果再来一个新对象，Eden区又满了，则再进行minor gc，移动到A区，然后清空Eden和B区，如果多次这样，有些长久的对象就会在不断的Survivor A和Survivor B区之间来回移动，多次之后，虚拟机默认15次后，就会将这些对象全部移动到老年区区。
4.  如果新对象在eden区放不下，则直接放入老年代
5.  如果Survivor 区放不下的对象，也直接放入老年代
6.  发现老年代也满了，则进行一次Full gc (major gc)

### 火车回收算法
垃圾收集算法一个很大的缺点就是难以控制垃圾回收所占用的CPU时间，以及何时需要进行垃圾回收。火车算法是分代收集器所用的算法，目的是在成熟对象空间中提供限定时间的渐进收集。目前应用于SUN公司的Hotspot虚拟机上。
在火车算法中，内存被分为块，多个块组成一个集合。为了形象化，一节车厢代表一个块，一列火车代表一个集合。
 注意每个车厢大小相等，但每个火车包含的车厢数不一定相等。垃圾收集以车厢为单位，收集顺序按被创建的先后顺序进行。

具体参考：[http://blog.csdn.net/feier7501/article/details/21748615](http://blog.csdn.net/feier7501/article/details/21748615)

## 常用垃圾回收机制
1. 标记－清除收集器
这种收集器首先遍历对象图并标记可到达的对象，然后扫描堆栈以寻找未标记对象并释放它们的内存。这种收集器一般使用单线程工作并停止其他操作。
2. 标记－压缩收集器
有时也叫标记－清除－压缩收集器，与标记－清除收集器有相同的标记阶段。在第二阶段，则把标记对象复制到堆栈的新域中以便压缩堆栈。这种收集器也停止其他操作。
3. 复制收集器
这种收集器将堆栈分为两个域，常称为半空间。每次仅使用一半的空间，虚拟机生成的新对象则放在另一半空间中。垃圾回收器运行时，它把可到达对象复制到另一半空间，没有被复制的的对象都是不可达对象，可以被回收。这种方法适用于短生存期的对象，持续复制长生存期的对象由于多次拷贝，导致效率降低。缺点是只有一半的虚拟机空间得到使用。
4. 增量收集器
增量收集器把堆栈分为多个域，每次仅从一个域收集垃圾。这会造成较小的应用程序中断。
5. 分代收集器
这种收集器把堆栈分为两个或多个域，用以存放不同寿命的对象。虚拟机生成的新对象一般放在其中的某个域中。过一段时间，继续存在的对象将获得使用期并转入更长寿命的域中。分代收集器对不同的域使用不同的算法以优化性能。这样可以减少复制对象的时间。
6. 并发收集器
并发收集器与应用程序同时运行。这些收集器在某点上（比如压缩时）一般都不得不停止其他操作以完成特定的任务，但是因为其他应用程序可进行其他的后台操作，所以中断其他处理的实际时间大大降低。
7. 并行收集器
并行收集器使用某种传统的算法并使用多线程并行的执行它们的工作。在多CPU机器上使用多线程技术可以显著的提高java应用程序的可扩展性。
8.  自适应收集器
根据程序运行状况以及堆的使用状况，自动选一种合适的垃圾回收算法。这样可以不局限与一种垃圾回收算法。

## 几种垃圾收集器
常见的垃圾收集器有：serial收集器、Parallel收集器、Parallel Old 垃圾收集器、CMS（Concurrent Mark-Sweep）收集器、G1收集器.其中Serial收集器为串行收集器，其他均为并行收集器。

串行垃圾回收器（Serial Garbage Collector）
并行垃圾回收器（Parallel Garbage Collector）
并发标记扫描垃圾回收器（CMS Garbage Collector）
G1垃圾回收器（G1 Garbage Collector）

### Serial收集器->串行收集器 (-XX:+UseSerialGC)
最古老，最稳定，简单而高效，可能会产生较长的停顿。
Serial是一个`单线程`的收集器，它不仅仅只会使用一个CPU或一条线程去完成垃圾收集工作，并且在进行垃圾收集的同时，必须暂停其他所有的工作线程，直到垃圾收集结束。
Serial垃圾收集器虽然在收集垃圾过程中需要暂停所有其他的工作线程，但是它简单高效，对于限定单个CPU环境来说，没有线程交互的开销，可以获得最高的单线程垃圾收集效率，因此Serial垃圾收集器依然是java虚拟机运行在Client模式下默认的新生代垃圾收集器。

![java-gc-serial](/img/java-gc/java-gc-serial.jpg)

### Serial Old收集器
Serial Old是Serial垃圾收集的老年代版本。它同样是个单线程的收集器，使用标记-整理算法，这个收集器也主要是运行在Client默认的java虚拟机默认的年老代垃圾收集器。

### ParNew收集器 (-XX:+UseParallelGC)
ParNew收集器其实就是Serial收集器的`多线程`版本，除了使用多条线程进行垃圾收集之外，其余行为包括Serial收集器可用的所有控制参数、收集算法、Stop The World、对象分配规则、回收策略等都与Serial收集器完全一样，在实现上，这两种收集器也共用了相当多的代码。ParNew收集器是许多运行在Server模式下的虚拟机中首选的新生代收集器。
![ParNew收集器](http://upload-images.jianshu.io/upload_images/650075-483c1885e2d36f65.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Parallel Scavenge
Parallel Scavenge是一个新生代收集器，使用多线程和复制算法。相比其他收集器，只有这个收集器是针对系统吞吐量进行改进，适用于后台运算并且交互不多的程序。其他收集器则更关注改善收集时的停顿时间，适用于用户交互的程序。

### Parallel Old 垃圾收集器(-XX:+UseParallelOldGC)
Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记－整理”算法。
在注重吞吐量以及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge加Parallel Old收集器。

![Parallel](http://upload-images.jianshu.io/upload_images/650075-0f74222185d67afc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### CMS 收集器
CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。目前很大一部分的Java应用集中在互联网站或者B/S系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。
CMS收集器是基于“标记—清除”算法实现的。整个过程需要下面四个步骤。
1. 初始标记（CMS initial mark）
初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，需要“Stop The World”。
2. 并发标记（CMS concurrent mark）
并发标记阶段就是进行GC Roots Tracing的过程。
3. 重新标记（CMS remark）
重新标记阶段是为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短，仍然需要“Stop The World”。
4. 并发清除（CMS concurrent sweep）
并发清除阶段会清除对象。

** 优点：** 并发收集、低停顿。
**缺点：**
CMS收集器对CPU资源非常敏感，以为在并发阶段占用一部分线程（CPU资源），导致应用程序变慢，总吞吐量变低。CMS默认启动的回收线程数是`（CPU数量+3）/4`，也就是当CPU在4个以上时，并发回收时垃圾收集线程不少于25%的CPU资源，并且随着CPU数量的增加而下降。

CMS收集器无法处理浮动垃圾,可能出现“Concurrent Mode Failure”失败而导致另一次Full GC的产生。
也是由于在垃圾收集阶段用户线程还需要运行，那也就还需要预留有足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，需要预留一部分空间提供并发收集时的程序运作使用。要是CMS运行期间预留的内存无法满足程序需要，就会出现一次“Concurrent Mode Failure”失败，这时虚拟机将启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。

CMS是一款基于“标记—清除”算法实现的收集器，这意味着收集结束时会有大量空间碎片产生。空间碎片过多时，将会给大对象分配带来很大麻烦，往往会出现老年代还有很大空间剩余，但是无法找到足够大的连续空间来分配当前对象，不得不提前触发一次Full GC。

> 
**浮动垃圾：** 由于CMS并发清理阶段用户线程还在运行着，伴随程序运行自然就还会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS无法在当次收集中处理掉它们，只好留待下一次GC时再清理掉，本地无法清理的垃圾则称为浮动垃圾。

### G1 收集器
G1收集器是当前收集器技术发展最前沿的成果，一款面向服务端应用的垃圾收集器。基于标记-整理算法，也就是说不会产生内存碎片,可以精确控制停顿。基本不牺牲吞吐量的前提下完成低停顿的内存回收。这是由于它将新生代、老年代划分为多个区域，并维护一个每个区域收集的优先列表，保证了在有限的时间内可以获得最高的收集效率。收集的范围是整个JAVA堆。而不是在区分新生代，老年代。

执行过程：
1. 初始标记（Initial Marking）
初始标记阶段仅仅只是标记一下GC Roots能直接关联到的对象，并且修改TAMS（Next Top at Mark Start）的值，让下一阶段用户程序并发运行时，能在正确可用的Region中创建新对象，这阶段需要停顿线程，但耗时很短。
2.  并发标记（Concurrent Marking）
并发标记阶段是从GC Root开始对堆中对象进行可达性分析，找出存活的对象，这阶段耗时较长，但可与用户程序并发执行。
3. 最终标记（Final Marking）
最终标记阶段是为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程Remembered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set中，这阶段需要停顿线程，但是可并行执行。
4.  筛选回收（Live Data Counting and Evacuation）
筛选回收阶段首先对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划，这个阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分Region，时间是用户可控制的，而且停顿用户线程将大幅提高收集效率。

**特点：**
- 并行与并发
可使用多个CPU来缩短Stop-The-World停顿的时间，部分其他收集器原本需要停顿Java线程执行的GC动作，G1收集器仍然可以通过并发的方式让Java程序继续执行。
- 分代收集
与其他收集器一样，分代概念在G1中依然得以保留。虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但它能够采用不同的方式去处理新创建的对象和已经存活了一段时间、熬过多次GC的旧对象以获取更好的收集效果。
- 空间整合
与CMS的“标记—清理”算法不同，G1从整体来看是基于“标记—整理”算法实现的收集器，从局部（两个Region之间）上来看是基于“复制”算法实现的，但无论如何，这两种算法都意味着G1运作期间不会产生内存空间碎片，收集后能提供规整的可用内存。这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次GC。
- 可预测的停顿
这是G1相对于CMS的另一大优势，降低停顿时间是G1和CMS共同的关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。

在G1之前的其他收集器进行收集的范围都是整个新生代或者老年代，而G1不再是这样。使用G1收集器时，Java堆的内存布局就与其他收集器有很大差别，它将整个Java堆划分为多个大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，它们都是一部分Region（不需要连续）的集合。

G1收集器之所以能建立可预测的停顿时间模型，是因为它可以有计划地避免在整个Java堆中进行全区域的垃圾收集。G1跟踪各个Region里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region（这也就是Garbage-First名称的来由）。这种使用Region划分内存空间以及有优先级的区域回收方式，保证了G1收集器在有限的时间内可以获取尽可能高的收集效率

## 几种收集器对比

### Serial收集器 VS ParNew收集器
ParNew收集器在单CPU的环境中绝对不会有比Serial收集器更好的效果，甚至由于存在线程交互的开销，该收集器在通过超线程技术实现的两个CPU的环境中都不能百分之百地保证可以超越Serial收集器。
然而，随着可以使用的CPU的数量的增加，它对于GC时系统资源的有效利用还是很有好处的。

### Parallel Scavenge收集器 VS CMS等收集器：
Parallel Scavenge收集器的特点是它的关注点与其他收集器不同，CMS等收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标则是达到一个可控制的吞吐量（Throughput）。
由于与吞吐量关系密切，Parallel Scavenge收集器也经常称为“吞吐量优先”收集器。

### Parallel Scavenge收集器 VS ParNew收集器：
Parallel Scavenge收集器与ParNew收集器的一个重要区别是它具有自适应调节策略。
> 
**GC自适应的调节策略（GC Ergonomics）:**
Parallel Scavenge收集器有一个参数-XX:+UseAdaptiveSizePolicy。当这个参数打开之后，就不需要手工指定新生代的大小、Eden与Survivor区的比例、晋升老年代对象年龄等细节参数了，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量，这种调节方式称为GC自适应的调节策略（GC Ergonomics）。

## 触发GC的类型
了解这些是为了解决实际问题，Java虚拟机会把每次触发GC的信息打印出来来帮助我们分析问题，所以掌握触发GC的类型是分析日志的基础。
> 
**GC_FOR_MALLOC** 表示是在堆上分配对象时内存不足触发的GC。
**GC_CONCURRENT** 当我们应用程序的堆内存达到一定量，或者可以理解为快要满的时候，系统会自动触发GC操作来释放内存。
**GC_EXPLICIT** 表示是应用程序调用System.gc、VMRuntime.gc接口或者收到SIGUSR1信号时触发的GC。
**GC_BEFORE_OOM** 表示是在准备抛OOM异常之前进行的最后努力而触发的GC。

## 垃圾回收的JVM配置
### 运行的垃圾回收器类型
|  配置  |    描述     |
|-------|------------|
|-XX:+UseSerialGC |	串行垃圾回收器|
|-XX:+UseParallelGC	| 并行垃圾回收器|
|-XX:+UseConcMarkSweepGC | 并发标记扫描垃圾回收器|
|-XX:ParallelCMSThreads= | 并发标记扫描垃圾回收器,**=为使用的线程数量**| 
| -XX:+UseG1GC | 	G1垃圾回收器 |

### GC的优化配置
|  配置	  |  描述  |
|---------|--------|
| -Xms | 初始化堆内存大小 |
| -Xmx | 堆内存最大值 |
| -Xmn | 新生代大小 |
| -XX:PermSize | 初始化永久代大小 |
| -XX:MaxPermSize | 永久代最大容量 |

**使用JVM GC参数的例子**
```java 
java -Xmx12m -Xms3m -Xmn1m -XX:PermSize=20m -XX:MaxPermSize=20m -XX:+UseSerialGC -jar java-application.jar
```
## 总结
内存回收与垃圾收集器在很多时候都是影响系统性能，并发能力的主要因素之一，虚拟机之所以提供多种不同的收集器及大量的参数调节，是因为只有根据具体实际需要，实现方式选择最优的收集方式才能保证最优的性能。没有固定收集器、参数组合，也没有最优的调优方法，虚拟机也没有什么必然的内存回收行为。因为直到现在为止还没有最好的收集器出现，更加没有万能的收集器。

新生代收集器：Serial、ParNew、Parallel Scavenge
老年代收集器：CMS、Serial Old、Parallel Old
收集器适用场景：
用户交互：ParNew、CMS
高吞吐量：Parallel Scavenge

![gc-collects](/img/java-gc/java-gc-collects.jpg)

上面有7中收集器，分为两块，上面为新生代收集器，下面是老年代收集器。如果两个收集器之间存在连线，就说明它们可以搭配使用。

用文字描述收集器可配合使用结果如下：
Serial/Serial Old、Serial/CMS、ParNew/Serial Old、ParNew/CMS、Parallel Scavenge/Parallel Old、Parallel Scavenge/Serial Old、CMS/Serial Old

## 附录：
### 垃圾收集器参数总结
```
-XX:+<option> 启用选项
-XX:-<option> 不启用选项
-XX:<option>=<number> 
-XX:<option>=<string>
```

| 参数 | 描述 |
|------|------|
|-XX:+UseSerialGC | Jvm运行在Client模式下的默认值，打开此开关后，使用Serial + Serial Old的收集器组合进行内存回收 |
|-XX:+UseParNewGC |	打开此开关后，使用ParNew + Serial Old的收集器进行垃圾回收 |
|-XX:+UseConcMarkSweepGC | 使用ParNew + CMS +  Serial Old的收集器组合进行内存回收，Serial Old作为CMS出现“Concurrent Mode Failure”失败后的后备收集器使用。|
|-XX:+UseParallelGC |	Jvm运行在Server模式下的默认值，打开此开关后，使用Parallel Scavenge +  Serial Old的收集器组合进行回收 |
|-XX:+UseParallelOldGC | 使用Parallel Scavenge +  Parallel Old的收集器组合进行回收 |
| -XX:SurvivorRatio |	新生代中Eden区域与Survivor区域的容量比值，默认为8，代表Eden:Subrvivor = 8:1 |
| -XX:PretenureSizeThreshold |	直接晋升到老年代对象的大小，设置这个参数后，大于这个参数的对象将直接在老年代分配 |
| -XX:MaxTenuringThreshold |	晋升到老年代的对象年龄，每次Minor GC之后，年龄就加1，当超过这个参数的值时进入老年代|
|-XX:UseAdaptiveSizePolicy |	动态调整java堆中各个区域的大小以及进入老年代的年龄 |
| -XX:+HandlePromotionFailure |	是否允许新生代收集担保，进行一次minor gc后, 另一块Survivor空间不足时，将直接会在老年代中保留|
|-XX:ParallelGCThreads | 设置并行GC进行内存回收的线程数 |
|-XX:GCTimeRatio |	GC时间占总时间的比列，默认值为99，即允许1%的GC时间，仅在使用Parallel Scavenge 收集器时有效 |
| -XX:MaxGCPauseMillis | 设置GC的最大停顿时间，在Parallel Scavenge 收集器下有效 |
| -XX:CMSInitiatingOccupancyFraction |	设置CMS收集器在老年代空间被使用多少后出发垃圾收集，默认值为68%，仅在CMS收集器时有效，-XX:CMSInitiatingOccupancyFraction=70 |
| -XX:+UseCMSCompactAtFullCollection | 由于CMS收集器会产生碎片，此参数设置在垃圾收集器后是否需要一次内存碎片整理过程，仅在CMS收集器时有效 |
| -XX:+CMSFullGCBeforeCompaction | 设置CMS收集器在进行若干次垃圾收集后再进行一次内存碎片整理过程，通常与UseCMSCompactAtFullCollection参数一起使用 |
| -XX:+UseFastAccessorMethods | 原始类型优化 |
| -XX:+DisableExplicitGC | 是否关闭手动System.gc |
| -XX:+CMSParallelRemarkEnabled | 降低标记停顿 |
| -XX:LargePageSizeInBytes | 内存页的大小不可设置过大，会影响Perm的大小，-XX:LargePageSizeInBytes=128m |

**Sun/oracle JDK GC组合方式**

|  ~  | 新生代GC方式 | 老年代和持久代GC方式 |
|-----|--------------|----------------------|
| -XX:+UseSerialGC | Serial 串行GC |	Serial Old 串行GC |
| -XX:+UseParallelGC | Parallel Scavenge 并行回收GC	| Serial Old  并行GC | 
| -XX:+UseConcMarkSweepGC |	ParNew 并行GC |	CMS并发GC, 当出现“Concurrent Mode Failure”时采用Serial Old 串行GC |
| -XX:+UseParNewGC | ParNew 并行GC | Serial Old 串行GC |
| -XX:+UseParallelOldGC | Parallel Scavenge 并行回收GC |	Parallel Old 并行GC |
| -XX:+UseConcMarkSweepGC -XX:+UseParNewGC | Serial 串行GC	| CMS 并发GC 当出现“Concurrent Mode Failure”时采用Serial Old 串行GC |

> 
参考文章：
[理解Java垃圾回收机制(1)](http://www.jayfeng.com/2016/03/11/%E7%90%86%E8%A7%A3Java%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6/)
[理解Java垃圾回收机制(2)](http://blog.jobbole.com/80499/)
[深入理解JVM : Java垃圾收集器](http://www.jianshu.com/p/50d5c88b272d)