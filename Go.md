# Go语言笔记
    main 和 init函数均自动调用。
| 维度     | init   | main   |
|:---------|:------:|-------:|
| 调用方式     | 自动调用（初始化包里的变量）  | 自动调用（程序入口）    |
| 定义位置   | 可在任意包中，支持多个定义  | 仅 main 包中，且唯一     |
| 执行顺序   | （复杂）同一文件中的 init 按代码顺序从上到下执行。<br/>同一包不同文件按文件名字符串顺序（字典序）执行。<br/>不同包的 init 按导入依赖关系执行：先被依赖的包先执行，无依赖时按 import 的逆序执行  | init执行完执行     |


## 指针
定义方式：
    
    //定义变量i,指针p1,p1接受i的内存地址
    i := 1
    var p1 *int
    p1 = &i
    
    //定义指针p2,p2储存p1的内存地址，即指针的指针
    p2 := &p1

    fmt.Println(p1) //输出i的内存地址
    fmt.Println(p2) //输出p1的内存地址

    fmt.Println(*p1,"--",**p2)//输出1--1，访问指针和访问指针的指针。

需要注意*访问指针可能会只想空指针。

指针和uintptr转换关系：*T <---> unsafe.Pointer <---> uintptr

    a := "Hello, world!"
    upA := uintptr(unsafe.Pointer(&a))
    upA += 1

    c := (*uint8)(unsafe.Pointer(upA))
    fmt.Println(*c)

## 结构体struct
#### 用于聚合多个不同类型字段的数据结构，Go当中没有类的概念
#### 首字母大写的字段或方法可在包外访问，否则仅限包内

语法：

        type 结构体名 struct {
            field1 int
            Person  // 匿名字段（继承Person字段）
            // ...
        }

实例化方式:

        p := Person{Name: "Alice", Age: 25} 
        p := new(Person)  // *Person 指针类型
        user := struct{Name string; Age int}{Nzame: "Bob"}//匿名结构

标签：
        
        //用作序列化
        type User struct {
            Name string `json:"name"`
            Age  int    `json:"age"`
        }

结构体方法：

        // 值接收者（操作副本）
        func (p Person) GetName() string {
            return p.Name
        }

        // 指针接收者（操作实际对象）
        func (p *Person) SetAge(age int) {
            p.Age = age
        }        

## 常量和枚举

    const a int = 1 //基础定义
    //定义多个
    const (
        h    byte = 3
        i         = "value"
        j, k      = "v", 4
        l, m      = 5, false
    )

    //枚举
    type Weekday int
    const (
        Sunday Weekday = iota // 0
        Monday                // 1
        Tuesday               // 2
    )
    
    //_ 跳值
    const (
        A = iota // 0
        _        // 跳过，iota=1
        B        // 2
    )

    // 位掩码（左移操作）
    const (
        FlagUp = 1 << iota // 1<<0=1
        FlagBroadcast      // 1<<1=2
    )

    // 数学表达式（如数量级定义）
    const (
        KB = 1 << (10 * iota) // 1<<10=1024
        MB                    // 1<<20
    )

    // 多常量同行声明
    const (
        A, B = iota, iota+1 // A=0, B=1
        C, D                // C=1, D=2
    )

    // 插值与重置
    const (
        A = 100        // iota=0（但显式赋值为100）
        B = iota       // iota=1 → B=1
        C              // iota=2 → C=2
    )

    const (
        A = iota       // 0
        B = "test"     // iota=1（但显式赋值）
        C = iota       // iota=2 → C=2
    )

## channel
    先进先出（FIFO）
    ch := make(chan int)      // 无缓冲通道（同步模式）
    chBuf := make(chan int, 3) // 有缓冲通道（异步模式，容量3）

发送阻塞：无缓冲通道需双方就绪，否则阻塞；有缓冲通道在缓冲区满时阻塞。

