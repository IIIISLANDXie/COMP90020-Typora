# L1 Time and Global States

#### Introduction

为什么需要时钟和全局状态？

- 每个计算机有自己不同的时钟，导致进程记录事件的时间戳可能会混乱
- 即使是在一开始就设置好了相同的时钟，clock drift（时钟漂移）总是会出现（可能因为温度等因素）



#### Assumption

分布式系统中有N个进程，每个进程都有自己的物理时钟，并且没有shared memory

进程间只能通过message通讯

事件间（events）可以被排序（e → e' means e happened before e'）



#### **Monotonicity** condition 单调性

- A clock always advances: $t'>t=>C(t')>C(t)$
- A clock does <span style='color:red'>not obey</span> the monotonicity condition and/or the bounds on its drift is called <span style='color:red'>faulty clock</span>

- **Faulty clock**

  - If a clock totally <span style='color:red'>stops running</span>, it has a <span style='color:red'>crash failure</span>
  - Otherwise, it has an <span style='color:red'>arbitrary failure</span>

  - A correct clock is not necessarily an accurate clock!



#### Synchronization

- **External Synchronization**

  - 外部同步：系统和世界时钟同步

    S表示UTC的source时钟，C表示本地时钟，如果$|S(t) - C_{i}(t)| < D$  for i = 1,2,...,N，那么称其在D bound中是精确的

- **Internal Synchronization**

  - 内部同步：系统内部进程间时钟同步

    C表示本地时钟，如果$|C_i(t) - C_j(t)| < D$  for i = 1,2,...,N，那么称其在D bound中是精确的

- **Synchronous distributed system**
  - 执行过程的每个步骤的事件已知其上限和下限
  - 在已知的有限时间内接收到通过通道传递的每条信息
  - 每个过程都有一个本地时钟，其实时漂移速率有已知界限



#### Cristian’s Method

- **适用范围**

  - 应用于<span style='color:red'>没有全局时钟</span>或<span style='color:red'>进程时钟之间没有严格同步</span>的<span style='color:red'>异步系统</span>，通过估计往返时间来调整时钟

- **限制条件**

  - A single time server might fail 可能会出现单点失败，need to use of a group of synchronized servers 需要群组同步服务器
  - It does not deal with faulty servers 不涉及故障服务器

- **基本流程**

  - 时间服务器S从UTC源接收信号
  - 进程p发送请求消息$m_r$给时间服务器S，请求当前时间
  - 服务器S在消息$m_t$中回复当前时间t给进程p
  - 进程p设置它的时钟为$t+T_{round}/2$，其中$T_{round}$是消息往返的总时间

- **精度计算**

  - **The earliest time** 

    进程p发送$m_r$之后的最小延迟时间min

  - **The latest time**

    服务器S发送$m_t$之后的$T_{round}-min$

  - **时间范围**

    当消息$m_t$到达进程p时，服务器S的时间范围时$[t + min, t + T_{round} - min]$

  - **精确度**

    $±(T_{round}/2 – min)$

<img src="https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p1.png" style="zoom:45%;" />



#### Berkeley Algorithm

- **适用范围**
  - 用于一组计算机的内部同步
- **算法规则**
  - master轮询其他slaves的时钟
  - master利用round trip times（往返时间）估算slaves的时钟值
  - 算法取平均值（剔除超出平均往返时间的时钟或故障时钟）
  - 算法向slaves发送需要的调整
    - 为什么不直接更新时钟？因为要保证时钟的monotonicity condition



#### Network Time Protocol 网络事件协议

- **目的**
  - 用于同步clients之间的时钟到UTC
- **用法**
  - Primary server连接UTC源
  - Secondary server和primary server同步，以此类推
  - 用户电脑是同步过程中最低级的server
- **Failure 故障处理**
  - 同步子网络可以reconfigure（重新配置）
  - primary失去UTC源，它会变成secondary
  - secondary失去primary，它可以使用其他primary

