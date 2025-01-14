---
sort: 3
---
# 网线到网络设备

<figure>
    <img src="./images/network.png" width=800>
</figure>

前面探索了客户端中的协议栈和网卡，介绍了发送网络包，也就是将网络包转换成电信号通过网线传输出去的过程。

这部分关注信号发出去以后经过的网络设备，即集线器、交换机、路由器。

- 信号在网线和集线器中传输
- 交换机的包转发操作
- 路由器的包转发操作


## 网线与集线器


### 每个包都是独立传输的

从计算机发送出来的网络包会通过集线器、路由器等设备被转发，最终到达目的地。转发设备会根据包头部中的控制信息，在转发设备内部一个写有转发规则的表中进行查询，以此来判断包的目的地，然后将包朝目的地的方向进行转发。快递员在送快递时只看快递单，不看里面的内容，同样地，转发设备在进行转发时也不看数据的内容。因此，无论包里面装的是应用程序的数据或者是 TCP 协议的控制信息，都不会对包的传输操作本身产生影响。换句话说，HTTP 请求的方法，TCP 的确认响应和序号，客户端和服务器之间的关系，这一切都与包的传输无关。因此，所有的包在传输到目的地的过程中都是独立的，相互之间没有任何关联。

**每个包都是独立传输的**

有了这个概念之后，这部分探索一下网络包在进入互联网之前经历的传输过程。这里假设客户端计算机连接的局域网结构是像下图这样的。

<figure>
    <img src="./images/局域网结构.png" width=500>
</figure>

也就是说，网络包从客户端计算机发出之后，要经过集线器、交换机和路由器最终进入互联网。实际上，我们家里用的路由器已经集成了集线器和交换机的功能，像图上这样使用独立设备的情况很少见。不过，把每个功能独立出来更容易理解，而且理解了这种模式之后，也就能理解集成了多种功能的设备了，因此我们这里将所有功能独立出来，逐个来进行探索。

### 防止网线中的信号衰减很重要

本章的探索从信号流出网卡进入网线开始。网卡中的 PHY（MAU）模块负责将包转换成电信号，信号通过 RJ-45 接口进入双绞线，这部分的放大图如下图右侧部分所示。

<figure>
    <img src="./images/PHY.png" width=900>
    <figcaption>网卡与集线器连接</figcaption>
</figure>

以太网信号的本质是正负变化的电压，可以认为网卡的 PHY（MAU）模块就是一个从正负两个信号端子输出信号的电路。PHY（MAU）模块直接连接上图右侧中的 RJ-45 接口，信号从这个接口中的 1 号和 2 号针脚流入网线。然后，信号会通过网线到达集线器的接口，这个过程就是单纯地传输电信号而已。

但是，信号到达集线器的时候并不是跟刚发送出去的时候一模一样。集线器收到的信号有时会出现衰减。信号在网线的传输过程中，能量会逐渐损失。网线越长，信号衰减就越严重。

<figure>
    <img src="./images/信号衰减.png" width=460>
</figure>

而且，信号损失能量并非只是变弱而已。以太网中的信号波形是方形的，但损失能量会让信号的拐角变圆，这是因为电信号的频率越高，能量的损失率越大。信号的拐角意味着电压发生剧烈的变化，而剧烈的变化意味着这个部分的信号频率很高。高频信号更容易损失能量，因此本来剧烈变化的部分就会变成缓慢的变化，拐角也就变圆了。

即便线路条件很好，没有噪声，信号在传输过程中依然会发生失真，如果再加上噪声的影响，失真就会更厉害。噪声根据强度和类型会产生不同的影响，无法一概而论，但如果本来就已经衰减的信号再进一步失真，就会出现对 0 和 1 的误判，这就是产生通信错误的原因。

### 双绞是为了抑制噪声

局域网网线使用的是双绞线，其中“双绞”的意思就是以两根信号线为一组缠绕在一起，这种拧麻花一样的设计是为了抑制噪声的影响。

那么双绞线为什么能够抑制噪声呢？首先，我们来看看噪声是如何产生的。产生噪声的原因是网线周围的电磁波，当电磁波接触到金属等导体时，在其中就会产生电流。因此，如果网线周围存在电磁波，就会在网线中产生和原本的信号不同的电流。由于信号本身也是一种带有电压变化的电流，其本质和噪声产生的电流是一样的，所以信号和噪声的电流就会混杂在一起，导致信号的波形发生失真，这就是噪声的影响。

影响网线的电磁波分为两种。一种是由电机、荧光灯、CRT 显示器等设备泄漏出来的电磁波，这种电磁波来自网线之外的其他设备，我们来看看双绞线如何抑制这种电磁波的影响。首先，信号线是用金属做成的，当电磁波接触到信号线时，会沿电磁波传播的右旋方向产生电流，这种电流会导致波形发生失真。如果我们将信号线缠绕在一起，信号线就变成了螺旋形，其中两根信号线中产生的噪声电流方向就会相反，从而使得噪声电流相互抵消，噪声就得到了抑制（图（a））。当然，即便信号线变成螺旋形，里面的信号依然可以原样传输，也就是说，信号没有变，只是噪声被削弱了。

另一种电磁波是从网线中相邻的信号线泄漏出来的。由于传输的信号本身就是一种电流，当电流流过时就会向周围发出电磁波，这些电磁波对于其他信号线来说就成了噪声。这种内部产生的噪声称为串扰crosstalk）。

这种噪声的强度其实并不高，但问题是噪声源的距离太近了。距离发生源越远，电磁波就会因扩散而变得越弱，但在同一根网线中的信号线之间距离很近，这些电磁波还没怎么衰减就已经接触到了相邻的信号线。因此，尽管信号线产生的电磁波十分微弱，也能够在相邻的信号线中产生感应电流。

要抑制这种噪声，关键在于双绞线的缠绕方式。在一根网线中，每一对信号线的扭绞间隔（节距）都有一定的差异，这使得在某些地方正信号线距离近，另一些地方则是负信号线距离近。由于正负信号线产生的噪声影响是相反的，所以两者就会相互抵消（图（b））。从网线整体来看，正负的分布保持平衡，自然就会削弱噪声的影响。

<figure>
    <img src="./images/双绞线抑制噪声.png" width=530>
</figure>

通过将信号线缠绕在一起的方式，噪声得到了抑制，从结果来看提升了网线的性能，除此之外还有其他一些工艺也能够帮助提升性能。例如在信号线之间加入隔板保持距离，以及在外面包裹可阻挡电磁波的金属屏蔽网等。有了这些工艺的帮助，我们现在可以买到性能指标不同的各种网线。网线的性能是以“类”来区分的，现在市售双绞线的主要种类如表所示。

