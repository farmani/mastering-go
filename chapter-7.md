# Go Concurrency
The key component of the Go concurrency model is the goroutine, which is the minimum executable entity in Go. Everything in Go is executed as a goroutine, either transparently or consciously. Each executable Go program has at least one goroutine, which is used for running the main() function of the main package. Each goroutine is executed on a single OS thread according to the instructions of the Go scheduler, which is responsible for the execution of goroutines. The OS scheduler does not dictate how many threads the Go runtime is going to create because the Go runtime will spawn enough threads to ensure that GOMAXPROCS threads are available to run Go code.
However, goroutines cannot directly communicate with each other. Data sharing in Go is implemented using either channels or shared memory. Channels act as the glue that connects multiple goroutines. Remember that although goroutines can process data and execute commands, they cannot communicate directly with each other, but they can communicate in other ways, including channels, local sockets, and shared memory. On the other hand, channels cannot process data or execute code but can send data to goroutines, receive data from goroutines, or have a special purpose.

When you combine multiple channels and goroutines you can create data flows, which in Go terminology are also called pipelines. So, you might have a goroutine that reads data from a database and sends it to a channel and a second goroutine that reads from that channel, processes that data, and sends it to another channel in order to be read from another goroutine, before making modifications to the data and storing it to a different database.

Excerpt From
Mastering Go
Mihalis Tsoukalos
This material may be protected by copyright.

### Processes, threads, and goroutines
A process is an OS representation of a running program, while a program is a binary file on disk that contains all the information necessary for creating an OS process. The binary file is written in a specific format (ELF on Linux) and contains all the instructions the CPU is going to run as well as a plethora of other useful sections. That program is loaded into memory and the instructions are executed, creating a running process. So, a process carries with it additional resources such as memory, opened file descriptions, and user data as well as other types of resources that are obtained during runtime.
A thread is a smaller and lighter entity than a process. Processes consist of one or more threads that have their own flow of control and stack. A quick and simplistic way to differentiate a thread from a process is to consider a process as the running binary file and a thread as a subset of a process.

A goroutine is the minimum Go entity that can be executed concurrently. The use of the word minimum is very important here, as goroutines are not autonomous entities like UNIX processes—goroutines live in OS threads that live in OS processes. The good thing is that goroutines are lighter than threads, which, in turn, are lighter than processes—running thousands or hundreds of thousands of goroutines on a single machine is not a problem. Among the reasons that goroutines are lighter than threads is because they have a smaller stack that can grow, they have a faster startup time, and they can communicate with each other through channels with low latency.
In practice, this means that a process can have multiple threads as well as lots of goroutines, whereas a goroutine needs the environment of a process to exist. So, to create a goroutine, you need to have a process with at least one thread. The OS takes care of the process and thread scheduling, while Go creates the necessary threads and the developer creates the desired number of goroutines.



### The Go scheduler
The OS kernel scheduler is responsible for the execution of the threads of a program. Similarly, the Go runtime has its own scheduler, which is responsible for the execution of the goroutines using a technique known as m:n scheduling, where m goroutines are executed using n OS threads using multiplexing. The Go scheduler is the Go component responsible for the way and the order in which the goroutines of a Go program get executed. This makes the Go scheduler a really important part of the Go programming language. The Go scheduler is executed as a goroutine.

> Be aware that as the Go scheduler only deals with the goroutines of a single program, its operation is much simpler, cheaper, and faster than the operation of the kernel scheduler.

Go uses the fork-join concurrency model. The fork part of the model, which should not be confused with the fork(2) system call, states that a child branch can be created at any point of a program. Analogously, the join part of the Go concurrency model is where the child branch ends and joins with its parent. Keep in mind that both sync.Wait() statements and channels that collect the results of goroutines are join points, whereas each new goroutine creates a child branch.

The fair scheduling strategy, which is pretty straightforward and has a simple implementation, shares all load evenly among the available processors. At first, this might look like the perfect strategy because it does not have to take many things into consideration while keeping all processors equally occupied. However, it turns out that this is not exactly the case because most distributed tasks usually depend on other tasks. Therefore, some processors are underutilized, or equivalently, some processors are utilized more than others. A goroutine is a task, whereas everything after the calling statement of a goroutine is a continuation. In the work-stealing strategy used by the Go scheduler, a (logical) processor that is underutilized looks for additional work from other processors.

When it finds such jobs, it steals them from the other processor or processors, hence the name. Additionally, the work-stealing algorithm of Go queues and steals continuations. A stalling join, as is suggested by its name, is a point where a thread of execution stalls at a join and starts looking for other work to do.
Although both task stealing and continuation stealing have stalling joins, continuations happen more often than tasks; therefore, the Go scheduling algorithm works with continuations rather than tasks.
The main disadvantage of continuation stealing is that it requires extra work from the compiler of the programming language. Fortunately, Go provides that extra help and therefore uses continuation stealing in its work-stealing algorithm. One of the benefits of continuation stealing is that you get the same results when using function calls instead of goroutines or a single thread with multiple goroutines. This makes perfect sense, as only one thing is executed at any given point in both cases.

The Go scheduler works using three main kinds of entities: OS threads (M), which are related to the OS in use; goroutines (G); and logical processors (P). The number of processors that can be used by a Go program is specified by the value of the GOMAXPROCS environment variable—at any given time, there are at most GOMAXPROCS processors.

The next figure shows that there are two different kinds of queues: a global run queue and a local run queue attached to each logical processor. Goroutines from the global queue are assigned to the queue of a logical processor in order to get executed at some point.

![](scheduler.jpg)

Each logical processor can have multiple threads, and the stealing occurs between the local queues of the available logical processors. Finally, keep in mind that the Go scheduler is allowed to create more OS threads when needed. OS threads are pretty expensive in terms of resources, which means that dealing too much with OS threads might slow down your Go applications.