- **实现方法**
  - Multicast
    - 利用high speed LAN（高传输速度）的机器多播它的时间给其他用户
    - not very accurate
  - Procedure call
    - 接受其他电脑的请求（like Cristian's algorithm）
    - higher accuracy
  - Symmetric
    - 一对服务器互相交换时间信息
    - <span style='color:red'>need </span>very high accuracies
    - message之间有non-negligible delay（不可忽视的延迟）
- **peers的信息交换**
  - use UDP
  - 每个信息都带有近期事件的时间戳
    - 上一个消息的发送和接受的本地时间
    - 当前消息发送的本地时间
  - 接收方记录时间$T_{i}$
- **准确性**
  - o代表offset（偏移量），d代表delay（延迟），filter pairs $<o_i,d_i>$，用于评估peers的可靠性
  - higher level更倾向被选择，因为它们靠近UTC（意味着延迟更小）
  - o越小越好，d越小越好

<img src="https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p2.png" style="zoom:20%;" />



#### Logical Time & Clocks (Lamport 1978)

- 产生原因
  - 我们并不需要absolute order
  - causality relationship between events has to be preserved
- **happened-before**，表达为$\rightarrow$
  - a $\rightarrow$ b means a happened before b
  - a $\rightarrow_p$ b means a happened before b within a process p
  - **Definition**
    - HB1: If $\exist$ process p: a $\rightarrow_p$ b, then a $\rightarrow$ b
    - HB2: For any message m: a = send(m) $\rightarrow$ b = receive(m)
    - HB3: If a $\rightarrow$ b and b $\rightarrow$ c, then a $\rightarrow$​ c
    - 先发生的事件不一定是代表会后发生事件的cause
    - $\rightarrow$ is a partial order, it cannot use in different processes which do not exchange messages
      - a $\rightarrow$ b（at $p_1$)，c $\rightarrow$ d（at $p_2$)
      - b $\rightarrow$ c because of $m_1$, a and e are concurrent，write as a $||$ e
      - <img src="https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p3.png" style="zoom:45%;" />
  - **Calculation**
    - $L_i$ is used in logical timestamps and incremented by 1 before each event
    - Each process has its logical clock initialized to zero
    - $e\rightarrow e'$ implies $L(e) \textless L(e') $, but $L(e) \textless L(e') $ not implies $e\rightarrow e'$
    - Above example is
      - p1: L(a) = 1, L(b) = 2
      - p2: L(c) = 3, L(d) = 4
      - p3: L(e) = 1, L(f) = 5
- **Vector Clock**
  - vector timestamps are used to timestamp local events
  - 对每一个event使用向量来记录它发生的时间戳
  - **性质**
    - $V = V’$ iff $V [j] = V’ [j]$ for $j = 1, 2, ..., N$
    - $V <= V’$ iff $V [j] \leq V’ [j]$ for $j = 1, 2, ..., N$
    - $V < V’$ iff $V \leq V’$ and $V != V’$
    - $e \rightarrow e’$ iff $V(e) < V(e’)$（注意这里是可以互逆的，因为vector完全限制了event的顺序）
    - <img src="https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p4.png" style="zoom:45%;" />
- Vector clock是对Lamport clock的改进



#### Global State

- **Aim**
  - Determine whether a <span style='color:red'>particular property</span> is <span style='color:red'>true</span> of a DS as it executes（评估分布式系统在运行时是否满足某个<span style='color:red'>特定属性</span>）
    - 特定性质是指需要验证的一些系统行为或状态，例如
      - Consistancy（一致性）
      - Deadlock（死锁）
      - Reachablity（资源可达性）
  - Use logical time to construct a global view of the system state（使用逻辑时间表达因果关系，从而构建系统状态的全局视图）
- **用于解决**
  - **Distributed garbage collection**
    - 是否在系统的任何地方references（引用）了某个对象？
    - 检查该过程的目标是检查系统中的对象是否还在被引用，从而决定是否回收
    - 未被引用的成为垃圾
    - 引用可能存在于local process，another process，communication channel
    - <img src="https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p5.png" style="zoom:50%;" />
  - **Deadlock detection 死锁检测**
    - exist "wait for" relation between processes
    - 全局状态提供了所有进程和资源的当前状态，使得我们能够检测到进程间的循环等待，从而识别死锁。
    - <img src="https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p6.png" style="zoom:50%;" />
  - **Termination detection 终止检测**
    - 判断分布式系统中的所有进程是否都已经完成其任务并进入终止状态
    - <img src="https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p7.png" style="zoom:50%;" />
  - **Debugging 故障检修**
    - 发现和修复错误的过程
- **Cut** 割
  - Cut assemble（集合）a meaningful global state from local states recorded at different times
  - Cut 是系统全局历史的一个子集，是包括从系统启动到某个时间节点的所有进程的状态
  - Frontier of a cut
    - 切片的frontier指每个进程中的到cut为止最后一个状态（或者说是进程的最新状态）
  - Consistent cut
    - 一致性cut指对于cut中的如果事件e在cut里，并且$e'\rightarrow e$，那么e'也必须在cut里（有果必有因）（但是有因可以没有果）
    - <img src="https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p8.png" style="zoom:50%;" />



- **Consistent Global State**
  - 根据consistent cut得到的global state
  - A DS evolves through consistent global states
- **Run**
  - 对于事件的发生顺序来说，全局历史总排序和和进程内本地历史排序是一致的（全局中可能是abcde顺序，但是在进程中是ac、bed，先后顺序不变）
- **Linearization (consistent run)**
  - 根据happened-before对全局历史进行排序（并发的事件无需排序或任意排序）
- **Reachable state**
  - A state S' is reachable from another state S if there is a linearization through these states



#### SnapShot 快照

- **Motivation**
  - a snapshot return a <span style='color:red'>configuration </span>of an execution in the same computation（返回相同计算中的配置）
- **用途**
  - Restarting after a failure（失败重启）
  - Off-line determination of stable properties（分析稳定性）
  - Debugging
  - Evaluate stable global predicates（用于评估全局谓词的稳定性）
- **挑战**
  - 在不停止系统的情况下进行snapshot
  - 收集本地snapshot
- **目的**
  - Store information locally

- **获取snapshot**
  - 需要区分basic message（基本信息：进程间通信的信息）和control message（控制信息：用于协调快照开始结束的信息）
- **Assumption**
  - Neither channels nor processes fail（通讯可靠）
  - Channels are unidirectional and messages arrive in order（FIFO)
  - There is a path between ant two processes（进程间都有通道）
  - Any process can initiate the snapshot algorithm（任何一个进程都可以开始快照算法）
  - The processes can continue to work while the snapshot takes place（进程进行间可以快照）
  - A process that has received a marker message records its state within a finite time（进程收到标记信息后可以在有限时间内记录自己state）
  - A process sends marker messages over each outgoing channel with in a finite time（进程收到标记信息后可以在有限时间内发出标记信息）
  - 如果进程$p_i$和进程$p_j$之间有通道，那么$p_i$记录自己state后的有限时间内$p_j$会记录自己state