<figure>
    <img src="./images/网线种类.png" width=460>
</figure>

### 集线器将信号发往所有线路

当信号到达集线器后，会被广播到整个网络中。以太网的基本架构就是将包发到所有的设备，然后由设备根据接收方 MAC 地址来判断应该接收哪些包，而集线器就是这一架构的忠实体现，它就是负责按照以太网的基本架构将信号广播出去。下面来看看它的工作方式。

集线器的内部结构如图左侧部分所示。

<figure>
    <img src="./images/PHY.png" width=900>
    <figcaption>网卡与集线器连接</figcaption>
</figure>

首先，在每个接口的后面装有和网卡中的 PHY（MAU）功能相同的模块，但如果它们像网卡端一样
采用直连式接线，是无法正常接收信号的。要正常接收信号，必须将“发送线路”和“接收线路”连接起来才行。在图中，集线器中的 PHY（MAU）模块与接口之间采用交叉接线的原因正是在于此。

集线器的接口中有一个 MDI/MDI-X 切换开关，现在你应该知道它是干什么用的了吧？ MDI 就是对 RJ-45 接口和信号收发模块进行直连接线，而 MDI-X 则是交叉接线。由于集线器的接口一般都是 MDI-X 模式，要将两台集线器相连时，就需要将其中一台改成 MDI 模式（图 （a））。

<figure>
    <img src="./images/交叉线的使用.png">
</figure>

如果集线器上没有 MDI 切换开关，而且所有的接口又都是 MDI-X 时，可以用交叉网线连接两台集线器。所谓交叉网线，就是一种将发送和接收信号线反过来接的网线（图 3.6）。

<figure>
    <img src="./images/交叉线.png">
</figure>

此外，交叉网线也可以像图（b）这样用于将两台计算机直接连接起来。网卡不仅可以连接集线器，因为网卡的 PHY（MAU）模块和集线器都是一样的，所以两台计算机的网卡也可以相互连接，只要将一侧的发送信
号线和另一侧的接收信号线连起来就可以收发数据了。

信号到达集线器的 PHY（MAU）模块后，会进入中继电路。中继电路的基本功能就是将输入的信号广播到集线器的所有端口上。当然，也有一些产品具有信号整形、错误抑制等功能，但基本上就是将输入的信号原封不动地输出到网线接口。

接下来，信号从所有接口流出，到达连接在集线器上的所有设备。然后，这些设备在收到信号之后会通过 MAC 头部中的接收方 MAC 地址判断是不是发给自己的，如果是发给自己的就接受，否则就忽略。这样，网络包就能够到达指定 MAC 地址的接收方了。

由于集线器只是原封不动地将信号广播出去，所以即便信号受到噪声的干扰发生了失真，也会原样发送到目的地。这时，接收信号的设备，也就是交换机、路由器、服务器等，会在将信号转换成数字信息后通过 FCS 校验发现错误，并将出错的包丢弃。当然，丢弃包并不会影响数据的传输，因为丢弃的包不会触发确认响应。因此协议栈的 TCP 模块会检测到丢包，并对该包进行重传。


## 交换机

### 交换机根据地址表进行转发

下面来看一下包是如何通过交换机的。交换机的设计是将网络包原样转发到目的地，内部结构图如下

<figure>
    <img src="./images/交换机结构.png" width=490>
</figure>

首先，信号到达网线接口，并由 PHY（MAU）模块进行接收，这一部
分和集线器是相同的。也就是说，它的接口和 PHY（MAU）模块也是以
MDI-X 模式进行连接的，当信号从双绞线传入时，就会进入 PHY（MAU）
模块的接收部分。

接下来，PHY（MAU）模块会将网线中的信号转换为通用格式，然后传递给 MAC 模块。MAC 模块将信号转换为数字信息，然后通过包末尾的 FCS 校验错误，如果没有问题则存放到缓冲区中。这部分操作和网卡基本相同，大家可以认为交换机的每个网线接口后面都是一块网卡。网线接口和后面的电路部分加在一起称为一个端口，也就是说交换机的一个端口就相当于计算机上的一块网卡。但交换机的工作方式和网卡有一点不同。网卡本身具有 MAC 地址，并通过核对收到的包的接收方 MAC 地址判断是不是发给自己的，如果不是发给自己的则丢弃；相对地，交换机的端口不核对接收方 MAC 地址，而是直接接收所有的包并存放到缓冲区中。因此，和网卡不同，交换机的端口不具有 MAC 地址。

将包存入缓冲区后，接下来需要查询一下这个包的接收方 MAC 地址是否已经在 MAC 地址表中有记录了。MAC 地址表主要包含两个信息，一个是设备的 MAC 地址，另一个是该设备连接在交换机的哪个端口上。以图中的地址表为例，MAC 地址和端口是一一对应的，通过这张表就能够判断出收到的包应该转发到哪个端口。举个例子，如果收到的包的接收方 MAC 地址为 00-02-B3-1C-9C-F9，则与表中的第 3 行匹配，根据端口列的信息，可知这个地址位于 8 号端口上，然后就可以通过交换电路将包发送到相应的端口了。

现在来看看交换电路到底是如何工作的。交换电路的结构如图所示，它可以将输入端和输出端连接起来。

<figure>
    <img src="./images/交换电路.png" width=500>
</figure>

其中，信号线排列成网格状，每一个交叉点都有一个交换开关，交换开关是电子控制的，通过切换开关的状态就可以改变信号的流向。交换电路的输入端和输出端分别连接各个接收端口和发送端口，网络包通过这个网格状的电路在端口之间流动。举个例子，假设现在要将包从 2 号端口发送到 7 号端口，那么信号会从输入端的 2 号线进入交换电路，这时，如果让左起的 6 个开关水平导通，然后将第 7 个开关切换为垂直导通，信号就会像图上一样流到输出端 7 号线路，于是网络包就被发送到了 7 号端口。每个交叉点上的交换开关都可以独立工作，因此只要路径不重复，就可以同时传输多路信号。

当网络包通过交换电路到达发送端口时，端口中的 MAC 模块和 PHY（MAU）模块会执行发送操作，将信号发送到网线中，这部分和网卡发送信号的过程是一样的。根据以太网的规则，首先应该确认没有其他设备在发送信号，也就是确认信号收发模块中的接收线路没有信号进来。如果检测到其他设备在发送信号，则需要等待信号发送完毕；如果没有其他信号，或者其他信号已经发送完毕，这时就可以将包的数字信息转换为电信号发送出去。在发送信号的过程中，还需要对接收信号进行监控，这一点和网卡也是一样的。如果在发送过程中检测到其他设备发送信号，就意味着出现了信号碰撞，这时需要发送阻塞信号以停止网络中所有的发送操作，等待一段时间后再尝试重新发送，这一步和网卡也是一样的。