#### The GOMAXPROCS environment variable
The GOMAXPROCS environment variable allows you to set the number of OS threads (CPUs) that can execute user-level Go code simultaneously. Starting with Go version 1.5, the default value of GOMAXPROCS should be the number of logical cores available in your machine. There is also the runtime.GOMAXPROCS() function, which allows you to set and get the value of GOMAXPROCS programmatically.
If you decide to assign a value to GOMAXPROCS that is smaller than the number of the cores in your machine, you might affect the performance of your program. However, using a GOMAXPROCS value that is larger than the number of the available cores does not necessarily make your Go programs run faster.

```go
func main() {
    fmt.Print("You are using ", runtime.Compiler, " ")
    fmt.Println("on a", runtime.GOARCH, "machine")
    fmt.Println("Using Go version", runtime.Version())
    fmt.Printf("GOMAXPROCS: %d\n", runtime.GOMAXPROCS(0))
}
```
What happens with the runtime.GOMAXPROCS(0) call? runtime.GOMAXPROCS() always returns the previous value of the maximum number of CPUs that can be executing simultaneously. When the parameter of runtime.GOMAXPROCS() is equal to or bigger than 1, then runtime.GOMAXPROCS() also changes the current setting. As we are using 0, our call does not alter the current setting.

You can change the value of GOMAXPROCS on the fly using the next technique:

```shell
$ export GOMAXPROCS=100; go run maxprocs.go
You are using gc on a amd64 machine
Using Go version go1.16.2
GOMAXPROCS: 100
```
you will most likely not need to change GOMAXPROCS


#### Concurrency and parallelism
It is a common misconception that concurrency is the same thing as parallelism—this is just not true! Parallelism is the simultaneous execution of multiple entities of some kind, whereas concurrency is a way of structuring your components so that they can be executed independently when possible.

In a valid concurrent design, adding concurrent entities makes the whole system run faster because more things can be executed in parallel. So, the desired parallelism comes from a better concurrent expression and implementation of the problem. The developer is responsible for taking concurrency into account during the design phase of a system and will benefit from a potential parallel execution of the components of the system. So, the developer should not think about parallelism but about breaking things into independent components that solve the initial problem when combined.
Even if you cannot run your functions in parallel on your machine, a valid concurrent design still improves the design and the maintainability of your programs.

## Goroutines
You can define, create, and execute a new goroutine using the go keyword followed by a function name or an anonymous function. The go keyword makes the function call return immediately, while the function starts running in the background as a goroutine and the rest of the program continues its execution. You cannot control or make any assumptions about the order in which your goroutines are going to be executed because that depends on the scheduler of the OS, the Go scheduler, and the load of the OS.

### Creating a goroutine

```go
func main() {
    go func (x int) {
        fmt.Printf("%d ", x)
    }(10)
    go printme(15)
    time.Sleep(time.Second)
    fmt.Println("Exiting...")
}
```
The (10) at the end is how you pass a parameter to an anonymous function.

This is how you execute a function as a goroutine. As a general rule of thumb, the functions that you execute as goroutines do not return any values directly. Exchanging data with goroutines happens via the use of shared memory or channels or some other mechanism.

As a Go program does not wait for its goroutines to end before exiting, we need to delay it manually, which is the purpose of the time.Sleep() call. We will correct that shortly in order to wait for all goroutines to finish before exiting.
### Creating multiple goroutines

```go
fmt.Printf("Going to create %d goroutines.\n", count)
for i := 0; i < count; i++ {
 	go func(x int) {
        fmt.Printf("%d ", x)
    }(i)
}
time.Sleep(time.Second)
fmt.Println("\nExiting...")
```
There is nothing prohibiting you from using a for loop to create multiple goroutines

### Waiting for your goroutines to finish
you also need to wait for them to finish before the main() function ends.

The synchronization process begins by defining a sync.WaitGroup variable and using the Add(), Done() and Wait() methods. If you look at the source code of the sync Go package, and more specifically at the waitgroup.go file, you see that the sync.WaitGroup type is nothing more than a structure with two fields:

```go
type WaitGroup struct {
    noCopy noCopy
    state1 [3]uint32
}
```
Each call to sync.Add() increases a counter in the state1 field, which is an array with three uint32 elements. Notice that it is really important to call sync.Add() before the go statement in order to prevent any race conditions—we are going to learn about race conditions in the Race conditions section. When each goroutine finishes its job, the sync.Done() function should be executed in order to decrease the same counter by one. Behind the scenes, sync.Done() runs a Add(-1) call. The Wait() method waits until that counter becomes 0 in order to return. The return of Wait() inside the main() function means that main() is going to return and the program ends.

> You can call Add() with a positive integer value other than 1 in order to avoid calling Add(1) multiple times. This can be handy when you know the number of goroutines you are going to create in advance. Done() does not support that functionality.

```go
var waitGroup sync.WaitGroup
fmt.Printf("%#v\n", waitGroup)
for i := 0; i < count; i++ {
    waitGroup.Add(1)
	go func(x int) {
        defer waitGroup.Done()
		fmt.Printf("%d ", x)
    }(i)
}
fmt.Printf("%#v\n", waitGroup)
waitGroup.Wait()
```

This is where you create a sync.WaitGroup variable that you are going to use. The fmt.Printf() call prints the contents of the sync.WaitGroup structure—you do not usually do that but it is good for learning more about the sync.WaitGroup structure.

We call Add(1) just before we create the goroutine in order to avoid race conditions.

The Done() call is going to be executed just before the anonymous function returns because of the defer keyword.

The Wait() function is going to wait for the counter in the waitGroup variable to become 0 before it returns, which is what we want to achieve.

When the Wait() function returns, the fmt.Println() statement is going to be executed.

> Remember that using more goroutines in a program is not a panacea for performance, as more goroutines, in addition to the various calls to sync.Add(), sync.Wait(), and sync.Done(), might slow down your program due to the extra housekeeping that needs to be done by the Go scheduler.


