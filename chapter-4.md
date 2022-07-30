# Reflection and Interfaces
**_Interfaces_** are about expressing abstractions and identifying and defining behaviors that can be shared among different _data types_.

Interfaces work with _**methods on types**_ or **_type methods_**, which are like functions attached to given data types, which in Go are _usually structures_. Remember that once you implement the required type methods of an interface, that interface is satisfied _implicitly_, which is also the case with the empty interface that is explained in this chapter.

Another handy Go feature is **_reflection_**, which _allows you to examine the structure of a data type at execution time_. However, as reflection is an advanced Go feature, you do not need to use it on a regular basis.

## Reflection
You might be wondering how you can find out the names of the fields of a structure at execution time. In such cases, you need to use reflection. _Apart from enabling you to print the fields and the values of a structure, reflection also allows you to explore and manipulate unknown structures like the ones created from decoding JSON data_.

The two main questions:

* Why was reflection included in Go?
* When should I use reflection?
    
To answer the first question, reflection allows you to dynamically learn the `type` of an arbitrary object along with information about its structure. Go provides the `reflect` package for working with reflection. Remember when we said in a previous chapter that `fmt.Println()` is clever enough to understand the data types of its parameters and act accordingly? Well, behind the scenes, the `fmt` package uses reflection to do that.

As far as the second question is concerned, reflection allows you to handle and work with data types that do not exist at the time at which you write your code but might exist in the future, which is when we use an existing package with user-defined data types.

Additionally, reflection might come in handy when you have to work with data types that do not implement a common interface and therefore have an uncommon or unknown behavior, this does not mean that they have bad or erroneous behavior, just uncommon behavior such as a user-defined structure.

> The introduction of generics in Go might make the use of reflection less frequent in some cases, because with generics you can work with different data types more easily and without the need to know their exact data types in advance. However, nothing beats reflection for fully exploring the structure and the data types of a variable.

The most useful parts of the `reflect` package are two data types named `reflect.Value` and `reflect.Type`. Now, `reflect.Value` is used for storing values of any type, whereas `reflect.Type` is used for representing Go types. There exist two functions named `reflect.TypeOf()` and `reflect.ValueOf()` that return the `reflect.Type` and `reflect.Value` values, respectively. Note that `reflect.TypeOf()` returns the _actual type_ of variable—if we are examining a structure, it returns the **name** of the structure.

As structures are really important in Go, the `reflect` package offers the `reflect.NumField()` method for listing the number of fields in a structure as well as the `Field()` method for getting the `reflect.Value` value of a specific field of a structure.

The `reflect` package also defines the `reflect.Kind` data type, which is used for representing the specific _data type_ of a variable: `int`, `struct`, etc. The documentation of the `reflect` package lists all possible values of the `reflect.Kind` data type. The `Kind()` function returns the _kind_ of a variable.

Last, the `Int()` and `String()` methods return the integer and string value of a `reflect.Value`, respectively.

> Reflection code can look unpleasant and hard to read sometimes. Therefore, according to the Go philosophy, you should rarely use reflection unless it is absolutely necessary because despite its cleverness, it does not create clean code.

### Learning the internal structure of a Go structure

```go
package main
import (
    "fmt"
    "reflect"
)
type Secret struct {
    Username string
    Password string
}
type Record struct {
    Field1 string
    Field2 float64
    Field3 Secret
}
func main() {
    A := Record{"String value", -12.123, Secret{"Mihalis", "Tsoukalos"}}
	r := reflect.ValueOf(A)
	fmt.Println("String value:", r.String())
	iType := r.Type()
	fmt.Printf("i Type: %s\n", iType)
	fmt.Printf("The %d fields of %s are\n", r.NumField(), iType)
	for i := 0; i < r.NumField(); i++ {
		fmt.Printf("\t%s ", iType.Field(i).Name)
		fmt.Printf("\twith type: %s ", r.Field(i).Type())
		fmt.Printf("\tand value _%v_\n", r.Field(i).Interface())
		// Check whether there are other structures embedded in Record
		k := reflect.TypeOf(r.Field(i).Interface()).Kind()
		// Need to convert it to string in order to compare it
		if k.String() == "struct" {
			fmt.Println(r.Field(i).Type())
		}
		// Same as before but using the internal value
		if k == reflect.Struct {
			fmt.Println(r.Field(i).Type())
		}
	}
}
```
In order to check the **_data type_** of a variable with a `string`, we need to convert the data type into a `string` variable first.

You can also use the internal representation of a data type during checking. However, this makes less sense than using a `string` value.

`main.Record` is the **_full unique name_** of the structure as defined by Go—main is the package name and Record is the struct name. This happens so that Go can differentiate between the elements of different packages.

If you were to make changes to the values of the structure fields, you would use the `Elem()` method and pass the structure as a pointer to `ValueOf()`—remember that pointers allow you to make changes to the actual variable. There exist methods that allow you to modify an existing value. In our case, we are going to use `SetString()` for modifying a string field and `SetInt()` for modifying an int field.

### Changing structure values using reflection
what is more practical is being able to change values in the Go structure

```go
package main
import (
    "fmt"
    "reflect"
)
type T struct {
    F1 int
    F2 string
    F3 float64
}
func main() {
    A := T{1, "F2", 3.0}
    fmt.Println("A:", A)
    r := reflect.ValueOf(&A).Elem()
    fmt.Println("String value:", r.String())
    typeOfA := r.Type()
    for i := 0; i < r.NumField(); i++ {
        f := r.Field(i)
        tOfA := typeOfA.Field(i).Name
        fmt.Printf("%d: %s %s = %v\n", i, tOfA, f.Type(), f.Interface())
        k := reflect.TypeOf(r.Field(i).Interface()).Kind()
        if k == reflect.Int {
            r.Field(i).SetInt(-100)
        } else if k == reflect.String {
            r.Field(i).SetString("Changed!")
        }
    }
    fmt.Println("A:", A)
}
```
With the use of `Elem()` and a pointer to variable A, variable A can be modified if needed.

We are using `SetInt()` for modifying an integer value and `SetString()` for modifying a string value.

### The three disadvantages of reflection
reflection should be used sparingly for three main reasons:

* The first reason is that extensive use of reflection will make your programs **_hard to read_** and maintain. A potential solution to this problem is good documentation, but developers are notorious for not having the time to write proper documentation. 
* The second reason is that the Go code that uses reflection makes your programs **_slower_**. Generally speaking, Go code that works with a particular data type is always faster than Go code that uses reflection to dynamically work with any Go data type. Additionally, such dynamic code makes it difficult for tools to refactor or analyze your code. 
* The last reason is that reflection **_errors cannot be caught at build time_** and are reported at runtime as panics, which means that _reflection errors can potentially crash your programs_. This can happen months or even years after the development of a Go program! One solution to this problem is extensive testing before a dangerous function call. However, this adds even more Go code to your programs, which makes them even slower.

## Type methods
A `type method` is a function that is attached to a specific data type. Although type methods (or methods on types) are in reality `functions`, they are defined and used in a slightly different way.

> The methods on types feature gives some object-oriented capabilities to Go, which is very handy and is used extensively in Go. Additionally, interfaces require type methods to work.
    
Defining new type methods is as simple as creating new functions, provided that you follow certain rules that associate the function with a data type.

### Creating type methods
Having a data type called ar2x2, you can create a type method named FunctionName for it as follows:

```go
func (a ar2x2) FunctionName(parameters) <return values> {
    ...
}
```

The `(a ar2x2)` part is what makes the `FunctionName()` function a type method because it associates `FunctionName()` with the `ar2x2` data type. No other data type can use that function. However, you are free to implement `FunctionName()` for other data types or as a regular function. If you have a `ar2x2` variable named `varAr`, you can invoke `FunctionName()` as `varAr.FunctionName(...)`, which looks like selecting the field of a structure variable.

You are not obligated to develop type methods if you do not want to. In fact, each type method can be rewritten as a regular function. Therefore, `FunctionName()` can be rewritten as follows:

```go
func FunctionName(a ar2x2, parameters...) <return values> {
    ...
}
```

_Have in mind that under the hood, the Go compiler does turn methods into regular function calls with the self value as the first parameter_. However, interfaces require the use of type methods to work.

> The expressions used for selecting a field of a structure or a type method of a data type, which would replace the ellipsis after the variable name above, are called selectors.
    
_Performing calculations between matrices of a given size is one of the rare cases where using an array instead of a slice makes more sense because you do not have to modify the size of the matrices._ Some might argue that using a slice instead of an array pointer is a better practice—you are allowed to use what makes more sense to you.

Most of the time, and when there is such a need, _the results of a type method are saved in the variable that invoked the type method_—in order to implement that for the `ar2x2` data type, we pass a pointer to the array that invoked the type method, like func (a *ar2x2).

### Using type methods

The `Add()` function and the `Add()` method use the exact same algorithm for adding two matrices. The only difference between them is the way they are being called and the fact that the function returns an array whereas the method saves the result to the calling variable.

> If you are defining type methods for a structure, you should make sure that the names of the type methods do not conflict with any field name of the structure because the Go compiler will reject such ambiguities.

```go
package main
import (
    "fmt"
    "os"
    "strconv"
)
type ar2x2 [2][2]int

// Add Traditional function
func Add(a, b ar2x2) ar2x2 {
    c := ar2x2{}
    for i := 0; i < 2; i++ {
        for j := 0; j < 2; j++ {
            c[i][j] = a[i][j] + b[i][j]
        }
    }
    return c
}

// Add Type method
func (a *ar2x2) Add(b ar2x2) {
	for i := 0; i < 2; i++ {
		for j := 0; j < 2; j++ {
			a[i][j] = a[i][j] + b[i][j]
		}
	}
}

// Subtract Type method
func (a *ar2x2) Subtract(b ar2x2) {
    for i := 0; i < 2; i++ {
        for j := 0; j < 2; j++ {
            a[i][j] = a[i][j] - b[i][j]
        }
    }
}

// Multiply Type method
func (a *ar2x2) Multiply(b ar2x2) {
    a[0][0] = a[0][0]*b[0][0] + a[0][1]*b[1][0]
    a[1][0] = a[1][0]*b[0][0] + a[1][1]*b[1][0]
    a[0][1] = a[0][0]*b[0][1] + a[0][1]*b[1][1]
    a[1][1] = a[1][0]*b[0][1] + a[1][1]*b[1][1]
}

func main() {
    if len(os.Args) != 9 {
        fmt.Println("Need 8 integers")
        return
    }
    k := [8]int{}
    for index, i := range os.Args[1:] {
        v, err := strconv.Atoi(i)
        if err != nil {
            fmt.Println(err)
            return
        }
        k[index] = v
    }
    a := ar2x2{{k[0], k[1]}, {k[2], k[3]}}
    b := ar2x2{{k[4], k[5]}, {k[6], k[7]}}
    fmt.Println("Traditional a+b:", Add(a, b))
    a.Add(b)
    fmt.Println("a+b:", a)
    a.Subtract(a)
    fmt.Println("a-a:", a)
    a = ar2x2{{k[0], k[1]}, {k[2], k[3]}}
    
    a.Multiply(b)
    fmt.Println("a*b:", a)
    a = ar2x2{{k[0], k[1]}, {k[2], k[3]}}
    b.Multiply(a)
    fmt.Println("b*a:", b)
}
```

What happens is that the `ar2x2` variable that called the `Add()` method is going to be modified and hold the result—this is the reason for using a pointer when defining the type method.

## Interfaces

An interface is a Go mechanism for defining behavior that is implemented using a set of methods. Interfaces play a key role in Go and can simplify the code of your programs when they have to deal with multiple data types that perform the same task. But remember, interfaces should not be unnecessarily complex. If you decide to create your own interfaces, then you should begin with a common behavior that you want to be used by multiple data types.

Interfaces work with methods on types (or type methods), which are like functions attached to given data types, which in Go are usually structures (although we can use any data type we want).

Once you implement the required type methods of an interface, that interface is satisfied implicitly.

The empty interface is defined as just `interface{}`. As the empty interface has no methods, it means that it is already implemented by all data types.

A Go `interface type` defines (or describes) the behavior of other types by specifying a set of methods that need to be implemented for supporting that behavior. For a data type to satisfy an interface, it needs to implement all the type methods required by that interface. _Therefore, interfaces are `abstract types` that specify a set of methods that need to be implemented so that another type can be considered an instance of the interface._ So, an interface is two things: a set of methods and a type. _Have in mind that small and well-defined interfaces are usually the most popular ones._

> As a rule of thumb, only create a new interface when you want to share a common behavior between two or more concrete data types. This is basically **_duck typing_**.

The biggest advantage you get from interfaces is that if needed, you can pass a variable of a data type that implements a particular interface to any function that expects a parameter of that specific interface, which saves you from having to write separate functions for each supported data type. However, Go offers an alternative to this with the recent addition of generics.

Interfaces can also be used for providing a kind of polymorphism in Go, which is an object-oriented concept. Polymorphism offers a way of accessing objects of different types in the same uniform way when they share a common behavior.

Lastly, interfaces can be used for composition. In practice, this means that you can combine existing interfaces and create new ones that offer the combined behavior of the interfaces that were brought together.

There is nothing prohibiting you from including additional methods in the definition of the ABC interface if the combination of existing interfaces does not describe the desired behavior accurately.

**_When you combine existing interfaces, it is better that the interfaces do not contain methods with the same name._**

> What you should keep in mind is that there is no need for an interface to be impressive and require the implementation of a large number of methods. In fact, the fewer methods an interface has, the more generic and widely used it can be, which improves its usefulness and therefore its usage.

### The `sort.Interface` interface

The `sort` package contains an interface named `sort.Interface` that allows you to sort slices according to your needs and your data, provided that you implement `sort.Interface` for the custom data types stored in your slices.

```go
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}
```

The `Len()` method returns the length of the slice that will be sorted and helps the interface to process all slice elements whereas the `Less()` method, which compares and sorts elements in pairs, defines how elements are going to be compared and therefore sorted. The return value of `Less()` is bool, which means that `Less()` only cares about whether the element at index i is bigger or not than the element at index j in the way that the two elements are being compared. Lastly, the `Swap()` method is used for swapping two elements of the slice, which is required for the sorting algorithm to work.

```go
package main
import (
    "fmt"
    "sort"
)
type S1 struct {
    F1 int
    F2 string
    F3 int
}
// S2 We want to sort records based on the value of F3.F1,
// Which is equivalent to S1.F1 as F3 is an S1 structure
type S2 struct {
    F1 int
    F2 string
    F3 S1
}

type S2slice []S2

// Len Implementing sort.Interface for S2slice
func (a S2slice) Len() int {
	return len(a)
}

// Less What field to use when comparing
func (a S2slice) Less(i, j int) bool {
	return a[i].F3.F1 < a[j].F3.F1
}

func (a S2slice) Swap(i, j int) {
	a[i], a[j] = a[j], a[i]
}

func main() {
	data := []S2{
		S2{1, "One", S1{1, "S1_1", 10}},
		S2{2, "Two", S1{2, "S1_1", 20}},
		S2{-1, "Two", S1{-1, "S1_1", -20}},
	}
	fmt.Println("Before:", data)
	sort.Sort(S2slice(data))
	fmt.Println("After:", data)
	// Reverse sorting works automatically
	sort.Sort(sort.Reverse(S2slice(data)))
	fmt.Println("Reverse:", data)
}
```

You need to have a slice because **_all sorting operations work on slices_**. It is for this slice, which should be a new data type that in this case is called S2slice, that you are going to implement the three type methods of the `sort.Interface`.

Once you have implemented `sort.Interface`, you'll see that `sort.Reverse()`, which is used for reverse sorting your slice, works automatically

### The empty interface

`Print()` requires an `interface{}` parameter that can accept both S1 and S2 variables. The `fmt.Println(s)` statement inside `Print()` can work with both S1 and S2.

> If you create a function that accepts one or more `interface{}` parameters, and you run a statement that can only be applied to a limited number of data types, things will not work out well. As an example, not all `interface{}` parameters can be multiplied by 5 or be used in `fmt.Printf()` with the `%d` control string.

```go
package main
import "fmt"
type S1 struct {
    F1 int
    F2 string
}
type S2 struct {
    F1 int
    F2 S1
}
func Print(s interface{}) {
    fmt.Println(s)
}
func main() {
    v1 := S1{10, "Hello"}
    v2 := S2{F1: -1, F2: v1}
    Print(v1)
    Print(v2)
	// Printing an integer
	Print(123)
	// Printing a string
	Print("Go is the best!")
}
```

Using the empty interface is easy as soon as you realize that you can pass any type of variable in the place of an `interface{}` parameter, and you can return any data type as an `interface{}` return value. However, **_with great power comes great responsibility_**—you should be very careful with `interface{}` parameters and their return values, because in order to use their real values you have to be sure about their underlying data type.

### Type assertions and type switches

A `type assertion` is a mechanism for working with the underlying concrete value of an interface. This mainly happens because interfaces are virtual data types without their own values—interfaces just define behavior and do not hold data of their own. But what happens when you do not know the data type before attempting a type assertion?

The answer is by using `type switches`. Type switches use `switch` blocks for data types and allow you to differentiate between type assertion values, which are data types, and process each data type the way you want. On the other hand, in order to use the empty interface in type switches, you need to use type assertions.

> You can have type switches for all kinds of interfaces and data types in general.

Therefore, the real work begins once you enter the function, because this is where you need to define the supported data types and the actions that take place for each supported data type.

Type assertions use the `x.(T)` notation, where x is an interface type and T is a type, and help you **_extract the value that is hidden behind the empty interface_**. For a type assertion to work, **_x should not be nil and the dynamic type of x should be identical to the T type_**.

```go
package main
import "fmt"
type Secret struct {
    SecretValue string
}
type Entry struct {
    F1 int
    F2 string
    F3 Secret
}
func Teststruct(x interface{}) {
    // type switch
    switch T := x.(type) {
    case Secret:
        fmt.Println("Secret type")
    case Entry:
        fmt.Println("Entry type")
    default:
        fmt.Printf("Not supported type: %T\n", T)
    }
}
func Learn(x interface{}) {
	switch T := x.(type) {
	default:
		fmt.Printf("Data type: %T\n", T)
	}
}
func main() {
	A := Entry{100, "F2", Secret{"myPassword"}}
	Teststruct(A)
	Teststruct(A.F3)
	Teststruct("A string")
	Learn(12.23)
	Learn('€')
}
```
The `Learn()` function prints the data type of its input parameter.

Strictly speaking, type assertions allow you to perform two main tasks:

* Checking whether an interface value keeps a particular type. When used this way, a type assertion returns two values: the `underlying value` and a `bool` value. The underlying value is what you might want to use. However, _**it is the value of the bool variable that tells you whether the type assertion was successful or not**_ and therefore whether you can use the underlying value or not. Checking whether a variable named `aVar` is of the `int` type requires the use of the `aVar.(int)` notation, which returns two values. If successful, it returns the real int value of aVar and true. Otherwise, it returns false as the second value, which means that the type assertion was not successful and that the real value could not be extracted. 
* Using the concrete value stored in an interface or assigning it to a new variable. This means that if there is a `float64` variable in an interface, a type assertion allows you to get that value.

> The functionality offered by the `reflect` package helps Go identify the underlying data type and the real value of an `interface{}` variable.

As explained, trying to extract the concrete value from an interface using a type assertion can have two outcomes:

* If you use the correct concrete data type, you get the underlying value without any issues
* If you use an incorrect concrete data type, your program will panic

```go
package main
import (
    "fmt"
)
func returnNumber() interface{} {
    return 12
}
func main() {
    anInt := returnNumber()
	number := anInt.(int)
    number++
    fmt.Println(number)
    // The next statement would fail because there
    // is no type assertion to get the value:
    // anInt++
    // The next statement fails but the failure is under 
    // control because of the ok bool variable that tells
    // whether the type assertion is successful or not
    value, ok := anInt.(int64)
    if ok {
        fmt.Println("Type assertion successful: ", value)
    } else {
        fmt.Println("Type assertion failed!")
    }
    // The next statement is successful but 
    // dangerous because it does not make sure that
    // the type assertion is successful.
    // It just happens to be successful
    i := anInt.(int)
    fmt.Println("i:", i)
    // The following will PANIC because anInt is not bool
    _ = anInt.(bool)
}
```

_The last statement panics the program because the anInt variable does not hold a bool value._

The reason for the panic is written onscreen: panic: interface conversion: interface {} is int, not bool.

### The `map[string]interface{}` map

Remember that the biggest advantage you get from using a `map[string]interface{}` map or any map that stores an `interface{}` value in general, is that you still have your data in its original state and data type. _If you use `map[string]string` instead, or anything similar, then any data you have is going to be converted into a string, which means that you are going to lose information about the original data type and the structure of the data you are storing in the map._

```go
package main
import (
    "encoding/json"
    "fmt"
    "os"
)
var JSONrecord = `{
    "Flag": true,
    "Array": ["a","b","c"],
    "Entity": {
      "a1": "b1",
      "a2": "b2",
      "Value": -456,
      "Null": null
    },
    "Message": "Hello Go!"
  }`

func typeSwitch(m map[string]interface{}) {
	for k, v := range m {
		switch c := v.(type) {
		case string:
			fmt.Println("Is a string!", k, c)
		case float64:
			fmt.Println("Is a float64!", k, c)
		case bool:
			fmt.Println("Is a Boolean!", k, c)
		case map[string]interface{}:
			fmt.Println("Is a map!", k, c)
			typeSwitch(v.(map[string]interface{}))
		default:
			fmt.Printf("...Is %v: %T!\n", k, c)
		}
	}
	return
}

func exploreMap(m map[string]interface{}) {
	for k, v := range m {
		embMap, ok := v.(map[string]interface{})
		// If it is a map, explore deeper
		if ok {
			fmt.Printf("{\"%v\": \n", k)
			exploreMap(embMap)
			fmt.Printf("}\n")
		} else {
			fmt.Printf("%v: %v\n", k, v)
		}
	}
}

func main() {
	if len(os.Args) == 1 {
		fmt.Println("*** Using default JSON record.")
	} else {
		JSONrecord = os.Args[1]
	}
	JSONMap := make(map[string]interface{})
	err := json.Unmarshal([]byte(JSONrecord), &JSONMap)
	if err != nil {
		fmt.Println(err)
		return
	}
	exploreMap(JSONMap)
	typeSwitch(JSONMap)
}
```

If a `map` is found, then we recursively call `typeSwitch()` on the new map in order to examine it even more.

The `exploreMap()` function inspects the contents of its input map. If a map is found, then we call `exploreMap()` on the new map recursively in order to examine it on its own.

`json.Unmarshal()` processes JSON data and converts it into a Go value. Although this value is usually a Go structure, in this case we are using a map as specified by the map[string]interface{} variable. Strictly speaking, the second parameter of json.Unmarshal() is of the empty interface data type, which means that its data type can be anything.

> `map[string]interface{}` is extremely handy for storing JSON records when you do not know their schema in advance. In other words, `map[string]interface{}` is good at storing arbitrary JSON data of unknown schema.

The error message in the third execution is generated by `json.Unmarshal()` as it cannot understand the schema of the JSON record.

### The error data type

```go
type error interface {
    Error() string
}
```

So, in order to satisfy the error interface you just need to implement the `Error() string` type method.

However, the crucial question is when you should implement the error interface on your own instead of using the default one. The answer to that question is when you want to give more context to an error condition.

When there is nothing more to read from a file, Go returns an `io.EOF` error, which, strictly speaking, is not an error condition but a logical part of reading a file. If a file is totally empty, you still get `io.EOF` when you try to read it. However, this might cause problems in some situations, and you might need to have a way of differentiating between a totally empty file and a file that has been read fully and there is nothing more to read. One way of dealing with that issue is with the help of the error interface.