### MAC地址表的维护

交换机在转发包的过程中，还需要对 MAC 地址表的内容进行维护，维护操作分为两种。

第一种是收到包时，将发送方 MAC 地址以及其输入端口的号码写入MAC 地址表中。由于收到包的那个端口就连接着发送这个包的设备，所以只要将这个包的发送方 MAC 地址写入地址表，以后当收到发往这个地址的包时，交换机就可以将它转发到正确的端口了。交换机每次收到包时都会执行这个操作，因此只要某个设备发送过网络包，它的 MAC 地址就会被记录到地址表中。

另一种是删除地址表中某条记录的操作，这是为了防止设备移动时产生问题。比如，我们在开会时会把笔记本电脑从办公桌拿到会议室，这时设备就发生了移动。从交换机的角度来看，就是本来连接在某个端口上的笔记本电脑消失了。这时如果交换机收到了发往这台已经消失的笔记本电脑的包，那么它依然会将包转发到原来的端口，通信就会出错，因此必须想办法删除那些过时的记录。然而，交换机没办法知道这台笔记本电脑已经从原来的端口移走了。因此地址表中的记录不能永久有效，而是要在一段时间不使用后就自动删除。

那么当笔记本电脑被拿到会议室之后，会议室里的交换机又会如何工作呢？只要笔记本电脑连接到会议室的交换机，交换机就会根据笔记本电脑发出的包来更新它的地址表。因此，对于目的地的交换机来说，不需要什么特别的措施就可以正常工作了。

综合来看，为了防止终端设备移动产生问题，只需要将一段时间不使用的过时记录从地址表中删除就可以了。

过时记录从地址表中删除的时间一般为几分钟，因此在过时记录被删除之前，依然可能有发给该设备的包到达交换机。这时，交换机会将包转发到老的端口，通信就会发生错误，这种情况尽管罕见，但的确也有可能发生。不过大家不必紧张，遇到这样的情况，只要重启一下交换机，地址表就会被清空并更新正确的信息，然后网络就又可以正常工作了。

总之，交换机会自行更新或删除地址表中的记录，不需要手动维护。当地址表的内容出现异常时，只要重启一下交换机就可以重置地址表，也不需要手动进行维护。


### 特殊操作

上面介绍了交换机的基本工作方式，下面来看一些特殊情况下的操作。比如，交换机查询地址表之后发现记录中的目标端口和这个包的源端口是同一个端口。当像下图这样用集线器和交换机连接在一起时就会遇到这样的情况。

<figure>
    <img src="./images/不向源端口转发网络包.png" width=380>
</figure>

那么这种情况要怎么处理呢？首先，计算机 A 发送的包到达集线器后会被集线器转发到所有端口上，也就是会到达交换机和计算机 B。这时，交换机转发这个包之后，这个包会原路返回集线器，然后，集线器又把包转发到所有端口，于是这个包又到达了计算机 A 和计算机 B。所以计算机 B 就会收到两个相同的包，这会导致无法正常通信。因此，当交换机发现一个包要发回到原端口时，就会直接丢弃这个包。

还有另外一种特殊情况，就是地址表中找不到指定的 MAC 地址。这可能是因为具有该地址的设备还没有向交换机发送过包，或者这个设备一段时间没有工作导致地址被从地址表中删除了。这种情况下，交换机无法判断应该把包转发到哪个端口，只能将包转发到除了源端口之外的所有端口上，无论该设备连接在哪个端口上都能收到这个包。这样做不会产生什么问题，因为以太网的设计本来就是将包发送到整个网络的，然后只有相应的接收者才接收包，而其他设备则会忽略这个包。

有人会说：“这样做会发送多余的包，会不会造成网络拥塞呢？”其实完全不用过于担心，因为发送了包之后目标设备会作出响应，只要返回了响应包，交换机就可以将它的地址写入地址表，下次也就不需要把包
发到所有端口了。局域网中每秒可以传输上千个包，多出一两个包并无大碍。此外，如果接收方 MAC 地址是一个广播地址，那么交换机会将包发送到除源端口之外的所有端口。

```note
广播地址（broadcast address）是一种特殊的地址，将广播地址设为接收方地址时，包会发送到网络中所有的设备。MAC 地址中的 FF:FF:FF:FF:FF:FF 和 IP 地址中的 255.255.255.255 都是广播地址。
```

### 全双工模式可以同时进行发送和接收

全双工模式是交换机特有的工作模式，它可以同时进行发送和接收操作，集线器不具备这样的特性。

使用集线器时，如果多台计算机同时发送信号，信号就会在集线器内部混杂在一起，进而无法使用，这种现象称为碰撞，是以太网的一个重要特征。不过，只要不用集线器，就不会发生碰撞。

而使用双绞线时，发送和接收的信号线是各自独立的，因此在双绞线中信号不会发生碰撞。网线连接的另一端，即交换机端口和网卡的 PHY（MAU）模块以及 MAC 模块，其内部发送和接收电路也是各自独立的，信号也不会发生碰撞。因此，只要不用集线器，就可以避免信号碰撞了。

如果不存在碰撞，也就不需要半双工模式中的碰撞处理机制了。也就是说，发送和接收可以同时进行。然而，以太网规范中规定了在网络中有信号时要等该信号结束后再发送信号，因此发送和接收还是无法同时进行。于是，人们对以太网规范进行了修订，增加了一个无论网络中有没有信号都可以发送信号的工作模式，同时规定在这一工作模式下停用碰撞检测。这种工作模式就是全双工模式。在全双工模式下，无需等待其他信号结束就可以发送信号，因此它比半双工模式速度要快。由于双方可以同时发送数据，所以可同时传输的数据量也更大，性能也就更高。

<figure>
    <img src="./images/全双工模式.png" width=400>
</figure>

### 自动协商：确定最优的传输速率

随着全双工模式的出现，如何在全双工和半双工模式之间进行切换的问题也产生了。在全双工模式刚刚出现的时候，还需要手动进行切换，但这样实在太麻烦，于是后来出现了自动切换工作模式的功能。这一功能可以由相互连接的双方探测对方是否支持全双工模式，并自动切换成相应的工作模式。此外，除了能自动切换工作模式之外，还能探测对方的传输速率并进行自动切换。这种自动切换的功能称为自动协商。

在以太网中，当没有数据在传输时，网络中会填充一种被称为连接脉冲的脉冲信号。在没有数据信号时就填充连接脉冲，这使得网络中一直都有一定的信号流过，从而能够检测对方是否在正常工作，或者说网线有没有正常连接。以太网设备的网线接口周围有一个绿色的 LED 指示灯，它表示是否检测到正常的脉冲信号。如果绿灯亮，说明 PHY（MAU）模块以及网线连接正常。

