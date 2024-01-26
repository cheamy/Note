# context

## context是什么

是go用来在G(goroutine)之间传递信息的并发安全的包

1. 数据传递: 用于G之间传递上下问信息, 比如传递请求的trace_id, 以便于追踪全局唯一请求
2. 取消控制: 通过取消信号和超时时间来控制子goroutine的退出, 防止goroutine泄露



## context底层实现

接口:

| 接口名   | 说明                              |
| -------- | --------------------------------- |
| context  | context的接口定义, 四个基本方法   |
| canceler | context的取消接口, 定义了两个方法 |

实现:

| 结构名    | 说明                                      |
| --------- | ----------------------------------------- |
| emptyCtx  | 一个空的context, 用作根context            |
| cancelCtx | 可以通过取消函数来取消的context           |
| timerCtx  | 可以通过定时器和deadline定时取消的context |
| valueCtx  | 可以存储键值对的context                   |

方法:

| 函数名       | 说明                        |
| ------------ | --------------------------- |
| Background   | 返回一个根context(emptyCtx) |
| TODO         | 返回一个根context(emptyCtx) |
| withConcel   | 派生出一个concelCtx         |
| withDeadline | 派生出一个timerCtx          |
| withTimeout  | 派生出一个timerCtx          |
| withValue    | 派生出一个valueCtx          |

### 接口说明

#### context接口

``` go
type Context interface {
    Deadline() (deadline time.Time, ok bool) 	// 设置截止时间
    Done() <-chan struct{}						// 返回一个channel, 如果当前context已被关闭货到达截止时间, 这个channel会被关闭, 用于表示该context是否结束.
    Err() error									// 返回此context结束的原因
    Value(key interface{}) interface{}			// 从context中获取键对应的值
}
```

### conceler接口

``` go
type canceler interface {
	cancel(removeFromParent bool, err error)	//创建conceler接口实例的goroutine, 可以调用cancel方法通知被创建的gorouting退出
	Done() <-chan struct{}						//返回一个channel, 后续被创建的goroutine可以通过监听这个channel来完成退出
}
```



### 实现说明

#### emptyCtx

空的 context，永远不会被 cancel，没有存储值，也没有 deadline。

它被包装成：

```go
var (
    background = new(emptyCtx)
    todo       = new(emptyCtx)
)
```

``` go
func Background() Context {
	return background
}
func TODO() Context {
	return todo
}
```

background 通常用在 main 函数中，作为所有 context 的根节点。

todo 通常用在并不知道传递什么 context的情形。例如，调用一个需要传递 context 参数的函数，你手头并没有其他 context 可以传递，这时就可以传递 todo。这常常发生在重构进行中，给一些函数添加了一个 Context 参数，但不知道要传什么，就用 todo “占个位子”，最终要换成其他 context。

#### cancelCtx

```go
type cancelCtx struct {
	Context

	mu       sync.Mutex            // protects following fields
	done     atomic.Value          // of chan struct{}, created lazily, closed by first cancel call
	children map[canceler]struct{} // set to nil by the first cancel call
	err      error                 // set to non-nil by the first cancel call
}
```

这是一个可以取消的 Context，实现了 canceler 接口。它直接将接口 Context 作为它的一个匿名字段, 因此一定实现了context接口. 在withCancel()派生出来的cancelCtx中, 成员Context是该派生的父context

``` go
func (c *cancelCtx) Done() <-chan struct{} {
	d := c.done.Load()
	if d != nil {
		return d.(chan struct{})
	}
	c.mu.Lock()
	defer c.mu.Unlock()
	d = c.done.Load()
	if d == nil {
		d = make(chan struct{})
		c.done.Store(d)
	}
	return d.(chan struct{})
}
```

- c.done 是“懒汉式”创建，只有调用了 Done() 方法的时候才会被创建

  由于是只读的, 所以只有在关闭后才能读到值(零值), 结合select在关闭时及时获取关闭的信息

- cancel取消当前context, 还会遍历当前context所有子context, 递归取消, 递归取消完所有子context之后, 会将自身从父context的children map中移除

- 使用withCancel创建实例时, 会将创建出来的context和父context关联起来, 已达到级联取消的效果

  1. 当父context的done为空或已经被取消, 就不用关联, 直接取消当前子context
  2. 当父context可以被取消, 但还没被取消, 并且父context可以提取标准的concelCtx结构, 则将子context加入到父context的children map中(若没有children map, 先创建再加入)
  3. 当父context可以被取消, 但还没被取消, 且父context不能提取标准的concelCtx结构, 则新启一个goroutine用来监听父context的通信管道的取消信号



#### timerCtx

不仅可以通过取消函数 , 也可以设置一个截止时间deadline, 在deadline时自动取消context

withDeadline: 设置何时关闭

withTimeout: 设置活跃时间

**如果设置的截止时间晚于父context的截止时间, 则不会创建timerCtx, 会直接创建一个可取消的context, 关联父context, 如果设置的截止时间早于父context的截止时间, 会创建一个正常的timerCtx**

#### valueCtx

对 key 的要求是可比较，因为之后需要通过 key 取出 context 中的值，可比较是必须的。

通过层层传递 context，最终形成这样一棵树：

![img](F:\Note\实习冲刺\语言\Go\context.assets\v2-b0dbe9549219fca2c96f1699fb61fbc0_720w.webp)

取值会顺着链路一直往上找，比较当前节点的 key 是否是要找的 key，如果是，则直接返回 value。否则，一直顺着 context 往前，最终找到根节点（一般是 emptyCtx），直接返回一个 nil。所以用 Value 方法的时候要判断结果是否为 nil。

因为查找方向是往上走的，所以，父节点没法获取子节点存储的值，子节点却可以获取父节点的值。

`WithValue` 创建 context 节点的过程实际上就是创建链表节点的过程。两个节点的 key 值是可以相等的，但它们是两个不同的 context 节点。查找的时候，会向上查找到最后一个挂载的 context 节点，也就是离得比较近的一个父节点 context。所以，整体上而言，用 `WithValue` 构造的其实是一个低效率的链表。