### What if the number of Add() and Done() calls differ?

this subsection tells you what happens when these two numbers do not agree with each other.

The cause of the error message can be found in the output: panic: sync: negative WaitGroup counter—caused by more calls to Done() than calls to Add(). Note that sometimes addDone.go does not produce any error messages and terminates just fine. This is an issue with concurrent programs in general—they do not always crash or misbehave as the order of execution can change, which might change the behavior of the program. This makes debugging even more difficult.

fatal error: all goroutines are asleep - deadlock!. This means that the program should wait indefinitely for a goroutine to finish, that is, for a Done() call that is never going to happen.

### Creating multiple files with goroutines

```go
var waitGroup sync.WaitGroup
for i := start; i <= end; i++ {
    waitGroup.Add(1)
    filepath := fmt.Sprintf("%s/%s%d", path, filename, i)
    go func(f string) {
        defer waitGroup.Done()
        createFile(f)
    }(filepath)
}
waitGroup.Wait()
```
## Channels
A channel is a communication mechanism that, among other things, allows goroutines to exchange data. Firstly, each channel allows the exchange of a particular data type, which is also called the element type of the channel, and secondly, for a channel to operate properly, you need someone to receive what is sent via the channel. You should declare a new channel using make() and the chan keyword (make(chan int)), and you can close a channel using the close() function. You can declare the size of a channel by writing something like make(chan int, 1).

A pipeline is a virtual method for connecting goroutines and channels so that the output of one goroutine becomes the input of another goroutine using channels to transfer your data. One of the benefits that you get from using pipelines is that there is a constant data flow in your program, as no goroutine or channel has to wait for everything to be completed in order to start their execution. Additionally, you use fewer variables and therefore less memory space because you do not have to save everything as a variable. Finally, the use of pipelines simplifies the design of the program and improves its maintainability.

### Writing to and reading from a channel
Writing the value val to channel ch is as easy as writing ch <- val. The arrow shows the direction of the value, and you will have no problem with this statement as long as both var and ch are of the same data type.
You can read a single value from a channel named c by executing <-c. In this case, the direction is from the channel to the outer world. You can save that value into a variable using aVar := <-c.

```go
package main
import (
    "fmt"
    "sync"
)
func writeToChannel(c chan int, x int) {
    c <- x
    close(c)
}

func printer(ch chan bool) {
    ch <- true
}

func main() {
	c := make(chan int, 1)
	var waitGroup sync.WaitGroup
	waitGroup.Add(1)
	go func(c chan int) {
		defer waitGroup.Done()
		writeToChannel(c, 10)
		fmt.Println("Exit.")
	}(c)
	fmt.Println("Read:", <-c)
	_, ok := <-c
	if ok {
		fmt.Println("Channel is open!")
	} else {
		fmt.Println("Channel is closed!")
	}
	
	waitGroup.Wait()
	var ch chan bool = make(chan bool)
	for i := 0; i < 5; i++ {
		go printer(ch)
	}

	// Range on channels
	// IMPORTANT: As the channel c is not closed,
	// the range loop does not exit on its own.
	n := 0
	for i := range ch {
		fmt.Println(i)
		if i == true {
			n++
		}
		if n > 2 {
			fmt.Println("n:", n)
			close(ch)
			break
		}
	}
	
	for i := 0; i < 5; i++ {
		fmt.Println(<-ch)
	}
}
```
This channel is buffered with a size of 1. This means that as soon as we fill that buffer, we can close the channel and the goroutine is going to continue its execution and return. A channel that is unbuffered has a different behavior: when you try to send a value to that channel, it blocks forever because it is waiting for someone to fetch that value. In this case, we definitely want a buffered channel in order to avoid any blocking.

The previous code shows a technique for determining whether a channel is closed or not. In this case, we are ignoring the read value—if the channel was open, then the read value would be discarded.

The range keyword works with channels! However, a range loop on a channel only exits when the channel is closed or using the break keyword.


When trying to read from a closed channel, we get the zero value of its data type, so this for loop works just fine and does not cause any issues.

there is a logical issue with it, which we will explain and resolve in the Race conditions section. Additionally, if we run channels.go multiple times, it might crash. However, most of the time it does not, which makes debugging even more challenging.

### Receiving from a closed channel
Reading from a closed channel returns the zero value of its data type. However, if you try to write to a closed channel, your program is going to crash in a bad way (panic).

```go
func main() {
    willClose := make(chan complex64, 10)

	// Write some data to the channel
    willClose <- -1
    willClose <- 1i

        // Read data and empty channel
    <-willClose
    <-willClose
    close(willClose)

    // Read again – this is a closed channel
    read := <-willClose
    fmt.Println(read)
}
```
### Channels as function parameters
When using a channel as a function parameter, you can specify its direction; that is, whether it is going to be used for sending or receiving data. In my opinion, if you know the purpose of a channel in advance, you should use this capability because it makes your programs more robust. You will not be able to send data accidentally to a channel from which you should only receive data or receive data from a channel to which you should only be sending data. As a result, if you declare that a channel function parameter is going to be used for reading only and you try to write to it, you get an error message that will most likely save you from nasty bugs in the future.

```go
func printer(ch chan<- bool) {
    ch <- true
}

func writeToChannel(c chan<- int, x int) {
    fmt.Println("1", x)
    c <- x
    fmt.Println("2", x)
}

func f2(out <-chan int, in chan<- int) {
    x := <-out
    fmt.Println("Read (f2):", x)
    in <- x
    return
}
```
This happens even if the function is not being used.

