# Go 与分布式系统
文章中的图片来源于腾讯云的cos

当今的软件开发领域，微服务架构与分布式架构兴起，每个服务都需要负责特定的业务功能与业务流程，
去保证系统更高的可扩展性和灵活性。这时，go语言凭借着简洁的语法，高效的执行速度成为主流开发语
言之一，在构建高性能的应用程序时非常关键。并且，Go语言天生支持并发编程和高并发处理。Go语言的
并发模型基于协程（goroutine）和通道（channel），使得并发编程变得简单而高效。这种模型非常适
合处理大规模数据和高并发的分布式应用场景，满足了微服务架构的需求。在大数据和云计算时代，高并
发和分布式应用是常态，而Go语言正是为这种需求而设计的。
就像是《Go语言编程设计》这本书中写到，go语言设计就是为了程序员开发更舒适，更早的下班，所以语
法简洁，入门比较简单。那么为什么go语言能在高并发或是说高性能场景下这么受欢迎呢？
首先，在 Go 语言中，创建 goroutine 的成本非常低，每个 goroutine 只需几千字节的额外内存。正
因如此，即使是同时运行数百甚至数千个 goroutine 也成为可能。通过通道实现的 goroutine 间通信
是 Go 并发性的一大亮点。Go 运行时处理了所有的复杂性。借助 goroutine 和基于通道的并发方法，
利用所有可用的 CPU 内核和处理并发 IO 非常简便，无需复杂的开发。相比之下，与 Python 或 Java 
相比，在 Go 中只需极少的样板代码即可在一个 goroutine 中运行函数。只需使用关键字 "go" 即可添
加函数调用：
package main

import (
    "fmt"
    "time"
)
func say(s string) {
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}
func main() {
    go say("world")
    say("hello")
}
Go 的并发模型非常易于上手，相较于 Node.js 也更具趣味性；在 Node.js 中，开发者必须密切关注异
步代码的处理。
并发性的另一个优秀特性是竞态条件检测器，这使得在异步代码中轻松发现潜在的竞态条件问题。

其次，go语言对protocol buffers和grpc有一定的支持，能够很好的通过 RPC 进行通信的微服务器。只
需要写一个清单就能定义 RPC 调用发生的情况和参数，然后从该清单将自动生成服务器和客户端代码。
这样产生代码不仅快速，同时网络占用也非常少。