在双绞线以太网规范最初制定的时候，只规定了按一定间隔发送脉冲信号，这种信号只能用来确认网络是否正常。后来，人们又设计出了如图这样的具有特定排列的脉冲信号，通过这种信号可以将自身的状态告知对方。自动协商功能就利用了这样的脉冲信号，即通过这种信号将自己能够支持的工作模式和传输速率相互告知对方，并从中选择一个最优的组合。

<figure>
    <img src="./images/无数据时网络信号.png" width=450>
</figure>

看一个具体的例子。假设现在连接双方的情况如表 3.2 所示，网卡一方支持所有的速率和工作模式，而交换机只支持到 100 Mbit/s 全双工模式。当两台设备通电并完成硬件初始化之后，就会开始用脉冲信号发送自己支持的速率和工作模式。当对方收到信号之后，会通过读取脉冲信号的排列来判断对方支持的模式，然后看看双方都支持的模式有哪些。表是按照优先级排序的，因此双方都支持的模式就是第 3 行及以下的部分。越往上优先级越高，因此在本例中 100 Mbit/s 全双工模式就是最优组合，于是双方就会以这个模式开始工作。

<figure>
    <img src="./images/自动协商.png" width=500>
</figure>

### 交换机可同时执行多个转发操作

交换机只将包转发到具有特定 MAC 地址的设备连接的端口，其他端
口都是空闲的。如图

<figure>
    <img src="./images/交换机结构.png" width=490>
</figure>

当包从最上面的端口发送到最下面的端口时，其他端口都处于空闲状态，这些端口可以传输其他的包，因此交换机可以同时转发多个包。

相对地，集线器会将输入的信号广播到所有的端口，如果同时输入多个信号就会发生碰撞，无法同时传输多路信号，因此从设备整体的转发能力来看，交换机要高于集线器。


## 路由器

### 路由器的基本知识

<figure>
    <img src="./images/局域网结构.png" width=550>
</figure>

网络包经过集线器和交换机之后，现在到达了路由器，并在此被转发到下一个路由器。这一步转发的工作原理和交换机类似，也是通过查表判断包转发的目标。不过在具体的操作过程上，路由器和交换机是有区别的。因为路由器是基于 IP 设计的，而交换机是基于以太网设计的。IP 和以太网的区别在很多地方都会碰到，我们稍后再具体讲，现在先来看看路由器的概况。

首先，路由器的内部结构如图所示。这张图已经画得非常简略了，大家只要看明白路由器包括转发模块和端口模块两部分就可以了。

<figure>
    <img src="./images/路由器的结构.png" width=500>
</figure>

其中转发模块负责判断包的转发目的地，端口模块负责包的收发操作。这一分工模式在前面介绍计算机内部结构的时候也出现过，换句话说，路由器转发模块和端口模块的关系，就相当于协议栈的 IP 模块和网卡之间的关系。因此，大家可以将路由器的转发模块想象成 IP 模块，将端口模块想象成网卡。

通过更换网卡，计算机不仅可以支持以太网，也可以支持无线局域网，路由器也是一样。如果路由器的端口模块安装了支持无线局域网的硬件，就可以支持无线局域网了。此外，计算机的网卡除了以太网和无线局域网之外很少见到支持其他通信技术的品种，而路由器的端口模块则支持除局域网之外的多种通信技术，如 ADSL、FTTH，以及各种宽带专线等，只要端口模块安装了支持这些技术的硬件即可。

看懂了内部结构之后，大家应该能大致理解路由器的工作原理了吧。路由器在转发包时，首先会通过端口将发过来的包接收进来，这一步的工作过程取决于端口对应的通信技术。对于以太网端口来说，就是按照以太网规范进行工作，而无线局域网端口则按照无线局域网的规范工作，总之就是委托端口的硬件将包接收进来。接下来，转发模块会根据接收到的包的 IP 头部中记录的接收方 IP 地址，在路由表中进行查询，以此判断转发目标。然后，转发模块将包转移到转发目标对应的端口，端口再按照硬件的规则将包发送出去，也就是转发模块委托端口模块将包发送出去的意思。

这就是路由器的基本原理，下面再做一些补充。刚才我们讲到端口模块会根据相应通信技术的规范来执行包收发的操作，这意味着端口模块是以实际的发送方或者接收方的身份来收发网络包的。以以太网端口为例，路由器的端口具有 MAC 地址，因此它就能够成为以太网的发送方和接收方。端口还具有 IP 地址，从这个意义上来说，它和计算机的网卡是一样的。当转发包时，首先路由器端口会接收发给自己的以太网包，然后查询转发目标，再由相应的端口作为发送方将以太网包发送出去。这一点和交换机是不同的，交换机只是将进来的包转发出去而已，它自己并不会成为发送方或者接收方。

```note
路由器的各个端口都具有 MAC 地址和 IP 地址。
```

### 路由表中的信息

在“查表判断转发目标”这一点上，路由器和交换机的大体思路是类似的，不过具体的工作过程有所不同。交换机是通过 MAC 头部中的接收方 MAC 地址来判断转发目标的，而路由器则是根据 IP 头部中的 IP 地址来判断的。由于使用的地址不同，记录转发目标的表的内容也会不同。

关于细节我们留到后面再讲，现在先来大致介绍一下。路由器中的表叫作路由表，其中包含的信息如下图所示。

<figure>
    <img src="./images/路由表3.png" width=550>
</figure>

最左侧的目标地址列记录的是接收方的信息。这里可能不是很容易理解，实际上这里的 IP 地址只包含表示子网的网络号部分的比特值，而表示主机号部分的比特值全部为 0。路由器会将接收到的网络包的接收方 IP 地址与路由表中的目标地址进行比较，并找到相应的记录。交换机在地址表中只匹配完全一致的记录，而路由器则会忽略主机号部分，只匹配网络
号部分。打个比方，路由器在转发包的时候只看接收方地址属于哪个区，×× 区发往这一边，×× 区发往那一边。

在匹配地址的过程中，路由器需要知道网络号的比特数，因此路由表中还有一列子网掩码。通过这个值就可以判断出网络号的比特数。

上面这些介绍可以帮助大家大致理解路由器的工作方式，如果要进一步深入，还需要再思考一些问题。刚才我们说过，目标地址列中的 IP 地址表示子网，但也有一些例外，有时地址本身的子网掩码和路由表中的子网掩码是不一致的，这是路由聚合的结果。路由聚合会将几个子网合并成一个子网，并在路由表中只产生一条记录。

