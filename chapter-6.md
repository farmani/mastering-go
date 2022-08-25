# Telling a UNIX System What to Do
Go. Systems programming involves working with files and directories, process control, signal handling, network programming, system files, configuration files, and file input and output (I/O)
often Go software is executed in a Docker environment—Docker images use the Linux operating system, which means that you might need to develop your utilities with the Linux operating system in mind.

### stdin, stdout, and stderr
Every UNIX operating system has three files open all the time for its processes. Remember that UNIX considers everything, even a printer or your mouse, as a file. UNIX uses file descriptors, which are positive integer values, as an internal representation for accessing open files, which is much prettier than using long paths. So, by default, all UNIX systems support three special and standard filenames: /dev/stdin, /dev/stdout, and /dev/stderr, which can also be accessed using file descriptors 0, 1, and 2, respectively. These three file descriptors are also called standard input, standard output, and standard error, respectively. Additionally, file descriptor 0 can be accessed as /dev/fd/0 on a macOS machine and as both /dev/fd/0 and /dev/pts/0 on a Debian Linux machine.
Go uses os.Stdin for accessing standard input, os.Stdout for accessing standard output, and os.Stderr for accessing standard error. Although you can still use /dev/stdin, /dev/stdout, and /dev/stderr or the related file descriptor values for accessing the same devices, it is better, safer, and more portable to stick with os.Stdin, os.Stdout, and os.Stderr.


### UNIX processes
a process is an execution environment that contains instructions, user data and system data parts, and other types of resources that are obtained during runtime. On the other hand, a program is a binary file that contains instructions and data that are used for initializing the instruction and user data parts of a process. Each running UNIX process is uniquely identified by an unsigned integer, which is called the process ID of the process.
There are three process categories: user processes, daemon processes, and kernel processes. User processes run in user space and usually have no special access rights. Daemon processes are programs that can be found in the user space and run in the background without the need for a terminal. Kernel processes are executed in kernel space only and can fully access all kernel data structures.
The C way of creating new processes involves the calling of the fork(2) system call. The return value of fork(2) allows the programmer to differentiate between a parent and a child process. Although you can fork a new process in Go using the exec package, Go does not allow you to control threads—Go offers goroutines, which the user can create on top of threads that are created and handled by the Go runtime.

### Handling UNIX signals
UNIX signals offer a very handy way of interacting asynchronously with your applications. However, UNIX signal handling requires the use of Go channels that are used exclusively for this task. 

A goroutine is the smallest executable Go entity. In order to create a new goroutine you have to use the go keyword followed by a predefined function or an anonymous function—the methods are equivalent. A channel in Go is a mechanism that among other things allows goroutines to communicate and exchange data.

> In order for a goroutine or a function to terminate the entire Go application, it should call os.Exit() instead of return. However, most of the time, you should exit a goroutine or a function using return because you just want to exit that specific goroutine or function and not stop the entire application.

There exists a dedicated channel that receives all signals, as defined by the signal.Notify() function. Go channels can have a capacity—the capacity of this particular channel is 1 in order to be able to receive and keep one signal at a time. This makes perfect sense as a signal can terminate a program and there is no need to try to handle another signal at the same time. There is usually an anonymous function that is executed as a goroutine and performs the signal handling and nothing else. The main task of that goroutine is to listen to the channel for data. Once a signal is received, it is sent to that channel, read by the goroutine, and stored into a variable—at this point the channel can receive more signals. That variable is processed by a switch statement.

> Some signals cannot be caught, and the operating system cannot ignore them. So, the SIGKILL and SIGSTOP signals cannot be blocked, caught, or ignored and the reason for this is that they allow privileged users as well as the UNIX kernel to terminate any process they desire.

