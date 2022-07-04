# Chapter 3: Composite Data Types

* Go offers support for `maps` and `structures`, which are _composite data types_. The general idea is that if an `array` or a `slice` cannot do the job, you might need to look at `maps`. If a `map` cannot help you, then you should consider creating and using a `structure`.
* Maps can use keys of different data types whereas structures can group multiple data types and create new ones.

## Maps

* Both arrays and slices limit you to using positive integers as indexes. Maps are powerful data structures because they allow you to use indexes of various data types as keys to look up your data as long as these keys are **_comparable_**. A practical rule of thumb is that you should use a map when you are going to need indexes that _are not positive integer numbers or when the integer indexes have big gaps_.

> Although `bool` variables are _comparable_, it makes no sense to use a bool variable as the key to a Go map because it only allows for two distinct values. Additionally, although floating point values are comparable, precision issues caused by the internal representation of such values might create bugs and crashes, _so you might want to avoid using floating point values as keys to Go maps_.

* Although this is not always the case, working with maps in Go is fast, as you can access all elements of a map in _linear time_. Inserting and retrieving elements from a map is fast and does not depend on the **_cardinality_** of the map. 
* You can create a new map variable using either `make()` or a `map` literal. Creating a new map with string keys and int values using `make()` is as simple as writing `make(map[string]int)` and assigning its return value to a variable.

```go
m := map[string]int {
    "key1": -1
    "key2": 123
}
```

* The map literal version is faster when you want to add data to a map at the time of creation.
* You should make **_no assumptions_** about the order of the elements inside a map. _Go randomizes keys when iterating over a map_—this is done on purpose and is an intentional part of the language design.
* You can find the length of a map, which is the number of keys in the map, using the `len()` function, which also works with arrays and slices; and you can delete a key and value pair from a map using the `delete()` function, which accepts two arguments: the name of the map and the name of the key, in that order. 
* Additionally, you can tell whether a key k exists on a map named aMap by the _second return value_ of the `v, ok := aMap[k]` statement. If ok is set to `true`, then k exists, and its value is v. If it does not exist, v will be set to the zero value of its data type, which depends on the definition of the map. If you try to get the value of a key that does not exist in a map, _Go will not complain about it_ and returns the zero value of the data type of the value.

### Storing to a nil map

* You are allowed to assign a map variable to `nil`. In that case, you will not be able to use that variable until you assign it to a new map variable. Put simply, _if you try to store data on a nil map, your program will crash_. 
* At this point aMap points to nil, which is a synonym for nothing.

```go
func main() {
    aMap := map[string]int{}
    aMap["test"] = 1
	fmt.Println("aMap:", aMap)
	aMap = nil
	fmt.Println("aMap:", aMap)
	if aMap == nil {
		fmt.Println("nil map!")
		aMap = map[string]int{}
	}
	aMap["test"] = 1
	// This will crash!
	aMap = nil
	aMap["test"] = 1
}
```

* Testing whether a map points to `nil` before using it is a good practice.

> In real-world applications, if a function accepts a map argument, then it should check that the map is not `nil` before working with it.

### Iterating over maps

* When `for` is combined with the `range` keyword it implements the functionality of `foreach` loops found in other programming languages and allows you to iterate over all the elements of a map without knowing its size or its keys. When `range` is applied on a map, it returns key and value pairs in that order.

```go
package main

import "fmt"

func main() {
	aMap := make(map[string]string)
	aMap["123"] = "456"
	aMap["key"] = "A value"
	// range works with maps as well
	for key, v := range aMap {
		fmt.Println("key:", key, "value:", v)
	}
	for _, v := range aMap {
		fmt.Print(" # ", v)
	}
	fmt.Println()
}

```
> As you already know, you should make no assumptions about the order that the key and value pairs of a map will be returned in from a for and range loop.

## Structures

* Structures are the more versatile data types in Go, and they can even be associated with functions, which are called `methods`.

> Structures, as well as other user-defined data types, are usually defined **_outside_** the `main()` function or any other package function so that they have a global scope and are available to the entire Go package. Therefore, unless you want to make clear that a type is only useful within the current local scope and is not expected to be used elsewhere, you should write the definitions of new data types outside functions.

### Defining new structures

