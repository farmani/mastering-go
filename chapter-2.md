# Chapter 2: Basic Go Data Types

* all Go variables should have a data type that is determined either implicitly or explicitly.

## The error data type

* Go provides a special data type for representing ***error conditions*** and ***error messages*** named error—in practice, *this means that Go treats errors as values*. In order to program successfully in Go, you should be aware of the error conditions that might occur with the functions and methods you are using and handle them accordingly.
* Go follows the next convention about `error` values: *if the value of an `error` variable is nil, then there was no error*. 
* Atoi (in `strconv.Atoi()`) stands for ASCII to Int
> If you want to learn more about `strconv.Atoi()`, you should execute `go doc strconv.Atoi` in your terminal window.
* Should you wish to return a custom error, you can use `errors.New()` from the `errors` package. This usually happens inside a function other than `main()` because `main()` does not return anything to any other function. Additionally, a good place to define your custom errors is inside the Go packages you create.
> You will most likely work with errors in your programs without needing the functionality of the errors package. Additionally, you do not need to define custom error messages unless you are creating big applications or packages.
* You can use the `fmt.Errorf()` function, which simplifies the creation of custom error messages—the `fmt.Errorf()` function returns an error value just like `errors.New()`.
* You should have a global error handling tactic in each application that should not change. In practice, this means the following:
	* 	All error messages should be handled at the same level, which means that all errors should either be returned to the calling function or be handled at the place they occurred.
	* It should be clearly documented how to handle critical errors. This means that there will be situations where a critical error should terminate the program and other times where a critical error might just create a warning message onscreen.
	* It is considered a good practice to send all error messages to the log service of your machine because this way the error messages can be examined at a later time. However, this is not always true, so exercise caution when setting this up—for example, cloud native apps do not work that way.

* The error data type is actually defined as an interface

```go
package main

import (
	"errors"
	"fmt"
	"os"
	"strconv"
)

// Custom error message with errors.New()
func check(a, b int) error {
	if a == 0 && b == 0 {
		return errors.New("this is a custom error message")
	}
	return nil
}

// Custom error message with fmt.Errorf()
func formattedError(a, b int) error {
	if a == 0 && b == 0 {
		return fmt.Errorf("a %d and b %d. UserID: %d", a, b, os.Getuid())
	}
	return nil
}
func main() {
	err := check(0, 10)
	if err == nil {
		fmt.Println("check() ended normally!")
	} else {
		fmt.Println(err)
	}
	err = check(0, 0)
	if err.Error() == "this is a custom error message" {
		fmt.Println("Custom error detected!")
	}
	err = formattedError(0, 0)
	if err != nil {
		fmt.Println(err)
	}
	i, err := strconv.Atoi("-123")
	if err == nil {
		fmt.Println("Int value is", i)
	}
	i, err = strconv.Atoi("Y123")
	if err != nil {
		fmt.Println(err)
	}
}
```
* The user ID of the user that executed the program with a call to `os.Getuid()`.
* The basic data types of Go that can be logically divided into two main categories: ***numeric*** data types and ***non-numeric*** data-types. Go also supports the bool data type, which can have a value of `true` or `false` only.

## Numeric data types
* Go supports `integer`, `floating-point`, and `complex` number values in various versions depending on the memory space they consume—this saves memory and computing time. Integer data types can be either signed or unsigned, which is not the case for floating point numbers.

| Data Type  | Description                       | 
|------------|-----------------------------------|
| int8       | 8-bit signed integer              | 
| int16      | 16-bit signed integer             |
| int32      | 32-bit signed integer             |
| int64      | 64-bit signed integer             |
| int        | 32- or 64-bit signed integer      |
| uint8      | 8-bit unsigned integer            |
| uint16     | 16-bit unsigned integer           |
| uint32     | 32-bit unsigned integer           |
| uint64     | 64-bit unsigned integer           |
| uint       | 32- or 64-bit unsigned integer    |
| float32    | 32-bit floating-point number      |
| float64    | 64-bit floating-point number      |
| complex64  | Complex number with float32 parts |
| complex128 | Complex number with float64 parts |

* The `int` and `uint` data types are special as they are the most efficient sizes for signed and unsigned integers on a given platform and can be either 32 or 64 bits each—their size is defined by Go itself. The `int` data type is the most widely used data type in Go due to its versatility.

```go
func main() {
    c1 := 12 + 1i
    c2 := complex(5, 7)
    fmt.Printf("Type of c1: %T\n", c1)
    fmt.Printf("Type of c2: %T\n", c2)
	
	var c3 complex64 = complex64(c1 + c2)
	fmt.Println("c3:", c3)
	fmt.Printf("Type of c3: %T\n", c3)
	cZero := c3 - c3
	fmt.Println("cZero:", cZero)
	
	x := 12
	k := 5
	fmt.Println(x)
	fmt.Printf("Type of x: %T\n", x)
	div := x / k
	fmt.Println("div", div)
	
	var m, n float64
	m = 1.223
	fmt.Println("m, n:", m, n)
	y := 4 / 2.3
	fmt.Println("y:", y)
	divFloat := float64(x) / float64(k)
	fmt.Println("divFloat", divFloat)
	fmt.Printf("Type of divFloat: %T\n", divFloat)
}
```
* Unless you are into mathematics, you will most likely not use `complex` numbers in your programs.
* Although cZero is equal to zero, it is still a `complex` number and a `complex64` variable.
* When you divide two integer values, you get an `integer` result even when the division is not perfect. This means that if this is not what you want, you should take extra care.
* As `n` does not have an initial value, it is automatically assigned with the zero value of its data type, which is 0 for the `float64`
* `divFloat := float64(x) / float64(k)` This is a type conversion where two integers (x and k) are converted to `float64` values. As the division between two `float64` values is a `float64` value, we get the result in the desired data type.
* The output shows that both c1 and c2 are `complex128` values, which is the preferred `complex` data type for the machine on which the code was executed. However, c3 is a `complex64` value because it was created using `complex64()`.

