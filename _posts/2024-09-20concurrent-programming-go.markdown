Concurrent and Parallel Programming in Go

September 2024

Pedro Ruiz Pareja

Most people think concurrency and parallelism are the same. But they’re not, and understanding the difference is crucial if you’re going to write efficient Go programs. Concurrency is about dealing with lots of things at once, while parallelism is about doing lots of things at once. These concepts are intertwined, but they solve different problems.

**Concurrency** is like handling multiple tasks by interleaving them. Imagine you’re a juggler. You’re not holding all the balls at the same time, but you're handling them in a way that appears simultaneous. It’s about structuring your program to manage multiple tasks effectively. **Parallelism**, on the other hand, is like having multiple jugglers, each juggling their own set of balls at the same time. It’s about executing multiple tasks literally at the same time.

Go was designed with concurrency in mind. Rob Pike, one of the creators of Go, once said, “Concurrency is not parallelism.” Go provides powerful concurrency primitives, like goroutines and channels, that make concurrent programming easy and fun. Let’s dive into how you can use these in Go.

### Goroutines: Lightweight Threads

Goroutines are the cornerstone of concurrency in Go. They’re like threads, but much lighter. Creating a goroutine is as simple as adding the `go` keyword before a function call. Here’s a basic example:

```go
package main

import (
    "fmt"
    "time"
)

func sayHello() {
    fmt.Println("Hello, World!")
}

func main() {
    go sayHello()
    time.Sleep(1 * time.Second) // Give the goroutine time to run
}
```

In this example, `sayHello` runs as a goroutine. The `main` function sleeps for a second to give the goroutine time to execute. Without this sleep, the program might exit before `sayHello` gets a chance to run.

### Channels: Communication Between Goroutines

Channels are Go’s way of enabling communication between goroutines. Think of them as pipes that connect goroutines, allowing them to send and receive data. Here’s a simple example:

```go
package main

import "fmt"

func sum(a int, b int, result chan int) {
    result <- a + b // Send sum to channel
}

func main() {
    result := make(chan int)
    go sum(1, 2, result)
    fmt.Println(<-result) // Receive result from channel
}
```

In this example, the `sum` function sends the sum of `a` and `b` to the `result` channel. The `main` function receives and prints the result. Channels are typed, so you can only send specific types of data through them.

### Select: Waiting on Multiple Channels

The `select` statement in Go is like a switch statement for channels. It lets you wait on multiple channel operations. Here’s an example:

```go
package main

import (
    "fmt"
    "time"
)

func sendValue(ch chan int, value int) {
    time.Sleep(time.Second)
    ch <- value
}

func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)

    go sendValue(ch1, 1)
    go sendValue(ch2, 2)

    select {
    case v1 := <-ch1:
        fmt.Println("Received from ch1:", v1)
    case v2 := <-ch2:
        fmt.Println("Received from ch2:", v2)
    }
}
```

In this example, `sendValue` sends a value to a channel after sleeping for a second. The `select` statement in `main` waits for a value from either `ch1` or `ch2` and prints it. This way, you can handle multiple channel operations without blocking.

### Parallelism in Go

While Go’s concurrency primitives are powerful, they don’t automatically give you parallelism. Parallelism in Go is achieved by running goroutines on multiple OS threads. Go’s runtime scheduler handles this for you, but you can control the level of parallelism with the `runtime.GOMAXPROCS` function. Here’s an example:

```go
package main

import (
    "fmt"
    "runtime"
    "sync"
)

func printNumbers(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    for i := 0; i < 5; i++ {
        fmt.Printf("Goroutine %d: %d\n", id, i)
    }
}

func main() {
    runtime.GOMAXPROCS(2) // Set the number of OS threads to 2
    var wg sync.WaitGroup

    for i := 0; i < 3; i++ {
        wg.Add(1)
        go printNumbers(i, &wg)
    }

    wg.Wait()
}
```

In this example, `runtime.GOMAXPROCS(2)` sets the number of OS threads to 2, allowing up to 2 goroutines to run in parallel. The `printNumbers` function prints numbers from 0 to 4, and we start 3 goroutines in `main`. The `sync.WaitGroup` ensures that `main` waits for all goroutines to finish.

### Real-World Application: Web Scraping

Let’s look at a real-world example: web scraping. Suppose you want to scrape multiple websites concurrently. Here’s how you can do it in Go:

```go
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
    "sync"
)

func fetch(url string, wg *sync.WaitGroup) {
    defer wg.Done()
    resp, err := http.Get(url)
    if err != nil {
        fmt.Println(err)
        return
    }
    body, err := ioutil.ReadAll(resp.Body)
    resp.Body.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Printf("Fetched %d bytes from %s\n", len(body), url)
}

func main() {
    var wg sync.WaitGroup
    urls := []string{
        "https://example.com",
        "https://golang.org",
        "https://godoc.org",
    }

    for _, url := range urls {
        wg.Add(1)
        go fetch(url, &wg)
    }

    wg.Wait()
}
```

In this example, the `fetch` function retrieves the content of a URL and prints the number of bytes fetched. The `main` function starts a goroutine for each URL and waits for all of them to finish using a `sync.WaitGroup`.

### Conclusion

Concurrency and parallelism are fundamental concepts in Go, and mastering them will make you a better Go programmer. Goroutines and channels are powerful tools that let you write concurrent programs easily. While Go’s runtime scheduler handles most of the work for you, understanding how to control concurrency and parallelism will help you write more efficient and effective programs.

If you’re new to Go, I recommend experimenting with these concepts. Start with simple examples and gradually move to more complex ones. The more you practice, the more intuitive concurrent and parallel programming will become.

For further reading, check out these references:

- [The Go Programming Language Specification](https://golang.org/ref/spec)
- [Effective Go](https://golang.org/doc/effective_go.html)
- [Go Concurrency Patterns](https://blog.golang.org/pipelines)
- [The Go Blog](https://blog.golang.org/)

Happy coding!