并且，go语言强大的GC能力也能够有效地管理内存，避免内存泄漏和内存碎片问题来保证程序运行的稳定
性和性能。其在1.3版本前通过先启动 STW 暂停，然后执行标记，停止 STW，最后再执行数据回收。的
方式进行GC。而在1.5版本后的三色标记法比较有意思，大体流程是：程序开始，全部标记为白色，1）所
有的对象放到白色集合，2）遍历一次根节点，得到灰色节点，3）遍历灰色节点，将可达的对象，从白色
标记灰色，遍历之后的灰色标记成黑色，4）由于并发特性，此刻外界向在堆中的对象发生添加对象，以
及在栈中的对象添加对象，在堆中的对象会触发插入屏障机制，栈中的对象不触发，5）由于堆中对象插
入屏障，则会把堆中黑色对象添加的白色对象改成灰色，栈中的黑色对象添加的白色对象依然是白色，
6）循环第 5 步，直到没有灰色节点，7）在准备回收白色前，重新遍历扫描一次栈空间，加上 STW 暂
停保护栈，防止外界干扰（有新的白色会被添加成黑色）在 STW 中，将栈中的对象一次三色标记，直到
没有灰色，8）停止 STW，清除白色。至于删除写屏障，则是遍历灰色节点的时候出现可达的节点被删
除，这个时候触发删除写屏障，这个可达的被删除的节点也是灰色，等循环三色标记之后，直到没有灰色
节点，然后清理白色，删除写屏障会造成一个对象即使被删除了最后一个指向它的指针也依旧可以活过这
一轮，在下一轮 GC 中被清理掉。而在最近的版本中，使用的是混合写屏障规则，1）GC 开始将栈上的
对象全部扫描并标记为黑色(之后不再进行第二次重复扫描，无需 STW)，2）GC 期间，任何在栈上创建
的新对象，均为黑色。3）被删除的对象标记为灰色。4）被添加的对象标记为灰色。
最后，每个goroutine的初始栈大小只有2KB。这相比于传统线程的栈大小要小很多。这意味着可以创建
更多的goroutine而不会消耗大量的内存。并且goroutine可以轻松地共享堆内存，而不需要为每个
goroutine分配单独的堆内存空间。说到这就不得不提出go语言伟大的调度设计，GMP模型，G 代表着 
goroutine，P 代表着上下文处理器，M 代表 thread 线程，在 GPM 模型，有一个全局队列（Global 
Queue）：存放等待运行的 G，还有一个 P 的本地队列：也是存放等待运行的 G，但数量有限，不超过 
256 个。GPM 的调度流程从 go func()开始创建一个 goroutine，新建的 goroutine 优先保存在 P 的
本地队列中，如果 P 的本地队列已经满了，则会保存到全局队列中。M 会从 P 的队列中取一个可执行
状态的 G 来执行，如果 P 的本地队列为空，就会从其他的 MP 组合偷取一个可执行的 G 来执行，当 
M 执行某一个 G 时候发生系统调用或者阻塞，M 阻塞，如果这个时候 G 在执行，runtime 会把这个线
程 M 从 P 中摘除，然后创建一个新的操作系统线程来服务于这个 P，当 M 系统调用结束时，这个 G 
会尝试获取一个空闲的 P 来执行，并放入到这个 P 的本地队列，如果这个线程 M 变成休眠状态，加入
到空闲线程中，然后整个 G 就会被放入到全局队列中。
以上种种条件下，国内很多公司选择使用go语言进行开发，而在最近的JDK21也来了，java也带来了协
程，是通过ForkJoinPool来实现的，也是具有work stealing机制的。并且go语言的错误处理非常烦人，
其目前最受欢迎的gozero框架相比于spring也有一定的差距。目前看来，go语言论剑”的能力，仍然需要
精进。

接下来，提到分布式就不得不说起蠕虫病毒发明人之一的Robert Morris教授的课程https://pdos.
csail.mit.edu/6.824/schedule.html。油管网址：https://www.youtube.com/channel/
UC_7WrbZTCODu1o_kfUMq88g。于国内也有搬运视频，可以省去翻译的麻烦：https://www.bilibili.
com/video/av87684880/?vd_source=8f5ffc48c9f4c1e2ff5541afb4be9550。

首先来说分布式系统中的一些基本概念，
大型互相协作的计算机驱动力：并行提高性能，容错，空间上分布的设备之间需要协调，限制出错域，提
高安全性。
实现implications:rpc,threads,锁。分布式基础的架构：存储，通信与运算。
其三个指标：可扩展性-指增加设备就能够提高性能的能力。可用性-在特定的故障范围内，系统仍然能够
提供服务，系统仍然是可用的。一致性：强一致性，所有机子都能读到最近的一次写入，通信代价很高。
弱一致性：允许读取出旧的数据，性能高。

这里参考《Go语言高级编程》。举几个例子吧，插入数据库之前，需要给这些消息、订单先打上一个ID，
然后再插入到数据库。对这个id的要求是希望其中能带有一些时间信息，这样即使后端的系统对消息进行
了分库分表，也能够以时间顺序对这些消息进行排序。
比如说sonyflake，Sony公司开源的一个项目。

Sequence ID和之前的定义一致，Machine ID其实就是节点id。sonyflake与众不同的地方在于其在启动
阶段的配置参数：

func NewSonyflake(st Settings) *Sonyflake
Settings数据结构如下：