- **snapshot的内容**
  - Each process should record any messages that 
    - arrived after it recorded its state（进程开始记录之后到达的信息）
    - before the sender recorded its own state（发送者记录其状态之前的信息）
- **启动方式**
  - use special marker message（特殊标记信息）
  - 进程启动的时候假装自己收到了一个marker message
- **算法规则**
  - **接收规则**
    - 当进程p通过通信管道c收到一个marker message
      - 如果是<span style='color:red'>第一次</span>收到marker message则
        - 记录自己的state
        - 记录<span style='color:red'>c</span>为<span style='color:red'>empty set</span>
        - 记录其他所有<span style='color:red'>到达管道</span>的信息
      - 如果不是第一次收到marker message则
        - 进程记录<span style='color:red'>自从上次记录自己state后从c收到的所有信息集合</span>
  - **发送规则**
    - 进程p<span style='color:red'>记录自己state之后</span>，对于<span style='color:red'>每一个发送管道c</span>，在<span style='color:red'>发送任何信息之前</span>，进程<span style='color:red'>先发送一个marker message</span>（进程若再次收到marker也不会再发送）

- **终止条件**
  - 每一个进程在其所有通道上都收到了标记后终止
- **算法定义**
  - 对于发起者来说，快照会记录本地状态和从发送marker之后一直到所有信道收到marker为止，收到的消息历史
  - 对于非发起者来说，快照会记录本地状态和从第一次收到marker之后一直到所有信道收到marker为止，收到的消息历史
  - 对于所有被进程发送的非marker消息，都不在本次快照内，因为它们影响的是下一次快照的本地状态
