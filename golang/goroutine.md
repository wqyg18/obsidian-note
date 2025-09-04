```go
package main  
  
import (  
    "fmt"  
    "time")  
  
func printMessage(msg string) {  
    for i := 0; i < 5; i++ {  
       fmt.Println(msg, i)  
       time.Sleep(time.Millisecond * 500)  
    }  
}  
func main() {  
    go printMessage("Goroutine 1")  
    go printMessage("Goroutine 2")  
  
    fmt.Println("main done")  
}
```
```md
 ┌───────────┐
 │  main()   │
 └─────┬─────┘
       │
       ▼
   创建 goroutine1 ──────► [等待调度执行]
       │
       ▼
   创建 goroutine2 ──────► [等待调度执行]
       │
       ▼
   打印 "main done"
       │
       ▼
   main 结束 → 程序退出

```
### 关键点：

1. **main 本身也是一个 goroutine**，叫做 `main goroutine`。
    
2. 当你 `go printMessage(...)` 时，新的子 goroutine 只是被 _注册到调度器_，并不保证立刻执行。
    
3. **调度器是抢占式的**，但调度时机不可预测，所以有时候子 goroutine 来得及执行，有时候来不及。
    
4. 如果 `main` 很快就执行完（比如直接 `fmt.Println("1")` 就退出），子 goroutine 可能还没跑，整个程序就结束了。
    

---

### 解决办法：让 main 等待子 goroutine

- **用 `time.Sleep`：** 暂停一下，给调度器机会运行子 goroutine（不推荐，粗糙）。
    
- **用 `sync.WaitGroup`：** 让 main 等待所有子 goroutine 完成（推荐，常用）。