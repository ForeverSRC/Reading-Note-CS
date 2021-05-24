## 第3章 Go语言并发组件

## goroutine

goroutine是一个更高级别的抽象，称为协程。

gorouitne没有定义自己的暂停方法或者再运行点。

Go语言的runtime会观察goroutine的运行时行为，并在它们阻塞时自动挂起它们，在不阻塞时恢复它们。

Go语言遵循一个*fork-join*的并发模型：

* fork：在程序中的任意一点，将执行的子分支与其父节点同时运行
* join：在将来某时，并发的分支将会合并在一起

Go语言中，可使用`sync.WaitGroup`创建join点。

## sync包

### WaitGroup

当不关心并发操作的结果或有其他方法收集结果时，WaitGroup是等待一组并发操作完成的好方法。

可以将WaitGroup视为一个并发安全的计数器。

### 互斥锁和读写锁

在defer语句中调用`Unlock`的目的是：确保即使出现了panic，调用也总是发生。

### Pool

使用Pool时需要记住：

* 实例化`sync.Pool`，使用new创建一个成员变量，在调用时是线程安全的
* 收到一个来自Get的实例时，不要对所接收的对象的状态做出任何假设
* 用完一个从Pool中取出的对象时，一定要调用Put，否则无法复用此实例，通常情况下采用defer完成
* Pool内的分布必须大致均匀

## Channel

| 操作  | Channel状态 | 结果                                                 |
| ----- | ----------- | ---------------------------------------------------- |
| Read  | nil         | 阻塞                                                 |
|       | 打开且非空  | 输出值                                               |
|       | 打开但空    | 阻塞                                                 |
|       | 关闭        | <默认值>,false                                       |
|       | 只写        | 编译错误                                             |
| Write | nil         | 阻塞                                                 |
|       | 打开且填满  | 阻塞                                                 |
|       | 打开且不满  | 写入值                                               |
|       | 关闭的      | **panic**                                            |
|       | 只读        | 编译错误                                             |
| close | nil         | **panic**                                            |
|       | 打开且非空  | 关闭；读取成功，直到通道耗尽，然后读取产生值的默认值 |
|       | 打开但空    | 关闭；读到生产者的默认值                             |
|       | 关闭        | **panic**                                            |
|       | 只读        | 编译错误                                             |

组织与构建健壮稳定的使用channel的代码

分配channel所有权：

* 包括：实例化、写入、关闭channel的goroutine
* 利用单向channel声明，区分channel的拥有者和channel的使用者：
  * channel所有者对channel(`chan` 或`chan<-`)有一个写访问时图
  * channel使用者只对channel有一个只读视图(`<-chan`)

拥有channel的goroutine应具备如下特征：

1. 实例化channel
2. 执行写操作，或将所有权传递给另一个goroutine
3. 关闭channel
4. 通过只读channel暴露

作为一个channel的消费者需要担心的事：

1. 知道channel何时关闭
2. 正确处理阻塞



