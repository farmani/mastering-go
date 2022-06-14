# Chapter 1: A Quick Introduction to Go

## Introducing Go

- Although Go is a general-purpose programming language, it is primarily used for writing system tools, command-line
  utilities, web services, and software that work over networks.

- What you get with Go, apart from its syntax and tools, is a pretty rich standard library and a type system that tries
  to save you from easy mistakes such as implicit type conversions, unused variables and unused packages. The Go
  compiler catches most of these easy mistakes and refuses to compile until you do something about them. Additionally,
  the Go compiler can find difficult to catch mistakes such as race conditions

- Go is ***portable by design***—this means that you should not worry about the operating system you are using.

- many services written in Go are executed in a Docker environment—Docker images use the Linux operating system, which
  means that you should program your utilities having the Linux operating system in mind.

### The advantages of Go

- Go manages OS threads for you and has a powerful runtime that allows you to spawn lightweight units of work (***
  goroutines***) that communicate with each other using ***channels***.

- Go's executable binaries are statically linked, which means that once they are generated, they do not depend on any
  shared libraries and include all required information.

- Although Go supports pointers, it does not support pointer arithmetic like C, unless you use the ***unsafe*** package,
  which is the root of many bugs and security holes.

- Although Go is not an object-oriented programming language, Go interfaces are very versatile and allow you to mimic
  some of the capabilities of object-oriented languages such as ***polymorphism***, ***encapsulation***, and ***
  composition***.

- Although Go is a very practical and competent programming language, it is not perfect

    - Go has no direct support for object-oriented programming, which is a popular programming paradigm.

    - Although goroutines are lightweight, they are not as powerful as OS threads. Depending on the application you are
      trying to implement, there might exist some rare cases where goroutines will not be appropriate for the job.

    - There are times when you need to handle memory allocation manually—Go cannot do that. In practice, this means that
      Go will not allow you to perform any memory management manually.

### The go doc and godoc utilities

- Two of these tools are the go doc subcommand and godoc utility, which allow you to see the documentation of existing
  Go functions and packages without needing an internet connection.

- As ***godoc*** is not installed by default, you might need to install it by
  running `go get golang.org/x/tools/cmd/godoc`.

- The ***go doc*** command can be executed as a normal command-line application that displays its output on a terminal,
  and ***godoc*** as a command-line application that starts a web server. In the latter case, you need a web browser to
  look at the Go documentation. The first utility is similar to the UNIX `man(1)` command, but for Go functions and
  packages.

> The number after the name of a UNIX program or system call refers to the section of the manual a manual page belongs
> to.

- in order to find information about the `Printf()` function of the `fmt` package:

```shell
$ go doc fmt.Printf
```

- you can find information about the entire `fmt` package:

```shell
$ go doc fmt
```

- The second utility requires executing `godoc` with the `-http` parameter. You can omit the equals sign in the
  presented command and put a space character in its place.

```shell
$ godoc -http=:8001
```

> Note that port numbers 0-1023 are restricted and can only be used by the root user, so it is better to avoid choosing
> one of those and pick something else, provided that it is not already in use by a different process

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

- Each Go source code begins with a package declaration. In this case, the name of the package is `main`, which has a
  special meaning in Go.

- Packages that are not part of the standard Go library are imported using their ***full internet path***.

- If you are creating an executable application is a `main()` function. Go considers this the entry point to the
  application and begins the execution of the application with the code found in the `main()` function of the main
  package.

- Two characteristics make `hw.go` an autonomous source file that can generate an executable binary: the name of the
  package, which should be ***main***, and the presence of the `main()` function.

#### Introducing functions

- There is a global Go rule that also applies to function and variable names and is valid for all packages except
  main: ***everything that begins with a lowercase letter is considered private and is accessible in the current package
  only.*** The only exception to this rule is package names, which can begin with either lowercase or uppercase letters.
  Having said that, _I am not aware of a Go package that begins with an uppercase letter!_

#### Introducing packages

- The package keyword helps you define the name of a new package, which can be anything you want with just one
  exception: if you are creating an executable application and not just a package that will be shared by other
  applications or packages, you should name your package ***main***.

- Packages of the standard Go library are imported by name (os) without the need for a hostname and a path, whereas
  external packages are imported using their full internet paths, like `github.com/spf13/cobra`.

## Running Go code

- There are two ways to execute Go code: as a _compiled language_ using `go build` or as a _scripting language_
  using `go run`.

- The generated executable is automatically named after the source code filename without the `.go` file extension.
  Therefore, because of the hw.go source file, the executable will be called hw. In case this is not what you want, go
  build supports the `-o` option, which allows you to change the filename and the path of the generated executable file.

- If no source files are provided, go build looks for a `main` package in the current directory.

```shell
$ go build hw.go
$ ./hw
Hello World!
```

- The `go run` command builds the named Go package, which in this case is the main package implemented in a single file,
  creates a temporary executable file, executes that file, and deletes it once it is done—to our eyes, this looks like
  using a scripting language.

```shell
$ go run hw.go
Hello World!
```

- If you want to test your code, then using `go run` is a better choice. However, if you want to create and distribute
  an
  executable binary, then go build is the way to go.

- if you are going to import a package, you should use some of this functionality—there are some exceptions to this rule
  that mainly have to with initializing connections, but they are not important for now.