要搞清楚这个问题，我们还是看一个例子。如图 3.14 所示，

<figure>
    <img src="./images/路由聚合.png" width=400>
</figure>

我们现在有 3 个子网，分别为10.10.1.0/24、10.10.2.0/24、10.10.3.0/24，路由器 B 需要将包发往这 3 个子网。在这种情况下，路由器 B 的路由表中原本应该有对应这 3 个子网的 3 条记录，但在这个例子中，无论发往任何一个子网，都是通过路由器 A 来进行转发，因此我们可以在路由表中将这 3 个子网合并成 10.10.0.0/16，这样也可以正确地进行转发，但我们减少了路由表中的记录数量，这就是路由聚合。经过路由聚合，多个子网会被合并成一个子网，子网掩码会发生变化，同时，目标地址列也会改成聚合后的地址。

```note
10.10.1.0/24，这个24的意思是子网掩码的高24位都是一。即

- 10.10.1.0/24 = 10.10.1.0/255.255.255.0
- 10.10.2.0/24 = 10.10.2.0/255.255.255.0
- 10.10.3.0/24 = 10.10.3.0/255.255.255.0
- 10.10.0.0/16 = 10.10.0.0/255.255.0.0
```

相对地，还有另外一些情况，如将一个子网进行细分并注册在路由表中，然后拆分成多条记录。

从结果上看，路由表的子网掩码列只是用来在匹配目标地址时告诉路由器应该匹配多少个比特。而且，目标地址中的地址和实际子网的网络号可能并不完全相同，但即便如此，路由器依然可以正常工作。

此外，通过上述方法，我们也可以将某台具体计算机的地址写入路由表中，这时的子网掩码为 255.255.255.255，也就是说地址中的全部 32 个比特都为 1。这样一来，主机号部分比特全部为 0 可以表示一个子网，主机号部分比特不全部为 0 可以表示某一台计算机，两种情况可以用相同的规则来处理。

关于目标地址和子网掩码我们先讲到这里。接下来在子网掩码的右边还有网关和接口两列，它们表示网络包的转发目标。根据目标地址和子网掩码匹配到某条记录后，路由器就会将网络包交给接口列中指定的网络接口（即端口），并转发到网关列中指定的 IP 地址。

最后一列是跃点计数，它表示距离目标 IP 地址的距离是远还是近。这个数字越小，表示距离目的地越近；数字越大，表示距离目的地越远。

路由表记录维护的方式和交换机也有所不同。交换机中对 MAC 地址表的维护是包转发操作中的一个步骤，而路由器中对路由表的维护是与包转发操作相互独立的，也就是说，在转发包的过程中不需要对路由表的内容进行维护。

对路由表进行维护的方法有几种，大体上可分为以下两类。
- （a）由人手动维护路由记录
- （b） 根据路由协议机制，通过路由器之间的信息交换由路由器自行维护路由表的记录

其中（b）中提到的路由协议有很多种，例如 RIP、OSPC、BGP 等都属于路由协议。


### 路由器的包接收操作

下面我们来看一看路由器的整个工作过程。首先，路由器会接收网络包。路由器的端口有各种不同的类型，这里我们只介绍以太网端口是如何接收包的。以太网端口的结构和计算机的网卡基本相同，接收包并存放到缓冲区中的过程也和网卡几乎没有区别。

首先，信号到达网线接口部分，其中的 PHY（MAU）模块和 MAC 模块将信号转换为数字信息，然后通过包末尾的 FCS 进行错误校验，如果没问题则检查 MAC 头部中的接收方 MAC 地址，看看是不是发给自己的包，如果是就放到接收缓冲区中，否则就丢弃这个包。如果包的接收方 MAC 地址不是自己，说明这个包是发给其他设备的，如果接收这个包就违反了以太网的规则。

**路由器的端口都具有 MAC 地址，只接收与自身地址匹配的包，遇到不匹配的包则直接丢弃**。

### 查询路由表确定输出端口

完成包接收操作之后，路由器就会丢弃包开头的 MAC 头部。MAC 头部的作用就是将包送达路由器，其中的接收方 MAC 地址就是路由器端口的 MAC 地址。因此，当包到达路由器之后，MAC 头部的任务就完成了，于是 MAC 头部就会被丢弃。

接下来，路由器会根据 MAC 头部后方的 IP 头部中的内容进行包的转发操作。转发操作分为几个阶段，首先是查询路由表判断转发目标。关于具体的工作过程，我们还是来看一个实际的例子，还是这个例子

<figure>
    <img src="./images/路由表3.png" width=550>
</figure>

假设地址为 10.10.1.101 的计算机要向地址为 192.168.1.10 的服务器发送一个包，这个包先到达图中的路由器。判断转发目标的第一步，就是根据包的接收方 IP 地址查询路由表中的目标地址栏，以找到相匹配的记录。就像前面讲过的一样，这个匹配并不是匹配全部 32 个比特，而是根据子网掩码列中的值判断网络号的比特数，并匹配相应数量的比特。例如，图的第 3 行，子网掩码列为 255.255.255.0，就表示需要匹配从左起 24 个比特。网络包的接收方 IP 地址和路由表中的目标地址左起 24 个比特的内容都是192.168.1，因此两者是匹配的，该行记录就是候选转发目标之一。

按照这样的规则，我们可能会匹配到多条候选记录。在这个例子中，第 3、4、5 行都可以匹配。其中，路由器首先寻找网络号比特数最长的一条记录。网络号比特数越长，说明主机号比特数越短，也就意味着该子网内可分配的主机数量越少，即子网中可能存在的主机数量越少，这一规则的目的是尽量缩小范围，所以根据这条记录判断的转发目标就会更加准确。

第 3 行 192.168.1.0/255.255.255.0 表示一个子网，第 4 行 192.168.1.10/255.255.255.255 表示一台服务器。相比服务器所属的子网来说，直接指定服务器本身的地址时范围更小，因此这里应该选择第 4 行作为转发目标。

按照最长匹配原则筛选后，如果只剩一条候选记录，则按照这条记录的内容进行转发。然而，有时候路由表中会存在网络号长度相同的多条记录，例如考虑到路由器或网线的故障而设置的备用路由就属于这种情况。这时，需要根据跃点计数的值来进行判断。跃点计数越小说明该路由越近，因此应选择跃点计数较小的记录。