## Race conditions
A data race condition is a situation where two or more running elements, such as threads and goroutines, try to take control of or modify a shared resource or shared variable of a program. Strictly speaking, a data race occurs when two or more instructions access the same memory address, where at least one of them performs a write (change) operation. If all operations are read operations, then there is no race condition. In practice, this means that you might get different output if you run your program multiple times, and that is a bad thing.
Using the -race flag when running or building Go source files executes the Go race detector, which makes the compiler create a modified version of a typical executable file. This modified version can record all accesses to shared variables as well as all synchronization events that take place, including calls to sync.Mutex and sync.WaitGroup, which are presented later on in this chapter. After analyzing the relevant events, the race detector prints a report that can help you identify potential problems so that you can correct them.

### The Go race detector
`go run -race`

```shell
$ go run -race channels.go 
Exit.
Read: 10
Channel is closed!
true
true
true
n: 3
==================
WARNING: DATA RACE
Write at 0x00c00006e010 by main goroutine:
  runtime.closechan()
      /usr/local/Cellar/go/1.16.2/libexec/src/runtime/chan.go:355 +0x0
  main.main()
      /Users/mtsouk/ch07/channels.go:54 +0x46c
Previous read at 0x00c00006e010 by goroutine 12:
  runtime.chansend()
      /usr/local/Cellar/go/1.16.2/libexec/src/runtime/chan.go:158 +0x0
  main.printer()
      /Users/mtsouk/ch07/channels.go:14 +0x47
Goroutine 12 (running) created at:
  main.main()
      /Users/mtsouk/ch07/channels.go:40 +0x2b4
==================
false
false
false
false
false
Found 1 data race(s)
exit status 66
```
The issue is that we cannot be sure about what is going to happen and in which order

```go
func printer(ch chan<- bool, times int) {
    for i := 0; i < times; i++ {
        ch <- true
    }
    close(ch)
}
```
The first thing to notice here is that instead of using multiple goroutines for writing to the desired channel, we use a single goroutine. A single goroutine writing to a channel followed by the closing of that channel cannot create any race conditions because things happen sequentially.

```go
func main() {
    // This is an unbuffered channel
    var ch chan bool = make(chan bool)
    // Write 5 values to channel with a single goroutine
    go printer(ch, 5)
    // IMPORTANT: As the channel c is closed,
    // the range loop is going to exit on its own.
    for val := range ch {
        fmt.Print(val, " ")
    }
    fmt.Println()
    for i := 0; i < 15; i++ {
        fmt.Print(<-ch, " ")
    }
    fmt.Println()
}
```
## The select keyword
The select keyword is really important because it allows you to listen to multiple channels at the same time. A select block can have multiple cases and an optional default case, which mimics the switch statement. It is good for select blocks to have a timeout option just in case. Last, a select without any cases (select{}) waits forever.
In practice, this means that select allows a goroutine to wait on multiple communication operations. So, select gives you the power to listen to multiple channels using a single select block. As a consequence, you can have nonblocking operations on channels, provided that you have implemented your select blocks appropriately.

A select statement is not evaluated sequentially, as all of its channels are examined simultaneously. If none of the channels in a select statement are ready, the select statement blocks (waits) until one of the channels is ready. If multiple channels of a select statement are ready, then the Go runtime makes a random selection from the set of these ready channels.

```go
wg.Add(1)
go func() {
    gen(0, 2*n, createNumber, end)
    wg.Done()
}()
func gen(min, max int, createNumber chan int, end chan bool) {
    time.Sleep(time.Second)
    for {
        select {
            case createNumber <- rand.Intn(max-min) + min:
            case <-end:
                fmt.Println("Ended!")
                // return
            case <-time.After(4 * time.Second):
                fmt.Println("time.After()!")
                return
        }
    }
}
```
The right thing to do here is add the return statement for gen() to finish. But let us imagine that you have forgotten to add the return statement. 

As stated earlier, select does not require a default branch. You can consider the third branch of the select statement as a clever default branch. This happens because time.After() waits for the specified duration (4 * time.Second) to elapse and then prints a message and properly ends gen() with return.

## Timing out a goroutine
There are times that goroutines take more time than expected to finish—in such situations, we want to time out the goroutines so that we can unblock the program.

### Timing out a goroutine – inside main()
```go
func main() {
    c1 := make(chan string)
    go func() {
        time.Sleep(3 * time.Second)
        c1 <- "c1 OK"
    }()
    select {
    case res := <-c1:
        fmt.Println(res)
    case <-time.After(time.Second):
        fmt.Println("timeout c1")
    }
    c2 := make(chan string)
    go func() {
        time.Sleep(3 * time.Second)
        c2 <- "c2 OK"
    }()
    select {
    case res := <-c2:
        fmt.Println(res)
    case <-time.After(4 * time.Second):
        fmt.Println("timeout c2")
    }
}
```
The purpose of the time.After() call is to wait for the desired time before being executed—if another branch is executed, the waiting time resets. In this case, we are not interested in the actual value returned by time.After() but in the fact that the time.After() branch was executed, which means that the waiting time has passed.

The reason for saying "most likely" is that Linux is not a real-time OS and sometimes the OS scheduler plays strange games, especially when it has to deal with a high load and has to schedule lots of tasks.

Excerpt From
Mastering Go
Mihalis Tsoukalos
This material may be protected by copyright.
### Timing out a goroutine – outside main()
Once again, the select block holds the logic of the time out.

## Go channels revisited
the zero value of the channel type is nil, and that if you send a message to a closed channel, the program panics. However, if you try to read from a closed channel, you get the zero value of the type of that channel. So, after closing a channel, you can no longer write to it, but you can still read from it. To be able to close a channel, the channel must not be receive-only.
a nil channel always blocks, which means that both reading and writing from nil channels blocks. This property of channels can be very useful when you want to disable a branch of a select statement by assigning the nil value to a channel variable. Finally, if you try to close a nil channel, your program is going to panic.

The previous statement defines a nil channel named c of type string.