接收阻塞：无数据时阻塞，接收已关闭通道返回零值（通过v, ok := <-ch判断关闭状态）。

关闭通道：close(ch)，关闭后发送会panic，接收仍可取剩余数据

    // 无缓冲通道导致死锁（未配对）
    ch := make(chan int)
    ch <- 1 // 阻塞直到接收端就绪，若无接收者则deadlock

    // 有缓冲通道允许短暂异步
    ch := make(chan int, 3)
    ch <- 1; ch <- 2 // 不阻塞，缓冲区未满

参数限制仅发送或仅接受

    //仅发送数据
    func <method_name>(<channel_name> chan <- <type>)

    //仅接收数据
    func <method_name>(<channel_name> <-chan <type>)

## sync包
    互斥锁（Mutex）
    var mutex sync.Mutex
    mutex.Lock()   // 加锁
    defer mutex.Unlock() // 解锁
    // 临界区代码

    WaitGroup
    var wg sync.WaitGroup
    wg.Add(3)       // 添加3个任务
    go func() {
        defer wg.Done() // 任务完成，计数器-1
    }()
    wg.Wait()       // 阻塞直到计数器归零

# [第二周](#SecondWeek)
## 不要通过共享内存来通信，而应该通过通信来共享内存
- [shareMemoryByCommunicating](https://github.com/bgscr/Study_Notes/blob/main/homework-task/task4/shareMemoryByCommunicating/main.go)
- [shareMemoryByCommunicatingV2](https://github.com/bgscr/Study_Notes/blob/main/homework-task/task4/shareMemoryByCommunicatingV2/main.go)


## 指针
    pointer methods，使用指针 作为方法接收者，则必须通过 指针 调用此方法。
    value methods，使用值 作为方法接收者，则既能通过 值 也能通过指针调用此方法。

    这有个例外情况。当 value 是addressable的，golang编译器会自动将通过 值 调用pointer methods的代码转换成通过 指针 调用。

    简单理解为，常量无法寻址，但变量肯定会存储在内存某个地方，可以被寻址

    下面的值不能被寻址(addresses):
    bytes in strings：字符串中的字节
    map elements：map中的元素
    dynamic values of interface values (exposed by type assertions)：接口的动态值
    constant values：常量
    literal values：字面值
    package level functions：包级别的函数
    methods (used as function values)：方法
    intermediate values：中间值
    function callings
    explicit value conversions
    all sorts of operations, except pointer dereference operations, but including:
    channel receive operations
    sub-string operations
    sub-slice operations
    addition, subtraction, multiplication, and division, etc.
    注意， &T{}相当于tmp := T{}; (&tmp)的语法糖，所以&T{}可合法不意味着T{}可寻址。
    下面的值可以寻址:
    variables
    fields of addressable structs
    elements of addressable arrays
    elements of any slices (whether the slices are addressable or not)
    pointer dereference operations
 
## 下划线:_  ,blank identifier的使用技巧
	var _ json.Marshaler = (*RawMessage)(nil)
	在此声明中，我们调用了一个 *RawMessage 转换并将其赋予了 Marshaler，以此来要求 *RawMessage 实现 Marshaler，这时其属性就会在编译时被检测。 若 json.Marshaler 接口被更改，此包将无法通过编译， 而我们则会注意到它需要更新



## select原理
	编译器会对select有不同的case的情况进行优化以提高性能。首先，编译器对select没有case、有单case和单case+default的情况进行单独处理，这些处理或者直接调用运行时函数，或者直接转成对channel的操作，或者以非阻塞的方式访问channel，多种灵活的处理方式能够提高性能，尤其是避免对channel的加锁。

	对最常出现的select有多case的情况，会调用runtime.selectgo()函数来获取执行case的索引，并生成 if 语句执行该case的代码。

	selectgo函数的执行分为四个步骤：
        首先，随机生成一个遍历case的轮询顺序 pollorder 并根据 channel 地址生成加锁顺序 lockorder，随机顺序能够避免channel饥饿，保证公平性，加锁顺序能够避免死锁和重复加锁；
        然后，根据 pollorder 的顺序查找 scases 是否有可以立即收发的channel，如果有则获取case索引进行处理；
        再次，如果pollorder顺序上没有可以直接处理的case，则将当前 goroutine 加入各 case 的 channel 对应的收发队列上并等待其他 goroutine 的唤醒；
        最后，当调度器唤醒当前 goroutine 时，会再次按照 lockorder 遍历所有的case，从中查找需要被处理的case索引进行读写处理，同时从所有case的发送接收队列中移除掉当前goroutine。


## vet
	使用该工具检查语法或规范问题


## sync
	第一次使用后不能复制的，noCopy，使用go vet监测出现复制锁的情况
	// noCopy may be added to structs which must not be copied
	// after the first use.
	//
	// See https://golang.org/issues/8005#issuecomment-190753527
	// for details.
	//
	// Note that it must not be embedded, due to the Lock and Unlock methods.
	type noCopy struct{}

	// Lock is a no-op used by -copylocks checker from `go vet`.
	func (*noCopy) Lock()   {}
	func (*noCopy) Unlock() {}

### fast-path 和 slow-path
    fast-path：一段针对常见操作或最佳情况进行优化的代码路径。在这条路径上，通常执行步骤最少、效率最高。所以 fast path 通常在设计上避免了昂贵的操作（如加锁、IO 操作等）以提高性能。

    slow-path：用于处理较为罕见或复杂的情况，通常执行步骤较多、性能较低。这类路径通常在少数情况下才会被执行，比如当代码需要处理边缘情况或复杂的操作时。

    slow-path 分离出来，单独定义一个函数，目的是为了对 fast-path 进行内联优化。
    将 slow-path 逻辑放在单独的 doSlow 函数中可以使 Do 方法的快路径更简洁，这样还有助于 Go 编译器对 fast-path 进行内联优化（即直接嵌入到调用处），从而减少函数调用的开销，提高性能。

### Mutex   
    
	// A Mutex is a mutual exclusion lock.
	//
	// See package [sync.Mutex] documentation.
	type Mutex struct {
		state int32
		sema  uint32
	}

	state int32 不同位分别表示了不同的状态：
	mutexLocked — 表示互斥锁的锁定状态；
	mutexWoken — 表示从正常模式被从唤醒；
	mutexStarving — 当前的互斥锁进入饥饿状态；
	waitersCount — 当前互斥锁上等待的 Goroutine 个数

### 先判断能否进入自旋锁，进入自旋锁的条件：	
	old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter)
	非饥饿并且已锁状态，runtime_canSpin(多 CPU 、当前 Goroutine 为了获取该锁进入自旋的次数小于四次、当前机器上至少存在一个正在运行的处理器 P 并且处理的运行队列为空)

### 没有进入自旋锁时的操作
	new := old
		// Don't try to acquire starving mutex, new arriving goroutines must queue.
		//非饥饿模式则变成锁的状态
		if old&mutexStarving == 0 {
			new |= mutexLocked
		}
		//如果是锁的状态或者饥饿的状态时，排队数量+1
		if old&(mutexLocked|mutexStarving) != 0 {
			new += 1 << mutexWaiterShift
		}
		// The current goroutine switches mutex to starvation mode.
		// But if the mutex is currently unlocked, don't do the switch.
		// Unlock expects that starving mutex has waiters, which will not
		// be true in this case.
		//更新饥饿模式
		if starving && old&mutexLocked != 0 {
			new |= mutexStarving
		}
		//唤醒状态
		if awoke {
			// The goroutine has been woken from sleep,
			// so we need to reset the flag in either case.
			if new&mutexWoken == 0 {
				throw("sync: inconsistent mutex state")
			}
			new &^= mutexWoken
		}


### 获取锁
	如果没有通过 CAS 获得锁，会调用 runtime.sync_runtime_SemacquireMutex 通过信号量保证资源不会被两个 Goroutine 获取。

	runtime.sync_runtime_SemacquireMutex 会在方法中不断尝试获取锁并陷入休眠等待信号量的释放，一旦当前 Goroutine 可以获取信号量，它就会立刻返回，sync.Mutex.Lock的剩余代码也会继续执行。

	在正常模式下，这段代码会设置唤醒和饥饿标记、重置迭代次数并重新执行获取锁的循环；
	在饥饿模式下，当前 Goroutine 会获得互斥锁，如果等待队列中只存在当前 Goroutine，互斥锁还会从饥饿模式中退出；

	if atomic.CompareAndSwapInt32(&m.state, old, new) {
	    if old&(mutexLocked|mutexStarving) == 0 {
	        break // locked the mutex with CAS
	    }
	    queueLifo := waitStartTime != 0
	    if waitStartTime == 0 {
	        waitStartTime = runtime_nanotime()
	    }

	    runtime_SemacquireMutex(&m.sema, queueLifo, 1)
	    // 如果等待时间超过 1ms 则切换到饥饿模式
	    starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
	    old = m.state
	    if old&mutexStarving != 0 {
	        if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
	            throw("sync: inconsistent mutex state")
	        }
	        delta := int32(mutexLocked - 1<<mutexWaiterShift)
	        if !starving || old>>mutexWaiterShift == 1 {
	            delta -= mutexStarving
	        }
	        atomic.AddInt32(&m.state, delta)
	        break
	    }
	    awoke = true
	    iter = 0
	} else {
	    old = m.state
	}