如果在路由表中无法找到匹配的记录，路由器会丢弃这个包，并通过 ICMP 消息告知发送方。这里的处理方式和交换机不同，原因在于网络规模的大小。交换机连接的网络最多也就是几千台设备的规模，这个规模并不大。如果只有几千台设备，遇到不知道应该转发到哪里的包，交换机可以将包发送到所有的端口上，虽然这个方法很简单粗暴，但不会引发什么问题。然而，路由器工作的网络环境就是互联网，它的规模是远远大于以太网的，全世界所有的设备都连接在互联网上，而且规模还在持续扩大，未来的互联网里到底会有多少设备，我们谁都说不准。在如此庞大的网络中，如果将不知道应该转发到哪里的包发送到整个网络上，那就会产生大量的网络包，造成网络拥塞。因此，路由器遇到不知道该转发到哪里的包，就会直接丢弃。

### 找不到匹配路由时选择默认路由

既然如此，那么是不是所有的转发目标都需要配置在路由表中才行呢？如果是公司或者家庭网络，这样的做法也没什么问题，但互联网中的转发目标可能超过 20 万个，如果全部要配置在路由表中实在是不太现实。

其实，大家不必担心，因为之前的 路由表中的最后一行的作用就相当于把所有目标都配置好了。这一行的子网掩码为 0.0.0.0，关键就在这里，子网掩码 0.0.0.0 的意思是网络包接收方 IP 地址和路由表目标地址的匹配中需要匹配的比特数为 0，换句话说，就是根本不需要匹配。只要将子网掩码设置为 0.0.0.0，那么无论任何地址都能匹配到这一条记录，这样就不会发生不知道要转发到哪里的问题了。

只要在这一条记录的网关列中填写接入互联网的路由器地址，当匹配不到其他路由时，网络包就会被转发到互联网接入路由器。因此这条记录被称为默认路由，这一行配置的网关地址被称为默认网关。在计算机的 TCP/IP 设置窗口中也有一个填写默认网关的框，意思是一样的。计算机上也有一张和路由器一样的路由表，其中默认网关的地址就是我们在设置窗口中填写的地址。

这样一来，无论目标地址是表示一个子网还是表示某台设备，都可以用相同的方法查找出转发目标，而且也避免了不知道转发到哪里的问题。

### 包的有效期

从路由表中查找到转发目标之后，网络包就会被转交给输出端口，并最终发送出去，但在此之前，路由器还有一些工作要完成。

第一个工作是更新 IP 头部中的 TTL（Time to Live，生存时间）字段（前面的某个表里有）。TTL 字段表示包的有效期，包每经过一个路由器的转发，这个值就会减 1，当这个值变成 0 时，就表示超过了有效期，这个包就会被丢弃。

这个机制是为了防止包在一个地方陷入死循环。如果路由表中的转发目标都配置正确，应该不会出现这样的情况，但如果其中的信息有问题，或者由于设备故障等原因切换到备用路由时导致暂时性的路由混乱，就会出现这样的情况。

发送方在发送包时会将 TTL 设为 64 或 128，也就是说包经过这么多路由器后就会“寿终正寝”。现在的互联网即便访问一台位于地球另一侧的服务器，最多也只需要经过几十个路由器，因此只要包被正确转发，就可以在过期之前到达目的地。


### 通过分片功能拆分大网络包

路由器的端口并不只有以太网一种，也可以支持其他局域网或专线通信技术。不同的线路和局域网类型各自能传输的最大包长度也不同，因此输出端口的最大包长度可能会小于输入端口。即便两个端口的最大包长度相同，也可能会因为添加了一些头部数据而导致包的实际长度发生变化，ADSL、FTTH 等宽带接入技术中使用的 PPPoE 协议就属于这种情况。无论哪种情况，一旦转发的包长度超过了输出端口能传输的最大长度，就无法直接发送这个包了。

遇到这种情况，可以使用 IP 协议中定义的分片功能对包进行拆分，缩短每个包的长度。需要注意的是，这里说的分片和第 2 章介绍的 TCP 对数据进行拆分的机制是不同的。TCP 拆分数据的操作是在将数据装到包里之前进行的，换句话说，拆分好的一个数据块正好装进一个包里。从 IP 分片的角度来看，这样一个包其实是一个未拆分的整体，也就是说，分片是对一个完整的包再进行拆分的过程。

分片操作的过程如图所示。

<figure>
    <img src="./images/分片功能.png" width=530>
</figure>

首先，我们需要知道输出端口的 MTU ，看看这个包能不能不分片直接发送。最大包长度是由端口类型决定的，用这个最大长度减掉头部的长度就是 MTU，将 MTU 与要转发的包长度进行比较。如果输出端口的 MTU 足够大，那么就可以不分片直接发送；如果输出端口的 MTU 太小，那么就需要将包按照这个 MTU 进行分片，但在此之前还需要看一下 IP 头部中的标志字段，确认是否可以分片。

如果查询标志字段发现不能分片，那么就只能丢弃这个包，并通过 ICMP 消息通知发送方。否则，就可以按照输出端口 MTU 对数据进行依次拆分了。在分片中，TCP 头部及其后面的部分都是可分片的数据，尽管 TCP 头部不属于用户数据，但从 IP 来看也是 TCP 请求传输的数据的一部分。数据被拆分后，每一份数据前面会加上 IP 头部，其大部分内容都和原本的 IP 头部一模一样，但其中有部分字段需要更新，这些字段用于记录分片相关的信息。

### 路由器的发送操作和计算机相同

到这里，发送前的准备工作就完成了，接下来就会进入包的发送操作。

这一步操作取决于输出端口的类型。如果是以太网端口，则按照以太网的规则将包转换为电信号发送出去；如果是 ADSL 则按照 ADSL 的规则来转换，以此类推。在家庭网络中，路由器后面一般连接 ADSL 等线路接入互联网，因此路由器会根据接入网的规则来发送包。不过，要理解具体的操作过程，需要先理解相应的通信线路，比较复杂，因此我们留到下一章探索互联网内部时再讲解。这里，我们假设路由器位于公司等局域网的内部，即输出端口也是以太网，看看这种情况是如何操作的。

以太网的包发送操作是根据以太网规则来进行的，即便设备种类不同，规则也是相同的。也就是说，其基本过程和协议栈中的 IP 模块发送包的过程是相同的，即在包前面加上 MAC 头部，设置其中的一些字段，然后将完成的包转换成电信号并发送出去。下面来简单复习一下这个过程。

首先，为了判断 MAC 头部中的 MAC 地址应该填写什么值，我们需要根据路由表的网关列判断对方的地址。如果网关是一个 IP 地址，则这个 IP 地址就是我们要转发到的目标地址；如果网关为空，则 IP 头部中的接收方 IP 地址就是要转发到的目标地址。知道对方的 IP 地址之后，接下来需要通过 ARP 根据 IP 地址查询 MAC 地址，并将查询的结果作为接收方 MAC 地址。路由器也有 ARP 缓存，因此首先会在 ARP 缓存中查询，如果找不到则发送 ARP 查询请求。接下来是发送方 MAC 地址字段，这里填写输出端口的 MAC 地址。