```go
package main
import (
    "fmt"
    "os"
    "os/signal"
    "syscall"
    "time"
)
func handleSignal(sig os.Signal) {
    fmt.Println("handleSignal() Caught:", sig)
}
func main() {
	fmt.Printf("Process ID: %d\n", os.Getpid())
	// We create a channel with data of type os.Signal because all channels must have a type.
	sigs := make(chan os.Signal, 1)
	// The below statement means handle all signals that can be handled.
	signal.Notify(sigs)
	start := time.Now()
	go func() {
		for {
			// Wait until you read data (<-) from the sigs channel and store it in the sig variable.
			sig := <-sigs
			switch sig {
				case syscall.SIGINT:
                    duration := time.Since(start)
                    fmt.Println("Execution time:", duration)
				case syscall.SIGINFO:
				    handleSignal(sig)
					// do not use return here because the goroutine exits
					// but the time.Sleep() will continue to work!
					os.Exit(0)
                default:
				    fmt.Println("Caught:", sig)
			}
			}
	}()
	for {
		fmt.Print("+")
		time.Sleep(10 * time.Second)
	}
}
```
On Linux machines, you should replace syscall.SIGINFO with another signal such as syscall.SIGUSR1 or syscall.SIGUSR2 because syscall.SIGINFO is not available on Linux (https://github.com/golang/go/issues/1653).

The endless for loop at the end of the main() function is for emulating the operation of a real program. Without an endless for loop, the program exits almost immediately.

The second line of output was generated by pressing Ctrl + C on the keyboard, which on UNIX machines sends the syscall.SIGINT signal to the program. The third line of output was caused by executing kill -USR1 74252 on a different terminal. The last line in the output was generated by the kill -9 74252 command. As the KILL signal, which is also represented by the number 9, cannot be handled, it terminates the program, and the shell prints the killed message.

### Handling two signals
If you want to handle a limited number of signals, instead of all of them, you should replace the signal.Notify(sigs) statement with the next statement:

```go
signal.Notify(sigs, syscall.SIGINT, syscall.SIGINFO)
```
## File I/O
> The io/ioutil package (https://golang.org/pkg/io/ioutil/) is deprecated in Go version 1.16. Existing Go code that uses the functionality of io/ioutil will continue to work but it is better to stop using that package.

### The io.Reader and io.Writer interfaces

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```
This definition, which should be revisited when we want one of our data types to satisfy the io.Reader interface, tells us the following:

- The Reader interface requires the implementation of a single method
- The parameter of Read() is a byte slice
- The return values of Read() are an integer and an error
    
The Read() method takes a byte slice as input, which is going to be filled with data up to its length, and returns the number of bytes read as well as an error variable.

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

The previous definition, which should be revisited when we want one of our data types to satisfy the io.Writer interface and to write to a file, reveals the following information:

- The interface requires the implementation of a single method
- The parameter of Write() is a byte slice
- The return values of Write() are an integer and an error value
    
The Write() method takes a byte slice, which contains the data that you want to write, as input and returns the number of bytes written and an error variable.

### Using and misusing io.Reader and io.Writer

```go
package main
import (
    "bufio"
    "fmt"
    "io"
)

type S1 struct {
    F1 int
    F2 string
}
type S2 struct {
    F1   S1
    text []byte
}

// Using pointer to S1 for changes to be persistent when the method exits
func (s *S1) Read(p []byte) (n int, err error) {
    fmt.Print("Give me your name: ")
    fmt.Scanln(&p)
    s.F2 = string(p)
    return len(p), nil
}

func (s *S1) Write(p []byte) (n int, err error) {
    if s.F1 < 0 {
        return -1, nil
    }
    for i := 0; i < s.F1; i++ {
        fmt.Printf("%s ", p)
    }
    fmt.Println()
    return s.F1, nil
}

func (s S2) eof() bool {
    return len(s.text) == 0
}
func (s *S2) readByte() byte {
    // this function assumes that eof() check was done before
    temp := s.text[0]
    s.text = s.text[1:]
    return temp
}

func (s *S2) Read(p []byte) (n int, err error) {
    if s.eof() {
        err = io.EOF
        return
    }
    l := len(p)
    if l > 0 {
        for n < l {
            p[n] = s.readByte()
            n++
            if s.eof() {
                s.text = s.text[0:0]
                break
            }
        }
    }
    return
}

func main() {
    s1var := S1{4, "Hello"}
    fmt.Println(s1var)
    
    buf := make([]byte, 2)
    _, err := s1var.Read(buf)
    
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("Read:", s1var.F2)
    _, _ = s1var.Write([]byte("Hello There!"))
    
    s2var := S2{F1: s1var, text: []byte("Hello world!!")}
    // Read s2var.text
    r := bufio.NewReader(&s2var)
    
    for {
        n, err := r.Read(buf)
        if err == io.EOF {
            break
        
        } else if err != nil {
            fmt.Println("*", err)
            break
        }
        fmt.Println("**", n, string(buf[:n]))
    }
}
```
Recall that Go allows you to name the return values of a function—in that case, a return statement without any additional arguments automatically returns the current value of each named return variable in the order they appear in the function signature. The Read() method uses that feature.

### Buffered and unbuffered file I/O
Buffered file I/O happens when there is a buffer for temporarily storing data before reading data or writing data. Thus, instead of reading a file byte by byte, you read many bytes at once. You put the data in a buffer and wait for someone to read it in the desired way.
Unbuffered file I/O happens when there is no buffer to temporarily store data before actually reading or writing it—this can affect the performance of your programs.
The next question that you might ask is how to decide when to use buffered and when to use unbuffered file I/O. When dealing with critical data, unbuffered file I/O is generally a better choice because buffered reads might result in out-of-date data and buffered writes might result in data loss when the power of your computer is interrupted.

However, keep in mind that buffered readers can also improve performance by reducing the number of system calls needed to read from a file or socket, so there can be a real performance impact on what the programmer decides to use.

Excerpt From
Mastering Go
Mihalis Tsoukalos
This material may be protected by copyright.
## Reading text files
### Reading a text file line by line
First, you create a new reader to the desired file using a call to bufio.NewReader(). Then, you use that reader with bufio.ReadString() in order to read the input file line by line. The trick is done by the parameter of bufio.ReadString(), which is a character that tells bufio.ReadString() to keep reading until that character is found. Constantly calling bufio.ReadString() when that parameter is the newline character (\n) results in reading the input file line by line.

```go
func lineByLine(file string) error {
    f, err := os.Open(file)
    if err != nil {
        return err
    }
    defer f.Close()
    r := bufio.NewReader(f)
    for {
        line, err := r.ReadString('\n')
        if err == io.EOF {
            break
        } else if err != nil {
            fmt.Printf("error reading file %s", err)
            break
        }
        fmt.Print(line)
    }
    return nil
}
```
bufio.ReadString() returns two values: the string that was read and an error variable.
The use of fmt.Print() instead of fmt.Println() for printing the input line shows that the newline character is included in each input line.

### Reading a text file word by word
regexp.MustCompile("[^\\s]+") statement states that we use whitespace characters to separate one word from another.

```go
func wordByWord(file string) error {
    f, err := os.Open(file)
    if err != nil {
        return err
    }
    defer f.Close()
    r := bufio.NewReader(f)
    for {
        line, err := r.ReadString('\n')
        if err == io.EOF {
            break
        } else if err != nil {
            fmt.Printf("error reading file %s", err)
            return err
        }
        r := regexp.MustCompile("[^\\s]+")
		// This is where you apply the regular expression to split the line variable into fields.
        words := r.FindAllString(line, -1)
		// This for loop just prints the fields of the words slice. If you want to know the number of words found in the input line, you can just find the value of the len(words) call.
        for i := 0; i < len(words); i++ {
            fmt.Println(words[i])
        }
    }
    return nil
}
```

### Reading a text file character by character
You take each line that you read and split it using a for loop with range, which returns two values. You discard the first, which is the location of the current character in the line variable, and you use the second. However, that value is a rune, which means that you have to convert it into a character using string().

```go
func charByChar(file string) error {
    f, err := os.Open(file)
    if err != nil {
        return err
    }
    defer f.Close()
    r := bufio.NewReader(f)
    for {
        line, err := r.ReadString('\n')
        if err == io.EOF {
            break
        } else if err != nil {
            fmt.Printf("error reading file %s", err)
            return err
        }
        for _, x := range line {
            fmt.Println(string(x))
        }
    }
    return nil
}
```
Note that, due to the fmt.Println(string(x)) statement, each character is printed in a distinct line, which means that the output of the program is going to be large. If you want a more compressed output, you should use the fmt.Print() function instead.

The use of the head(1) utility without any parameters limits the output to just 10 lines.
### Reading from /dev/random
The purpose of the /dev/random system device is to generate random data, which you might use for testing your programs or, in this case, as the seed for a random number generator. Getting data from /dev/random can be a little bit tricky, and this is the main reason for specifically discussing it here.

You need encoding/binary because you are reading binary data from /dev/random that you convert into an integer value.

```go
package main
import (
    "encoding/binary"
    "fmt"
    "os"
)

func main() {
	f, err := os.Open("/dev/random")
	defer f.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
	var seed int64
	binary.Read(f, binary.LittleEndian, &seed)
	fmt.Println("Seed:", seed)
}
```
There are two representations named little endian and big endian that have to do with the byte order in the internal representation. In our case, we are using little endian. The endian-ness has to do with the way different computing systems order multiple bytes of information.

> A real-world example of endian-ness is how different languages read text in different ways: European languages tend to be read from left to right, whereas Arabic texts are read from right to left.

In a big endian representation, bytes are read from left to right, while little endian reads bytes from right to left. For the 0x01234567 value, which requires 4 bytes for storing, the big endian representation is 01 | 23 | 45 | 67 whereas the little endian representation is 67 | 45 | 23 | 01.

### Reading a specific amount of data from a file
The numeric value that is given as a command-line argument specifies the size of the buffer that is going to be used for reading.

```go
func readSize(f *os.File, size int) []byte {
    buffer := make([]byte, size)
    n, err := f.Read(buffer)
    // io.EOF is a special case and is treated as such
    if err == io.EOF {
        return nil
    }
    if err != nil {
        fmt.Println(err)
        return nil
    }
    return buffer[0:n]
}
```
All the magic happens in the definition of the buffer variable because this is where we define the maximum amount of data that we want to read. Therefore, each time you invoke readSize(), the function is going to read from f at most size characters.

The remaining code is about error conditions; io.EOF is a special and expected condition that should be treated separately and return the read characters as a byte slice to the caller function.

## Writing to a file

os.Create() returns an *os.File value associated with the file path that is passed as a parameter. Note that if the file already exists, os.Create() truncates it.

The fmt.Fprintf() function, which requires a string variable, helps you write data to your own files using the format you want. The only requirement is having an io.Writer to write to. In this case, a valid *os.File variable, which satisfies the io.Writer interface, does the job.

The os.WriteString() method writes the contents of a string to a valid *os.File variable.

os.OpenFile() provides a better way to create or open a file for writing. os.O_APPEND is saying that if the file already exists, you should append to it instead of truncating it. os.O_CREATE is saying that if the file does not already exist, it should be created. Last, os.O_WRONLY is saying that the program should open the file for writing only.

The Write() method gets its input from a byte slice, which is the Go way of writing. All previous techniques used strings, which is not the best way, especially when working with binary data. However, using strings instead of byte slices is more practical as it is more convenient to manipulate string values than the elements of a byte slice, especially when working with Unicode characters. On the other hand, using string values increases allocation and can cause a lot of garbage collection pressure.

```go
package main
import (
    "bufio"
    "fmt"
    "io"
    "os"
)
func main() {
    buffer := []byte("Data to write\n")
    f1, err := os.Create("/tmp/f1.txt")
	if err != nil {
		fmt.Println("Cannot create file", err)
		return
	}
	defer f1.Close()
	fmt.Fprintf(f1, string(buffer))
	f2, err := os.Create("/tmp/f2.txt")
	if err != nil {
		fmt.Println("Cannot create file", err)
		return
	}
	defer f2.Close()
	n, err := f2.WriteString(string(buffer))
	fmt.Printf("wrote %d bytes\n", n)
	f3, err := os.Create("/tmp/f3.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	w := bufio.NewWriter(f3)
	n, err = w.WriteString(string(buffer))
	fmt.Printf("wrote %d bytes\n", n)
	w.Flush()
	f := "/tmp/f4.txt"
	f4, err := os.Create(f)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer f4.Close()
	for i := 0; i < 5; i++ {
		n, err = io.WriteString(f4, string(buffer))
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Printf("wrote %d bytes\n", n)
	}
	// Append to a file
	f4, err = os.OpenFile(f, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer f4.Close()
	// Write() needs a byte slice
	n, err = f4.Write([]byte("Put some more data at the end.\n"))
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("wrote %d bytes\n", n)
}
```
### Working with JSON
Go allows you to add support for JSON fields in Go structures using tags, which is the subject of the Structures and JSON subsection. Tags control the encoding and decoding of JSON records to and from Go structures.

### Using Marshal() and Unmarshal()
Marshaling is the process of converting a Go structure into a JSON record. You usually want that for transferring JSON data via computer networks or for saving it on disk. Unmarshaling is the process of converting a JSON record given as a byte slice into a Go structure. You usually want that when receiving JSON data via computer networks or when loading JSON data from disk files.

> The number one bug when converting JSON records into Go structures and vice versa is not making the required fields of your Go structures exported. When you have issues with marshaling and unmarshaling, begin your debugging process from there.

```go
package main
import (
    "encoding/json"
    "fmt"
)
type UseAll struct {
    Name    string `json:"username"`
    Surname string `json:"surname"`
    Year    int    `json:"created"`
}

func main() {
	useall := UseAll{Name: "Mike", Surname: "Tsoukalos", Year: 2021}
	// Regular Structure
	// Encoding JSON data -> Convert Go Structure to JSON record with fields
	t, err := json.Marshal(&useall)
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("Value %s\n", t)
	}
	// Decoding JSON data given as a string
	str := `{"username": "M.", "surname": "Ts", "created":2020}`
	// Convert string into a byte slice
	jsonRecord := []byte(str)
	// Create a structure variable to store the result
	temp := UseAll{}
	err = json.Unmarshal(jsonRecord, &temp)
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("Data type: %T with value %v\n", temp, temp)
	}
}
```
What the previous metadata tells us is that the Name field of the UseAll structure is translated to username in the JSON record, and vice versa
Other than this, you treat and use UseAll as a regular Go structure.

The json.Marshal() function requires a pointer to a structure variable—its real data type is an empty interface variable—and returns a byte slice with the encoded information and an error variable.

However, as json.Unmarshal() requires a byte slice, you need to convert that string into a byte slice before passing it to json.Unmarshal().

The json.Unmarshal() function requires the byte slice with the JSON record and a pointer to the Go structure variable that is going to store the JSON record and returns an error variable.


### Structures and JSON
without including any empty fields—the next code illustrates how to perform that task with the use of omitempty:

```go
// Ignoring empty fields in JSON
type NoEmpty struct {
    Name    string `json:"username"`
    Surname string `json:"surname"`
    Year    int    `json:"creationyear,omitempty"`
}
```
Last, imagine that you have some sensitive data on some of the fields of a Go structure that you do not want to include in the JSON records. You can do that by including the "-" special value in the desired json: structure tags.

```go
// Removing private fields and ignoring empty fields
type Password struct {
    Name     string `json:"username"`
    Surname  string `json:"surname,omitempty"`
    Year     int    `json:"creationyear,omitempty"`
    Pass     string `json:"-"`
}
```

### Reading and writing JSON data as streams
The good thing is that Go supports the processing of multiple JSON records as streams instead of individual records, which is faster and more efficient.

```go
// DeSerialize decodes a serialized slice with JSON records
func DeSerialize(e *json.Decoder, slice interface{}) error {
    return e.Decode(slice)
}
```
The function writes the slice, which is of the interface{} data type and is given as a parameter, and gets its input from the buffer of the *json.Decoder parameter. The *json.Decoder parameter along with its buffer is defined in the main() function in order to avoid allocating it all the time and therefore losing the performance gains and efficiency of using this type

```go
// Serialize serializes a slice with JSON records
func Serialize(e *json.Encoder, slice interface{}) error {
    return e.Encode(slice)
}
```
The Serialize() function accepts two parameters, a *json.Encoder and a slice of any data type, hence the use of interface{}. The function processes the slice and writes the output to the buffer of the json.Encoder—this buffer is passed as a parameter to the encoder at the time of its creation.

> Both the Serialize() and DeSerialize() functions can work with any type of JSON record due to the use of interface{}.

### Pretty printing JSON records
As there exist two ways to read JSON records, individually and as a stream, there exist two ways to pretty print JSON data: as single JSON records and as a stream.

```go
func PrettyPrint(v interface{}) (err error) {
    b, err := json.MarshalIndent(v, "", "\t")
    if err == nil {
        fmt.Println(string(b))
    }
    return err
}
```

The json.NewEncoder() function returns a new Encoder that writes to a writer that is passed as a parameter to json.NewEncoder(). An Encoder writes JSON values to an output stream. Similarly to json.MarshalIndent(), the SetIndent() method allows you to apply a customizable indent to a stream.

```go
func JSONstream(data interface{}) (string, error) {
    buffer := new(bytes.Buffer)
    encoder := json.NewEncoder(buffer)
    encoder.SetIndent("", "\t")
    err := encoder.Encode(data)
    if err != nil {
        return "", err
    }
	
    return buffer.String(), nil
}
```

## Working with XML
The idea behind XML and Go is the same as with JSON and Go. You put tags in Go structures in order to specify the XML tags and you can still serialize and deserialize XML records using xml.Unmarshal() and xml.Marshal(), which are found in the encoding/xml package.

```go
package main
import (
    "encoding/xml"
    "fmt"
)
type Employee struct {
    XMLName   xml.Name `xml:"employee"`
    ID        int      `xml:"id,attr"`
    FirstName string   `xml:"name>first"`
    LastName  string   `xml:"name>last"`
    Height    float32  `xml:"height,omitempty"`
    Address
    Comment string `xml:",comment"`
}
type Address struct {
	City, Country string
}
func main() {
	r := Employee{ID: 7, FirstName: "Mihalis", LastName: "Tsoukalos"}
	r.Comment = "Technical Writer + DevOps"
	r.Address = Address{"SomeWhere 12", "12312, Greece"}
	// As is the case with JSON, xml.MarshalIndent() is for beautifying the output.
	output, err := xml.MarshalIndent(&r, "  ", "    ")

	if err != nil {
		fmt.Println("Error:", err)
	}
	output = []byte(xml.Header + string(output))
	fmt.Printf("%s\n", output)
}
```

However, there is additional information regarding the name and the type of each XML element. The XMLName field provides the name of the XML record, which in this case will be employee.
A field with the tag ",comment" is a comment and it is formatted as such in the output. A field with the tag attr appears as an attribute to the provided field name (which is id in this case) in the output. The name>first notation tells Go to embed the first tag inside a tag called name.
Lastly, a field with the omitempty option is omitted from the output if it is empty. An empty value is any of 0, false, a nil pointer or interface, and any array, slice, map, or string with a length of zero.

### Converting JSON to XML and vice versa
```go
type XMLrec struct {
	Name    string `xml:"username"`
	Surname string `xml:"surname,omitempty"`
	Year    int    `xml:"creationyear,omitempty"`
}
type JSONrec struct {
	Name    string `json:"username"`
	Surname string `json:"surname,omitempty"`
	Year    int    `json:"creationyear,omitempty"`
}
```
## Working with YAML
The Go standard library does not include support for YAML files, which means that you should look at external packages for YAML support. There exist three main packages that allow you to work with YAML from Go:

- [https://github.com/kylelemons/go-gypsy](https://github.com/kylelemons/go-gypsy)
- [https://github.com/go-yaml/yaml](https://github.com/go-yaml/yaml)
- [https://github.com/goccy/go-yaml](https://github.com/go-yaml/yaml)
```go
var yamlfile = `
image: Golang
matrix:
  docker: python
  version: [2.7, 3.9]
`
type Mat struct {
    DockerImage string    `yaml:"docker"`
    Version     []float32 `yaml:",flow"`
}
type YAML struct {
    Image  string
    Matrix Mat
}
```
The Version field is a slice of float32 values. As there is no name for the Version field, the name is going to be version. The flow keyword says that the marshal is using a flow style, which is useful for structs, sequences, and maps.
The go mod init command initializes and writes a new go.mod file in the current directory whereas the go mod tidy command synchronizes go.mod with the source code.

> If you want to play it safe and you are using packages that do not belong to the standard library, then developing inside ~/go/src, committing to a GitHub repository, and using Go modules for all dependencies might be the best option. However, this does not mean that you must develop your own packages in the form of Go modules.

## The viper package
Flags are specially formatted strings that are passed into a program to control its behavior.

All viper projects follow a pattern. First, you initialize viper and then you define the elements that interest you. After that, you get these elements and read their values in order to use them. The desired values can be taken either directly, as happens when you are using the flag package from the standard Go library, or indirectly using configuration files. When using formatted configuration files in the JSON, YAML, TOML, HCL, or Java properties format, viper does all the parsing for you, which saves you from having to write and debug lots of Go code. viper also allows you to extract and save values in Go structures. However, this requires that the fields of the Go structure match the keys of the configuration file.

The general rule is to use the features of Viper that simplify your code. Put simply, if your command-line utility requires too many command-line parameters and flags, then it would be better to use a configuration file instead.

### Using command-line flags
Starting from Go version 1.16, using modules is the default behavior, which the viper package needs to use. So, you need to put useViper.go, which is the name of the source file, inside ~/go for things to work.

```go
package main
import (
    "fmt"
    "github.com/spf13/pflag"
    "github.com/spf13/viper"
)
func aliasNormalizeFunc(f *pflag.FlagSet, n string) pflag.NormalizedName {
	switch n {
	case "pass":
		n = "password"
		break
	case "ps":
		n = "password"
		break
	}
	return pflag.NormalizedName(n)
}

func main() {
	pflag.StringP("name", "n", "Mike", "Name parameter")
	pflag.StringP("password", "p", "hardToGuess", "Password")
	pflag.CommandLine.SetNormalizeFunc(aliasNormalizeFunc)
	pflag.Parse()
	viper.BindPFlags(pflag.CommandLine)
	name := viper.GetString("name")
	password := viper.GetString("password")
	fmt.Println(name, password)
	// Reading an Environment variable
	viper.BindEnv("GOMAXPROCS")
	val := viper.Get("GOMAXPROCS")
	if val != nil {
		fmt.Println("GOMAXPROCS:", val)
	}

	// Setting an Environment variable
	viper.Set("GOMAXPROCS", 16)
	val = viper.Get("GOMAXPROCS")
	fmt.Println("GOMAXPROCS:", val)
}
```
We need to import both the pflag and viper packages as we are going to use the functionality from both of them.
The aliasNormalizeFunc() function is used for creating additional aliases for a flag—in this case an alias for the --password flag. According to the existing code, the --password flag can be accessed as either --pass or --ps.

In the preceding code, we create a new flag called name that can also be accessed as -n. Its default value is Mike and its description that appears in the usage of the utility is Name parameter.

Additionally, we register a normalization function for generating aliases for the password flag.

The pflag.Parse() call should be used after all command-line flags are defined—its purpose is to parse the command-line flags into the defined flags.
Additionally, the viper.BindPFlags() call makes all flags available to the viper package—strictly speaking, we say that the viper.BindPFlags() call binds an existing set of pflag flags (pflag.FlagSet) to viper.

The previous commands show how to read the values of two string command-line flags.

The viper package can work with environment variables. We first need to call viper.BindEnv() to tell viper the environment variable that interests us and then we can read its value by calling viper.Get(). If GOMAXPROCS is not already set, the fmt.Println() call will not get executed.
Similarly, we can change the current value of an environment variable using viper.Set().
The good thing is that viper automatically provides usage information:

```shell
$ go run useViper.go --help
Usage of useViper:
  -n, --name string       Name parameter (default "Mike")
  -p, --password string   Password (default "hardToGuess")
pflag: help requested
exit status 2
```
### Reading JSON configuration files
Using text files for storing configuration details can be very helpful when writing complex applications that require lots of data and setup.

There is an important point here: although we are using a JSON file to store the configuration, the Go structure uses mapstructure instead of json for the fields of the JSON configuration file.

Keep in mind that viper does not check whether the configuration file actually exists and is readable. If the file cannot be found or read, viper.ReadInConfig() acts like processing an empty configuration file.


```go
package main
import (
    "encoding/json"
    "fmt"
    "os"
    "github.com/spf13/viper"
)
type ConfigStructure struct {
    MacPass     string `mapstructure:"macos"`
    LinuxPass   string `mapstructure:"linux"`
    WindowsPass string `mapstructure:"windows"`
    PostHost    string `mapstructure:"postgres"`
    MySQLHost   string `mapstructure:"mysql"`
    MongoHost   string `mapstructure:"mongodb"`
}
var CONFIG = ".config.json"
func main() {
	if len(os.Args) == 1 {
		fmt.Println("Using default file", CONFIG)
	} else {
		CONFIG = os.Args[1]
	}
	viper.SetConfigType("json")
	viper.SetConfigFile(CONFIG)
	fmt.Printf("Using config: %s\n", viper.ConfigFileUsed())
	viper.ReadInConfig()

	if viper.IsSet("macos") {
		fmt.Println("macos:", viper.Get("macos"))
	} else {
		fmt.Println("macos not set!")
	}

	if viper.IsSet("active") {
		value := viper.GetBool("active")
		if value {
			postgres := viper.Get("postgres")
			mysql := viper.Get("mysql")
			mongo := viper.Get("mongodb")
			fmt.Println("P:", postgres, "My:", mysql, "Mo:", mongo)
		}
	} else {
		fmt.Println("active is not set!")
	}

	if !viper.IsSet("DoesNotExist") {
		fmt.Println("DoesNotExist is not set!")
	}

	var t ConfigStructure
	err := viper.Unmarshal(&t)
	if err != nil {
		fmt.Println(err)
		return
	}
	PrettyPrint(t)
}
```
The viper.IsSet() call checks whether a key named macos can be found in the configuration. If it is set, it reads its value using viper.Get("macos")

As the active key should hold a Boolean value, we use viper.GetBool() for reading it.

The call to viper.Unmarshal() allows you to put the information from the JSON configuration file into a properly defined Go structure—this is optional but handy.

Excerpt From
Mastering Go
Mihalis Tsoukalos
This material may be protected by copyright.
## The cobra package
cobra also supports persistent flags and local flags, which are flags that are available to all commands and flags that are available to given commands only, respectively. Also, by default, cobra uses viper for parsing its command-line arguments.
All cobra projects follow the same development pattern. You use the cobra utility, then you create commands, and then you make the desired changes to the generated Go source code files in order to implement the desired functionality.

```shell
$ GO111MODULE=on go get -u -v github.com/spf13/cobra/cobra
```
The previous command downloads the cobra binary and the required dependencies using Go modules even if you are using a Go version older than 1.16.

> It is not necessary to know about all of the supported environment variables such as GO111MODULE, but sometimes they can help you resolve tricky problems with your Go installation. So, if you want to learn about your current Go environment, you can use the go env command.

As the cobra package works better with modules, we define the project dependencies using Go modules. In order to specify that a Go project uses Go modules, you should execute go mod init. This command creates two files named go.sum and go.mod.

### A utility with three commands
cobra add command, which is used for adding new commands to a cobra project.

```shell
$ ~/go/bin/cobra add three
```
The previous commands create three new files in the cmd folder named one.go, two.go, and three.go, which are the initial implementations of the three commands.
The first thing you should usually do is delete unwanted code from root.go and change the messages of the utility and each command as described in the Short and Long fields. However, if you want you can leave the source files unchanged.

### Adding command-line flags
Global command-line flags are defined in the ./cmd/root.go file. We are going to define two global flags named directory, which is a string, and depth, which is an unsigned integer.
Both global flags are defined in the init() function of ./cmd/root.go.

```go
rootCmd.PersistentFlags().StringP("directory", "d", "/tmp", "Path to use.")
rootCmd.PersistentFlags().Uint("depth", 2, "Depth of search.")
viper.BindPFlag("directory", rootCmd.PersistentFlags().Lookup("directory"))
viper.BindPFlag("depth", rootCmd.PersistentFlags().Lookup("depth"))
```
We use rootCmd.PersistentFlags() to define global flags followed by the data type of the flag.

if you want to add a shortcut to it, you should use the UintP() method instead. After defining the two flags, we pass their control to viper by calling viper.BindPFlag()

As both of them are available in the cobra project, we call viper.GetString("directory") to get the value of the directory flag and viper.GetUint("depth") to get the value of the depth flag.

Last, we add a command-line flag that is only available to the two command using the next line in the ./cmd/two.go file:

```go
twoCmd.Flags().StringP("username", "u", "Mike", "Username value")
```
As this is a local flag available to the two command only, we can get its value by calling cmd.Flags().GetString("username") inside the ./cmd/two.go file only.

### Creating command aliases
you need to add an extra field named Aliases in the cobra.Command structure of each command—the data type of the Aliases field is string slice.

```go
var oneCmd = &cobra.Command{
    Use:     "one",
    Aliases: []string{"cmd1"},
    Short:   "Command one",
```
Please keep in mind that the internal name of the one command is oneCmd—the other commands have analogous internal names.

> If you accidentally put the cmd1 alias, or any other alias, in multiple commands, the Go compiler will not complain. However, only its first occurrence gets executed.

### Creating subcommands
The way to create them using the cobra utility is the following:

```shell
$ ~/go/bin/cobra add list -p 'threeCmd'
```

The previous commands create two new files inside ./cmd named delete.go and list.go. The -p flag is followed by the internal name of the command you want to associate the subcommands with. The internal name of the three command is threeCmd. You can verify that these two commands are associated with the three command as follows:

```shell
$ go run main.go three delete
delete called
```
> At this point you might wonder what happens when you want to create two subcommands with the same name for two different commands. In that case, you create the first subcommand and rename its file before creating the second one.

## Finding cycles in a UNIX file system

If you are wondering why we are using a string and not a byte slice or some other kind of slice as the key for the visited map, it is because maps cannot have slices as keys because slices are not comparable.

The filepath.Walk() function does not traverse symbolic links by design in order to avoid cycles.

The utility uses IsDir(), which is a function that helps you to identify directories

the utility uses os.Lstat() because it can handle symbolic links. Additionally, os.Lstat() returns information about the symbolic link without following it, which is not the case with os.Stat()—in this case we do not want to automatically follow symbolic links.

First, we make sure that the path actually exists, and then we call os.Lstat().

The filepath.EvalSymlinks() function is used for finding out where symbolic links point to. If that destination is another directory, then the code that follows makes sure that it is going to be visited as well using an additional call to filepath.Walk().

The call to filepath.Abs() returns the absolute path of the path that is given as a parameter.
```go
func walkFunction(path string, info os.FileInfo, err error) error {
    fileInfo, err := os.Stat(path)
    if err != nil {
        return nil
    }
    fileInfo, _ = os.Lstat(path)
    mode := fileInfo.Mode()
    // Find regular directories first
    if mode.IsDir() {
        abs, _ := filepath.Abs(path)
        _, ok := visited[abs]
        if ok {
            fmt.Println("Found cycle:", abs)
            return nil
        }
        visited[abs]++
        return nil
    }

	// Find symbolic links to directories
    if fileInfo.Mode()&os.ModeSymlink != 0 {
        temp, err := os.Readlink(path)
        if err != nil {
            fmt.Println("os.Readlink():", err)
            return err
        }
        newPath, err := filepath.EvalSymlinks(temp)
        if err != nil {
            return nil
        }
		
        linkFileInfo, err := os.Stat(newPath)
        if err != nil {
            return err
        }
        
		linkMode := linkFileInfo.Mode()
        if linkMode.IsDir() {
            fmt.Println("Following...", path, "-->", newPath)
            abs, _ := filepath.Abs(newPath)
            _, ok := visited[abs]
            if ok {
                fmt.Println("Found cycle!", abs)
                return nil
            }
            visited[abs]++
            err = filepath.Walk(newPath, walkFunction)
            if err != nil {
                return err
            }
            return nil
        }
    }
    return nil
}
```

## New to Go 1.16
Go 1.16 came with some new features including embedding files in Go binaries as well as the introduction of the os.ReadDir() function, the os.DirEntry type, and the io/fs package.
### Embedding files
appeared in Go 1.16 that allows you to embed static assets into Go binaries. The allowed data types for keeping an embedded file are string, []byte, and embed.FS. This means that a Go binary may contain a file that you do not have to manually download when you execute the Go binary
```go
package main
import (
    _ "embed"
    "fmt"
    "os"
)

	//go:embed static/image.png
	var f1 []byte

    //go:embed static/textfile
    var f2 string

func writeToFile(s []byte, path string) error {
    fd, err := os.OpenFile(path, os.O_CREATE|os.O_WRONLY, 0644)
    if err != nil {
        return err
    }
    defer fd.Close()
    n, err := fd.Write(s)
    if err != nil {
        return err
    }
    fmt.Printf("wrote %d bytes\n", n)
    return nil
}

func main() {
    arguments := os.Args
    if len(arguments) == 1 {
        fmt.Println("Print select 1|2")
        return
    }
    fmt.Println("f1:", len(f1), "f2:", len(f2))
    switch arguments[1] {
        case "1":
            filename := "/tmp/temporary.png"
            err := writeToFile(f1, filename)
            if err != nil {
                fmt.Println(err)
                return
            }
        case "2":
            fmt.Print(f2)
            default:
            fmt.Println("Not a valid option!")
    }
}
```
You need the embed package in order to embed any files in your Go binaries. As the embed package is not directly used, you need to put _ in front of it so that the Go compiler won't complain.

You need to begin a line with //go:embed, which denotes a Go comment but is treated in a special way, followed by the path to the file you want to embed. In this case we embed static/image.png, which is a binary file.

using a byte slice is recommended for binary files because we are going to directly use that byte slice to save that binary file.

In this case we save the contents of a plain text file, which is static/textfile, in a string variable named f2.

The switch block is responsible for returning the desired file to the user—in the case of static/textfile, the file contents are printed on the screen. For the binary file, we decided to store it as /tmp/temporary.png.

### ReadDir and DirEntry
- os.ReadDir(), which is a new function, returns []DirEntry. This means that it cannot directly replace ioutil.ReadDir(), which returns []FileInfo. Although neither os.ReadDir() nor os.DirEntry offers any new functionality, they make things faster and simpler, which is important. 
- The os.ReadFile() function directly replaces ioutil.ReadFile(). 
- The os.WriteFile() function can directly replace ioutil.WriteFile(). 
- Similarly, os.MkdirTemp() can replace ioutil.TempDir() without any changes. However, as the os.TempDir() name was already taken, the new function name is different. 
- The os.CreateTemp() function is the same as ioutil.TempFile(). Although the name os.TempFile() was not taken, the Go people decided to name it os.CreateTemp() in order to be on par with os.MkdirTemp().

> Both os.ReadDir() and os.DirEntry can be found as fs.ReadDir() and fs.DirEntry in the io/fs package for working with the file system interface found in io/fs.

Both fs.WalkDir() and filepath.WalkDir() are using DirEntry instead of FileInfo. This means that in order to see any performance improvements when walking directory trees, you need to change filepath.Walk() calls to filepath.WalkDir() calls.

If it is a file, then we just need to get its size. This involves calling Info() to get general information about the file and then Size() to get the size of the file:

```go
func GetSize(path string) (int64, error) {
    contents, err := os.ReadDir(path)
    if err != nil {
        return -1, err
    }
    var total int64
    for _, entry := range contents {
        // Visit directory entries
        if entry.IsDir() {
            temp, err := GetSize(filepath.Join(path, entry.Name()))
            if err != nil {
                return -1, err
            }
            total += temp
            // Get size of each non-directory entry
        } else {
            info, err := entry.Info()
            if err != nil {
                return -1, err
            }
            // Returns an int64 value
            total += info.Size()
        }
    }
    return total, nil
}
```
### The io/fs package
Put simply, io/fs offers a read-only file system interface named FS. Note that embed.FS implements the fs.FS interface, which means that embed.FS can take advantage of some of the functionality offered by the io/fs package. This means that your applications can create their own internal file systems and work with their files


The ReadFile() function is used for retrieving a file, which is identified by its file path, from the embed.FS file system as a byte slice, which is returned from the extract() function.

searchString is a global variable that holds the search string. When a match is found, the matching path is printed on screen.

we make a call to fs.Stat() in order to get more details about it.

```go
func list(f embed.FS) error {
    return fs.WalkDir(f, ".", walkFunction)
}
func walkFunction(path string, d fs.DirEntry, err error) error {
    if err != nil {
        return err
    }
    fmt.Printf("Path=%q, isDir=%v\n", path, d.IsDir())
    return nil
}
func extract(f embed.FS, filepath string) ([]byte, error) {
    s, err := fs.ReadFile(f, filepath)
    if err != nil {
        return nil, err
    }
    return s, nil
}
func walkSearch(path string, d fs.DirEntry, err error) error {
    if err != nil {
        return err
    }
    if d.Name() == searchString {
        fileInfo, err := fs.Stat(f, path)
        if err != nil {
            return err
        }
        fmt.Println("Found", path, "with size", fileInfo.Size())
        return nil
    }
    return nil
}

```
## Updating the phone book application

```shell
$ cd ~/go/src/github.com/mactsouk
$ git clone git@github.com:mactsouk/phonebook.git
$ cd phonebook
```

### Using cobra

The first task after cloning the GitHub repository, which at this point is almost empty, is to run the cobra init command with the appropriate parameters.

```shell
$ ~/go/bin/cobra init --pkg-name github.com/mactsouk/phonebook
```

Then, you should create the structure of the application using the cobra binary. Once you have the structure, it is easy to know what you have to implement. The structure of the application is based on the supported commands.

```shell
$ ~/go/bin/cobra add list
```

After that you should declare that we want to use Go modules by executing the next command:

```shell
$ go mod init
```

If needed you can run go mod tidy after go mod init. At this point executing go run main.go should download all required package dependencies and generate the default cobra output.

## Exercises
Use the functionality of byCharacter.go, byLine.go, and byWord.go in order to create a simplified version of the wc(1) UNIX utility.
Create a full version of the wc(1) UNIX utility using the viper package for processing command-line options.
Create a full version of the wc(1) UNIX utility using commands instead of command-line options with the help of the cobra package.
Modify JSONstreams.go to accept user data or data from a file.
Modify embedFiles.go in order to save the binary file at a user-selected location.
Modify ioFS.go in order to get the desired command as well as the search string as a command-line argument.
Make ioFS.go a cobra project.
The byLine.go utility uses ReadString('\n') to read the input file. Modify the code to use Scanner (https://golang.org/pkg/bufio/#Scanner) for reading.
Similarly, byWord.go uses ReadString('\n') to read the input file—modify the code to use Scanner instead.
Modify the code of yaml.go in order to read the YAML data from an external file.

## Additional resources
- The viper package: [https://github.com/spf13/viper](https://github.com/spf13/viper)
- The cobra package: [https://github.com/spf13/cobra](https://github.com/spf13/viper)
- The documentation of encoding/json: [https://golang.org/pkg/encoding/json](https://github.com/spf13/viper)
- The documentation of io/fs: [https://golang.org/pkg/io/fs/](https://github.com/spf13/viper)
- Endian-ness: [https://en.wikipedia.org/wiki/Endianness](https://github.com/spf13/viper)
- Go 1.16 release notes: [https://golang.org/doc/go1.16](https://github.com/spf13/viper)