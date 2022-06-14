# Chapter 1: A Quick Introduction to Go

## Introducing Go

- Although Go is a general-purpose programming language, it is primarily used for writing system tools, command-line utilities, web services, and software that work over networks.

- What you get with Go, apart from its syntax and tools, is a pretty rich standard library and a type system that tries to save you from easy mistakes such as implicit type conversions, unused variables and unused packages. The Go compiler catches most of these easy mistakes and refuses to compile until you do something about them. Additionally, the Go compiler can find difficult to catch mistakes such as race conditions

- Go is portable by design—this means that you should not worry about the operating system you are using.

- many services written in Go are executed in a Docker environment—Docker images use the Linux operating system, which means that you should program your utilities having the Linux operating system in mind.

### The advantages of Go

- Go manages OS threads for you and has a powerful runtime that allows you to spawn lightweight units of work (goroutines) that communicate with each other using channels.

- Go's executable binaries are statically linked, which means that once they are generated, they do not depend on any shared libraries and include all required information.

- although Go supports pointers, it does not support pointer arithmetic like C, unless you use the unsafe package, which is the root of many bugs and security holes.

- Although Go is not an object-oriented programming language, Go interfaces are very versatile and allow you to mimic some of the capabilities of object-oriented languages such as polymorphism, encapsulation, and composition.

- Although Go is a very practical and competent programming language, it is not perfect:

    - Go has no direct support for object-oriented programming, which is a popular programming paradigm.

    - Although goroutines are lightweight, they are not as powerful as OS threads. Depending on the application you are trying to implement, there might exist some rare cases where goroutines will not be appropriate for the job.

    - There are times when you need to handle memory allocation manually—Go cannot do that. In practice, this means that Go will not allow you to perform any memory management manually.

### The go doc and godoc utilities

- Two of these tools are the go doc subcommand and godoc utility, which allow you to see the documentation of existing Go functions and packages without needing an internet connection.

- As godoc is not installed by default, you might need to install it by running go get golang.org/x/tools/cmd/godoc.

- The go doc command can be executed as a normal command-line application that displays its output on a terminal, and godoc as a command-line application that starts a web server. In the latter case, you need a web browser to look at the Go documentation. The first utility is similar to the UNIX man(1) command, but for Go functions and packages.

> The number after the name of a UNIX program or system call refers to the section of the manual a manual page belongs to.

- in order to find information about the Printf() function of the fmt package:

```shell
$ go doc fmt.Printf
```

- you can find information about the entire fmt package:

```shell
$ go doc fmt
```

- The second utility requires executing godoc with the -http parameter: You can omit the equals sign in the presented command and put a space character in its place.

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

- Each Go source code begins with a package declaration. In this case, the name of the package is main, which has a special meaning in Go.

- Packages that are not part of the standard Go library are imported using their full internet path. The next important thing if you are creating an executable application is a main() function. Go considers this the entry point to the application and begins the execution of the application with the code found in the main() function of the main package.

- Two characteristics make hw.go an autonomous source file that can generate an executable binary: the name of the package, which should be main, and the presence of the main() function.

#### Introducing functions

- There is a global Go rule that also applies to function and variable names and is valid for all packages except main: everything that begins with a lowercase letter is considered private and is accessible in the current package only. “The only exception to this rule is package names, which can begin with either lowercase or uppercase letters. Having said that, I am not aware of a Go package that begins with an uppercase letter!

#### Introducing packages

- The package keyword helps you define the name of a new package, which can be anything you want with just one exception: if you are creating an executable application and not just a package that will be shared by other applications or packages, you should name your package main.

- Packages of the standard Go library are imported by name (os) without the need for a hostname and a path, whereas external packages are imported using their full internet paths, like github.com/spf13/cobra.

## Running Go code