type Settings struct {
    StartTime      time.Time
    MachineID      func() (uint16, error)
    CheckMachineID func(uint16) bool
}
StartTime选项之前的Epoch差不多，如果不设置的话，默认是从2014-09-01 00:00:00 +0000 UTC开
始。
MachineID可以由用户自定义的函数，如果用户不定义的话，会默认将本机IP的低16位作为machine id。
CheckMachineID是由用户提供的检查MachineID是否冲突的函数。这里的设计还是比较巧妙的，如果有另
外的中心化存储并支持检查重复的存储，那就可以按照自己的想法随意定制这个检查MachineID是否冲突
的逻辑。如果公司有现成的Redis集群，那么就可以很轻松地用Redis的集合类型来检查冲突。
redis 127.0.0.1:6379> SADD base64_encoding_of_last16bits MzI0Mgo=
(integer) 1
redis 127.0.0.1:6379> SADD base64_encoding_of_last16bits MzI0Mgo=
(integer) 0
使用起来也比较简单，有一些逻辑简单的函数就略去实现了：
package main
import (
    "fmt"
    "os"
    "time"

    "github.com/sony/sonyflake"
)
func getMachineID() (uint16, error) {
    var machineID uint16
    var err error
    machineID = readMachineIDFromLocalFile()
    if machineID == 0 {
        machineID, err = generateMachineID()
        if err != nil {
            return 0, err
        }
    }
    return machineID, nil
}
func checkMachineID(machineID uint16) bool {
    saddResult, err := saddMachineIDToRedisSet()
    if err != nil || saddResult == 0 {
        return true
    }
    err := saveMachineIDToLocalFile(machineID)
    if err != nil {
        return true
    }
    return false
}
func main() {
    t, _ := time.Parse("2006-01-02", "2018-01-01")
    settings := sonyflake.Settings{
        StartTime:      t,
        MachineID:      getMachineID,
        CheckMachineID: checkMachineID,
    }

    sf := sonyflake.NewSonyflake(settings)
    id, err := sf.NextID()
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    fmt.Println(id)
}
普通的进程内加入锁机制：
var wg sync.WaitGroup
var l sync.Mutex
for i := 0; i < 1000; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        l.Lock()
        counter++
        l.Unlock()
    }()
}
wg.Wait()
某些场景下，希望一个任务有单一的执行者。而不像计数器场景一样，所有goroutine都执行成功。后来
的goroutine在抢锁失败后，需要放弃其流程。这时候就需要trylock了。