- You either use a variable or you do not declare it at all.

- There is only one way to format curly braces in Go.

- Coding blocks in Go are embedded in curly braces even if they contain just a single statement or no statements at all.

- Go functions can return multiple values.

- You cannot automatically convert between different data types, even if they are of the same kind.

- Go requires the use of semicolons as statement terminators in many contexts, and the compiler automatically inserts
  the required semicolons when it thinks that they are necessary.
  Therefore, putting the opening curly brace ({) in its own line will make the Go compiler insert a semicolon at the end
  of the previous line (func main()), which is the main cause of the error message

## Important characteristics of Go

### Defining and using variables

- You can declare a new variable using the `var` keyword followed by the variable name, followed by the desired data type

- If you want, you can follow that declaration with `=` and an initial value for your variable. If there is an initial
  value given, you can omit the data type and the compiler will guess it for you.

- Very important Go rule: ***if no initial value is given to a variable, the Go compiler will automatically initialize
  that variable to the zero value of its data type.***

- The official name for `:=` is ***short assignment statement***, and it is very frequently used in Go, especially for getting
  the return values from functions and for loops with the range keyword.

- The short assignment statement can be used in place of a `var` declaration with an implicit type. You rarely see the use
  of `var` in Go; the `var` keyword is ***mostly used for declaring global or local variables without an initial value***. The
  _reason for the former is that every statement that exists outside the code of a function must begin with a keyword
  such as `func` or `var`_.

- This means that the short assignment statement cannot be used outside a function environment because it is not
  available there. Last, you might need to use `var` when you want to be explicit about the data type. For example, when
  you want int8 or int32 instead of int.

- Only `const` (when the value of a variable is not going to change) and `var` work for global variables, which are
  variables that are defined outside a function and are not embedded in curly braces.

### Printing variables

- If you want Go to take care of the printing, then you might want to use the `fmt.Println()` function. However, there are
  times that you want to have full control over how data is going to get printed. In such cases, you might want to use
  `fmt.Printf()`.

- If someone requires the use of control sequences that specify the data type of the variable that is going to get
  printed. Additionally, the `fmt.Printf()` function allows you to format the generated output

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

> Personally, I prefer to make global variables stand out by either beginning them with an uppercase letter or using all
> capital letters.

- The `float64()` type cast converts the value of AnotherGlobal to float64. Note that AnotherGlobal continues to be `int`.

- Go does not allow implicit data conversions like C.

- For conversions that are not straightforward (for example, string to int), there exist specialized functions that
  allow you to catch issues with the conversion in the form of an `error` variable that is returned by the function.

## Controlling program flow

- `if` statements use no parenthesis for embedding the conditions that need to be examined because Go does not use
  parentheses in general. As expected, `if` has support for `else` and `else if` statements.

- The `switch` statement has two different forms. ***In the first form, the switch statement has an expression that is being
  evaluated, whereas in the second form, the switch statement has no expression to evaluate. In that case, expressions
  are evaluated in each case statement, which increases the flexibility of switch.*** The main benefit you get from `switch`
  is that when used properly, it simplifies complex and hard-to-read if-else blocks.

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

- The order of the `case` statements is ***important** because only the first match is executed. The `fallthrough` keyword tells
  Go that after this branch is executed, it will continue with the next branch

- We need to convert user input into an integer value using a separate call, which in this case is a call to
  `strconv.Atoi()`.

- the `default` branch is there, which is a good practice because it can catch unexpected values.

## Iterating with for loops and range

- Go supports `for` loops as well as the `range` keyword for iterating over all the elements of `arrays`, `slices`, and `maps`.

- Depending on how you write a `for` loop, it can function as a `while` loop or an `infinite` loop. Moreover, for loops
  can implement the functionality of JavaScript's `forEach` function when combined with the `range` keyword.

> You need to put curly braces around a for loop even if it contains a single statement or no statements at all

- A `for` loop can be exited with a `break` keyword, and you can skip the current iteration with the `continue` keyword.
  When used with `range`, `for` loops allow you to visit all the elements of a slice or an `array` without knowing the
  size of the data structure.

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

- If you want to ignore either of these return values, which is not the case here, you can use `_` in the place of the
  value that you want to ignore. If you just need the index, you can leave out the second value from range entirely
  without using `_`.

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

- The `fmt.Scanln()` function can help you read user input while the program is already running and store it to
  a `string` variable, which is _passed as a pointer_ to `fmt.Scanln()`. The `fmt` package contains additional functions
  for reading user input from the console (`os.Stdin`), from files or from argument lists.

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

- By default, command-line arguments in Go are stored in the `os.Args` slice. Go also offers the flag package for parsing
  command-line arguments, but there are better and more powerful alternatives.

- It is important to know that the `os.Args` slice is properly initialized by Go and is available to the program when
  referenced. The `os.Args` slice contains string values:.

- The first command-line argument stored in the `os.Args` slice is always the _name of the executable_. The remaining
  command-line arguments are what comes after the name of the executable—the various command-line arguments are
  automatically separated by _space characters_ unless they are included in double or single quotes.

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

- We use the `error` variable returned by `strconv.ParseFloat()` to make sure that the call to `strconv.ParseFloat()`
  was successful, and we have a valid numeric value to process.