# go项目

![QQ20250227-221634](C:\Users\86138\Desktop\QQ20250227-221634.png)



![QQ20250227-221703](C:\Users\86138\Desktop\QQ20250227-221703.png)



https://mp.weixin.qq.com/s/ZW8czTaZ0lY5vb4HUbWEAw

 [什么是rpc.pdf](Q:\BaiduNetdiskDownload\慕课网体系课-2022-2023年Go开发工程师全新版-3888元-完结\课程资料\课程资料\Go工程师体系课全新版电子教程\第4周 从0开始理解rpc和grpc\simpread-8._什么是_rpc_Go_工程师体系课全新版_-_慕课网【加微信.赠送精品IT课程】.pdf) 

# 看完一遍pdf文件

# 1.go的net包

# 2.rpc原理 stub

protoc --proto_path=. --go-grpc_out=. user.proto

protoc -I . --go_out=. --go-grpc_out=. user.proto

# 3.协程

好的，我会为你提供更详细的 Go 协程操作大全，涵盖协程的基本使用、同步、通道、信号量控制、错误处理等内容，并结合更多示例来详细讲解每个操作的实现。

1. **创建协程**

Go 协程（goroutine）是由 Go 运行时管理的轻量级线程。通过 `go` 关键字可以启动一个新的协程。

示例：基本的协程使用

```go
package main

import "fmt"

func greet() {
	fmt.Println("Hello from Goroutine!")
}

func main() {
	go greet() // 启动一个新的协程
	fmt.Println("Main function is running")
}
```

- **解释**：调用 `go greet()` 会异步启动一个新的协程执行 `greet` 函数。主程序继续执行 `fmt.Println("Main function is running")`，并不会等待协程执行完毕。因此，主函数的输出可能在协程输出之前或者之后。

2. **同步协程：`sync.WaitGroup`**

有时我们希望等待所有的协程执行完成再继续执行后续操作。Go 提供了 `sync.WaitGroup` 来实现这一点。`WaitGroup` 通过 `Add` 来指定等待的协程数，通过 `Done` 来标记协程完成，通过 `Wait` 等待所有协程完成。

示例：使用 `sync.WaitGroup` 等待协程完成

```go
package main

import (
	"fmt"
	"sync"
)

func task(wg *sync.WaitGroup, id int) {
	defer wg.Done() // 在任务完成后调用 Done，通知 WaitGroup 该协程已完成
	fmt.Printf("Task %d completed\n", id)
}

func main() {
	var wg sync.WaitGroup

	// 启动 3 个协程
	for i := 1; i <= 3; i++ {
		wg.Add(1) // 增加等待的协程数
		go task(&wg, i)
	}

	wg.Wait() // 等待所有协程完成
	fmt.Println("All tasks completed.")
}
```

- **解释**：`wg.Add(1)` 为每个启动的协程添加一个计数器，`defer wg.Done()` 会在每个协程执行完成后调用 `Done` 来减少计数，最后主函数通过 `wg.Wait()` 等待所有协程完成。

3. **协程间通信：使用通道（Channel）**

Go 使用通道（Channel）在协程之间进行通信。通过 `chan` 创建通道，并使用 `<-` 进行发送和接收。

示例：基本通道使用

```go
package main

import "fmt"

func sendData(ch chan int) {
	ch <- 42 // 向通道发送数据
}

func main() {
	ch := make(chan int) // 创建一个无缓冲通道

	go sendData(ch)
	data := <-ch // 从通道接收数据
	fmt.Println("Received data:", data)
}
```

- **解释**：创建一个通道 `ch`，启动一个协程发送数据。主程序通过 `<-ch` 接收数据并打印。

4. **带缓冲区的通道**

带缓冲区的通道可以在没有接收者的情况下存储一定数量的数据。你可以在创建通道时指定缓冲区大小。

示例：带缓冲区的通道

```go
package main

import "fmt"

func sendData(ch chan int) {
	for i := 0; i < 3; i++ {
		ch <- i // 向通道发送数据
	}
	close(ch) // 关闭通道
}

func main() {
	ch := make(chan int, 2) // 创建一个容量为 2 的缓冲区通道

	go sendData(ch)
	for data := range ch { // 从通道接收数据，直到通道关闭
		fmt.Println("Received:", data)
	}
}
```