```go
type emptyFile struct {
    Ended bool
    Read  int
}
// Implement error interface
func (e emptyFile) Error() string {
    return fmt.Sprintf("Ended with io.EOF (%t) but read (%d) bytes", e.Ended, e.Read)
}

// Check values
func isFileEmpty(e error) bool {
    // Type assertion
    v, ok := e.(emptyFile)
    if ok {
        if v.Read == 0 && v.Ended == true {
            return true
        }
    }
    return false
}

func readFile(file string) error {
    var err error
    fd, err := os.Open(file)
    if err != nil {
        return err
    }
    defer fd.Close()
    reader := bufio.NewReader(fd)
    n := 0
    for {
        line, err := reader.ReadString('\n')
        n += len(line)
        if err == io.EOF {
            // End of File: nothing more to read
            if n == 0 {
                return emptyFile{true, n}
            }

            break
        } else if err != nil {
            return err
        }
    }
    return nil
}

func main() {
    flag.Parse()
    if len(flag.Args()) == 0 {
        fmt.Println("usage: errorInt <file1> [<file2> ...]")
        return
    }
    for _, file := range flag.Args() {
        err := readFile(file)
        if isFileEmpty(err) {
            fmt.Println(file, err)
        } else if err != nil {
            fmt.Println(file, err)
        } else {
            fmt.Println(file, "is OK.")
        }
    }
}
```

This is a `type assertion` for getting an `emptyFile` structure from the error variable.

If you are dealing with multiple error variables, you should add a `type switch` to the `isFileEmpty()` function after the type assertion.

This kind of context is added to the emptyFile structure and returned as an error value.

_The order we do the checking in is important because only the first match is executed. This means that we have to go from more specific cases to more generic conditions._

### Writing your own interfaces

Creating your own interfaces is easy. For reasons of simplicity, we include our own interface in the main package. However, this is rarely the case as we usually want to share our interfaces, which means that interfaces are usually included in Go packages other than main.

```go
type Shape2D interface {
    Perimeter() float64
}
```

in order for a data type to satisfy the Shape2D interface, it needs to implement a type method named `Perimeter()` that returns a `float64` value.

The code that follows presents the simplest way of using an interface, which is by calling its method directly, as if it was a function, to get a result. Although this is allowed, _it is rarely the case as we usually create functions that accept interface parameters in order for these functions to be able to work with multiple data types_.

> The code uses a handy technique for quickly finding out whether a given variable is of a given data type that was presented earlier in assertions.go. In this case, we examine whether a variable is of the Shape2D interface by using the `interface{}(a).(Shape2D)` notation, where `a` is the variable that is being examined and Shape2D is the data type against the variable being checked.

```go
type circle struct {
    R float64
}
func (c circle) Perimeter() float64 {
    return 2 * math.Pi * c.R
}

func main() {
    a := circle{R: 1.5}
    fmt.Printf("R %.2f -> Perimeter %.3f \n", a.R, a.Perimeter())
    _, ok := interface{}(a).(Shape2D)
    if ok {
        fmt.Println("a is a Shape2D!")
    }
}
```

The `interface{}(a).(Shape2D)` notation checks whether the `a` variable satisfies the Shape2D interface without using its underlying value (circle{R: 1.5}).

The fact that Go considers interfaces as data types allows us to create slices with elements that satisfy a given interface without getting any error messages.

This kind of scenario can be useful in various cases because it illustrates how to store elements with different data types that all satisfy a common interface on the same slice and how to sort them using `sort.Interface`. Put simply, the presented utility sorts different structures with different numbers and names of fields that all share a common behavior through an interface implementation.

The `sort.Interface` interface is implemented for the shapes data type, which is defined as a slice of Shape3D elements.

As floating-point numbers have a lot of decimal points, printing is implemented using a separate function named `PrintShapes()` that uses an `fmt.Printf("%.2f ", v)` statement to specify the number of decimal points that are displayed onscreen

```go
package main
import (
    "fmt"
    "math"
    "math/rand"
    "sort"
    "time"
)
const min = 1
const max = 5
func rF64(min, max float64) float64 {
    return min + rand.Float64()*(max-min)
}

type Shape3D interface {
    Vol() float64
}

type Cube struct {
    x float64
}
type Cuboid struct {
    x float64
    y float64
    z float64
}
type Sphere struct {
    r float64
}
func (c Cube) Vol() float64 {
    return c.x * c.x * c.x
}

func (c Cuboid) Vol() float64 {
    return c.x * c.y * c.z
}

func (c Sphere) Vol() float64 {
    return 4 / 3 * math.Pi * c.r * c.r * c.r
}

type shapes []Shape3D

// Implementing sort.Interface
func (a shapes) Len() int {
	return len(a)
}
func (a shapes) Less(i, j int) bool {
	return a[i].Vol() < a[j].Vol()
}
func (a shapes) Swap(i, j int) {
	a[i], a[j] = a[j], a[i]
}

func PrintShapes(a shapes) {
	for _, v := range a {
		switch v.(type) {
		case Cube:
			fmt.Printf("Cube: volume %.2f\n", v.Vol())
		case Cuboid:
			fmt.Printf("Cuboid: volume %.2f\n", v.Vol())
		case Sphere:
			fmt.Printf("Sphere: volume %.2f\n", v.Vol())
		default:
			fmt.Println("Unknown data type!")
		}
	}
	fmt.Println()
}
func main() {
	data := shapes{}
	rand.Seed(time.Now().Unix())

	for i := 0; i < 3; i++ {
		cube := Cube{rF64(min, max)}
		cuboid := Cuboid{rF64(min, max), rF64(min, max), rF64(min, max)}
		sphere := Sphere{rF64(min, max)}
		data = append(data, cube)
		data = append(data, cuboid)
		data = append(data, sphere)
	}
	PrintShapes(data)
	// Sorting
	sort.Sort(shapes(data))
	PrintShapes(data)
	// Reverse sorting
	sort.Sort(sort.Reverse(shapes(data)))
	PrintShapes(data)
}
```