* When you define a new structure, you group a set of values into a single data type, which allows you to pass and receive this set of values as a single entity. A structure has fields, and each field has its own data type, which can even be another structure or slice of structures. Additionally, as a structure is a new data type, it is defined using the `type` keyword followed by the name of the structure and ending with the `struct` keyword, which signifies that we are defining a new structure.

```go
type Entry struct {
    Name    string
    Surname string
    Year    int
}
```

> The `type` keyword allows you to define new data types or create aliases for existing ones. Therefore, you are allowed to say `type myInt int` and define a new data type called myInt that is an alias for int. However, Go considers myInt and int as totally different data types that you cannot _compare directly_ even though they hold the same kind of values. Each structure defines a new data type, hence the use of the `type` keyword.

* the fields of a structure usually begin with an uppercase letter—this depends on what you want to do with the fields. 
* These three fields can be accessed with the dot notation as `V.Name`, `V.Surname`, and `V.Year`, where `V` is the name of the variable holding the instance of the Entry structure. A structure literal named p1 can be defined as `p1 := aStructure{"fmt", 12, -2}`. 
* There exist two ways to work with structure variables. The first one is as regular variables and the second one is as pointer variables that point to the memory address of a structure. Both ways are equally good and are usually embedded into separate functions because they allow you to initialize some or all of the fields of structure variables properly and/or do any other tasks you want before using the structure variable. As a result, there exist two main ways to create a new structure variable using a function. The first one returns a regular structure variable whereas the second one returns a pointer to a structure. Each one of these two ways has two variations. The first variation returns a structure instance that is initialized by the Go compiler, whereas the second variation returns a structure instance that is initialized by the user. 
* The order in which you put the fields in the definition of a structure type is **_significant_** for the type identity of the defined structure. Put simply, **_two structures with the same fields will not be considered identical in Go if their fields are not in the same order_**.

### Using the new keyword

* Additionally, you can create new structure instances using the `new()` keyword: `pS := new(Entry)`. The new() keyword has the following properties:

  * It allocates the proper memory space, which depends on the data type, and then it zeroes it 
  * It always returns a `pointer` to the allocated memory 
  * It works for all data types except `channel` and `map`

```go
package main

import "fmt"

type Entry struct {
	Name    string
	Surname string
	Year    int
}

// Initialized by Go
func zeroS() Entry {
	return Entry{}
}

// Initialized by the user
func initS(N, S string, Y int) Entry {
	if Y < 2000 {
		return Entry{Name: N, Surname: S, Year: 2000}
	}
	return Entry{Name: N, Surname: S, Year: Y}
}

// Initialized by Go - returns pointer
func zeroPtoS() *Entry {
	t := &Entry{}
	return t
}

// Initialized by the user - returns pointer
func initPtoS(N, S string, Y int) *Entry {
	if len(S) == 0 {
		return &Entry{Name: N, Surname: "Unknown", Year: Y}
	}
	return &Entry{Name: N, Surname: S, Year: Y}
}

func main() {
	s1 := zeroS()
	p1 := zeroPtoS()
	fmt.Println("s1:", s1, "p1:", *p1)
	s2 := initS("Mihalis", "Tsoukalos", 2020)
	p2 := initPtoS("Mihalis", "Tsoukalos", 2020)
	fmt.Println("s2:", s2, "p2:", *p2)
	fmt.Println("Year:", s1.Year, s2.Year, p1.Year, p2.Year)
	pS := new(Entry)
	fmt.Println("pS:", pS)
}

```

* Now is a good time to remind you of an important Go rule: If no initial value is given to a variable, the Go compiler automatically initializes that variable to the zero value of its data type. For structures, this means that a structure variable without an initial value is initialized to the zero values of each one of the data types of its fields. 
* The `new(Entry)` call returns a `pointer` to an Entry structure. Generally speaking, when you have to initialize lots of structure variables, it is considered a good practice to create a function for doing so as this is less error-prone.

### Slices of structures
```go
package main

import (
	"fmt"
	"strconv"
)

type record struct {
	Field1 int
	Field2 string
}

func main() {
	S := []record{}
	for i := 0; i < 10; i++ {
		text := "text" + strconv.Itoa(i)
		temp := record{Field1: i, Field2: text}
		S = append(S, temp)
	}
	// Accessing the fields of the first element
	fmt.Println("Index 0:", S[0].Field1, S[0].Field2)
	fmt.Println("Number of structures:", len(S))
	sum := 0
	for _, k := range S {
		sum += k.Field1
	}
	fmt.Println("Sum:", sum)
}

```
## Regular expressions and pattern matching