## Non-numeric data types
* Go has support for `Strings`, `Characters`, `Runes`, `Dates`, and `Times`. However, Go does not have a dedicated `char` data type.

> For Go, dates and times are the same thing and are represented by the same data type. However, it is up to you to determine whether a time and date variable contains valid information or not.


### Strings, Characters, and Runes
* A Go `string` is just a **_collection_** of `bytes` and can be accessed as a whole or as an array. A single `byte` can store any ASCII character—however, multiple bytes are usually needed for storing a single Unicode character.
* A `rune` is an `int32` value that is used for representing a single Unicode code point, which is an integer value that is used for representing single Unicode characters or, less frequently, providing formatting information.

> Although a `rune` is an `int32` value, you cannot compare a `rune` with an `int32` value. Go considers these two data types as totally different.

* You can create a new `byte` slice from a given string by using a `[]byte("A String")` statement. Given a byte slice variable b, you can convert it into a string using the `string(b)` statement. 
* When working with byte slices that contain Unicode characters, the number of bytes in a byte slice is not always connected to the number of characters in the byte slice, because most Unicode characters require more than one byte for their representation. As a result, when you try to print each single byte of a byte slice using `fmt.Println()` or `fmt.Print()`, the output is not text presented as characters but integer values. If you want to print the contents of a byte slice as text, you should either print it using `string(byteSliceVar)` or using `fmt.Printf()` with `%s` to tell `fmt.Printf()` that you want to print a string.
* You can define a `rune` using single quotes: `r := '€'` and you can print the integer value of the bytes that compose it as `fmt.Println(r)`—in this case, the integer value is `8364`. Printing it as a single Unicode character requires the use of the `%c` control string in `fmt.Printf()`.

```go
func main() {
    aString := "Hello World! €"
    fmt.Println("First character", string(aString[0]))
	
	// Runes
    // A rune
    r := '€'
    fmt.Println("As an int32 value:", r)
    // Convert Runes to text
    fmt.Printf("As a string: %s and as a character: %c\n", r, r)
    // Print an existing string as runes
    for _, v := range aString {
        fmt.Printf("%x ", v)
    }
    fmt.Println()
	
	// Print an existing string as characters
    for _, v := range aString {
        fmt.Printf("%c", v)
    }
    fmt.Println()
}
```
* What makes this a `rune` is the use of single quotes around the `€` character. The `%c` control string in `fmt.Printf()` prints a rune as a character.
* You can convert an `integer` value into a `string` in two main ways: using `string()` and using a function from the `strconv` package. However, the two methods are fundamentally different. The `string()` function converts an integer value into _a Unicode code point, which is a single character_, whereas functions such as `strconv.FormatInt()` and `strconv.Itoa()` convert an integer value into a string value _with the same representation and the same number of characters_.
```go
input := strconv.Itoa(n)
input = strconv.FormatInt(int64(n), 10)
input = string(n)
```
* `unicode.IsPrint()`, can help you to identify the parts of a string that are printable using runes.
```go
for i := 0; i < len(sL); i++ {
	if unicode.IsPrint(rune(sL[i])) {
		fmt.Printf("%c\n", sL[i])
	} else {
		fmt.Println("Not printable!")
	}
}
```
* if `unicode.IsPrint()` returns `true` then a `rune` is printable.
* This utility is very handy for filtering your input or filtering data before printing it on screen, storing it in log files, transferring it on a network, or storing it in a database.

> If you are working with text and text processing, you definitely need to learn all the gory details and functions of the `strings` package, so make sure that you experiment with all these functions and create many examples that will help you to clarify things.

