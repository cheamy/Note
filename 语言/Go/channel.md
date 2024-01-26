# channel

## 定义

channel是用来手法数据的管道

## 初始化

声明:

``` go
var channel_name chan channel_type
var channel_name [size]chan channel_type
```

声明后初始化:

``` go
channel_name = make(chan channel_type)
channel_name = make(chan channel_type, size)
```

或者:

``` go
channel_name := make(chan channel_type)
channel_name := make(chan channel_type, size)
```

单向channel:

``` go
type RChannel = <-chan int
type SChannel = chan<- int

var ch = make(chan int)
var rec RChannel = ch
var send SChannel = ch
```

## 写操作

### case1:  channel中有读等待goroutine

1. 加锁(runtime.Mutex)
2. 从recvq(读等待队列)弹出队列头部的sudog, 进入send流程
3. 将要写入的数据拷贝到该sudog的elem数据容器上
4. 释放锁
5. 唤醒sudog绑定的goroutine

### case2: channel中没有读等待goroutine,且环形缓冲数组里面有剩余空间

1. 加锁
2. 将数据写入到senx指向的位置
3. sendx++, qcount++
4. 释放锁

### case3: channel中没有读等待goroutine,且没有剩余空间存放数据

1. 加锁(什么时候释放还不知道)
2. 获取一个sudog结构, 绑定对应的channel, goroutine, 还有ep指针
3. 将sudog放入channel的写等待队列
4. runtime.gopark 挂起当前的goroutine

### 特殊case

#### 写入的channel为nil

- 当写入的channel为nil时,会导致当前的goroutine永久性挂起,  如果当前的goroutine是main goroutine, 还会导致整个程序退出

#### channel已关闭还想进行写操作

- channel已关闭, 再向channel写, 会出现panic



## 读操作

### case1: channel中有写等待goroutine

1. 加锁
2. 从sendq中弹出队列头部的sudog, 并进入recv流程
3. 如果channel无缓冲区, 直接读取sudog里面的数据,并唤醒对应的goroutine
4. 如果channel有缓冲区, 读取环形缓冲区头部元素, 并将sudog中的元素写入到缓冲区,唤醒对应的goroutine
5. 释放锁

### case2: channel中无写等待gorouting且环形缓冲区有剩余元素

1. 加锁
2. 读取recvx指向的数据
3. recvx++, qcount--
4. 释放锁

### case3:channel中无写等待goroutine且无元素可以读

1. 加锁(暂时没找到在哪里释放锁)
2. 获取sudog结构, 绑定channel , goroutine, ep指针
3. 将sudog放入channel的读等待队列recvq
4. runtime.gopark 挂起对应的goroutine

### 特殊case

#### 读取的channel为nil

当读取channel为nil时, 会使当前goroutine永久挂起,如果是main goroutine, 会导致整个程序退出
#### 读取的channel已关闭

channel已关闭会正常读取里面的元素,如果没有剩余元素,则会得到对应类型的零值



## 关闭操作

1. 如果关闭一个nil的channel, 会发生panic
2. 加锁
3. 如果当前channel已关闭, 发生panic
4. 关闭channel(c.closed置为1)
5. 将sendq和recvq里面所有等待者加入到glist中
6. 释放锁
7. 唤醒glist中所有等待者



## select

select用于在一个goroutine中服务多个channel的读写操作

非阻塞型(含default):

``` go
select {
case <- ch1 :
    // 执行 ch1 case 的代码块
case ch2 <- value:
    // 执行 ch2 case 的代码块
default:
    // 如果上述 case 都不能执行，则执行 default 中的代码块
}
```

阻塞型(不含default):

``` go
select {
case ch1 <- value:
    // 执行 ch1 case 的代码块
case ch2 <- value:
    // 执行 ch2 case 的代码块
}
```



select会按随机顺序执行case, 直到某一个case完成操作. 如果所有的case都没有立即完成操作(即对应channel没有准备好), 则看有没有default分支, 有default则走default, 防止阻塞

如果没有default, 则将当前goroutine加入到所有case对应的等待队列中, 并挂起当前的goroutine, 等待唤醒. 

如果当前goroutine被某一个case上的channel操作唤醒, 还需要将当前goroutine从所有case对应channel的等待队列中剔除