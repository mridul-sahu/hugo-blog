+++
author = "Mridul Sahu"
categories = ["Go", "Concurrency"]
tags = ["programming", "pattern", "google-io"]
date = "2019-05-19"
description = "Writing Concurrent Programs using Go"
featured = "gophers.jpg"
featuredalt = "Concurrent code using Go"
featuredpath = "date"
title = "Golang Concurrency Patterns"
type = "post"
+++

## What is Concurrency?

Large programs are often made up of many smaller sub-programs. For example a web server handles requests made from web browsers and serves up HTML web pages in response. Each request is handled like a small program. It would be ideal for programs like these to be able to run their smaller components at the same time (in the case of the web server to handle multiple requests).

Concurrency is the composition of independently executing components. Concurrency helps us make progress on more than one task simultaneously. Thus programs running multiple components at the same time are called concurrent programs.

> Concurrency is a way to structure software, particularly as a way to write clean code that interacts well with the real world.</p>
> -- <cite>**Rob Pike**</cite>

## Are Concurrent and Parallel Programming same thing?

{{< img-post "date" "no-no.webp" "No" "center" >}}

**Concurrency** is when two or more tasks can start, run, and complete in overlapping time periods. It doesn't necessarily mean they'll ever both be running at the same instant. For example, multitasking on a single-core machine.

**Parallelism** is when tasks literally run at the same time, e.g., on a multicore processor.

Concurrent execution is the generalised form of parallel execution, i.e a concurrent program might run in parallel on a multiprocessor. Concurrency **enables** parallelism but **isn't** parallelism.

## Why Go?
Go has rich support for concurrency using goroutines and channels. Writing a concurrent code in go is as simple as creating a goroutine using the keyword `go` followed by a function invocation.

```go
go myConcurrentFunc()
```
Concurrent programming in many environments is made difficult by the subtleties required to implement correct access to shared variables. Go encourages a different approach in which shared values are passed around on channels and, in fact, never actively shared by separate threads of execution. Only one goroutine has access to the value at any given time. Data races cannot occur, by design. Go approach is: **"Don't communicate by sharing memory, share memory by communicating."**

Go has channels as a first class value. Goroutines don't talk to each other directly but rather talk to a channel and the other end could be something that processes the values that are being sent to it.

Goroutines behave like threads but technically; it is an abstraction over threads.

When we run a go program, go runtime will create few threads on a core on which all the goroutines are spawned. At any point in time, one thread can have multiple goroutines running on it, handled by go runtime which is much faster than traditional thread scheduling.

To compare some of the major differences between threads and goroutines here's a table below:

{{< gist mridul-sahu 1d7a8686c82ad397167153baf9f82cfe >}}

## Let's show some code!

Here are some concurrency patterns with example:

### [Generator](#generator) {#generator}
We launch the goroutine inside a function and return back to the caller the channel with which to communicate with the goroutine. 
```go
func BoringGenerator(msg string) <-chan string {
    c := make(chan string)
    
    go func() {
        for i := 0; ; i++ {
            c <- fmt.Sprintf("%s %d", msg, i)
            time.Sleep(time.Second)
        }
    }()
    return c
}
```

### [Fan In](#fan-in) {#fan-in}
Multiplexing multiple channels for input, we stitch together many channels into a single channel. 
```go
func BoringFanIn(chans ... <-chan string) <-chan string {
    c := make(chan string)
    for _, v := range chans {
        go func(<-chan string) {
            for {
                c <- <-v
            }
        }(v)
    }
    return c
}
```

### [Fan In (single goroutine)](#fan-in-single) {#fan-in-single}
If the number of input channels is known beforehand we can achieve a fan-in using single goroutine.
```go
func BoringFanInFixedArguments(input1, input2 <-chan string) <-chan string {
    c := make(chan string)

    go func() {
        for {
            select {
            case s := <-input1:
                c <- s
            case s := <-input2:
                c <- s
            }
        }
    }()
    return c
}
```

### [Fan Out (All)](#fan-out-all) {#fan-out-all}
If you have multiple channels processing input from a single channel we send data to all of then when available.
```go
func FanOutAll(input <-chan string, outs ... chan<- string) {
    go func() {
        for {
            msg := <-input
            for _, out := range outs {
                go func(o chan<- string) {
                    o <- msg
                }(out)
            }
        }
    }()
}
```

### [Fan Out (Any)](#fan-out-any) {#fan-out-any}
If you want only one of the known number of output channels to process a single channel we send data to whoever is available at the moment.
```go
func FanOutAny(input <-chan string, out1, out2 chan<- string) {
    go func() {
        for {
            msg := <-input
            select {
            case out1 <- msg:
            case out2 <- msg:
            }
        }
    }()
}
```

### [Turnout](#turnout) {#turnout}
Processing data from multiple input channels via anyone of the output channels.
```go
func Turnout(inp1, inp2 <-chan string, out1, out2 chan<- string) {
    c := BoringFanInFixedArguments(inp1, inp2)
    FanOutAny(c, out1, out2)
}
```

### [Receive Till Quit](#receive-till-quit) {#receive-till-quit}
Receiving data from an input channel till a quit signal is received.
```go
func BoringReceiveTillQuit(input <-chan string, quit <-chan bool) {
    for {
        select {
        case s := <-input:
            fmt.Print(s)
        case <-quit:
            fmt.Print("Oh ok that was nice talk")
            return
        }
    }
}
```

### [Synced Quit](#synced-quit) {#synced-quit}
Processing data till a quit signal is received and then acknowledge it.
```go
func BoringSyncedQuit(input <-chan string, quit chan bool) {
    for {
        select {
        case s := <-input:
            fmt.Print(s)
        case <-quit:
            fmt.Print("Well see ya, bye")
            quit <- true
        }
    }
}
```

### [Send When Available](#send-when-available) {#send-when-available}
Waiting for a request and then sending an output back when available (after processing the request).
```go
type Request struct {
    query string
    data  chan string
}

func SendWhenAvailable() chan<- Request {
    r := make(chan Request)

    go func(r <-chan Request) {
        for {
            request := <-r
            time.Sleep(time.Millisecond * 100)
            request.data <- fmt.Sprintf("%s is a very interesting question indeed!", request.query)
        }
    }(r)
    return r
}
```

### [Timeout](#timeout) {#timeout}
Processing inputs till an overall timeout occurs.
```go
func BoringTimeoutEntireConversation(input <-chan string) {
    timeout := time.After(time.Second * 10)
    for {
        select {
        case s := <-input:
            fmt.Print(s)
        case <-timeout:
            fmt.Print("That's it, I am done")
            return
        }
    }
}
```

### [Timeout Per Event](#timeout-per-event) {#timeout-per-event}
Process events till a timeout occurs in receiving an input event.
```go
func BoringTimeoutPerMessage(input <-chan string) {
    for {
        select {
        case s := <-input:
            fmt.Print(s)
        case <-time.After(time.Second * 3):
            fmt.Print("Too boring, I am leaving")
            return
        }
    }
}
```

## References

1. Check out the awesome talk on **_Go Concurrency Patterns_** (Google I/O - 2012) by **_Rob Pike_**:
    {{< youtube f6kdp27TYZs >}}

2. Another talk about **_Advanced Go Concurrency Patterns_** (Google I/O - 2013) by **_Sameer Ajmani_**:
    {{< youtube QDDwwePbDtw >}}

Feel free to checkout a list of examples using these patterns on my github {{< url-link "Github" "https://github.com/mridul-sahu/golang-concurrency-patterns" >}}:bowtie:
