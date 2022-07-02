# Chapter 2: “Basic Go Data Types”

* all Go variables should have a data type that is determined either implicitly or explicitly.

## The error data type

* Go provides a special data type for representing ***error conditions*** and ***error messages*** named error—in practice, *this means that Go treats errors as values*. In order to program successfully in Go, you should be aware of the error conditions that might occur with the functions and methods you are using and handle them accordingly.
* Go follows the next convention about `error` values: *if the value of an `error` variable is nil, then there was no error*. 
* Atoi (in `strconv.Atoi()`) stands for ASCII to Int
> If you want to learn more about `strconv.Atoi()`, you should execute `go doc strconv.Atoi` in your terminal window.
* Should you wish to return a custom error, you can use `errors.New()` from the `errors` package. This usually happens inside a function other than `main()` because `main()` does not return anything to any other function. Additionally, a good place to define your custom errors is inside the Go packages you create.
> You will most likely work with errors in your programs without needing the functionality of the errors package. Additionally, you do not need to define custom error messages unless you are creating big applications or packages.
* You can use the `fmt.Errorf()` function, which simplifies the creation of custom error messages—the `fmt.Errorf()` function returns an error value just like `errors.New()`.
* You should have a global error handling tactic in each application that should not change. In practice, this means the following:
	* 	All error messages should be handled at the same level, which means that all errors should either be returned to the calling function or be handled at the place they occurred.
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


## Go constants
### The constant generator iota
## Grouping similar data
### Arrays
### Slices
## Pointers
## Generating random numbers
## Updating the phone book application
## Exercises