### 解锁
	该过程会先使用atomic.AddInt32函数快速解锁，这时会发生下面的两种情况：
	如果该函数返回的新状态等于 0，当前 Goroutine 就成功解锁了互斥锁；
	如果该函数返回的新状态不等于 0，则进入 Slow path。
	func (m *Mutex) Unlock() {
		// Fast path: drop lock bit.
		new := atomic.AddInt32(&m.state, -mutexLocked)
		if new != 0 {
			// Outlined slow path to allow inlining the fast path.
			// To hide unlockSlow during tracing we skip one extra frame when tracing GoUnblock.
			m.unlockSlow(new)
		}
	}


	先校验锁状态的合法性 — 如果当前互斥锁已经被解锁过了会直接抛出异常 “sync: unlock of unlocked mutex” 中止当前程序。
	在正常模式下，上述代码会使用如下所示的处理过程：

		如果互斥锁不存在等待者或者互斥锁的 mutexLocked、mutexStarving、mutexWoken 状态不都为 0，那么当前方法可以直接返回，不需要唤醒其他等待者；
		如果互斥锁存在等待者，会通过runtime_Semrelease唤醒等待者并移交锁的所有权；

	在饥饿模式下，上述代码会直接调用runtime_Semrelease将当前锁交给下一个正在尝试获取锁的等待者，等待者被唤醒后会得到锁，在这时互斥锁还不会退出饥饿状态；
	func (m *Mutex) unlockSlow(new int32) {
	   if (new+mutexLocked)&mutexLocked == 0 {
	      throw("sync: unlock of unlocked mutex")
	   }
	   if new&mutexStarving == 0 {
	      old := new
	      for {
	         if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
	            return
	         }
	         new = (old - 1<<mutexWaiterShift) | mutexWoken
	         if atomic.CompareAndSwapInt32(&m.state, old, new) {
	            runtime_Semrelease(&m.sema, false, 1)
	            return
	         }
	         old = m.state
	      }
	   } else {
	      runtime_Semrelease(&m.sema, true, 1)
	   }
	}

	互斥锁的加锁，它涉及自旋、信号量以及调度等概念：

	如果互斥锁处于初始化状态，会通过置位 mutexLocked 加锁；
	如果互斥锁处于 mutexLocked 状态并且在普通模式下工作，会进入自旋，执行 30 次 PAUSE 指令消耗 CPU 时间等待锁的释放；
	如果当前 Goroutine 等待锁的时间超过了 1ms，互斥锁就会切换到饥饿模式；
	互斥锁在正常情况下会通过sync_runtime_SemacquireMutex将尝试获取锁的 Goroutine 切换至休眠状态，等待锁的持有者唤醒；
	如果当前 Goroutine 是互斥锁上的最后一个等待的协程或者等待的时间小于 1ms，那么它会将互斥锁切换回正常模式；

	互斥锁的解锁过程：
	当互斥锁已经被解锁时，调用 Mutex.Lock 会直接抛出异常；
	当互斥锁处于饥饿模式时，将锁的所有权交给队列中的下一个等待者，等待者会负责设置 mutexLocked 标志位；
	当互斥锁处于普通模式时，如果没有 Goroutine 等待锁的释放或者已经有被唤醒的 Goroutine 获得了锁，会直接返回；在其他情况下会通过sync.runtime_Semrelease 唤醒对应的 Goroutine；

