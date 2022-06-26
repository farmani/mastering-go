# Chapter 1: A Quick Introduction to Go

## Introducing Go

- Although Go is a general-purpose programming language, it is primarily used for writing system tools, command-line utilities, web services, and software that work over networks.

- What you get with Go, apart from its syntax and tools, is a pretty rich standard library and a type system that tries to save you from easy mistakes such as implicit type conversions, unused variables and unused packages. The Go compiler catches most of these easy mistakes and refuses to compile until you do something about them. Additionally, the Go compiler can find difficult to catch mistakes such as race conditions

- Go is ***portable by design***—this means that you should not worry about the operating system you are using.

- many services written in Go are executed in a Docker environment—Docker images use the Linux operating system, which means that you should program your utilities having the Linux operating system in mind.

### The advantages of Go

- Go manages OS threads for you and has a powerful runtime that allows you to spawn lightweight units of work (***goroutines***) that communicate with each other using ***channels***.

- Go's executable binaries are statically linked, which means that once they are generated, they do not depend on any shared libraries and include all required information.

- Although Go supports pointers, it does not support pointer arithmetic like C, unless you use the ***unsafe*** package, which is the root of many bugs and security holes.

- Although Go is not an object-oriented programming language, Go interfaces are very versatile and allow you to mimic some of the capabilities of object-oriented languages such as ***polymorphism***, ***encapsulation***, and ***composition***.

- Although Go is a very practical and competent programming language, it is not perfect
  
  - Go has no direct support for object-oriented programming, which is a popular programming paradigm.
  
  - Although goroutines are lightweight, they are not as powerful as OS threads. Depending on the application you are trying to implement, there might exist some rare cases where goroutines will not be appropriate for the job.
  
  - There are times when you need to handle memory allocation manually—Go cannot do that. In practice, this means that Go will not allow you to perform any memory management manually.

### The go doc and godoc utilities

- Two of these tools are the go doc subcommand and godoc utility, which allow you to see the documentation of existing Go functions and packages without needing an internet connection.

- As ***godoc*** is not installed by default, you might need to install it by running `go get golang.org/x/tools/cmd/godoc`.

- The ***go doc*** command can be executed as a normal command-line application that displays its output on a terminal, and ***godoc*** as a command-line application that starts a web server. In the latter case, you need a web browser to look at the Go documentation. The first utility is similar to the UNIX `man(1)` command, but for Go functions and packages.

> The number after the name of a UNIX program or system call refers to the section of the manual a manual page belongs to.

- in order to find information about the `Printf()` function of the `fmt` package:

```shell
$ go doc fmt.Printf
```

- you can find information about the entire `fmt` package:

```shell
$ go doc fmt
```

- The second utility requires executing `godoc` with the `-http` parameter. You can omit the equals sign in the presented command and put a space character in its place.

```shell
$ godoc -http=:8001
```

> Note that port numbers 0-1023 are restricted and can only be used by the root user, so it is better to avoid choosing one of those and pick something else, provided that it is not already in use by a different process

## Hello World!

```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("Hello World!")
}
```

- Each Go source code begins with a package declaration. In this case, the name of the package is `main`, which has a special meaning in Go.

- Packages that are not part of the standard Go library are imported using their ***full internet path***.

- If you are creating an executable application is a `main()` function. Go considers this the entry point to the application and begins the execution of the application with the code found in the `main()` function of the main package.

- Two characteristics make `hw.go` an autonomous source file that can generate an executable binary: the name of the package, which should be ***main***, and the presence of the `main()` function.

#### Introducing functions

- There is a global Go rule that also applies to function and variable names and is valid for all packages except main: ***everything that begins with a lowercase letter is considered private and is accessible in the current package only.*** The only exception to this rule is package names, which can begin with either lowercase or uppercase letters. Having said that, _I am not aware of a Go package that begins with an uppercase letter!_

#### Introducing packages

- The package keyword helps you define the name of a new package, which can be anything you want with just one exception: if you are creating an executable application and not just a package that will be shared by other applications or packages, you should name your package ***main***.

- Packages of the standard Go library are imported by name (os) without the need for a hostname and a path, whereas external packages are imported using their full internet paths, like `github.com/spf13/cobra`.

## Running Go code

- There are two ways to execute Go code: as a _compiled language_ using `go build` or as a _scripting language_ using `go run`.