还有一个以太类型字段，填写 0080（十六进制）。网络包完成后，接下来会将其转换成电信号并通过端口发送出去。这一步的工作过程和计算机也是相同的。例如，当以太网工作在半双工模式时，需要先确认线路中没有其他信号后才能发送，如果检测到碰撞，则需要等待一段时间后重发。如果以太网工作在全双工模式，则不需要确认线路中的信号，可以直接发送。

如果输出端口为以太网，则发送出去的网络包会通过交换机到达下一个路由器。由于接收方 MAC 地址就是下一个路由器的地址，所以交换机会根据这一地址将包传输到下一个路由器。接下来，下一个路由器会将包转发给再下一个路由器，经过层层转发之后，网络包就到达了最终的目的地。

### 路由器与交换机的关系

关于路由器的基本工作，也就是包转发，到这里就全部讲完了，下面来整理一下路由器与交换机的关系。

<figure>
    <img src="./images/IP包装到以太网包.png" width=350>
</figure>

要理解两者之间的关系，关键点在于计算机在发送网络包时，或者是路由器在转发网络包时，都需要在前面加上 MAC 头部。之前的讲解都是说在开头加上 MAC 头部，如果看图 3.16 大家可以发现，准确的说法应该是将 IP 包装进以太网包的数据部分中。也就是说，给包加上 MAC 头部并发送，从本质上说是将 IP 包装进以太网包的数据部分中，委托以太网去传输这些数据。IP 协议本身没有传输包的功能，因此包的实际传输要委托以太网来进行。路由器是基于 IP 设计的，而交换机是基于以太网设计的，因此 IP 与以太网的关系也就是路由器与交换机的关系。换句话说，路由器将包的传输工作委托给交换机来进行。当然，这里讲的内容只适用于原原本本实现 IP 和以太网机制的纯粹的路由器和交换机，实际的路由器有内置交换机功能的，比如用于连接互联网的家用路由器就属于这一种，对于这种路由器，上面内容可能就不适用了。但是，如果把这种“不纯粹”的路由器拆分成“纯粹”的路由器和“纯粹”的交换机，则它们各自都适用上面的内容。

从包的转发目标也可以看出路由器和交换机之间的委托关系。IP 并不是委托以太网将包传输到最终目的地，而是传输到下一个路由器。在创建 MAC 头部时，也是从 IP 的路由表中查找出下一个路由器的 IP 地址，并通过 ARP 查询出 MAC 地址，然后将 MAC 地址写入 MAC 头部中的，这表示 IP 对以太网的委托只是将包传输到下一个路由器就行了。当包到达下一个路由器后，下一个路由器又会重新委托以太网将包传输到再下一个路由器。随着这一过程反复执行，包就会最终到达 IP 的目的地，也就是通信的对象。

到这里我们已经梳理了路由器与交换机之间的关系。简单来说，IP（路由器）负责将包发送给通信对象这一整体过程，而其中将包传输到下一个路由器的过程则是由以太网（交换机）来负责的。当然，网络并非只有以太网一种，还有无线局域网，以及接入互联网的通信线路，它们和 IP 之间的关系又是什么样的呢？其实只要将以太网替换成无线局域网、互联网线路等通信规格就可以了。也就是说，如果和下一个路由器之间是通过无线局域网连接的，那么就委托无线局域网将包传输过去；如果是通过互联网线路连接的，那么就委托它将包传输过去。除了这里列举的例子之外，世界上还有很多其他类型的通信技术，它们之间的关系也是一样的，都是委托所使用的通信技术将包传输过去。IP 本身不负责包的传输，而是委托各种通信技术将包传输到下一个路由器，这样的设计是有重要意义的，即可以根据需要灵活运用各种通信技术，这也是 IP 的最大特点。正是有了这一特点，我们才能够构建出互联网这一规模巨大的网络。


## 路由器的其他功能

### 通过地址转换有效利用 IP 地址

刚才我们介绍了路由器的基本工作过程，现在的路由器除了这些基本功能之外，还有一些附加功能。下面来介绍两种最重要的功能——地址转换和包过滤。

首先，我们先了解一下地址转换功能出现的背景。所谓地址，就是用来识别每一台设备的标志，因此每台设备都应该有一个唯一不重复的地址，就好像如果很多人的地址都一样，那么快递员就不知道该把包裹送给谁了。

网络也是一样，本来互联网中所有的设备都应该有自己的固定地址，而且最早也确实是这样做的。比如，公司内网需要接入互联网的时候，应该向地址管理机构申请 IP 地址，并将它们分配给公司里的每台设备。换句话说，那个时候没有内网和外网的区别，所有客户端都是直接连接到互联网的。

尽管互联网原本是这样设计的，但进入 20 世纪 90 年代之后，互联网逐步向公众普及，接入互联网的设备数量也快速增长，如此一来，情况就发生了变化。如果还用原来的方法接入，过不了多久，可分配的地址就用光了。如果不能保证每台设备有唯一不重复的地址，就会从根本上影响网络包的传输，这是一个非常严重的问题。如果任由这样发展下去，不久的将来，一旦固定地址用光，新的设备就无法接入了，互联网也就无法继续发展了。

解决这个问题的关键在于固定地址的分配方式。举个例子，假如有 A、B 两家公司，它们的内网是完全独立的。这种情况下，两家公司的内网之间不会有网络包流动，即使 A 公司的某台服务器和 B 公司的某台客户端具有相同的 IP 地址也没关系，因为它们之间不会进行通信。只要在每家公司自己的范围内，能够明确判断网络包的目的地就可以了，是否和其他公司的内网地址重复无关紧要，只要每个公司的网络是相互独立的，就不会出现问题。

解决地址不足的问题，利用的就是这样的性质，即公司内部设备的地址不一定要和其他公司不重复。这样一来，公司内部设备就不需要分配固定地址了，从而大幅节省了 IP 地址。当然，就算是公司内网，也不是可以随便分配地址的，因此需要设置一定的规则，规定某些地址是用于内网的，这些地址叫作私有地址，而原来的固定地址则叫作公有地址。

私有地址的规则其实并不复杂，在内网中可用作私有地址的范围仅限以下这些。
- 10.0.0.0 ～ 10.255.255.255
- 172.16.0.0 ～ 172.31.255.255
- 192.168.0.0 ～ 192.168.255.255
  