## Working with two different CSV file formats

Remember that the records of each CSV format are stored using their own Go structure under a different variable name. As a result, we need to implement `sort.Interface` for both CSV formats and therefore for both slice variables.

The two supported formats are the following:
* Format 1: name, surname, telephone number, time of last access
* Format 2: name, surname, area code, telephone number, time of last access
    
As the two CSV formats that are going to be used have a different number of fields, the utility determines the format that is being used by the number of fields found in the first record that was read and acts accordingly. After that, the data will be sorted using `sort.Sort()`—the data type of the slice that keeps the data helps Go determine the sort implementation that is going to be used without any help from the developer.
    
> The main benefit you get from functions that work with empty interface variables is that you can add support for additional data types easily at a later time without the need to implement additional functions and without breaking existing code.

```go
func readCSVFile(filepath string) error {
    var firstLine bool = true
    var format1 = true
    for _, line := range lines {
        if firstLine {
            if len(line) == 4 {
                format1 = true
            } else if len(line) == 5 {
                format1 = false
        
            } else {
                return errors.New("Unknown File Format!")
            }
            firstLine = false

			if format1 {
                if len(line) == 4 {
                    temp := F1{
                        Name:       line[0],
                        Surname:    line[1],
                        Tel:        line[2],
                        LastAccess: line[3],
                    }
                    d1 = append(d1, temp)
                }
            } else {
                if len(line) == 5 {
                    temp := F2{
                        Name:       line[0],
                        Surname:    line[1],
                        Areacode:   line[2],
                        Tel:        line[3],
                        LastAccess: line[4],
                    }
                    d2 = append(d2, temp)
                }
            }
        }
		
        return nil
    }
}
```

The first line of the CSV file determines its format—therefore, we need a flag variable for specifying whether we are dealing with the first line (firstLine) or not. Additionally, we need a second variable for specifying the format we are working with (format1 is that variable).

The `sortData()` function accepts an empty interface parameter. The code of the function determines the data type of the slice that is passed as an empty interface to that function using a `type switch`. After that, a type assertion allows you to use the actual data stored under the empty interface parameter. Its full implementation is as follows:

```go
func sortData(data interface{}) {
    // type switch
    switch T := data.(type) {
    case Book1:
        d := data.(Book1)
        sort.Sort(Book1(d))
        list(d)
    case Book2:
        d := data.(Book2)
        sort.Sort(Book2(d))
        list(d)
    default:
        fmt.Printf("Not supported type: %T\n", T)
    }
}
```

Lastly, `list()` prints the data of the data variable using the technique found in `sortData()`. Although the code that handles Book1 and Book2 is the same as in sortData(), you still need a type assertion to get the data from the empty interface variable.

```go
func list(d interface{}) {
    switch T := d.(type) {
    case Book1:
        data := d.(Book1)
        for _, v := range data {
            fmt.Println(v)
        }
    case Book2:
        data := d.(Book2)
        for _, v := range data {
            fmt.Println(v)
        }
    default:
        fmt.Printf("Not supported type: %T\n", T)
    }
}
```

## Object-oriented programming in Go

As Go does not support all object-oriented features, it cannot replace an object-oriented programming language fully. However, it can mimic some object-oriented concepts.

* First of all, a Go structure with its type methods is like an object with its methods. 
* Second, interfaces are like abstract data types that define behaviors and objects of the same class, which is similar to polymorphism. 
* Third, Go supports encapsulation, which means it supports hiding data and functions from the user by making them private to the structure and the current Go package. 
* Lastly, combining interfaces and structures is like composition in object-oriented terminology.

```go
package main
import (
    "fmt"
)
type IntA interface {
    foo()
}
type IntB interface {
    bar()
}
type IntC interface {
    IntA
    IntB
}
```

The IntC interface combines interfaces IntA and IntB. If you implement IntA and IntB for a data type, then this data type implicitly satisfies IntC.