- **Correctness and Complexity**
  - **Complexity**
    - Let e be the number of edge（e是边数），d the diameter of network（d是网络直径），recording a single instance require $O(e)$ messages and $O(d)$ time
  - **Correctness**
    - **FINO**: NO message sent after the marker on that channel is recorded in the channel state(通道状态中不会记录在该通道标记后发送的任何信息)
    - 如果进程收到了一个message在marker之前，
      - 如果进程尚未拍摄快照，则把message记录在快照中
      - 否则，则把message记录在channel里
- **Snapshot taken by the <span style='color:red'>Chandy-Lamport Algorithm</span> correspond to <span style='color:red'>consistent global states</span>**

- **Consistent snapshot**
  - **presnapshot event**: Occurs at a process before the local snapshot at this process is taken（发生在某进程中，但该进程的本地快照尚未拍摄）
  - **postsnapshot event**: Occurs at a process after the local snapshot at this process is taken（该进程的本地快照拍摄完成后发生在该进程中）
  - 对于每一个presnapshot event e，所有的因果上在e之前的事件都应该是presnapshot
  - 如果有一个basic message被包含在信道中，那么只有可能是：相关发送事件是presnapshot，相关接收事件是postsnapshot
- **Reachability**
  - **Definition**
    - Let $S_{init}$ be the global state immediately before the first process recorded its state
    - Let $S_{final}$ be the global state when the snapshot terminates
    - Let $S_{snap}$ be the recorded global state
    - Let Sys = e0, e1, ..., en be the linearization of a system execution. Then there is a permutation（排列组合） Sys' = e0', e1', ..., en' of Sys such that $S_{init}$, $S_{snap}$, $S_{final}$ occur in Sys'
    - $S_{snap}$ is reachable from $S_{init}$ in Sys', and $S_{final}$ is reachable from $S_{snap}$ is Sys'
  - **Property**
    - Useful for detecting stable predicates
    - If a stable predicate is TRUE in $S_{snap}$, then is TRUE in $S_{final}$（因为该state只要是true，那么它到达的任何state都是true）
    - If a stable predicate is FALSE in $S_{snap}$, then is FALSE in $S_{init}$
- **Example**
  - 对于下面的图例来说，
  - 首先p1决定发起snapshot，记录自身状态<span style='color:red'>p1<\$1000, 0></span>，开始接收所有接收信道的消息
  - p1发送了订单(Order 10, $100)和marker message M，由于订单先于M，根据FIFO，p2会先收到order，后收到marker
  - p2处理完成5weights的订单，处理marker message，记录自身状态为<span style='color:red'>p2<\$50, 1995></span>，开始记录所有接收信道消息
  - p1从c1收到<span style='color:red'>c1\<five widgets></span>
  - 结束算法，c2没有信息<span style='color:red'>c2<></span>


![](https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p9.png)

#### Distributed Debugging

- A distributed system record its states（记录自己状态的分布式系统）
- Sent the states to an external server later（把状态发送到外部服务器）
- The external server then assembles global consistent states（外部服务器总结全局一致状态）

- Aim
  - 决定给定的predicate是否是definitely true at some point或者是使其possibly true的cases
  - Example
    - 随便来俩变量xy的差是非零
    - 今天下雨