* A regular expression is a sequence of characters that defines a search pattern. Every regular expression is **_compiled_** into a recognizer by building a generalized transition diagram called a finite automaton. A finite automaton can be either deterministic or nondeterministic. Nondeterministic means that more than one transition out of a state can be possible for the same input. A **_recognizer_** is a program that takes a string x as input and is able to tell whether x is a sentence of a given language. 
* A grammar is a set of production rules for strings in a formal language—the production rules describe how to create strings from the alphabet of the language that are valid according to the syntax of the language. A grammar does not describe the meaning of a string or what can be done with it in whatever context—it only describes its form. What is important here is to realize that grammars are at the heart of regular expressions because without a grammar, you cannot define or use a regular expression.

### About Go regular expressions
| Expression | Description                                           | 
|------------|-------------------------------------------------------|
| .          | Matches any character                                 | 
| *          | Means any number of times—cannot be used on its own   |
| ?          | Zero or one time—cannot be used on its own            |
| +          | Means one or more times—cannot be used on its own     |
| ^          | This denotes the beginning of the line                |
| $          | This denotes the end of the line                      |
| []         | [] is for grouping characters                         |
| [A-Z]      | This means all characters from capital A to capital Z |
| \d         | Any digit in 0-9                                      |
| \D         | A non-digit                                           |
| \w         | Any word character: [0-9A-Za-z_]                      |
| \W         | Any non-word character                                |
| \s         | A whitespace character                                |
| \S         | A non-whitespace character                            |

* The Go package responsible for defining regular expressions and performing pattern matching is called `regexp`. We use the `regexp.MustCompile()` function to create the regular expression and the `Match()` function to see whether the given string is a match or not. 
* The `regexp.MustCompile()` function parses the given regular expression and returns a `regexp.Regexp` variable that can be used for matching—`regexp.Regexp` is the representation of a compiled regular expression. The function panics if the expression cannot be parsed, which is good because you will know that your expression is invalid early in the process. The `re.Match()` method returns `true` if the given byte slice matches the re regular expression, which is a `regexp.Regexp` variable, and `false` otherwise.

> Creating separate functions for pattern matching can be handy because it allows you to reuse the functions without worrying about the context of the program.

* Keep in mind that although regular expressions and pattern matching look convenient and handy at first, they are the root of lots of bugs. My advice is to use the simplest regular expression that can solve your problem. However, if you can avoid using regular expressions at all, it would be much better in the long run!

### Matching names and surnames

```go
func matchNameSur(s string) bool {
    t := []byte(s)
    re := regexp.MustCompile(`^[A-Z][a-z]*$`)
    return re.Match(t)
}
```

### Matching integers

```go
func matchInt(s string) bool {
    t := []byte(s)
    re := regexp.MustCompile(`^[-+]?\d+$`)
    return re.Match(t)
}
```

### Matching the fields of a record

```go
func matchRecord(s string) bool {
    fields := strings.Split(s, ",")
    if len(fields) != 3 {
        return false
    }
    if !matchNameSur(fields[0]) {
        return false
    }
	if !matchNameSur(fields[1]) {
		return false
	}
	return matchTel(fields[2])
}
```

### Working with CSV files