trylock顾名思义，尝试加锁，加锁成功执行后续流程，如果加锁失败的话也不会阻塞，而会直接返回加
锁的结果。在Go语言中可以用大小为1的Channel来模拟trylock：
package main
import (
    "sync"
)
type Lock struct {
    c chan struct{}
}
func NewLock() Lock {
    var l Lock
    l.c = make(chan struct{}, 1)
    l.c <- struct{}{}
    return l
}
func (l Lock) Lock() bool {
    lockResult := false
    select {
    case <-l.c:
        lockResult = true
    default:
    }
    return lockResult
}
func (l Lock) Unlock() {
    l.c <- struct{}{}
}
var counter int
func main() {
    var l = NewLock()
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            if !l.Lock() {
                // log error
                println("lock failed")
                return
            }
            counter++
            println("current counter", counter)
            l.Unlock()
        }()
    }
    wg.Wait()
}
在分布式场景下，setnx很适合在高并发场景下，用来争抢一些“唯一”的资源。这种场景没有办法依赖具
体的时间来判断先后，因为不管是用户设备的时间，还是分布式场景下的各台机器的时间，都是没有办法
在合并后保证正确的时序的。哪怕是同一个机房的集群，不同的机器的系统时间可能也会有细微的差别。
所以，需要依赖于这些请求到达Redis节点的顺序来做正确的抢锁操作。
package main
import (
    "fmt"
    "sync"
    "time"

    "github.com/go-redis/redis"
)
func incr() {
    client := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",
        Password: "", // no password set
        DB:       0,  // use default DB
    })
    var lockKey = "counter_lock"
    var counterKey = "counter"
    resp := client.SetNX(lockKey, 1, time.Second*5)
    lockSuccess, err := resp.Result()

    if err != nil || !lockSuccess {
        fmt.Println(err, "lock result: ", lockSuccess)
        return
    }
    getResp := client.Get(counterKey)
    cntValue, err := getResp.Int64()
    if err == nil {
        cntValue++
        resp := client.Set(counterKey, cntValue, 0)
        _, err := resp.Result()
        if err != nil {
            // log err
            println("set value error!")
        }
    }
    println("current counter is ", cntValue)
    delResp := client.Del(lockKey)
    unlockSuccess, err := delResp.Result()
    if err == nil && unlockSuccess > 0 {
        println("unlock success!")
    } else {
        println("unlock failed", err)
    }
}
func main() {
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            incr()
        }()
    }
    wg.Wait()
}
简单说一下啊MapReduce，从名字来看，map-映射,Recude-压缩。是一个著名的分布式计算框架可以并行
的高效处理海量数据集，在大数据处理、并行计算、机器学习等方面都有很广泛的应用。
https://ldy-1311016186.cos.ap-beijing.myqcloud.comc7b438f61c9c533ae2634cf2646392d.png。
其编译模型：1.用户程序首先调用的MapReduce库将输入文件分成M个数据片度，每个数据片段的大小一
般从16MB到64MB(可以通过可选的参数来控制每个数据片段的大小)。然后用户程序在机群中创建大量的
程序副本。
2.这些程序副本中的有一个特殊的程序–master。副本中其它的程序都是worker程序，由master分配任
务。有M个Map任务和R个Reduce任务将被分配，master将一个Map任务或Reduce任务分配给一个空闲的
worker。
3.被分配了map任务的worker程序读取相关的输入数据片段，从输入的数据片段中解析出key/value 
pair，然后把key/value pair传递给用户自定义的Map函数，由Map函数生成并输出的中间key/value 
pair，并缓存在内存中。
4.缓存中的key/value pair通过分区函数分成R个区域，之后周期性的写入到本地磁盘上。缓存的key/
value pair在本地磁盘上的存储位置将被回传给master，由master负责把这些存储位置再传送给
Reduce worker。
5.当Reduce worker程序接收到master程序发来的数据存储位置信息后，使用RPC从Map worker所在主机
的磁盘上读取这些缓存数据。当Reduce worker读取了所有的中间数据后，通过对key进行排序后使得具
有相同key值的数据聚合在一起。由于许多不同的key值会映射到相同的Reduce任务上，因此必须进行排
序。如果中间数据太大无法在内存中完成排序，那么就要在外部进行排序。
6.Reduce worker程序遍历排序后的中间数据，对于每一个唯一的中间key值，Reduce worker程序将这
个key值和它相关的中间value值的集合传递给用户自定义的Reduce函数。Reduce函数的输出被追加到所
属分区的输出文件。
7.当所有的Map和Reduce任务都完成之后，master唤醒用户程序。在这个时候，在用户程序里的对
MapReduce调用才返回。
https://ldy-1311016186.cos.ap-beijing.myqcloud.com/c7b438f61c9c533ae2634cf2646392d.png
简单来说就是，单个Master主节点负责：任务创建与分配；多个Worker从节点负责：任务处理、状态汇
报。在映射阶段，master将大数据集分成小块，然后再不同的Mapper从结点上进行并行的计算处理，生
成键值对key-value pair（key-单词，value-当前次数）作为中间结果保存在文档中，并且将文档路径
告诉master。在reduce规约阶段，master会分配reducer对应的中间文件，这些中间的键值对会被
reducer合并，生成最终的计算结果。
相关文献：https://ldy-1311016186.cos.ap-beijing.myqcloud.com/mapreduce-osdi04.pdf。