```go
package main
func main() {
    var c chan string
	close(c)
}
```
### Buffered channels
These channels allow us to put jobs in a queue quickly in order to be able to deal with more requests and process requests later on. Moreover, you can use buffered channels as semaphores in order to limit the throughput of your application.
The presented technique works as follows: all incoming requests are forwarded to a channel, which processes them one by one. When the channel is done processing a request, it sends a message to the original caller saying that it is ready to process a new one. So, the capacity of the buffer of the channel restricts the number of simultaneous requests that it can keep.

```go
package main
import (
    "fmt"
)
func main() {
    numbers := make(chan int, 5)

    counter := 10
    for i := 0; i < counter; i++ {
        select {
        // This is where the processing takes place
        case numbers <- i * i:
			fmt.Println("About to process", i)
		default:
			fmt.Print("No space for ", i, " ")
		}
    }
    fmt.Println()
    for {
        select {
        case num := <-numbers:
            fmt.Print("*", num, " ")
        default:
            fmt.Println("Nothing left to read!")
            return
        }
    }
}
```
### nil channels
nil channels always block! Therefore, you should use them when you want that behavior on purpose!

We are making wg a global variable in order to be available from anywhere in the code and avoid passing it to every function that needs it.

The send() function keeps sending random numbers to the c channel. Do not confuse channel c, which is a channel function parameter, with channel t.C, which is part of timer t—you can change the name of the c variable but not the name of the C field. When the time of timer t expires, the timer sends a value to the t.C channel.
This triggers the execution of the relevant branch of the select statement, which assigns the value nil to channel c, prints the value of the sum variable and wg.Done() is executed, which is going to unblock wg.Wait() found in the main() function. Additionally, as c becomes nil, it stops/blocks send() from sending any data to it.

```go
package main
import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)
var wg sync.WaitGroup
func add(c chan int) {
	sum := 0
	t := time.NewTimer(time.Second)
	for {
		select {
		case input := <-c:
			sum = sum + input
		case <-t.C:
			c = nil
			fmt.Println(sum)
			wg.Done()
		}
	}
}
func send(c chan int) {
	for {
		c <- rand.Intn(10)
	}
}
func main() {
	c := make(chan int)
	rand.Seed(time.Now().Unix())
	wg.Add(1)
	go add(c)
	go send(c)
	wg.Wait()
}
```
### Worker pools
A worker pool is a set of threads that process jobs assigned to them.

threads do not usually die after serving a request because the cost of ending a thread and creating a new one is too high, whereas goroutines do die after finishing their job. Worker pools in Go are implemented with the help of buffered channels, because they allow you to limit the number of goroutines running at the same time.

```go
package main
import (
    "fmt"
    "os"
    "runtime"
    "strconv"
    "sync"
    "time"
)
type Client struct {
	id      int
	integer int
}
type Result struct {
	job    Client
	square int
}
var size = runtime.GOMAXPROCS(0)
var clients = make(chan Client, size)
var data = make(chan Result, size)
func worker(wg *sync.WaitGroup) {
	for c := range clients {
		square := c.integer * c.integer
		output := Result{c, square}
		data <- output
		time.Sleep(time.Second)
	}
	wg.Done()
}
func create(n int) {
	for i := 0; i < n; i++ {
		c := Client{i, i}
		clients <- c
	}
	close(clients)
}
func main() {
	if len(os.Args) != 3 {
		fmt.Println("Need #jobs and #workers!")
		return
	}
	nJobs, err := strconv.Atoi(os.Args[1])
	if err != nil {
		fmt.Println(err)
		return
	}
	nWorkers, err := strconv.Atoi(os.Args[2])
	if err != nil {
		fmt.Println(err)
		return
	}
	go create(nJobs)
	finished := make(chan interface{})
	go func() {
		for d := range data {
			fmt.Printf("Client ID: %d\tint: ", d.job.id)
			fmt.Printf("%d\tsquare: %d\n", d.job.integer, d.square)
		}
		finished <- true
		}()
	var wg sync.WaitGroup
	for i := 0; i < nWorkers; i++ {
		wg.Add(1)
		go worker(&wg)
	}
	wg.Wait()
	close(data)
	fmt.Printf("Finished: %v\n", <-finished)
}
```
Put simply, the Client structure holds the input data of each request, whereas Result holds the results of a request—if you want to process complex data, you should modify these structures.

The clients and data buffered channels are used to get new client requests and write the results, respectively. If you want your program to run faster, you can increase the value of size.

The delay that is introduced with time.Sleep() is not necessary, but it gives you a better sense of the way that the generated output is printed.

The finished channel is used for blocking the program and, therefore, needs no particular data type.

The finished <- true statement is used for unblocking the program as soon as the for range loop ends. The for range loop ends when the data channel is closed, which happens after wg.Wait(), which means after all workers have finished.
### Signal channels
A signal channel is one that is used just for signaling. Put simply, you can use a signal channel when you want to inform another goroutine about something. Signal channels should not be used for data transferring. 

#### Specifying the order of execution for your goroutines
have in mind that this technique works best when you are dealing with a small number of goroutines. The presented code example has four goroutines that we want to execute in the desired order—first, goroutine for function A(), then function B(), then C(), and finally, D().

```go
var wg sync.WaitGroup
func A(a, b chan struct{}) {
    <-a
    fmt.Println("A()!")
    time.Sleep(time.Second)
	close(b)
}
//Function A() is going to be blocked until channel a, which is passed as a parameter, is closed.

func B(a, b chan struct{}) {
    <-a
    fmt.Println("B()!")
    time.Sleep(3 * time.Second)
    close(b)
}

func C(a, b chan struct{}) {
    <-a
    fmt.Println("C()!")
    close(b)
}

func D(a chan struct{}) {
    <-a
    fmt.Println("D()!")
    wg.Done()
}
func main() {
    x := make(chan struct{})
    y := make(chan struct{})
    z := make(chan struct{})
    w := make(chan struct{})

    wg.Add(1)
    go func() {
        D(w)
    }()

    wg.Add(1)
    go func() {
        D(w)
    }()
    go A(x, y)
    wg.Add(1)
    go func() {
        D(w)
    }()
    go C(z, w)
    go B(y, z)

    wg.Add(1)
    go func() {
        D(w)
    }()
    // This triggers the process
    close(x)
    
    wg.Wait()
}
```
a channel can be closed only once.