### Once

    //第一个字段与结构体本身的指针地址是相同的，访问 Once 结构体无需指针偏移操作，就可以直接操作 done 属性
    type Once struct {
        _ noCopy

        // done indicates whether the action has been performed.
        // It is first in the struct because it is used in the hot path.
        // The hot path is inlined at every call site.
        // Placing done first allows more compact instructions on some architectures (amd64/386),
        // and fewer instructions (to calculate offset) on other architectures.
        done atomic.Uint32
        m    Mutex
    }

    //为了能够确保f已经执行完成之后，才通知其他goroutine不用再次执行，所以采用原子操作和互斥锁方法。
    func (o *Once) Do(f func()) {
        // Note: Here is an incorrect implementation of Do:
        //
        //	if o.done.CompareAndSwap(0, 1) {
        //		f()
        //	}
        //
        // Do guarantees that when it returns, f has finished.
        // This implementation would not implement that guarantee:
        // given two simultaneous calls, the winner of the cas would
        // call f, and the second would return immediately, without
        // waiting for the first's call to f to complete.
        // This is why the slow path falls back to a mutex, and why
        // the o.done.Store must be delayed until after f returns.

        if o.done.Load() == 0 {
            // Outlined slow-path to allow inlining of the fast-path.
            o.doSlow(f)
        }
    }

    func (o *Once) doSlow(f func()) {
        o.m.Lock()
        defer o.m.Unlock()
        if o.done.Load() == 0 {
            defer o.done.Store(1)
            f()
        }
    }