```go
func processA(s IntA) {
    fmt.Printf("%T\n", s)
}
type a struct {
    XX int
    YY int
}
// Satisfying IntA
func (varC c) foo() {
    fmt.Println("Foo Processing", varC)
}
// Satisfying IntB
func (varC c) bar() {
    fmt.Println("Bar Processing", varC)
}
type b struct {
    AA string
    XX int
}
// Structure c has two fields
type c struct {
    A a
    B b
}
// Structure compose gets the fields of structure a
type compose struct {
    field1 int
    a
}
// Different structures can have methods with the same name
func (A a) A() {
    fmt.Println("Function A() for A")
}
func (B b) A() {
    fmt.Println("Function A() for B")
}
func main() {
    var iC c = c{a{120, 12}, b{"-12", -12}}
    
    iC.A.A()
    iC.B.A()
    // The following will not work
    // iComp := compose{field1: 123, a{456, 789}}
    // iComp := compose{field1: 123, XX: 456, YY: 789}
    iComp := compose{123, a{456, 789}}
    fmt.Println(iComp.XX, iComp.YY, iComp.field1)
	
    iC.bar()
    processA(iC)
}
```

`IntC` is a composition of the `IntA` and `IntB` interfaces.

This new structure uses an anonymous structure (a), which means that it gets the fields of that anonymous structure.

Here we access a method of the `a` structure (A.A()) and a method of the `b` structure (B.A()).

When using an anonymous structure inside another structure, as we do with a{456, 789}, you can access the fields of the anonymous structure, which is the a{456, 789} structure, directly as `iComp.XX` and `iComp.YY`.

Although `processA()` works with `IntA` variables, it can also work with `IntC` variables because the `IntC` interface satisfies `IntA`!

## Updating the phone book application

More advanced Go packages such as `viper`, which is presented in Chapter 6, Telling a UNIX System What to Do, simplify the process of parsing command-line arguments with the use of command-line options such as `-f` followed by a file path or `--filepath`

Generally speaking, not having to recompile your software for user-defined data is considered a good practice.

Here is where we read the PHONEBOOK environment variable.

```go
func setCSVFILE() error {
    filepath := os.Getenv("PHONEBOOK")
    if filepath != "" {
        CSVFILE = filepath
    }
	
	_, err := os.Stat(CSVFILE)
    if err != nil {
        fmt.Println("Creating", CSVFILE)
        f, err := os.Create(CSVFILE)
        if err != nil {
            f.Close()
            return err
        }
        f.Close()
    }
	
    fileInfo, err := os.Stat(CSVFILE)
    mode := fileInfo.Mode()
    if !mode.IsRegular() {
        return fmt.Errorf("%s not a regular file", CSVFILE)
    }

	return nil
}
```

The first thing to decide when trying to sort data is the field that is going to be used for sorting. After that, we need to decide what we are going to do when two or more records have the same value in the main field used for sorting.

The code related to sorting using `sort.Interface` is the following:

You need to have a separate data type—sort.Interface is implemented for this data type.

```go
type PhoneBook []Entry
```
```go
var data = PhoneBook{}
```

As you have a separate data type for implementing `sort.Interface`, the data type of the data variable needs to change and become PhoneBook. Then `sort.Interface` is implemented for PhoneBook.

```go
// Implement sort.Interface
func (a PhoneBook) Len() int {
	return len(a)
}
// First based on surname. If they have the same
// surname take into account the name.
func (a PhoneBook) Less(i, j int) bool {
    if a[i].Surname == a[j].Surname {
        return a[i].Name < a[j].Name
    }
    return a[i].Surname < a[j].Surname
}

func (a PhoneBook) Swap(i, j int) {
    a[i], a[j] = a[j], a[i]
}
```

What we say here is that if the entries that are compared, which are Go structures, have the same Surname field value, then compare these entries using their Name field values.

The `Swap()` function has a standard implementation. After implementing the desired interface, we need to tell our code to sort the data, which happens in the implementation of the `list()` function:

```go
func list() {
    sort.Sort(PhoneBook(data))
    for _, v := range data {
        fmt.Println(v)
    }
}
```

## Exercises

* Create a slice of structures using a structure that you created and sort the elements of the slice using a field from the structure
* Integrate the functionality of sortCSV.go in phonebook.go
* Add support for a reverse command to phonebook.go in order to list its entries in reverse order
* Use the empty interface and a function that allows you to differentiate between two different structures that you create

## Additional resources
* The documentation of the reflect package: [https://golang.org/pkg/reflect/](https://golang.org/pkg/reflect/)
* The documentation of the sort package: [https://golang.org/pkg/sort/](https://golang.org/pkg/sort/)
* Working with errors in Go 1.13: [https://blog.golang.org/go1.13-errors](https://blog.golang.org/go1.13-errors)
* The implementation of the sort package: [https://golang.org/src/sort/](https://golang.org/src/sort/)