在制定私有地址规则时，这些地址属于公有地址中还没有分配的范围。换句话说，私有地址本身并没有什么特别的结构，只不过是将公有地址中没分配的一部分拿出来规定只能在内网使用它们而已。这个范围中的地址和其他公司重复也没关系，所以对于这些地址不作统一管理，不需要申请，任何人都可以自由使用。当然，如果在公司内部地址有重复就无法传输网络包了，因此必须避免在内网中出现重复的地址。

尽管这样的确能节省一部分地址，但仅凭这一点还无法完全解决问题。公司内网并不是完全独立的，而是需要通过互联网和其他很多公司相连接，所以当内网和互联网之间需要传输包的时候，问题就出现了，因为如果很多地方都出现相同的地址，包就无法正确传输了。

<figure>
    <img src="./images/地址转换.png" width=550>
</figure>

于是，当公司内网和互联网连接的时候，需要采用上图这样的结构，即将公司内网分成两个部分，一部分是对互联网开放的服务器，另一部分是公司内部设备。其中对互联网开放的部分分配公有地址，可以和互联网直接进行通信，这一部分和之前介绍的内容是一样的。相对地，内网部分则分配私有地址，内网中的设备不能和互联网直接收发网络包，而是通过一种特别的机制进行连接，这个机制就叫地址转换。

### 地址转换的基本原理

地址转换的基本原理是在转发网络包时对 IP 头部中的 IP 地址和端口号进行改写。具体的过程我们来看一个实际的例子，假设现在要访问 Web 服务器，看看包是如何传输的。

<figure>
    <img src="./images/改写ip地址.png" width=560>
</figure>

首先，TCP 连接操作的第一个包被转发到互联网时，将发送方 IP 地址从私有地址改写成公有地址。这里使用的公有地址是地址转换设备的互联网接入端口的地址。与此同时，端口号也需要进行改写，地址转换设备会随机选择一个空闲的端口。然后，改写前的私有地址和端口号，以及改写后的公有地址和端口号，会作为一组相对应的记录保存在地址转换设备内部的一张表中。

改写发送方 IP 地址和端口号之后，包就被发往互联网，最终到达服务器，然后服务器会返回一个包。服务器返回的包的接收包是原始包的发送方，因此返回的包的接收方就是改写后的公有地址和端口号。这个公有地址其实是地址转换设备的地址，因此这个返回包就会到达地址转换设备。

接下来，地址转换设备会从地址对应表中通过公有地址和端口号找到相对应的私有地址和端口号，并改写接收方信息，然后将包发给公司内网，这样包就能够到达原始的发送方了。在后面的包收发过程中，地址转换设备需要根据对应表查找私有地址和公有地址的对应关系，再改写地址和端口号之后进行转发。当数据收发结束，进入断开阶段，访问互联网的操作全部完成后，对应表中的记录就会被删除。

通过这样的机制，具有私有地址的设备就也可以访问互联网了。从互联网一端来看，实际的通信对象是地址转换设备（这里指的是路由器）。上面是以公司内网为例来进行介绍的，家庭网络中的工作过程也是完全相同的，只是规模不同而已。

### 改写端口号的原因

现在我们使用的地址转换机制是同时改写地址和端口号的，但早期的地址转换机制是只改写地址，不改写端口号的。用这种方法也可以让公司内网和互联网进行通信，而且这种方法更简单。

但是，使用这种方法的前提是私有地址和公有地址必须一一对应，也就是说，有多少台设备要上互联网，就需要多少个公有地址。当然，访问动作结束后可以删除对应表中的记录，这时同一个公有地址可以分配给其他设备使用，因此只要让公有地址的数量等于同时访问互联网的设备数量就可以了。然而公司人数一多，同时访问互联网的人数也会增加。一个几千人的公司里，有几百人同时访问互联网是很正常的，这样就需要几百个公有地址。

改写端口号正是为了解决这个问题。客户端一方的端口号本来就是从空闲端口中随机选择的，因此改写了也不会有问题。端口号是一个 16 比特的数值，总共可以分配出几万个端口，因此如果用公有地址加上端口的组合对应一个私有地址，一个公有地址就可以对应几万个私有地址，这种方法提高了公有地址的利用率。


### 从互联网访问公司内网

对于从公司内网访问互联网的包，即便其发送方私有地址和端口号没有保存在对应表中也是可以正常转发的，因为用来改写的公有地址就是地址转换设备自身的地址，而端口号只要随便选一个空闲的端口就可以了，这些都可以由地址转换设备自行判断。然而，对于从互联网访问公司内网的包，如果在对应表中没有记录就无法正常转发。因为如果对应表中没有记录，就意味着地址转换设备无法判断公有地址与私有地址之间的对应关系。

换个角度来看，这意味着对于没有在访问互联网的内网设备，是无法从互联网向其发送网络包的。而且即便是正在访问的设备，也只能向和互联网通信中使用的那个端口发送网络包，无法向其他端口发送包。也就是说，除非公司主动允许，否则是无法从互联网向公司内网发送网络包的。这种机制具有防止非法入侵的效果。

不过，有时候我们希望能够从互联网访问公司内网，这需要进行一些设置才能实现。之所以无法从互联网访问内网，是因为对应表里没有相应的记录，那么我们只要事先手动添加这样的记录就可以了。一般来说，用于外网访问的服务器可以放在地址转换设备的外面并为它分配一个公有地址，也可以将服务器的私有地址手动添加到地址转换设备中，这样就可以从互联网访问到这台具有私有地址的服务器了。

<figure>
    <img src="./images/互联网访问内网.png" width=560>
</figure>

### 路由器的包过滤功能

下面来介绍一下包过滤功能。包过滤也是路由器的一个重要附加功能，刚才的地址转换看起来有点复杂，不过包过滤的机制并不复杂。包过滤就是在对包进行转发时，根据 MAC 头部、IP 头部、TCP 头部的内容，按照事先设置好的规则决定是转发这个包，还是丢弃这个包。我们通常说的防火墙设备或软件，大多数都是利用这一机制来防止非法入侵的。

包过滤的原理非常简单，但要想设置一套恰当的规则来区分非法访问和正常访问，只阻止非法入侵而不影响正常访问，是非常不容易的。举个例子，为了防止从互联网非法入侵内网，我们可以将来自互联网的所有包都屏蔽掉，但是这会造成什么结果呢？正如我们第 2 章介绍过的 TCP 的工作过程一样，网络包是双向传输的，如果简单地阻止来自互联网的全部包，那么从内网访问互联网的操作也会无法正常进行。这个话题其实非常有趣，由于包过滤的使用方法和服务器的工作相关，所以我们在探索服务器时再详细介绍吧。

当网络包通过互联网接入路由器之后，它终于要进入互联网内部了，下一章将对这一部分进行探索。