* Go provides a dedicated package for working with CSV data named `encoding/csv` [https://golang.org/pkg/encoding/csv/](https://golang.org/pkg/encoding/csv/).

> There exist two very popular Go interfaces named `io.Reader` and `io.Write` that are to do with reading from files and writing to the files. Almost all reading and writing operations in Go use these two interfaces. The use of the same interface for readers allows readers to share some common characteristics but most importantly allows you to create your own readers and use them anywhere that Go expects an `io.Reader` reader. The same applies to writers that satisfy the `io.Write` interface.

* The `encoding/csv` package contains functions that can help you read and write CSV files. As we are dealing with small CSV files, we use `csv.NewReader(f).ReadAll()` to read the entire input file all at once. For bigger data files or if we wanted to check the input or make any changes to the input as we read it, it would have been better to read it line by line using `Read()` instead of `ReadAll()`. 
* Go assumes that the CSV file uses the comma character (,) for separating the different fields of each line. Should we wish to change that behavior, we should change the value of the Comma variable of the CSV reader or the writer depending on the task we want to perform. We change that behavior in the output CSV file, which separates its fields using the `tab` character.

> it is better if the input and output CSV files are using the same field delimiter.

```go
package main

import (
	"encoding/csv"
	"fmt"
	"os"
)

type Record struct {
	Name       string
	Surname    string
	Number     string
	LastAccess string
}

var myData = []Record{}

func readCSVFile(filepath string) ([][]string, error) {
	_, err := os.Stat(filepath)
	if err != nil {
		return nil, err
	}
	f, err := os.Open(filepath)
	if err != nil {
		return nil, err
	}
	defer f.Close()
	// CSV file read all at once
	// lines data type is [][]string
	lines, err := csv.NewReader(f).ReadAll()
	if err != nil {
		return [][]string{}, err
	}
	return lines, nil
}

func saveCSVFile(filepath string) error {
	csvfile, err := os.Create(filepath)
	if err != nil {
		return err
	}
	defer csvfile.Close()
	csvwriter := csv.NewWriter(csvfile)
	// Changing the default field delimiter to tab
	csvwriter.Comma = '\t'
	for _, row := range myData {
		temp := []string{row.Name, row.Surname, row.Number, row.LastAccess}
		_ = csvwriter.Write(temp)
	}
	csvwriter.Flush()
	return nil
}

func main() {
	if len(os.Args) != 3 {
		fmt.Println("csvData input output!")
		return
	}
	input := os.Args[1]
	output := os.Args[2]
	lines, err := readCSVFile(input)
	if err != nil {
		fmt.Println(err)
		return

	}
	// CSV data is read in columns - each line is a slice
	for _, line := range lines {
		temp := Record{
			Name:       line[0],
			Surname:    line[1],
			Number:     line[2],
			LastAccess: line[3],
		}
		myData = append(myData, temp)
		fmt.Println(temp)
	}
	err = saveCSVFile(output)
	if err != nil {
		fmt.Println(err)
		return
	}
}

```

* Have in mind that `csv.NewReader()` does separate the fields of each input line, which is the main reason for needing a slice with _**two**_ dimensions to store the input.

### Adding an index

* As a rule of thumb, you index a field that is going to be used for searching. There is no point in creating an index that is not going to be used for querying.

### The improved version of the phone book application

```go
package main

import (
	"encoding/csv"
	"fmt"
	"os"
	"strconv"
	"strings"
	"time"
)

type Record struct {
	Name       string
	Surname    string
	Number     string
	LastAccess string
}

var myData = []Record{}

func readCSVFile(filepath string) ([][]string, error) {
	_, err := os.Stat(filepath)
	if err != nil {
		return nil, err
	}
	f, err := os.Open(filepath)
	if err != nil {
		return nil, err
	}
	defer f.Close()
	// CSV file read all at once
	// lines data type is [][]string
	lines, err := csv.NewReader(f).ReadAll()
	if err != nil {
		return [][]string{}, err
	}
	return lines, nil
}

func saveCSVFile(filepath string) error {
	csvfile, err := os.Create(filepath)
	if err != nil {
		return err
	}
	defer csvfile.Close()
	csvwriter := csv.NewWriter(csvfile)
	// Changing the default field delimiter to tab
	csvwriter.Comma = '\t'
	for _, row := range myData {
		temp := []string{row.Name, row.Surname, row.Number, row.LastAccess}
		_ = csvwriter.Write(temp)
	}
	csvwriter.Flush()
	return nil
}

func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		fmt.Println("Usage: insert|delete|search|list <arguments>")
		return
	}
	// If the CSVFILE does not exist, create an empty one
	_, err := os.Stat(CSVFILE)
	// If error is not nil, it means that the file does not exist
	if err != nil {
		fmt.Println("Creating", CSVFILE)
		f, err := os.Create(CSVFILE)
		if err != nil {
			f.Close()
			fmt.Println(err)
			return
		}
		f.Close()
	}
	fileInfo, err := os.Stat(CSVFILE)
	// Is it a regular file?
	mode := fileInfo.Mode()
	if !mode.IsRegular() {
		fmt.Println(CSVFILE, "not a regular file!")
		return
	}
	err = readCSVFile(CSVFILE)
	if err != nil {
		fmt.Println(err)
		return
	}
	err = createIndex()
	if err != nil {
		fmt.Println("Cannot create index.")
		return
	}

	// Differentiating between the commands
	switch arguments[1] {
	case "insert":
		if len(arguments) != 5 {
			fmt.Println("Usage: insert Name Surname Telephone")
			return
		}
		t := strings.ReplaceAll(arguments[4], "-", "")
		if !matchTel(t) {
			fmt.Println("Not a valid telephone number:", t)
			return
		}

		temp := initS(arguments[2], arguments[3], t)
		// If it was nil, there was an error
		if temp != nil {
			err := insert(temp)

			if err != nil {
				fmt.Println(err)
				return
			}
		}
	case "delete":
		if len(arguments) != 3 {
			fmt.Println("Usage: delete Number")
			return
		}
		t := strings.ReplaceAll(arguments[2], "-", "")
		if !matchTel(t) {
			fmt.Println("Not a valid telephone number:", t)
			return
		}
		err := deleteEntry(t)
		if err != nil {
			fmt.Println(err)
		}
	case "search":
		if len(arguments) != 3 {
			fmt.Println("Usage: search Number")
			return
		}
		t := strings.ReplaceAll(arguments[2], "-", "")
		if !matchTel(t) {
			fmt.Println("Not a valid telephone number:", t)
			return
		}
		temp := search(t)
		if temp == nil {
			fmt.Println("Number not found:", t)
			return
		}
		fmt.Println(*temp)
	case "list":
		list()
	default:
		fmt.Println("Not a valid option")
	}
}
func createIndex() error {
	index = make(map[string]int)
	for i, k := range data {
		key := k.Tel
		index[key] = i
	}
	return nil
}

func deleteEntry(key string) error {
	i, ok := index[key]
	if !ok {
		return fmt.Errorf("%s cannot be found!", key)
	}
	data = append(data[:i], data[i+1:]...)
	// Update the index - key does not exist any more
	delete(index, key)
	err := saveCSVFile(CSVFILE)
	if err != nil {
		return err
	}
	return nil
}
func insert(pS *Entry) error {
	// If it already exists, do not add it
	_, ok := index[(*pS).Tel]
	if ok {
		return fmt.Errorf("%s already exists", pS.Tel)
	}
	data = append(data, *pS)
	// Update the index
	_ = createIndex()
	err := saveCSVFile(CSVFILE)
	if err != nil {
		return err
	}
	return nil
}

func search(key string) *Entry {
	i, ok := index[key]
	if !ok {
		return nil
	}
	data[i].LastAccess = strconv.FormatInt(time.Now().Unix(), 10)
	return &data[i]
}

```

* We have to create it for the rest of the program to use it. This is determined by the return value of the `os.Stat(CSVFILE)` call.

* The code returns `nil`, which is possible because the function returns a pointer to an Entry variable.

## Exercises

* Write a Go program that converts an existing array into a map.
* Write a Go program that converts an existing map into two slices—the first slice contains the keys of the map whereas the second slice contains the values. The values at index n of the two slices should correspond to a key and value pair that can be found in the original map.
* Make the necessary changes to nameSurRE.go to be able to process multiple command-line arguments.
* Change the code of intRE.go to process multiple command-line arguments and display totals of true and false results at the end.
* Make changes to csvData.go to separate the fields of a record based on the # character.
* Write a Go utility that converts os.Args into a slice of structures with fields for storing the index and the value of each command-line argument—you should define the structure that is
* going to be used on your own.
* Make the necessary changes to phoneBook.go in order to create the index based on the LastAccess field. Is this practical? Does it work? Why?
* Make changes to csvData.go in order to separate the fields of a record with a character that is given as a command-line argument.

## Additional resources

* The encoding/csv documentation: [https://golang.org/pkg/encoding/csv/](https://golang.org/pkg/encoding/csv/)
* The runtime package documentation: [https://golang.org/pkg/runtime/](https://golang.org/pkg/runtime/)