Although we run C() before B(), C() is going to finish after B() has finished.
The closing of the first channel is what triggers the execution of the goroutines because this unblocks A().
So, the four functions, which are executed as goroutines, are executed in the desired order, and, in the case of the last function, the desired number of times.

## Shared memory and shared variables
A mutex variable, which is an abbreviation of mutual exclusion variable, is mainly used for thread synchronization and for protecting shared data when multiple writes can occur at the same time. A mutex works like a buffered channel with a capacity of one, which allows at most one goroutine to access a shared variable at any given time. This means that there is no way for two or more goroutines to be able to update that variable simultaneously. Go offers the sync.Mutex and sync.RWMutex data types.
A critical section of a concurrent program is the code that cannot be executed simultaneously by all processes, threads, or, in this case, goroutines. It is the code that needs to be protected by mutexes. Therefore, identifying the critical sections of your code makes the whole programming process so much simpler that you should pay particular attention to this task. A critical section cannot be embedded into another critical section when both critical sections use the same sync.Mutex or sync.RWMutex variable.
avoid at almost any cost spreading mutexes across functions because that makes it really hard to see whether you are embedding or not.

### The sync.Mutex type
The sync.Mutex type is the Go implementation of a mutex. Its definition, which can be found in the mutex.go file of the sync directory, is as follows

```go
type Mutex struct {
    state int32
    sema  uint32
}
```

The definition of sync.Mutex is nothing special. All of the interesting work is done by the sync.Lock() and sync.Unlock() functions, which can lock and unlock a sync.Mutex variable, respectively. Locking a mutex means that nobody else can lock it until it has been released using the sync.Unlock() function.

```go
package main
import (
    "fmt"
    "os"
    "strconv"
    "sync"
    "time"
)
var m sync.Mutex
var v1 int
func change(i int) {
    m.Lock()
	time.Sleep(time.Second)
	v1 = v1 + 1
	if v1 == 10 {
		v1 = 0
		fmt.Print("* ")
	}
	m.Unlock()
}
func read() int {
	m.Lock()
	a := v1
	m.Unlock()
	return a
}
```
#### What happens if you forget to unlock a mutex?
Forgetting to unlock a sync.Mutex mutex creates a panic situation even in the simplest kind of a program. The same applies to the sync.RWMutex mutex, which is presented in the next section

```go
var m sync.Mutex
var w sync.WaitGroup
func function() {
    m.Lock()
    fmt.Println("Locked!")
}
```
As expected, the program crashes because of the deadlock. To avoid such situations, always remember to unlock any mutexes created in your program.

## The sync.RWMutex type

```go
type RWMutex struct {
    w           Mutex
	writerSem   uint32
	readerSem   uint32
readerCount int32
readerWait  int32
}
```
sync.RWMutex is based on sync.Mutex with the necessary additions and improvements. So, you might ask, how does sync.RWMutex improve sync.Mutex? Although a single function is allowed to perform write operations with a sync.RWMutex mutex, you can have multiple readers owning a sync.RWMutex mutex—this means that read operations are usually faster with sync.RWMutex. However, there is one important detail that you should be aware of: until all of the readers of a sync.RWMutex mutex unlock that mutex, you cannot lock it for writing, which is the small price you have to pay for the performance improvement you get for allowing multiple readers.
The functions that can help you to work with sync.RWMutex are RLock() and RUnlock(), which are used for locking and unlocking the mutex for reading purposes, respectively. The Lock() and Unlock() functions used in sync.Mutex should still be used when you want to lock and unlock a sync.RWMutex mutex for writing purposes. Finally, it should be apparent that you should not make changes to any shared variables inside an RLock() and RUnlock() block of code

sync.RWMutex is based on sync.Mutex with the necessary additions and improvements. So, you might ask, how does sync.RWMutex improve sync.Mutex? Although a single function is allowed to perform write operations with a sync.RWMutex mutex, you can have multiple readers owning a sync.RWMutex mutex—this means that read operations are usually faster with sync.RWMutex. However, there is one important detail that you should be aware of: until all of the readers of a sync.RWMutex mutex unlock that mutex, you cannot lock it for writing, which is the small price you have to pay for the performance improvement you get for allowing multiple readers.
The functions that can help you to work with sync.RWMutex are RLock() and RUnlock(), which are used for locking and unlocking the mutex for reading purposes, respectively. The Lock() and Unlock() functions used in sync.Mutex should still be used when you want to lock and unlock a sync.RWMutex mutex for writing purposes. Finally, it should be apparent that you should not make changes to any shared variables inside an RLock() and RUnlock() block of code

```go
var Password *secret
var wg sync.WaitGroup
type secret struct {
    RWM      sync.RWMutex
    password string
}

func Change(pass string) {
    fmt.Println("Change() function")
    Password.RWM.Lock()
    
    This is the beginning of the critical section.
    fmt.Println("Change() Locked")
    time.Sleep(4 * time.Second)
    Password.password = pass
    Password.RWM.Unlock()
    
    This is the end of the critical section.
    fmt.Println("Change() UnLocked")
}

func show () {
    defer wg.Done()
    Password.RWM.RLock()
    fmt.Println("Show function locked!")
    time.Sleep(2 * time.Second)
    fmt.Println("Pass value:", Password.password)
    defer Password.RWM.RUnlock()
}
```

## The atomic package
An atomic operation is an operation that is completed in a single step relative to other threads or, in this case, to other goroutines. This means that an atomic operation cannot be interrupted in the middle of it. The Go Standard library offers the atomic package, which, in some simple cases, can help you to avoid using a mutex. With the atomic package, you can have atomic counters accessed by multiple goroutines without synchronization issues and without worrying about race conditions. However, mutexes are more versatile than atomic operations.