- **解释**：缓冲区通道允许发送者在没有接收者的情况下存储最多 2 个数据。通过 `close(ch)` 关闭通道，避免阻塞接收操作。

5. **`select` 语句：选择多个通道**

`select` 语句允许你等待多个通道操作并选择其中一个最先完成的操作。这在需要同时监听多个通道时非常有用。

示例：`select` 使用

```go
package main

import "fmt"

func sendData(ch chan int) {
	ch <- 42
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)

	go sendData(ch1)
	go sendData(ch2)

	select {
	case data1 := <-ch1:
		fmt.Println("Received from ch1:", data1)
	case data2 := <-ch2:
		fmt.Println("Received from ch2:", data2)
	}
}
```

- **解释**：`select` 语句会阻塞，直到从 `ch1` 或 `ch2` 中接收到数据。一旦接收到数据，`select` 语句会退出并执行相应的 case。

6. **取消协程的执行：使用 `context` 包**

`context` 包用于处理协程的取消和超时等管理。在协程中通过上下文（`context.Context`）来传递取消信号。

示例：使用 `context` 取消协程

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func task(ctx context.Context) {
	select {
	case <-time.After(2 * time.Second):
		fmt.Println("Task completed")
	case <-ctx.Done():
		fmt.Println("Task canceled:", ctx.Err())
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	go task(ctx)

	time.Sleep(1 * time.Second) // 等待 1 秒
	cancel()                    // 取消任务
	time.Sleep(2 * time.Second) // 等待协程处理取消
}
```

- **解释**：`cancel()` 会通知协程停止执行。`task` 函数会监听 `ctx.Done()`，一旦接收到取消信号，协程会输出取消信息并停止执行。

7. **协程的错误处理：使用通道传递错误**

在并发编程中，错误处理是非常重要的。可以通过通道来将错误信息传递到主线程。

示例：协程中的错误处理

```go
package main

import (
	"fmt"
	"errors"
	"sync"
)

func riskyTask(ch chan error) {
	// 模拟错误
	ch <- errors.New("something went wrong")
}

func main() {
	ch := make(chan error)
	var wg sync.WaitGroup
	wg.Add(1)

	go func() {
		defer wg.Done()
		riskyTask(ch)
	}()

	wg.Wait() // 等待协程完成
	if err := <-ch; err != nil {
		fmt.Println("Error:", err)
	}
}
```

- **解释**：`riskyTask` 会模拟一个错误，并通过通道发送给主程序。主程序接收到错误后，打印错误信息。

8. **限制并发数：使用信号量（Semaphore）模拟**

当有多个协程需要并发执行时，我们有时需要限制并发的数量。可以通过通道模拟信号量来控制并发的协程数量。

示例：限制并发数

```go
package main

import (
	"fmt"
	"time"
)

func task(id int, sem chan struct{}) {
	sem <- struct{}{} // 协程开始，获取信号量
	defer func() { <-sem }() // 协程结束，释放信号量

	fmt.Printf("Task %d is running\n", id)
	time.Sleep(1 * time.Second)
}

func main() {
	sem := make(chan struct{}, 3) // 限制最多 3 个并发

	for i := 0; i < 10; i++ {
		go task(i, sem)
	}

	time.Sleep(5 * time.Second) // 等待协程完成
}
```

- **解释**：通过一个容量为 3 的缓冲通道 `sem` 来限制并发数，最多允许 3 个协程同时运行。

------

总结

Go 协程和相关的同步机制如通道、`WaitGroup`、`context` 等，使得并发编程变得简单而高效。以下是一些核心概念和技巧：

- **协程创建**：通过 `go` 启动异步任务。
- **同步**：使用 `sync.WaitGroup` 等工具等待协程完成。
- **通信**：使用通道在协程间传递数据。
- **选择通道**：使用 `select` 语句同时监听多个通道。
- **协程取消**：使用 `context` 取消协程执行。
- **错误处理**：通过通道或其他方式传递错误信息。
- **控制并发**：使用信号量等技术限制并发数。

掌握这些技巧，可以有效地编写高效且可维护的并发程序。

# 4.鸭子类型

# 5.gin