- The generated executable is automatically named after the source code filename without the `.go` file extension. Therefore, because of the hw.go source file, the executable will be called hw. In case this is not what you want, go build supports the `-o` option, which allows you to change the filename and the path of the generated executable file.

- If no source files are provided, go build looks for a `main` package in the current directory.

```shell
$ go build hw.go
$ ./hw
Hello World!
```

- The `go run` command builds the named Go package, which in this case is the main package implemented in a single file, creates a temporary executable file, executes that file, and deletes it once it is done—to our eyes, this looks like using a scripting language.

```shell
$ go run hw.go
Hello World!
```

- If you want to test your code, then using `go run` is a better choice. However, if you want to create and distribute an executable binary, then go build is the way to go.

## Important characteristics of Go

- if you are going to import a package, you should use some of this functionality—there are some exceptions to this rule that mainly have to with initializing connections, but they are not important for now.

- You either use a variable or you do not declare it at all.

- There is only one way to format curly braces in Go.

- Coding blocks in Go are embedded in curly braces even if they contain just a single statement or no statements at all.

- Go functions can return multiple values.

- You cannot automatically convert between different data types, even if they are of the same kind.

- Go requires the use of semicolons as statement terminators in many contexts, and the compiler automatically inserts the required semicolons when it thinks that they are necessary. Therefore, putting the opening curly brace ({) in its own line will make the Go compiler insert a semicolon at the end of the previous line (func main()), which is the main cause of the error message

### Defining and using variables

- You can declare a new variable using the `var` keyword followed by the variable name, followed by the desired data type

- If you want, you can follow that declaration with `=` and an initial value for your variable. If there is an initial value given, you can omit the data type and the compiler will guess it for you.

- Very important Go rule: ***if no initial value is given to a variable, the Go compiler will automatically initialize that variable to the zero value of its data type.***

- The official name for `:=` is ***short assignment statement***, and it is very frequently used in Go, especially for getting the return values from functions and for loops with the range keyword.

- The short assignment statement can be used in place of a `var` declaration with an implicit type. You rarely see the use of `var` in Go; the `var` keyword is ***mostly used for declaring global or local variables without an initial value***. The _reason for the former is that every statement that exists outside the code of a function must begin with a keyword such as `func` or `var`_.

- This means that the short assignment statement cannot be used outside a function environment because it is not available there. Last, you might need to use `var` when you want to be explicit about the data type. For example, when you want int8 or int32 instead of int.

- Only `const` (when the value of a variable is not going to change) and `var` work for global variables, which are variables that are defined outside a function and are not embedded in curly braces.

### Printing variables

- If you want Go to take care of the printing, then you might want to use the `fmt.Println()` function. However, there are times that you want to have full control over how data is going to get printed. In such cases, you might want to use `fmt.Printf()`.

- If someone requires the use of control sequences that specify the data type of the variable that is going to get printed. Additionally, the `fmt.Printf()` function allows you to format the generated output

```go
package main

import (
    "fmt"
    "math"
)

var Global int = 1234
var AnotherGlobal = -5678

func main() {
    var j int
    i := Global + AnotherGlobal
    fmt.Println("Initial j value:", j)
    j = Global
    // math.Abs() requires a float64 parameter
    // so we type cast it appropriately
    k := math.Abs(float64(AnotherGlobal))
    fmt.Printf("Global=%d, i=%d, j=%d k=%.2f.\n", Global, i, j, k)
}
```

> Personally, I prefer to make global variables stand out by either beginning them with an uppercase letter or using all capital letters.

- The `float64()` type cast converts the value of AnotherGlobal to float64. Note that AnotherGlobal continues to be `int`.

- Go does not allow implicit data conversions like C.

- For conversions that are not straightforward (for example, string to int), there exist specialized functions that allow you to catch issues with the conversion in the form of an `error` variable that is returned by the function.

## Controlling program flow

- `if` statements use no parenthesis for embedding the conditions that need to be examined because Go does not use parentheses in general. As expected, `if` has support for `else` and `else if` statements.

- The `switch` statement has two different forms. ***In the first form, the switch statement has an expression that is being evaluated, whereas in the second form, the switch statement has no expression to evaluate. In that case, expressions are evaluated in each case statement, which increases the flexibility of switch.*** The main benefit you get from `switch` is that when used properly, it simplifies complex and hard-to-read if-else blocks.

```go
package main

import (
    "fmt"
    "os"
    "strconv"
)

var Global int = 1234
var AnotherGlobal = -5678

func main() {
    if len(os.Args) != 2 {
        fmt.Println("Please provide a command line argument")
        return
    }
    argument := os.Args[1]

    // With expression after switch
    switch argument {
    case "0":
        fmt.Println("Zero!")
    case "1":
        fmt.Println("One!")
    case "2", "3", "4":
        fmt.Println("2 or 3 or 4")
        fallthrough
    default:
        fmt.Println("Value:", argument)
    }

    // This part of the program makes sure that you have a single command-line argument to process, which is accessed as os.Args[1]
    value, err := strconv.Atoi(argument)
    if err != nil {
        fmt.Println("Cannot convert to int:", argument)
        return
    }

    // No expression after switch
    switch {
    case value == 0:
        fmt.Println("Zero!")
    case value > 0:
        fmt.Println("Positive integer")
    case value < 0:
        fmt.Println("Negative integer")
    default:
        fmt.Println("This should not happen:", value)
    }
}
```

- The order of the `case` statements is ***important*** because only the first match is executed. The `fallthrough` keyword tells Go that after this branch is executed, it will continue with the next branch

- We need to convert user input into an integer value using a separate call, which in this case is a call to `strconv.Atoi()`.

- the `default` branch is there, which is a good practice because it can catch unexpected values.

## Iterating with for loops and range

- Go supports `for` loops as well as the `range` keyword for iterating over all the elements of `arrays`, `slices`, and `maps`.

- Depending on how you write a `for` loop, it can function as a `while` loop or an `infinite` loop. Moreover, for loops can implement the functionality of JavaScript's `forEach` function when combined with the `range` keyword.

> You need to put curly braces around a for loop even if it contains a single statement or no statements at all

- A `for` loop can be exited with a `break` keyword, and you can skip the current iteration with the `continue` keyword. When used with `range`, `for` loops allow you to visit all the elements of a slice or an `array` without knowing the size of the data structure.

- The following code shows how a `for` loop can simulate a `while` loop, which is not supported directly.

```go
package main

import (
    "fmt"
)

func main() {
    // For loop used as while loop
    i := 0
    for {
        if i == 10 {
            break
        }
        fmt.Print(i*i, " ")
        i++
    }
    fmt.Println()
}
```

- If you want to ignore either of these return values, which is not the case here, you can use `_` in the place of the value that you want to ignore. If you just need the index, you can leave out the second value from range entirely without using `_`.

```go
package main

import (
    "fmt"
)

func main() {
    // This is a slice but range also works with arrays
    aSlice := []int{-1, 2, 1, -1, 2, -2}
    for i, v := range aSlice {
        fmt.Println("index:", i, "value: ", v)
    }
}
```

## Getting user input

### Reading from standard input

- The `fmt.Scanln()` function can help you read user input while the program is already running and store it to a `string` variable, which is _passed as a pointer_ to `fmt.Scanln()`. The `fmt` package contains additional functions for reading user input from the console (`os.Stdin`), from files or from argument lists.

```go
package main

import (
    "fmt"
)

func main() {
    // Get User Input
    fmt.Printf("Please give me your name: ")
    var name string
    fmt.Scanln(&name)
    fmt.Println("Your name is", name)
}
```

### Working with command-line arguments

- By default, command-line arguments in Go are stored in the `os.Args` slice. Go also offers the flag package for parsing command-line arguments, but there are better and more powerful alternatives.

- It is important to know that the `os.Args` slice is properly initialized by Go and is available to the program when referenced. The `os.Args` slice contains string values.

- The first command-line argument stored in the `os.Args` slice is always the _name of the executable_. The remaining command-line arguments are what comes after the name of the executable—the various command-line arguments are automatically separated by _space characters_ unless they are included in double or single quotes.

```go
package main

import (
    "fmt"
    "os"
    "strconv"
)

func main() {
    arguments := os.Args
    if len(arguments) == 1 {
        fmt.Println("Need one or more arguments!")
        return
    }

    var min, max float64
    for i := 1; i < len(arguments); i++ {
        n, err := strconv.ParseFloat(arguments[i], 64)
        if err != nil {
            continue
        }

        if i == 1 {
            min = n
            max = n
            continue
        }

        if n < min {
            min = n
        }
        if n > max {
            max = n
        }
    }
    fmt.Println("Min:", min)
    fmt.Println("Max:", max)
}
```

- We use the `error` variable returned by `strconv.ParseFloat()` to make sure that the call to `strconv.ParseFloat()` was successful, and we have a valid numeric value to process.

## Using error variables to differentiate between input types

- Uses `error` variables to differentiate between various kinds of user input. For this technique to work, you should go from more *specific cases* to more *generic* ones.

```go
var total, nInts, nFloats int
invalid := make([]string, 0)
for _, k := range arguments[1:] {
    // Is it an integer?
    _, err := strconv.Atoi(k)
    if err == nil {
        total++
        nInts++
        continue
    }

    // Is it a float
    _, err = strconv.ParseFloat(k, 64)
    if err == nil {
        total++
        nFloats++
        continue
    }

    // Then it is invalid
    invalid = append(invalid, k)
}
```

- The `invalid` variable, which is a ***slice***, is used for keeping all non-numeric values.

- The call to `strconv.Atoi()` determines whether we are processing a valid ***int*** value or not. If so, we increase the total and nInts counters

- Similarly, if the examined string represents a valid floating-point value, the call to `strconv.ParseFloat()` is going to be successful and the program will update the relevant counters. Lastly, if a value is not numeric, it is added to the invalid slice with a call to `append()`.

> This is a common practice for keeping unexpected input in applications.

```shell
$ go run process.go 1 2 3
#read: 3 #ints: 3 #floats: 0

$ go run process.go 1 2.1 a    
#read: 2 #ints: 1 #floats: 1

$ go run process.go a 1 b
#read: 1 #ints: 1 #floats: 0
Too much invalid input: 2
a
```

## Understanding the Go concurrency model

- A `goroutine` is the smallest executable Go entity. In order to create a new `goroutine`, you have to use the `go` keyword followed by a *predefined function* or an *anonymous function*—both methods are equivalent as far as Go is concerned.

> **Note that you can only execute functions or anonymous functions as goroutines.**

- A `channel` in Go is a mechanism that, *among other things*, allows goroutines to communicate and exchange data.

- Although it is easy to create goroutines, there are other difficulties when dealing with concurrent programming including *goroutine synchronization* and *sharing data between goroutines*—this is a Go mechanism for avoiding side effects when running goroutines. As `main()` runs as a goroutine as well, you do not want `main()` to finish before the other goroutines of the program because when `main()` exits, the entire program along with any goroutines that have not finished yet will terminate. 
- Although goroutines are do not share any variables, they can share memory. The good thing is that there are various techniques for the `main()` function to ***wait*** for goroutines to exchange data through channels or, less frequently in Go, using shared memory.

- `time.Sleep()` as you can see in below is not the right way to synchronize goroutines—we will discuss the proper way to synchronize goroutines in Chapter 7, Go Concurrency

```go
package main

import (
    "fmt"
    "time"
)

func myPrint(start, finish int) {
    for i := start; i <= finish; i++ {
        fmt.Print(i, " ")
    }
    fmt.Println()
    time.Sleep(100 * time.Microsecond)
}
func main() {
    for i := 0; i < 5; i++ {
        go myPrint(i, 5)
    }
    time.Sleep(time.Second)
}
```

- the `go` keyword is used for creating goroutines.

- goroutines are *initialized in random order* and start *running in random order*. The *Go scheduler* is responsible for the execution of goroutines just like the OS scheduler is responsible for the execution of the OS threads.

- however, keep in mind that *Go concurrency is everywhere*, which is the main reason for including this section here. Therefore, as some error messages generated by the compiler talk about goroutines, you should not think that these goroutines were created by you.

## Developing the which(1) utility in Go

- Go can work with your operating system through a set of packages. A good way of learning a new programming language is by trying to implement simple versions of traditional UNIX utilities.

- The `os` package is for interacting with the underlying operating system, and the `path/filepath` package is used for working with the contents of the `PATH` variable that is read as a long string, depending on the number of directories it contains.

```go
package main

import (
    "fmt"
    "os"
    "path/filepath"
)
func main() {
    arguments := os.Args
    if len(arguments) == 1 {
        fmt.Println("Please provide an argument!")
        return
    }
    file := arguments[1]
    path := os.Getenv("PATH")
    pathSplit := filepath.SplitList(path)
    for _, directory := range pathSplit {
        fullPath := filepath.Join(directory, file)
        // Does it exist?
        fileInfo, err := os.Stat(fullPath)
        if err == nil {
            mode := fileInfo.Mode()
            // Is it a regular file?
            if mode.IsRegular() {
                // Is it executable?
                if mode&0111 != 0 {
                    fmt.Println(fullPath)
                    return
                }
            }
        }
    }
}
```

- We get the contents of the `PATH` environment variable and split it using `filepath.SplitList()`, which offers a portable way of separating a list of paths. Lastly, we iterate over all the directories of the PATH variable using a for loop with range as `filepath.SplitList()` returns a ***slice***.

- We construct the full path that we examine using `filepath.Join()` that *is used for concatenating the different parts of a path using an OS-specific separator—this makes `filepath.Join()` work in all supported operating systems*. In this part, we also get some lower-level information about the file—remember that in UNIX everything is a file, which means that we want to make sure that we are dealing with a regular file that is also executable.

- According to the UNIX philosophy and the way UNIX pipes work, utilities generate no output onscreen if they have nothing to say. However, an exit code of `0` means success whereas a non-zero exit code usually means failure.

```shell
$ go run which.go which
/usr/bin/which
$ go run which.go doesNotExist
```

## Logging information

> Logging and putting logging information in log files is a practical way of examining data and information from your software asynchronously either locally or at a central log server or using server software such as Elasticsearch, Beats, and Grafana Loki.

- Generally speaking, using a log file to write some information used to be considered a better practice than writing the same output on screen for two reasons: firstly, because the output does not get lost as it is stored on a file, and secondly, because you can search and process log files using UNIX tools

- As we usually run our services via systemd, *programs should log to stdout so systemd can put logging data in the journal*. https://12factor.net/logs offers more information about app logs. Additionally, in cloud native applications, we are encouraged to simply log to `stderr` and let the container system redirect the `stderr` stream to the desired destination.

- The UNIX logging service has support for two properties named ***logging level*** and ***logging facility***. 
	- The logging level is a value that specifies the **severity** of the log entry. There are various logging levels, including `debug`, `info`, `notice`, `warning`, `err`, `crit`, `alert`, and `emerg`, in reverse order of severity. The log package of the standard Go library does not support working with logging levels. 
	- 	The logging facility is like a category used for logging information. The value of the logging facility part can be one of `auth`, `authpriv`, `cron`, `daemon`, `kern`, `lpr`, `mail`, `mark`, `news`, `syslog`, `user`, `UUCP`, `local0`, `local1`, `local2`, `local3`, `local4`, `local5`, `local6`, or `local7` and is defined inside `/etc/syslog.conf`, `/etc/rsyslog.conf`, or another appropriate file depending on the server process used for system logging on your UNIX machine. ***This means that if a logging facility is not defined correctly, it will not be handled; therefore, the log messages you send to it might get ignored and therefore lost.***

- The log package sends log messages to ***standard error***. Part of the log package is the log/syslog package, which allows you to send log messages to the syslog server of your machine. Although by default log writes to standard error, the use of `log.SetOutput()` modifies that behavior. The list of functions for sending logging data includes `log.Printf()`, `log.Print()`, `log.Println()`, `log.Fatalf()`, `log.Fatalln()`, `log.Panic()`, `log.Panicln()` and `log.Panicf()`.

> **Logging is for application code, not library code. If you are developing libraries, do not put logging in them.**

- Writing to the main system log file is as easy as calling `syslog.New()` with the `syslog.LOG_SYSLOG` option. After that you need to tell your Go program that all logging information goes to the new logger—this is implemented with a call to the `log.SetOutput()` function.

```go
package main

import (
	"log"
	"log/syslog"
)

func main() {
	sysLog, err := syslog.New(syslog.LOG_SYSLOG, "systemLog.go")
	if err != nil {
		log.Println(err)
		return
	} else {
		log.SetOutput(sysLog)
		log.Print("Everything is fine!")
	}
}
```

- After the call to `log.SetOutput()`, all logging information goes to the syslog logger variable that sends it to `syslog.LOG_SYSLOG`. *Custom text for the log entries coming from that program is specified as the second parameter to the syslog.New() call*.

> Usually, you want to store logging data on user-defined files because they group relevant information, which makes them easier to process and inspect.

- on macOS Big Sur machine, for example, you will find entries like the following inside `/var/log/system.log`

```shell
Dec  5 16:20:10 iMac systemLog.go[35397]: 2020/12/05 16:20:10 Everything is fine!
Dec  5 16:43:18 iMac systemLog.go[35641]: 2020/12/05 16:43:18 Everything is fine!
```

- The number inside the brackets is the process ID of the process that wrote the log entry

- Similarly, if you execute `journalctl -xe` on a Linux machine, you can see entries similar to the next:

```shell
Dec 05 16:33:43 thinkpad systemLog.go[12682]: 2020/12/05 16:33:43 Everything is fine!
Dec 05 16:46:01 thinkpad systemLog.go[12917]: 2020/12/05 16:46:01 Everything is fine!
```

### log.Fatal() and log.Panic()

- The `log.Fatal()` function is used when something erroneous has happened and you just want to exit your program as soon as possible after reporting that bad situation. The call to `log.Fatal()` ***terminates*** a Go program at the point where `log.Fatal()` was called after printing an error message. *Additionally, it returns back a non-zero exit code, which in UNIX indicates an error*.

- There are situations where a program is about to fail for good and you want to have as much information about the failure as possible—`log.Panic()` implies that something really unexpected and unknown, such as not being able to find a file that was previously accessed or not having enough disk space, has happened. Analogous to the `log.Fatal()` function, `log.Panic()` prints a custom message and immediately terminates the Go program.

- Have in mind that `log.Panic()` is equivalent to a call to `log.Print()` followed by a call to `panic()`. `panic()` is a built-in function that stops the execution of the current function and begins panicking. After that, it returns to the caller function. On the other hand, `log.Fatal()` calls `log.Print()` and then `os.Exit(1)`, which is an immediate way of terminating the current program.

```go
package main

import (
	"log"
	"os"
)

func main() {
	if len(os.Args) != 1 {
		log.Fatal("Fatal: Hello World!")
	}
	log.Panic("Panic: Hello World!")
}
```

```shell
$ go run logs.go 
2020/12/03 18:39:26 Panic: Hello World!
panic: Panic: Hello World!
goroutine 1 [running]:
log.Panic(0xc00009ef68, 0x1, 0x1)
        /usr/lib/go/src/log/log.go:351 +0xae
main.main()
        /home/mtsouk/Desktop/mGo3rd/code/ch01/logs.go:12 +0x6b
exit status 2
$ go run logs.go 1
2020/12/03 18:39:30 Fatal: Hello World!
exit status 1
```

- the output of `log.Panic()` includes additional low-level information that, hopefully, will help you to resolve difficult situations that happened in your Go code.

### Writing to a custom log file

- Most of the time, and especially on applications and services that are deployed to production, you just need to write your logging data in a log file of your choice.

- The path of the log file that is used is hardcoded into the code using a global variable named `LOGFILE`.

- the `/tmp` directory is emptied after each system reboot.

- Additionally, at this point, this will save you from having to execute customLog.go with root privileges and from putting unnecessary files into your precious system directories.

```go
package main

import (
	"fmt"
	"log"
	"os"
	"path"
)

func main() {
	LOGFILE := path.Join(os.TempDir(), "mGo.log")
	f, err := os.OpenFile(LOGFILE, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)

	// The call to os.OpenFile() creates the log file for writing,
	// if it does not already exist, or opens it for writing
	// by appending new data at the end of it (os.O_APPEND)
	if err != nil {
		fmt.Println(err)
		return
	}

	defer f.Close()

	iLog := log.New(f, "iLog ", log.LstdFlags)
	iLog.Println("Hello there!")
	iLog.Println("Mastering Go 3rd edition!")
}

```

> The `defer` keyword tells Go to execute the statement just before the current function returns. This means that `f.Close()` is going to be executed just before `main()` returns.

> If you ever decide to use the code of customLog.go in a real application, you should change the path stored in `LOGFILE` into something that makes more sense.

```shell
$ cat /tmp/mGo.log
iLog 2020/12/05 17:31:07 Hello there!
iLog 2020/12/05 17:31:07 Mastering Go 3rd edition!
```

### Printing line numbers in log entries

The desired functionality is implemented with the use of `log.Lshortfile` in the parameters of `log.New()` or `SetFlags()`. The `log.Lshortfile` flag adds the filename as well as the line number of the Go statement that printed the log entry in the log entry itself. If you use `log.Llongfile` instead of `log.Lshortfile`, then you get the *full path* of the Go source file—usually, this is not necessary, especially when you have a really long path.

```go
package main

import (
	"fmt"
	"log"
	"os"
	"path"
)

func main() {
	LOGFILE := path.Join(os.TempDir(), "mGo.log")
	f, err := os.OpenFile(LOGFILE, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer f.Close()
	LstdFlags := log.Ldate | log.Lshortfile
	iLog := log.New(f, "LNum ", LstdFlags)
	iLog.Println("Mastering Go, 3rd edition!")
	iLog.SetFlags(log.Lshortfile | log.LstdFlags)
	iLog.Println("Another log entry!")
}
```

- In case you are wondering, you are allowed to change the format of the log entries during program execution—this means that when there is a reason, you can print more analytical information in the log entries. This is implemented with multiple calls to `iLog.SetFlags()`.

```shell
$ cat /tmp/mGo.log 
LNum 2020/12/05 customLogLineNumber.go:24: Mastering Go, 3rd edition!
LNum 2020/12/05 17:33:23 customLogLineNumber.go:27: Another log entry!
```

## Overview of Go generics

- The main idea behind generics in Go, as well as any other programming language that supports generics, is *not having to write special code for supporting multiple data types when performing the same task*.

- Go supports multiple data types in functions such as `fmt.Println()` using the empty interface and reflection

- demanding every programmer to write lots of code and implement lots of functions and methods for supporting multiple custom data types is not the optimal solution—generics comes into play for providing an alternative to the use of interfaces and reflection for supporting multiple data types

```go
package main

import (
	"fmt"
)

func Print[T any](s []T) {
	for _, v := range s {
		fmt.Print(v, " ")
	}
	fmt.Println()
}
func main() {
	Ints := []int{1, 2, 3}
	Strings := []string{"One", "Two", "Three"}
	Print(Ints)
	Print(Strings)
}
```

- we have a function named `Print()` that uses generics through a generics variable, which is specified by the use of `[T any]` after the function name and before the function parameters. Due to the use of `[T any]`, Print() can accept any slice of any data type and work with it.

> In Chapter 4, Reflection and Interfaces, you will learn about the empty interface and how it can be used for accepting data of any data type. However, the empty interface requires extra code for working with specific data types.

- I believe that generics should be used when they can create simpler code and designs. It is better to have repetitive straightforward code than optimal abstractions that slow down your applications.

- There are times that you need to limit the data types that are supported by a function that uses generics—this is not a bad thing as all data types do not share the same capabilities. *Generally speaking, generics can be useful when processing data types that share some characteristics*.

## Developing a basic phone book application


```go
package main

import (
	"fmt"
	"os"
	"path"
)

type Entry struct {
	Name    string
	Surname string
	Tel     string
}

var data = []Entry{}

func search(key string) *Entry {
	for i, v := range data {
		if v.Surname == key {
			return &data[i]
		}
	}
	return nil
}
func list() {
	for _, v := range data {
		fmt.Println(v)

	}
}
func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		exe := path.Base(arguments[0])
		fmt.Printf("Usage: %s search|list <arguments>\n", exe)
		return
	}

	data = append(data, Entry{"Mihalis", "Tsoukalos", "2109416471"})
	data = append(data, Entry{"Mary", "Doe", "2109416871"})
	data = append(data, Entry{"John", "Black", "2109416123"})

	// Differentiate between the commands
	switch arguments[1] {
	// The search command
	case "search":
		if len(arguments) != 3 {
			fmt.Println("Usage: search Surname")
			return
		}
		result := search(arguments[2])
		if result == nil {
			fmt.Println("Entry not found:", arguments[2])
			return
		}
		fmt.Println(*result)
	// The list command
	case "list":
		list()
	// Response to anything that is not a match
	default:
		fmt.Println("Not a valid option")
	}
}
```

- `Structures` group a set of values into a single data type, which allows you to pass and receive this set of values as a single entity.

- The exe variable holds the path to the executable file—it is a nice and professional touch to print the name of the executable binary in the instructions of the program

### Exercises
- Our version of which(1) stops after finding the first occurrence of the desired executable. Make the necessary changes to which.go in order to find all possible occurrences of the desired executable.
- The current version of which.go processes the first command-line argument only. Make the necessary changes to which.go in order to accept and search the PATH variable for multiple executable binaries.
- Read the documentation of the fmt package at https://golang.org/pkg/fmt/.