when using an atomic variable, all reading and writing operations of an atomic variable must be done using the functions provided by the atomic package in order to avoid race conditions.

```go
package main
import (
    "fmt"
    "sync"
    "sync/atomic"
)
type atomCounter struct {
    val int64
}

func (c *atomCounter) Value() int64 {
    return atomic.LoadInt64(&c.val)
}

func main() {
	X := 100
	Y := 4
	var waitGroup sync.WaitGroup
	counter := atomCounter{}
	for i := 0; i < X; i++ {
		waitGroup.Add(1)
		go func(no int) {
			defer waitGroup.Done()
			for i := 0; i < Y; i++ {
				atomic.AddInt64(&counter.val, 1)
			}
		}(i)
	}
	waitGroup.Wait()
	fmt.Println(counter.Value())
}
```
The atomic.AddInt64() function changes the value of the val field of the counter structure variable in a safe way

as stated before, the use of the atomic package for working with the shared variable offers a simple way of avoiding race conditions when changing the value of the shared variable.

## Sharing memory using goroutines
how to share data using a dedicated goroutine. Although shared memory is the traditional way that threads communicate with each other, Go comes with built-in synchronization features that allow a single goroutine to own a shared piece of data. This means that other goroutines must send messages to this single goroutine that owns the shared data, which prevents the corruption of the data. Such a goroutine is called a monitor goroutine. In Go terminology, this is sharing by communicating instead of communicating by sharing.

> Personally, I prefer to use a monitor goroutine instead of traditional shared memory techniques because the implementation with the monitor goroutine is safer, closer to the Go philosophy, and easier to understand.

```go
package main
import (
    "fmt"
    "math/rand"
    "os"
    "strconv"
    "sync"
    "time"
)
var readValue = make(chan int)
var writeValue = make(chan int)
func set(newValue int) {
    writeValue <- newValue
}

func read() int {
    return <-readValue
}

func monitor() {
	var value int
	for {
		select {
		case newValue := <-writeValue:
			value = newValue
			fmt.Printf("%d ", value)
		case readValue <- value:
		}
	}
}
func main() {
	if len(os.Args) != 2 {
		fmt.Println("Please give an integer!")
		return
	}
	n, err := strconv.Atoi(os.Args[1])
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("Going to create %d random numbers.\n", n)
	rand.Seed(time.Now().Unix())
	go monitor()

	var wg sync.WaitGroup
	for r := 0; r < n; r++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			set(rand.Intn(10 * n))
		}()
	}

	wg.Wait()
	fmt.Printf("\nLast value: %d\n", read())
}
```

The monitor() function contains the logic of the program with the endless for loop and the select statement. The first case receives data from the writeValue channel, sets the value variable accordingly, and prints that new value. The second case sends the value of the value variable to the readValue channel. As all traffic goes through monitor() and its select block, there is no way to have a race condition because there is a single instance of monitor() running.

## Closured variables and the go statement
about closured variables, which are variables inside closures, and the go statement. Notice that closured variables in goroutines are evaluated when the goroutine actually runs and when the go statement is executed in order to create a new goroutine. This means that closured variables are going to be replaced by their values when the Go scheduler decides to execute the relevant code

As i is a closured variable, it is evaluated at the time of execution
the for loop ends, so the value of i that is being used is 21. Lastly, the same issue also applies to Go channels, so be careful.

```go
func main() {
    for i := 0; i <= 20; i++ {
        i := i
        go func() {
            fmt.Print(i, " ")
        }()
    }
    time.Sleep(time.Second)
    fmt.Println()
    for i := 0; i <= 20; i++ {
        go func(x int) {
            fmt.Print(x, " ")
        }(i)
    }
    
    time.Sleep(time.Second)
    fmt.Println()
}
```
This is one way of correcting the issue. The valid yet bizarre i := i statement creates a new instance of the variable for the goroutine that holds the correct value.
This is a totally different way of correcting the race condition: pass the current value of i to the anonymous function as a parameter and everything is OK

## The context package
The main purpose of the context package is to define the Context type and support cancellation. Yes, you heard that right; there are times when, for some reason, you want to abandon what you are doing. However, it would be very helpful to be able to include some extra information about your cancellation decisions. The context package allows you to do exactly that

The Context type is an interface with four methods named Deadline(), Done(), Err(), and Value(). The good news is that you do not need to implement all of these functions of the Context interface—you just need to modify a Context variable using methods such as context.WithCancel(), context.WithDeadline(), and context.WithTimeout().

> All three of these functions return a derived Context (the child) and a CancelFunc() function. Calling the CancelFunc() function removes the parent's reference to the child and stops any associated timers. As a side effect, this means that the Go garbage collector is free to garbage collect the child goroutines that no longer have associated parent goroutines. For garbage collection to work correctly, the parent goroutine needs to keep a reference to each child goroutine. If a child goroutine ends without the parent knowing about it, then a memory leak occurs until the parent is canceled as well

we use context.Background() to initialize an empty Context. The other function that can create an empty Context is context.TODO()

The WithCancel() method returns a copy of parent context with a new Done channel. Notice that the cancel variable, which is a function, is one of the return values of context.CancelFunc(). The context.WithCancel() function uses an existing Context and creates a child with cancellation. The context.WithCancel() function also returns a Done channel that can be closed, either when the cancel() function is called, as shown in the preceding code, or when the Done channel of the parent context is closed

The cancel variable in f2() comes from context.WithTimeout(), which requires two parameters: a Context parameter and a time.Duration parameter. When the timeout period expires the cancel() function is called automatically.