```go
package main

import (
    "fmt"
    s "strings"
    "unicode"
)

var f = fmt.Printf

func main() {
	f("EqualFold: %v\n", s.EqualFold("Mihalis", "MIHAlis"))
	f("EqualFold: %v\n", s.EqualFold("Mihalis", "MIHAli"))

	f("Index: %v\n", s.Index("Mihalis", "ha"))
	f("Index: %v\n", s.Index("Mihalis", "Ha"))

	f("Prefix: %v\n", s.HasPrefix("Mihalis", "Mi"))
	f("Prefix: %v\n", s.HasPrefix("Mihalis", "mi"))
	f("Suffix: %v\n", s.HasSuffix("Mihalis", "is"))
	f("Suffix: %v\n", s.HasSuffix("Mihalis", "IS"))

	t := s.Fields("This is a string!")
	f("Fields: %v\n", len(t))
	t = s.Fields("ThisIs a\tstring!")
	f("Fields: %v\n", len(t))

	f("%s\n", s.Split("abcd efg", ""))
	f("%s\n", s.Replace("abcd efg", "", "_", -1))
	f("%s\n", s.Replace("abcd efg", "", "_", 4))
	f("%s\n", s.Replace("abcd efg", "", "_", 2))

	f("SplitAfter: %s\n", s.SplitAfter("123++432++", "++"))
	trimFunction := func(c rune) bool {
		return !unicode.IsLetter(c)
	}
	f("TrimFunc: %s\n", s.TrimFunc("123 abc ABC \t .", trimFunction))
}
```
* As we are going to use the `strings` package multiple times, we create a convenient alias for it named `s`. We do the same for the `fmt.Printf()` function where we create a global alias using a variable named `f`. These two shortcuts make code less populated with long, repeated lines of code. _You can use it when learning Go but this is not recommended in any kind of production software, as it makes code less readable._
* The `strings.EqualFold()` function compares two strings without considering their case and returns `true` when they are the same and `false` otherwise.
* The `strings.Index()` function checks whether the string of the second parameter can be found in the string that is given as the first parameter and returns the index where it was found for the first time. On an unsuccessful search, it returns `-1`.
* The `strings.HasPrefix()` function checks whether the given string, which is the first parameter, begins with the string that is given as the second parameter. In the previous code, the first call to `strings.HasPrefix()` returns `true`, whereas the second returns `false`.
* Similarly, the `strings.HasSuffix()` function checks whether the given string ends with the second string. Both functions _take into account the case of the input string and the case of the second parameter_.
* The handy `strings.Fields()` function splits the given string around one or more white space characters as defined by the `unicode.IsSpace()` function and returns a slice of substrings found in the input string. _If the input string contains white characters only, it returns an empty slice._
* The `strings.Split()` function allows you to split the given string according to the desired separator string—the `strings.Split()` function returns a string slice. Using `""` as the second parameter of `strings.Split()` allows you to process a string character by character.
* The `strings.Replace()` function takes four parameters. The first parameter is the string that you want to process. The second parameter contains the string that, if found, will be replaced by the third parameter of `strings.Replace()`. The last parameter is the maximum number of replacements that are allowed to happen. _If that parameter has a negative value, then there is no limit to the number of replacements that can take place_.
* The `strings.SplitAfter()` function splits its first parameter string into substrings based on the separator string that is given as the second parameter to the function. _The separator string is included in the returned slice._
* The last lines of code define a trim function named `trimFunction` that is used as the second parameter to `strings.TrimFunc()` in order to filter the given input based on the return value of the trim function—in this case, the trim function keeps all letters and nothing else due to the `unicode.IsLetter()` call.
* Visit the documentation page of the strings package at [https://golang.org/pkg/strings/](https://golang.org/pkg/strings/) for the complete list of available functions. You will see the functionality of the strings package in other places in this book.

### Times and dates

* The king of working with times and dates in Go is the `time.Time` data type, which represents an instant in time with **_nanosecond_** precision. Each `time.Time` value is associated with a **_location_** (time zone). 
* The `time.Now().Unix()` function returns the popular UNIX epoch time, which is the number of seconds that have elapsed since `00:00:00 UTC, January 1, 1970`. If you want to convert the UNIX time to the equivalent `time.Time` value, you can use the `time.Unix()` function.

> The `time.Since()` function calculates the time that has passed since a given time and returns a `time.Duration` variable—the duration data type is defined as `type Duration int64`. Although a Duration is, in reality, an int64 value, you cannot compare or convert a duration to an int64 value implicitly because Go does not allow implicit data type conversions.

* The single most important topic about Go and dates and times is the way Go parses a string in order to convert it into a date and a time.
* The function used for parsing is `time.Parse()` and its full signature is `Parse(layout, value string) (Time, error)`, where layout is the parse string and value is the input that is being parsed. The `time.Time` value that is returned is a moment in time with nanosecond precision and contains both date and time information.

The next table shows the most widely used strings for parsing dates and times.

| Parse Value | Meaning                              | 
|-------------|--------------------------------------|
| 03          | 12-hour value (12pm, 07am)           | 
| 15          | 24-hour value (23, 07)               |
| 04          | Minutes (55, 15)                     |
| 05          | Seconds (5, 23)                      |
| Mon         | Abbreviated day of week (Tue, Fri)   |
| Monday      | Day of week (Tuesday, Friday)        |
| 02          | Day of month (15, 31)                |
| 2006        | Year with 4 digits (2020, 2004)      |
| 06          | Year with the last 2 digits (20, 04) |
| Jan         | Abbreviated month name (Feb, Mar)    |
| January     | Full month name (July, August)       |
| MST         | Time zone (EST, UTC)                 |

* if you want to parse the _30 January 2020_ string and convert it into a Go date variable, you should match it against the `02 January 2006` string—you cannot use anything else in its place when matching a string with the 30 January 2020 format. Similarly, if you want to parse the _15 August 2020 10:00_ string, you should match it against the `02 January 2006 15:04` string. The documentation of the time package [https://golang.org/pkg/time/](https://golang.org/pkg/time/) contains even more detailed information about parsing dates and times—however, the ones presented here should be more than enough for regular use. 
* If you are matching a string that only contains the date, then your time will be set to `00:00` by Go and will most likely be incorrect. Similarly, when matching the time only, your date will be incorrect and should not be used.

> The formatting strings can be also used for printing dates and times in the desired format. So in order to print the current date in the 01-02-2006 format, you should use `time.Now().Format("01-02-2006")`.

```go
package main
import (
    "fmt"
    "os"
    "time"
)

func main() {
    start := time.Now()
    if len(os.Args) != 2 {
        fmt.Println("Usage: dates parse_string")
        return
    }
    dateString := os.Args[1]
	
	// Is this a date only?
	d, err := time.Parse("02 January 2006", dateString)
	if err == nil {
		fmt.Println("Full:", d)
		fmt.Println("Time:", d.Day(), d.Month(), d.Year())
	}


	// Is this a date + time?
	d, err = time.Parse("02 January 2006 15:04", dateString)
	if err == nil {
		fmt.Println("Full:", d)
		fmt.Println("Date:", d.Day(), d.Month(), d.Year())
		fmt.Println("Time:", d.Hour(), d.Minute())
	}

	// Is this a date + time with month represented as a number?
	d, err = time.Parse("02-01-2006 15:04", dateString)
	if err == nil {
		fmt.Println("Full:", d)
		fmt.Println("Date:", d.Day(), d.Month(), d.Year())
		fmt.Println("Time:", d.Hour(), d.Minute())
	}
	
	// Is it time only?
	d, err = time.Parse("15:04", dateString)
	if err == nil {
		fmt.Println("Full:", d)
		fmt.Println("Time:", d.Hour(), d.Minute())
	}

	t := time.Now().Unix()
	fmt.Println("Epoch time:", t)
	
	// Convert Epoch time to time.Time
	d = time.Unix(t, 0)
	fmt.Println("Date:", d.Day(), d.Month(), d.Year())
	fmt.Printf("Time: %d:%d\n", d.Hour(), d.Minute())
	duration := time.Since(start)
	fmt.Println("Execution time:", duration)
}
```

* you can access the individual fields of a variable that holds a valid date using `Day()`, `Month()`, and `Year()`.
* Note that it is compulsory that the string that is being examined contains the `-` and the `:` characters as specified in the `time.Parse()` call and that "02-01-2006 15:04" is different from "02/01/2006 1504".

> If a command-line argument such as `14 December 2020` contains space characters, you should put it in double quotes for the UNIX shell to treat it as a single command-line argument. Running `go run dates.go 14 December 2020` does not work.

* The presented utility accepts a date and a time and converts them into different time zones.

```go
loc, _ = time.LoadLocation("America/New_York")
fmt.Printf("New York Time: %s\n", now.In(loc))
```

## Go constants

* Go supports constants, which are variables that cannot change their values. Constants in Go are defined with the help of the `const` keyword. Generally speaking, constants can be either `global` or `local` variables.
* You might need to rethink your approach if you find yourself defining too many constant variables with a local scope.
* Strictly speaking, the value of a constant variable is defined at compile time, not at runtime.

### The constant generator iota
* The constant generator `iota` is used for declaring a sequence of related values that use incrementing numbers without the need to explicitly type each one of them.

> A Go type is a way of defining a new named type that uses the same underlying type as an existing type. This is mainly used for differentiating between different types that might use the same kind of data. The type keyword can be used for defining structures and interfaces.

```go
package main
import (
    "fmt"
)
type Digit int
type Power2 int
const PI = 3.1415926
const (
    C1 = "C1C1C1"
    C2 = "C2C2C2"
    C3 = "C3C3C3"
)

func main() {
	const s1 = 123
	var v1 float32 = s1 * 12
	fmt.Println(v1)
	fmt.Println(PI)
	// Having Digit type in here is because we want to have special enum type not simple int
	const (
		Zero Digit = iota
		One
		Two
		Three
		Four
	)

	fmt.Println(One)
	fmt.Println(Two)
	// To demonstrate this it’s better to show 1 as 00000001 and << operator shift all bits in it to left
	const (
		p2_0 Power2 = 1 << iota
		_
		p2_2
		_
		p2_4
		_
		p2_6
	)
	fmt.Println("2^0:", p2_0)
	fmt.Println("2^2:", p2_2)
	fmt.Println("2^4:", p2_4)
	fmt.Println("2^6:", p2_6)
}
```
> Although we are defining constants inside main(), constants can be normally found outside of main() or any other function or method.

* There is another constant generator `iota` here that is a little different from the previous one. Firstly, you can see the use of the underscore character in a `const` block with a constant generator `iota`, which allows you to _skip unwanted values_. Secondly, the value of `iota` always increments and can be used in expressions, which is what occurred in this case.

## Grouping similar data

> You can use `slices` instead of `arrays` almost anywhere in Go, but we are also demonstrating `arrays` because they can still be useful and because `slices` are implemented by Go using arrays!

### Arrays

* When defining an `array` variable, you must define its size. Otherwise, you should put `[...]` in the array declaration and let the Go compiler find out the length for you. So you can create an array with 4 string elements either as `[4]string{"Zero", "One", "Two", "Three"}` or as `[...]string{"Zero", "One", "Two", "Three"}`. If you put nothing in the square brackets, then a `slice` is going to be created instead. The (valid) indexes for that particular array are 0, 1, 2, and 3.
* You cannot **_change_** the size of an array after you have created it.
* When you pass an array to a function, what is happening is that Go creates a copy of that array and passes that copy to that function—therefore any changes you make to an array inside a function are lost when the function returns.
* As a result, arrays in Go are not very powerful, which is the main reason that Go has introduced an additional data structure named `slice` that is similar to an array but is dynamic in nature

### Slices

* Slices in Go are more powerful than `arrays` mainly because they are **_dynamic_**, which means that they can **grow** or **shrink** after creation if needed. Additionally, _any changes you make to a slice inside a function also affect the original slice_. But how does this happen? Strictly speaking, all parameters in Go are passed by value—there is no other way to pass parameters in Go.
* In reality, a slice value is a `header` that contains a pointer to an underlying `array` where the elements are actually stored, the length of the array, and its capacity. Note that the slice value does not include its elements, just a pointer to the underlying array. So, when you pass a slice to a function, Go makes a copy of that header and passes it to the function. This copy of the slice header includes the pointer to the underlying array. That slice header is defined in the `reflect` package [https://golang.org/pkg/reflect/#SliceHeader](https://golang.org/pkg/reflect/#SliceHeader) as follows:

```go
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}
```

* A side effect of passing the slice header is that it is faster to pass a slice to a function because Go does not need to make a copy of the slice and its elements, just the slice header.
* You can create a slice using `make()` or like an array without specifying its size or using [...]. If you do not want to initialize a slice, then using `make()` is better and faster. However, if you want to initialize it at the time of creation, then `make()` cannot help you. As a result, you can create a slice with three `float64` elements as `aSlice := []float64{1.2, 3.2, -4.5}`. Creating a slice with space for three float64 elements with `make()` is as simple as executing `make([]float64, 3)`. Each element of that slice has a value of `0`, which is the zero value of the `float64` data type.
* Both slices and arrays can have many dimensions—creating a slice with two dimensions with `make()` is as simple as writing `make([][]int, 2)`. This returns a slice with two dimensions where the _first dimension is 2 (rows) and the second dimension (columns) is unspecified and should be explicitly specified when adding data to it_.
* If you want to define and initialize a slice with two dimensions at the same time, you should execute something similar to `twoD := [][]int{{1, 2, 3}, {4, 5, 6}}`.
* You can find the length of an array or a slice using `len()`. 
* You can add new elements to a full slice using the `append()` function. `append()` automatically allocates the required memory space.

```go
package main
import "fmt"
func main() {
    // Create an empty slice
    aSlice := []float64{}
    // Both length and capacity are 0 because aSlice is empty
    fmt.Println(aSlice, len(aSlice), cap(aSlice))
    // Add elements to a slice
    aSlice = append(aSlice, 1234.56)
    aSlice = append(aSlice, -34.0)
    fmt.Println(aSlice, "with length", len(aSlice))
	// A slice with length 4
	t := make([]int, 4)
	t[0] = -1
	t[1] = -2
	t[2] = -3
	t[3] = -4
	// Now you will need to use append
	t = append(t, -5)
	fmt.Println(t)
	// A 2D slice
	// You can have as many dimensions as needed
	twoD := [][]int{{1, 2, 3}, {4, 5, 6}}
	// Visiting all elements of a 2D slice
	// with a double for loop
	for _, i := range twoD {
		for _, k := range i {
			fmt.Print(k, " ")
		}
		fmt.Println()
	}
	
	make2D := make([][]int, 2)
	fmt.Println(make2D)
	make2D[0] = []int{1, 2, 3, 4}
	make2D[1] = []int{-1, -2, -3, -4}
	fmt.Println(make2D)
}
```
* slices also have an additional property called `capacity` that can be found using the `cap()` function.

> The capacity of a slice is really important when you want to select a part of a slice or when you want to reference an array using a slice.
    
* The capacity shows how much a slice can be expanded without the need to allocate more memory and change the underlying array. Although after slice creation the capacity of a slice is handled by Go, a developer can define the capacity of a slice at creation time using the `make()` function—after that the capacity of the slice doubles each time the length of the slice is about to become bigger than its current capacity. The first argument of `make()` is the type of the slice and its dimensions, the second is its initial length and the third, _which is optional_, is the capacity of the slice. Although the data type of a slice cannot change after creation, the other two properties can change.
    
> Writing something like `make([]int, 3, 2)` generates an error message because at any given time the capacity of a slice (2) cannot be smaller than its length (3).
    
* What happens when you want to append a slice or an array to an existing slice? Go supports the `...` operator, which is used for exploding a slice or an array into multiple arguments before appending it to an existing slice.

```go
package main
import "fmt"
func main() {
    // Only length is defined. Capacity = length
    a := make([]int, 4)

	fmt.Println("L:", len(a), "C:", cap(a))
    // Initialize slice. Capacity = length
    b := []int{0, 1, 2, 3, 4}
    fmt.Println("L:", len(b), "C:", cap(b))

	// Same length and capacity
    aSlice := make([]int, 4, 4)
	
	// Add an element
	aSlice = append(aSlice, 5)

	fmt.Println(aSlice)
	// The capacity is doubled
	fmt.Println("L:", len(aSlice), "C:", cap(aSlice))
	// Now add four elements
	aSlice = append(aSlice, []int{-1, -2, -3, -4}...)
	
	fmt.Println(aSlice)
	// The capacity is doubled
	fmt.Println("L:", len(aSlice), "C:", cap(aSlice))
}
```
* The `...` operator expands `[]int{-1, -2, -3, -4}` into multiple arguments and append() appends each argument one by one to aSlice.

> Setting the correct capacity of a slice, if known in advance, will make your programs faster because Go will not have to allocate a new underlying array and have all the data copied over.

* In Go you select a part of a slice by defining two indexes, the first one is the beginning of the selection whereas the second one is the end of the selection, **_without including the element at that index_**, separated by `:`.

> If you want to process all the command-line arguments of a utility apart from the first one, which is its name, you can assign it to a new variable (arguments := os.Args) for ease of use and use the `arguments[1:]` notation to skip the first command-line argument.

* However, there is a variation where you can add a third parameter that controls the capacity of the resulting slice. So, using `aSlice[0:2:4]` selects the first 2 elements of a slice (at indexes 0 and 1) and creates a new slice with a maximum capacity of 4. _The resulting capacity is defined as the result of the 4-0 subtraction where 4 is the maximum capacity and 0 is the first index_—if the first index is omitted, it is automatically set to 0. In this case, the capacity of the result slice will be 4 because 4-0 equals 4. 
* If we would have used `aSlice[2:4:4]`, we would have created a new slice with the aSlice[2] and aSlice[3] elements and with a capacity of 4-2. _**Lastly, the resulting capacity cannot be bigger than the capacity of the original slice because in that case, you would need a different underlying array.**_

```go
package main
import "fmt"
func main() {
    aSlice := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    fmt.Println(aSlice)
    l := len(aSlice)
    // First 5 elements
    fmt.Println(aSlice[0:5])
	// Last 2 elements
	fmt.Println(aSlice[l-2 : l])
	// Last 2 elements
	fmt.Println(aSlice[l-2:])

	// First 5 elements
	t := aSlice[0:5:10]
	fmt.Println(len(t), cap(t))
	// Elements at indexes 2,3,4
	// Capacity will be 10-2
	t = aSlice[2:5:10]
	fmt.Println(len(t), cap(t))

	// Elements at indexes 0,1,2,3,4
	// New capacity will be 6-0
	t = aSlice[:5:6]
	fmt.Println(len(t), cap(t))
}
```
A byte slice is a slice of the byte data type ([]byte). Go knows that most byte slices are used to store strings and so makes it easy to switch between this type and the string type. 
What is special is that Go uses byte slices for performing file I/O operations because they allow you to determine with precision the amount of data you want to read or write to a file.”

> “As Go does not have a char data type, it uses byte and rune for storing character values. A single byte can only store a single ASCII character whereas a rune can store Unicode characters. However, a rune can occupy multiple bytes.”

```go
package main
import "fmt"
func main() {
    // Byte slice
    b := make([]byte, 12)
    fmt.Println("Byte slice:", b)

	
	b = []byte("Byte slice €") // This is how you convert a string into a byte slice.
    fmt.Println("Byte slice:", b)
		
    
	// Print byte slice contents as text
    fmt.Printf("Byte slice as text: %s\n", b)
    fmt.Println("Byte slice as text:", string(b))

    
	// Length of b
    fmt.Println("Length of b:", len(b))
}
```
* As Unicode characters like `€` need more than one `byte` for their representation, the length of the byte slice might not be the same as the length of the string that it stores. 
* The previous code shows how to print the contents of a `byte` slice as text using two techniques. The first one is by using the `%s` control string and the second one using `string()`.

* There is no default function for deleting an element from a slice, which means that if you need to delete an element from a slice, you must write your own code. Deleting an element from a slice can be tricky. The first technique virtually divides the original slice into two slices, split at the index of the element that needs to be deleted. Neither of the two slices includes the element that is going to be deleted. After that, we concatenate these two slices and creates a new one. The second technique copies the last element at the place of the element that is going to be deleted and creates a new slice by excluding the last element from the original slice.
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
        fmt.Println("Need an integer value.")
        return
    }
    index := arguments[1]
    i, err := strconv.Atoi(index)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("Using index", i)
    aSlice := []int{0, 1, 2, 3, 4, 5, 6, 7, 8}
    fmt.Println("Original slice:", aSlice)
    // Delete element at index i
    if i > len(aSlice)-1 {
        fmt.Println("Cannot delete element", i)
        return
    }
    // The ... operator auto expands aSlice[i+1:] so that
    // its elements can be appended to aSlice[:i] one by one
    aSlice = append(aSlice[:i], aSlice[i+1:]...)
    fmt.Println("After 1st deletion:", aSlice)
	// Delete element at index i
	if i > len(aSlice)-1 {
		fmt.Println("Cannot delete element", i)
		return
	}
	// Replace element at index i with last element
	aSlice[i] = aSlice[len(aSlice)-1]
	// Remove last element
	aSlice = aSlice[:len(aSlice)-1]
	fmt.Println("After 2nd deletion:", aSlice)
}
```

* We replace the element that we want to delete with the last element using the `aSlice[i] = aSlice[len(aSlice)-1]` statement, and then we remove the last element with the `aSlice = aSlice[:len(aSlice)-1]` statement.
* As mentioned before, behind the scenes, each slice is implemented using an underlying array. The length of the underlying array is the same as the capacity of the slice and there exist pointers that connect the slice elements to the appropriate array elements.
* Go allows you to reference an array or a part of an array using a slice. This has some strange capabilities including the fact that the changes to the slice affect the referenced array! However, **_when the capacity of the slice changes, the connection to the array ceases to exist!_** This happens because when the capacity of a slice changes, so does the underlying array, and the connection between the slice and the original array does not exist anymore.

```go
package main
import (
    "fmt"
)
func change(s []string) {
    s[0] = "Change_function"
}

func main() {
	a := [4]string{"Zero", "One", "Two", "Three"}
	fmt.Println("a:", a)
	var S0 = a[0:1]
	fmt.Println(S0)
	S0[0] = "S0"
	
	var S12 = a[1:3]
	fmt.Println(S12)
	S12[0] = "S12_0"
	S12[1] = "S12_1"
	fmt.Println("a:", a)
	
	// Changes to slice -> changes to array
	change(S12)
	fmt.Println("a:", a)

	// capacity of S0
	fmt.Println("Capacity of S0:", cap(S0), "Length of S0:", len(S0))
	// Adding 4 elements to S0
	S0 = append(S0, "N1")
	S0 = append(S0, "N2")
	S0 = append(S0, "N3")
	a[0] = "-N1"

	// Changing the capacity of S0
	// Not the same underlying array anymore!
	S0 = append(S0, "N4")
	fmt.Println("Capacity of S0:", cap(S0), "Length of S0:", len(S0))
	// This change does not go to S0
	a[0] = "-N1-"
	// This change does not go to S12
	a[1] = "-N2-"
	
	fmt.Println("S0:", S0)
	fmt.Println("a: ", a)
	fmt.Println("S12:", S12)
}
```
* As the slice and the array are connected, any changes you make to the slice will also affect the array even if the changes take place inside a function.
* As the capacity of `S0` changes, it is no longer connected to the same underlying array (a).
* However, array a and slice `S12` are still connected because the capacity of `S12` has not changed.

* Go offers the `copy()` function for copying an existing array to a slice or an existing slice to another slice. However, the use of `copy()` can be tricky because the destination slice is not auto-expanded if the source slice is bigger than the destination slice. Additionally, if the destination slice is bigger than the source slice, then `copy()` does not empty the elements from the destination slice that did not get copied. 
* Here we run the `copy(a1, a2)` command. In this case, the a2 slice is bigger than a1. After copy(a1, a2), a2 remains the same, which makes perfect sense as a2 is the input slice, whereas the first element of a2 is copied to the first element of a1 because a1 has space for a single element only.

* The `sort` package can sort slices of built-in data types without the need to write any extra code. Additionally, Go provides the `sort.Reverse()` function for sorting in the reverse order than the default. However, what is fascinating is that sort allows you to write your own sorting functions for custom data types by implementing the `sort.Interface` interface 
* So, you can sort a slice of integers saved as `sInts` by `typing sort.Ints(sInts)`. When sorting a slice of integers in reverse order using `sort.Reverse()`, you need to pass the desired slice to `sort.Reverse()` using `sort.IntSlice(sInts)` because the IntSlice type implements the `sort.Interface` internally, which allows you to sort in a different way than usual. The same applies to the other standard Go data types.


```go
package main
import (
    "fmt"
    "sort"
)
func main() {
	sInts := []int{1, 0, 2, -3, 4, -20}
	sFloats := []float64{1.0, 0.2, 0.22, -3, 4.1, -0.1}
	sStrings := []string{"aa", "a", "A", "Aa", "aab", "AAa"}
	fmt.Println("sInts original:", sInts)
	sort.Ints(sInts)
	fmt.Println("sInts:", sInts)
	sort.Sort(sort.Reverse(sort.IntSlice(sInts)))
	fmt.Println("Reverse:", sInts)
	fmt.Println("sFloats original:", sFloats)
	sort.Float64s(sFloats)
	fmt.Println("sFloats:", sFloats)
	sort.Sort(sort.Reverse(sort.Float64Slice(sFloats)))
	fmt.Println("Reverse:", sFloats)
	fmt.Println("sStrings original:", sStrings)
	sort.Strings(sStrings)
	fmt.Println("sStrings:", sStrings)
	sort.Sort(sort.Reverse(sort.StringSlice(sStrings)))
	fmt.Println("Reverse:", sStrings)
}
```
* As `sort.Interface` knows how to sort integers, it is trivial to sort them in reverse order. Sorting in reverse order is as simple as calling the `sort.Reverse()` function.

## Pointers

* Go has support for pointers but _not for pointer arithmetic_, which is the cause of many bugs and errors in programming languages like C. A pointer is the memory address of a variable. You need to _dereference_ a pointer in order to get its value—dereferencing is performed using the `*` character in front of the pointer variable. Additionally, you can get the memory address of a normal variable using an `&` in front of it.
* If a pointer variable points to an existing regular variable, then any changes you make to the stored value using the pointer variable will modify the regular variable.

> The format and the values of memory addresses might be different between different machines, different operating systems, and different architectures.

* The main benefit you get from pointers is that passing a variable to a function as a pointer (**_we can call that by reference_**) does not discard any changes you make to the value of that variable inside that function when the function returns. There exist times when you want that functionality because it simplifies your code, but the price you pay for that simplicity is being extra careful with what you do with a pointer variable. _Remember that slices are passed to functions without the need to use a pointer_—it is Go that passes the pointer to the underlying array of a slice and there is no way to change that behavior
* Pointers allow you to share data between functions. However, when sharing data between functions and goroutines, you should be extra careful with race condition issues.
* Pointers are also very handy when you want to tell the difference between the zero value of a variable and a value that is not set (`nil`). This is particularly useful with structures because pointers (and therefore pointers to structures), can have the `nil` value, which means that you can compare a pointer to a structure with the `nil` value, which is not allowed for normal structure variables.
* Having support for pointers and, more specifically, pointers to structures allows Go to support data structures such as `linked lists` and `binary trees`, which are widely used in computer science. Therefore, you are allowed to define a structure field of a `Node` structure as `Next *Node`, which is a pointer to another Node structure.

```go
package main
import "fmt"
type aStructure struct {
    field1 complex128
    field2 int
}

func processPointer(x *float64) {
	*x = *x * *x
}

func returnPointer(x float64) *float64 {
	temp := 2 * x
	return &temp
}
func bothPointers(x *float64) *float64 {
	temp := 2 * *x
	return &temp
}

func main() {
	var f float64 = 12.123
	fmt.Println("Memory address of f:", &f)
	
	// Pointer to f
	fP := &f
	fmt.Println("Memory address of f:", fP)
	fmt.Println("Value of f:", *fP)
	// The value of f changes
	processPointer(fP)
	fmt.Printf("Value of f: %.2f\n", f)
	
	// The value of f does not change
	x := returnPointer(f)
	fmt.Printf("Value of x: %.2f\n", *x)
	// The value of f does not change
	xx := bothPointers(fP)
	fmt.Printf("Value of xx: %.2f\n", *xx)
	
	// Check for empty structure
	var k *aStructure
	// This is nil because currently k points to nowhere
	fmt.Println(k)
	// Therefore you are allowed to do this:
	if k == nil {
		k = new(aStructure)
	}
	fmt.Printf("%+v\n", k)
	if k != nil {
		fmt.Println("k is not nil!")
	}
}
```

* The k variable is a pointer to an aStructure structure. As k points to nowhere, Go makes it point to nil, which is the zero value for pointers.
* As `k` is `nil`, we are allowed to assign it to an empty `aStructure` value with `new(aStructure)` without losing any data. Now, `k` is no longer nil but both fields of `aStructure` have the zero values of their data types.

## Generating random numbers

* Go can help you with that using the functionality of the `math/rand` package. Each random number generator needs a **_seed_** to start producing numbers. The **_seed_** is used for initializing the entire process and is extremely important because _if you always start with the same seed, you will always get the same sequence of pseudo-random numbers_.

* The `rand.Seed()` function is used for initializing a random number generator.

```go
func random(min, max int) int {
    return rand.Intn(max-min) + min
}
```

* `rand.Intn()` _generates non-negative random integers from 0 up to the value of its single parameter minus 1._

```go
func getString(len int64) string {
    temp := ""
    startChar := "!"
    var i int64 = 1
    for {
        myRand := random(MIN, MAX)
		newChar := string(startChar[0] + byte(myRand))
		temp = temp + newChar
		if i == len {
			break
		}
		i++
	}
	return temp
}
```

* The total number of printable characters in the ASCII table is 94. Therefore, the values of the MIN and MAX global variables, which are not shown here, are 0 and 94, respectively. 
* The `startChar` variable holds the first ASCII character that can be generated by the utility, which, in this case, is the exclamation mark, which has a decimal ASCII value of 33.
* The `string(startChar[0] + byte(myRand))` statement converts the random integers into characters in the desired range.
* If you intend to use these pseudo-random numbers for security-related work, it is important that you use the `crypto/rand` package, which implements a cryptographically secure pseudo-random number generator. You do not need to define a seed when using the `crypto/rand` package.

```go
func generateBytes(n int64) ([]byte, error) {
    b := make([]byte, n)
    _, err := rand.Read(b)
    if err != nil {
        return nil, err
    }
    return b, nil
}
```

* The `rand.Read()` function randomly generates numbers that occupy the entire b `byte` slice. You need to decode that byte slice using `base64.URLEncoding.EncodeToString(b)` in order to get a valid string without any control or unprintable characters.

## Exercises
* Create a function that concatenates two arrays into a new slice.
* Create a function that concatenates two arrays into a new array.
* Create a function that concatenates two slices into a new array.

## Additional resources

* The sort package documentation: [https://golang.org/pkg/sort/](https://golang.org/pkg/sort/)
* The time package documentation: [https://golang.org/pkg/time/](https://golang.org/pkg/time/)
* The crypto/rand package documentation: [https://golang.org/pkg/crypto/rand/](https://golang.org/pkg/crypto/rand/)
* The math/rand package documentation: [https://golang.org/pkg/math/rand/](https://golang.org/pkg/math/rand/)