### OnceFunc 如果抛出异常每次调用都是相同的异常，Once.Do只会有一次异常
    // OnceFunc returns a function that invokes f only once. The returned function
    // may be called concurrently.
    //
    // If f panics, the returned function will panic with the same value on every call.
    func OnceFunc(f func()) func() {
        var (
            once  Once
            valid bool
            p     any
        )
        // Construct the inner closure just once to reduce costs on the fast path.
        g := func() {
            defer func() {
                p = recover()
                if !valid {
                    // Re-panic immediately so on the first call the user gets a
                    // complete stack trace into f.
                    panic(p)
                }
            }()
            f()
            f = nil      // Do not keep f alive after invoking it.
            valid = true // Set only if f does not panic.
        }
        return func() {
            once.Do(g)
            if !valid {
                panic(p)
            }
        }
    }

### OnceValue 在OnceFunc的基础上有返回值
    // OnceValue returns a function that invokes f only once and returns the value
    // returned by f. The returned function may be called concurrently.
    //
    // If f panics, the returned function will panic with the same value on every call.
    func OnceValue[T any](f func() T) func() T {
        var (
            once   Once
            valid  bool
            p      any
            result T
        )
        g := func() {
            defer func() {
                p = recover()
                if !valid {
                    panic(p)
                }
            }()
            result = f()
            f = nil
            valid = true
        }
        return func() T {
            once.Do(g)
            if !valid {
                panic(p)
            }
            return result
        }
    }
