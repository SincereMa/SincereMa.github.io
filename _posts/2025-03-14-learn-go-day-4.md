---
layout: post
title: "Go语言高效学习-并发与工程化 (Day 4)"
date: "2025-03-14"
categories: "技术"
tags: "Go语言高效学习" 
thumb: "/assets/images/post/Learn-Go-Stage-2.png"
---

###### 针对NodeJS工程师的Go语言学习计划
###### 🔧 阶段二：并发与工程化（Days 4-7）
###### 目标：掌握Go的核心竞争力—并发与工程化开发流程 
###### Day 4：Goroutine与Channel


![](/assets/images/post/Learn-Go-full.png)

## 🚀 Go语言高效学习计划（NodeJS工程师版）
目标：2周快速掌握核心概念，上手大型项目；后续深入高级特性

本文涉及的代码链接：[Github](https://github.com/SincereMa/Go-Learn)


## 知识点梳理与对比

### 1. Goroutine (轻量级协程)

*   **概念**: Goroutine 是 Go 语言中并发执行的基本单位，类似于线程，但比线程更轻量级。您可以将 Goroutine 看作是在 Go 运行时（runtime）管理的轻量级线程。
*   **特点**:
    *   **轻量级**: 创建和销毁 Goroutine 的开销远小于线程。可以轻松创建成千上万个 Goroutine。
    *   **Go 运行时调度**: Goroutine 由 Go 运行时调度器自动管理，无需手动管理线程的生命周期。
    *   **非抢占式多任务处理**: Goroutine 之间的切换由 Go 运行时在发生阻塞操作（如 I/O、Channel 操作）时自动进行。
*  **与 Node.js 的 `worker_threads` 对比**
    
    | 特性        | Goroutine                                   | Node.js `worker_threads`                     |
    | ----------- | ------------------------------------------- | ------------------------------------------- |
    | 轻量级      | 更轻量，创建/销毁开销小                         | 相对较重，创建/销毁开销较大                       |
    | 调度        | Go 运行时自动调度，无需手动管理                   | 需要手动管理线程的创建、消息传递、销毁                |
    | 通信        | 主要通过 Channel, 也支持共享内存，但推荐Channel | 主要通过 `postMessage` 进行消息传递，也支持共享内存 |
    | 适用场景    | 高并发 I/O 密集型任务                           | CPU 密集型任务，或需要隔离执行环境的任务      |
    
*   **启动方式**: 使用 `go` 关键字即可启动一个新的 Goroutine。

    ```go
    package main

    import (
    	"fmt"
    	"time"
    )

    func myFunc() {
    	fmt.Println("Hello from a Goroutine!")
    }

    func main() {
    	go myFunc() // 启动一个新的 Goroutine 执行 myFunc
    	time.Sleep(time.Second) // 等待 Goroutine 执行
        fmt.Println("Hello from the main function!")
    }
    ```
*  **与Nodejs 示例对比**
    ```typescript
    // Node.js 使用 worker_threads
    const { Worker, isMainThread, parentPort } = require('worker_threads');

    if (isMainThread) {
      const worker = new Worker(__filename);
      worker.on('message', (message) => console.log(message));

      // 让 worker 有时间发送消息
      setTimeout(() => {
        console.log("Hello from the main thread")
      }, 1000);

    } else {
      parentPort.postMessage('Hello from a worker thread!');
    }

    ```

### 2. Channel (管道)

*   **概念**: Channel 是 Goroutine 之间通信的主要方式。它提供了一种类型安全、同步的机制来传递数据。
*   **类型**:
    *   **无缓冲 Channel**: 发送和接收操作是同步的，必须同时准备好才能进行数据传递。发送方会阻塞，直到接收方准备好接收；接收方会阻塞，直到发送方准备好发送。
    *   **有缓冲 Channel**: 发送方在缓冲区未满时不会阻塞，接收方在缓冲区非空时不会阻塞。
*   **创建**: 使用 `make` 函数创建 Channel。
    ```go
    ch := make(chan int)     // 无缓冲 int 类型 Channel
    chBuffered := make(chan string, 10) // 有缓冲 string 类型 Channel，缓冲区大小为 10
    ```
 *   **与Node.js的消息传递对比**

      | 特性        | Goroutine                                     | Node.js `worker_threads`                                        |
      | ---------- | --------------------------------------------- | ---------------------------------------------------------------- |
      | 消息        | 类型安全、同步的机制来传递数据                     | `postMessage` 进行消息传递, `SharedArrayBuffer` 在线程中共享内存. |

    
*   **操作**:
    *   **发送**: `ch <- value`
    *   **接收**: `value := <-ch`
    *   **关闭**: `close(ch)` (关闭后不能再发送数据，但仍可以接收已发送的数据)
*   **示例**

    ```go
    package main

    import "fmt"

    func main() {
    	// 无缓冲 Channel
    	ch := make(chan int)

    	go func() {
    		ch <- 10 // 发送数据
    	}()

      	value := <-ch // 接收数据
    	fmt.Println(value) // 输出: 10

    	// 有缓冲 Channel
    	chBuffered := make(chan string, 2)

    	chBuffered <- "Hello"
    	chBuffered <- "World"

    	fmt.Println(<-chBuffered) // 输出: Hello
    	fmt.Println(<-chBuffered) // 输出: World
      close(ch) // 示例：关闭通道

    }
    ```
*  **与Nodejs 示例对比**

    ```typescript
    // Node.js 使用 worker_threads 消息传递
    const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

    if (isMainThread) {
      const worker = new Worker(__filename, { workerData: 'Hello' }); // 传递初始数据
      worker.on('message', (message) => console.log(`Received: ${message}`));
      worker.postMessage('World'); // 主线程发送消息
    } else {
      console.log(`Received: ${workerData}`); // 工作线程接收初始数据
      parentPort.postMessage('from worker'); // 工作线程发送消息
    }
    ```

### 3. `select` 多路复用

*   **概念**: `select` 语句用于处理多个 Channel 的发送和接收操作。它会阻塞，直到其中一个 case 满足条件（即某个 Channel 可发送或接收）。
*   **特点**:
    *   **非确定性选择**: 如果多个 case 同时满足，`select` 会随机选择一个执行。
    *   **超时处理**: 可以使用 `time.After` 结合 `select` 实现超时控制。
    *   **default case**: 如果没有任何 case 满足，会执行 `default` case（如果存在）。
*   **示例**:

    ```go
    package main

    import (
    	"fmt"
    	"time"
    )

    func main() {
    	ch1 := make(chan string)
    	ch2 := make(chan string)

    	go func() {
    		time.Sleep(time.Second)
    		ch1 <- "Message from ch1"
    	}()

    	go func() {
    		time.Sleep(2 * time.Second)
    		ch2 <- "Message from ch2"
    	}()

    	for i := 0; i < 2; i++ {
    		select {
    		case msg1 := <-ch1:
    			fmt.Println(msg1)
    		case msg2 := <-ch2:
    			fmt.Println(msg2)
    		case <-time.After(3 * time.Second):
    			fmt.Println("Timeout")
    			return // 添加 return 语句避免继续循环
    		}
    	}
    }

    ```

## 实战：并发文件处理

下面是一个并发文件处理的示例，它结合了 Goroutine、Channel 和 `select`，并与 Node.js 的 `fs.promises` 链式调用进行了对比。

**Go 实现:**

```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
	"sync"
	"time"
)

// FileData 结构体，用于存储文件名和内容
type FileData struct {
	Name    string
	Content string
	Err     error
}

// processFile 函数处理单个文件， 并将结果发送到 Channel
func processFile(filePath string, resultChan chan<- FileData) { // 使用单向 Channel，限制只能发送
	content, err := os.ReadFile(filePath)
	resultChan <- FileData{Name: filepath.Base(filePath), Content: string(content), Err: err}
}

func main() {
	dir := "./test_files" // 假设要处理的文件都在这个目录下

	// 创建测试文件
	err := createTestFiles(dir)
	if err != nil {
		fmt.Println("创建测试文件出错", err)
		os.Exit(1)
	}

	// 创建一个有缓冲的 Channel
	resultChan := make(chan FileData, 10) // 缓冲区大小可以根据实际情况调整
	var wg sync.WaitGroup                 // 用于等待所有 Goroutine 完成

	// 遍历目录， 为每个文件启动一个 Goroutine
	files, err := os.ReadDir(dir) // 使用 os.ReadDir 读取目录下的文件
	if err != nil {
		fmt.Println("读取目录失败", err)
		os.Exit(1)
	}
	for _, file := range files {
		if !file.IsDir() { // 忽略子目录
			filePath := filepath.Join(dir, file.Name())
			wg.Add(1) // 增加 WaitGroup 计数器
			go func(fp string) {
				defer wg.Done()             // Goroutine 完成时减少计数器
				processFile(fp, resultChan) // 处理文件
			}(filePath)
		}
	}
	// 启动一个 Goroutine 来关闭 Channel
	go func() {
		wg.Wait()         // 等待所有文件处理 Goroutine 完成
		close(resultChan) // 关闭 Channel
	}()

	// 使用 select 监听 resultChan 和超时
	timeout := time.After(5 * time.Second) // 设置超时时间
	for {
		select {
		case result, ok := <-resultChan: // 从结果通道接收文件数据
			if !ok {
				// Channel 已关闭，所有文件处理完成
				fmt.Println("所有文件处理完成！")
				return
			}
			if result.Err != nil {
				fmt.Printf("处理文件 %s 出错: %v\n", result.Name, result.Err)
			} else {
				fmt.Println("文件内容读取成功", result.Name)
			}
		case <-timeout:
			fmt.Println("处理文件操作超时")
			return
		}
	}
}

// 创建测试文件
func createTestFiles(dir string) error {
	// 确保目录存在
	if _, err := os.Stat(dir); os.IsNotExist(err) {
		err := os.Mkdir(dir, 0755)
		if err != nil {
			panic(err)
		}
	}

	for i := 1; i <= 3; i++ {
		fileName := fmt.Sprintf("file%d.txt", i)
		content := []byte(fmt.Sprintf("This is the content of %s", fileName))
		err := os.WriteFile(filepath.Join(dir, fileName), content, 0644)
		if err != nil {
			return err
		}
	}
	return nil
}
```
1.  **创建测试文件**：首先检查`test_files`目录是否存在，如果不存在则创建该目录。然后，它创建三个测试文件（`file1.txt`、`file2.txt`、`file3.txt`），每个文件都包含一些示例文本内容。
2.  **文件处理**：
  *   定义了一个`FileData`结构体来存储每个文件的名称、内容以及处理过程中可能出现的错误。
  *   `resultChan := make(chan FileData, 10)`：创建了一个缓冲通道`resultChan`，用于接收处理文件的结果。
  *   使用`sync.WaitGroup`来等待所有处理文件的goroutine完成。
3.  **启动goroutine**：
  *   使用`os.ReadDir`读取`test_files`目录中的所有文件和子目录。
  *   遍历文件列表，对于每个文件，启动一个goroutine来处理它。
     *    `wg.Add(1)`：增加WaitGroup的计数器，表示有一个新的goroutine开始执行。

  *   `go func(fp string) { ... }(filePath)`：启动一个新的goroutine来处理文件。这里使用了一个匿名函数，并将文件路径`fp`作为参数传递给它。
  *   `defer wg.Done()`：在goroutine结束时减少WaitGroup的计数器。这确保了无论goroutine如何退出（正常完成或发生错误），计数器都会递减。
4.   **处理文件结果**：
    *  使用`select`多路复用从resultChan中读取结果，直到通道关闭:

5.  **运行结果**

```
文件内容读取成功 file1.txt
文件内容读取成功 file3.txt
文件内容读取成功 file2.txt
所有文件已处理完成！
```

我们通过对比Go和Node.js在并发模型、线程/协程通信以及文件处理方面的实现方式，提供了Goroutine和Channel的相关知识点。