```go
package main
import (
    "context"
    "fmt"
    "os"
    "strconv"
    "time"
)
func f1(t int) {
    c1 := context.Background()
    c1, cancel := context.WithCancel(c1)
    defer cancel()
	go func() {
		time.Sleep(4 * time.Second)
		cancel()
	}()
	select {
	case <-c1.Done():
		fmt.Println("f1() Done:", c1.Err())
		return
	case r := <-time.After(time.Duration(t) * time.Second):
		fmt.Println("f1():", r)
	}
	return
}

func f2(t int) {
	c2 := context.Background()
	c2, cancel := context.WithTimeout(c2, time.Duration(t)*time.Second)
	defer cancel()
	go func() {
		time.Sleep(4 * time.Second)
		cancel()
	}()
	select {
	case <-c2.Done():
		fmt.Println("f2() Done:", c2.Err())
		return
	case r := <-time.After(time.Duration(t) * time.Second):
		fmt.Println("f2():", r)
	}
	return
}
func f3(t int) {
	c3 := context.Background()
	deadline := time.Now().Add(time.Duration(2*t) * time.Second)
	c3, cancel := context.WithDeadline(c3, deadline)
	defer cancel()
	go func() {
		time.Sleep(4 * time.Second)
		cancel()
	}()
	select {
	case <-c3.Done():
		fmt.Println("f3() Done:", c3.Err())
		return
	case r := <-time.After(time.Duration(t) * time.Second):
		fmt.Println("f3():", r)
	}
	return
}

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Need a delay!")
		return
	}
	delay, err := strconv.Atoi(os.Args[1])
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("Delay:", delay)
	f1(delay)
	f2(delay)
	f3(delay)
}
```

The cancel variable in f3() comes from context.WithDeadline(). context.WithDeadline() requires two parameters: a Context variable and a time in the future that signifies the deadline of the operation. When the deadline passes, the cancel() function is called automatically

### Using context as a key/value store
we pass values in a Context and use it as a key-value store. In this case, we do not pass values into contexts in order to provide further information about why they were canceled

```go
package main
import (
    "context"
    "fmt"
)
type aKey string
func searchKey(ctx context.Context, k aKey) {
    v := ctx.Value(k)
	if v != nil {
		fmt.Println("found value:", v)
		return
	} else {
		fmt.Println("key not found:", k)
	}
}

func main() {
	myKey := aKey("mySecretValue")
	ctx := context.WithValue(context.Background(), myKey, "mySecret")
	
	searchKey(ctx, myKey)
	searchKey(ctx, aKey("notThere"))
	emptyCtx := context.TODO()
	searchKey(emptyCtx, aKey("notThere"))
}
```
The context.WithValue() function that is used in main() offers a way to associate a value with a Context. The next two statements search an existing context (ctx) for the values of two keys.

This time we create a context using context.TODO() instead of context.Background(). Although both functions return a non-nil, empty Context, their purposes differ. You should never pass a nil context—use the context.TODO() function to create a suitable context. Additionally, use the context.TODO() function when you are not sure about the Context that you want to use. The context.TODO() function signifies that we intend to use an operation context, without being sure about it yet

## The semaphore package
A semaphore is a construct that can limit or control the access to a shared resource. As we are talking about Go, a semaphore can limit the access of goroutines to a shared resource but originally, semaphores were used for limiting access to threads. Semaphores can have weights that limit the number of threads or goroutines that can have access to a resource.

```go
func (s *Weighted) Acquire(ctx context.Context, n int64) error
func (s *Weighted) Release(n int64)
```
```go
package main
import (
    "context"
    "fmt"
    "os"
    "strconv"
    "time"
    "golang.org/x/sync/semaphore"
)
var Workers = 4
var sem = semaphore.NewWeighted(int64(Workers))
func worker(n int) int {
	square := n * n
	time.Sleep(time.Second)
	return square
}

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Need #jobs!")
		return
	}
	nJobs, err := strconv.Atoi(os.Args[1])
	if err != nil {
		fmt.Println(err)
		return
	}
	// Where to store the results
	var results = make([]int, nJobs)
	// Needed by Acquire()
	ctx := context.TODO()
	for i := range results {
		err = sem.Acquire(ctx, 1)
		if err != nil {
			fmt.Println("Cannot acquire semaphore:", err)
			break
		}
		go func(i int) {
			defer sem.Release(1)
			temp := worker(i)
			results[i] = temp
		}(i)
	}
	err = sem.Acquire(ctx, int64(Workers))
	if err != nil {
		fmt.Println(err)
	}
	for k, v := range results {
		fmt.Println(k, "->", v)
	}
}
```
This is where we define the semaphore with a weight identical to the maximum number of goroutines that can be executed concurrently. This means that no more than Workers goroutines can acquire the semaphore at the same time.

If nJobs is bigger than Workers, then the Acquire() call is going to block and wait for Release() calls in order to unblock.

This is where we run the goroutines that do the job and write the results to the results slice. As each goroutine writes to a different slice element, there are not any race conditions.

This is a clever trick: we acquire all of the tokens so that the sem.Acquire() call blocks until all workers/goroutines have finished. This is similar in functionality to a Wait() call

Each line in the output shows the input value and the output value separated by ->. The use of the semaphore keeps things in order.

## Exercises
* Try to implement a concurrent version of wc(1) that uses a buffered channel.
* Try to implement a concurrent version of wc(1) that uses shared memory.
* Try to implement a concurrent version of wc(1) that uses semaphores.
* Try to implement a concurrent version of wc(1) that saves its output to a file.
* Modify wPools.go so that each worker implements the functionality of wc(1).

## Additional resources
* The documentation page of sync is at [https://golang.org/pkg/sync/](https://golang.org/pkg/sync/)
* Learn about semaphore at [https://pkg.go.dev/golang.org/x/sync/semaphore](https://pkg.go.dev/golang.org/x/sync/semaphore)
* Learn more about the Go scheduler by reading a series of posts starting with [https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html)
* The implementation of the Go scheduler: [https://golang.org/src/runtime/proc.go](https://golang.org/src/runtime/proc.go)