### sync.OnceValues 在sync.OnceValue的基础上返回两个返回值
    // OnceValues returns a function that invokes f only once and returns the values
    // returned by f. The returned function may be called concurrently.
    //
    // If f panics, the returned function will panic with the same value on every call.
    func OnceValues[T1, T2 any](f func() (T1, T2)) func() (T1, T2) {
        var (
            once  Once
            valid bool
            p     any
            r1    T1
            r2    T2
        )
        g := func() {
            defer func() {
                p = recover()
                if !valid {
                    panic(p)
                }
            }()
            r1, r2 = f()
            f = nil
            valid = true
        }
        return func() (T1, T2) {
            once.Do(g)
            if !valid {
                panic(p)
            }
            return r1, r2
        }
    }

### sync.Map
    type Map struct {
        mu    sync.Mutex        // 保护dirty的互斥锁
        read  atomic.Pointer[readOnly] // 原子操作的只读map
        dirty map[any]*entry    // 全量数据，操作需加锁
        misses int              // read未命中次数，触发dirty提升
    }

    type readOnly struct {
        m       map[any]*entry  // 只读数据
        amended bool            // 标记dirty存在read未包含的键
    }

    /*
    entry的指针p有三种状态：
    nil：已标记删除，但尚未同步到dirty。
    expunged：已从dirty中删除，不可恢复。
    正常值：有效数据
    */
    type entry struct {
        p atomic.Pointer[any]   // 原子操作的value指针，支持状态标记
    }

    read：原子操作的无锁map，支持高并发读。
    dirty：含全量数据，操作需加锁。当misses达到阈值（len(dirty)）时，dirty会替换为新的read

    进行删除的时候会现将extry的p设为nil
    LoadOrStore和Store，如果read和dirty没有key时，并且misses次数达到dirty的数量时，将重构dirty，此时会将nil状态的extry的p设置为expunged
    
    重构read的时候会跳过nil和expunged的值

#### Swap方法，Store实际调用的方法
    func (m *Map) Swap(key, value any) (previous any, loaded bool) {
        // 先尝试从 read map 获取 key 对应的 entry
        read := m.loadReadOnly()
        if e, ok := read.m[key]; ok { // 如果 key 存在
            // 尝试交换值
            if v, ok := e.trySwap(&value); ok { // 如果交换成功
                if v == nil { // 说明已被删除，但没有彻底删除（expunged）
                    return nil, false // 新增键值对成功
                }
                return *v, true // 交换成功
            }
        }

        // 未找到或需要修改 dirty map，需要加锁处理
        m.mu.Lock()
        read = m.loadReadOnly()
        if e, ok := read.m[key]; ok { // 如果 key 在 read map 中
            if e.unexpungeLocked() {
                // 如果 entry 之前被 expunged，意味着 dirty map 不为 nil 且该 entry 不在 dirty map 中
                m.dirty[key] = e
            }
            // 进行值交换
            if v := e.swapLocked(&value); v != nil {
                loaded = true
                previous = *v
            }
        } else if e, ok := m.dirty[key]; ok { // 如果 key 在 dirty map 中
            // 进行值交换
            if v := e.swapLocked(&value); v != nil {
                loaded = true
                previous = *v
            }
        } else { // key 即不在 read map 中，也不在 dirty map 中
            if !read.amended {
                // 添加第一个新 key 到 dirty map，需要先标记 read map 为不完整
                m.dirtyLocked()
                m.read.Store(&readOnly{m: read.m, amended: true})
            }
            // 在 dirty map 中创建新 entry
            m.dirty[key] = newEntry(value)
        }
        m.mu.Unlock()
        return previous, loaded
    }