- Chandy-Lamport snapshot algorithm
  - Best case: prove violation of these properties（证明这些性质违反的情况）



#### Possibly and Definitely

- Possibly $\varphi$ 
  - 定义：存在一个一致的全局状态，通过该状态的线性化，使得谓词 $\varphi$ 为真
  - 解释：如果在某个特定的时间点，系统状态可以排列成一个顺序，使得 $\varphi$ 成立，那么我们说 $\varphi$ 可能为真
- Definitely $\varphi$ 
  - 定义：对于所有线性化L，存在一个一致的全局状态，通过该状态的线性化使得 $\varphi$ 为真
  - 解释：任何时候、无论系统怎样排列，只要系统处于某个一致状态，这个谓词 $\varphi$ 都必须成立，那么我们说 $\varphi$ 必然为真
- 对于Chandy-Lamport算法来说，$\varphi (S_{snap})$ 可能为真，pos $\varphi$ 
- Inference
  - $\neg pos \varphi => def \neg \varphi$ （没有可能真意味着没有任何情况会真，推断出一定不真）
  - 反之不成立



#### Monitoring Algorithm（Marzullo-Neiger）

- Centralized Algorithm

  - One external observer (monitor M) connects to all processes and receives periodic state messages（包含它们本地state）
  - Monitor不干扰系统计算
  - Monitor从收到的message中总结出全局一致状态

- State collection

  - process send <span style='color:red'>initial state</span> to monitor
  - process send their local state when
    - local state changes a portion of global state, which affect the evaluation of $\varphi $（本地状态改变了全局变量的一部分，影响到 $\varphi$ 的计算）
    - local state change causes $\varphi$ to change its value（本地状态改变了 $\varphi$ 的值）

- 为了让monitor推断出一致性，必须maintain vector clocks

- 每个message中都包含vector clock value

- 定义 $V(s_i)$ 是从process i 收到的vector time stamp

  - $V(s_i)[i] \geq V(s_k)[i]$ $ \forall i,k$ 
    - 对于P1(x1,y1)，P2(x2,y2)来说，x1 $\geq$ x2，y1 $\leq$ y2
  - P2已知P1的事件数量 $\leq$ P1发送这个信息时的数量
  - Example
    - 图中x1和x2来说，x1大于等于x2不成立，y1小于等于y2成立，所以cut c1是inconsistent并且不构成违反$\varphi$
    - cut c2 is consistent
  - <img src="https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p10.png" style="zoom:50%;" />

- Global State for Execution

  - 计算 possibly $\varphi$
    - 初始化所有初始状态进入states集合
    - 如果所有states都是FALSE，则增加一层L，并加入states可到达的新状态states‘，赋值给states
    - 当找到一个S使得$\varphi$为真，输出"possibly $\varphi$"
  - 计算definitely $\varphi$
    - 如果初始时全部状态为真，直接输出"definitely $\varphi$"
    - 如果不是，增加L，并加入states可到达的新状态states‘，赋值给state，保留使得$\varphi$为FALSE的状态
    - 如果states为空，则输出"definitely $\varphi$"
  - <img src="https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p11.png" style="zoom:50%;" />
  - Evaluating definitely $\varphi$
  - <img src="https://raw.githubusercontent.com/IIIISLANDXie/COMP90020-Typora/main/L1/l1p12.png" style="zoom:50%;" />
  - Cost
    - Time complexity
      - N processes（N进程数）
      - k is the max number of messages per process（k表示每个进程的最大信息数）
        - Time cost $O(k^N)$
        - Space cost $O(kN)$

  - 在异步系统中
    - monitoring的观察可能不能透过全局
    - 全局状态中的任何两个进程状态都可能在任意时间段内发生过
  - 在同步系统中
    - 除了使用逻辑网络时钟外，还在同步网络中使用物理时钟，以限制需要考虑的状态数量
    - 在时钟同步的已知范围内，监控器只考虑可能同时发生的本地状态集

















