# Go Packages and Functions
Go packages
Downloading Go packages
Functions
Anonymous functions
Functions that return multiple values
The return values of a function can be named
Functions that accept other functions as parameters
Functions can return other functions
Variadic functions
The defer keyword
Developing your own packages
The init() function
Order of execution
Using GitHub to store Go packages
A package for working with a database
Getting to know your database
Storing the Go package
The design of the Go package
The implementation of the Go package
Testing the Go package
Modules
Creating better packages
Generating documentation
GitLab Runners and Go
The initial version of the configuration file
The final version of the configuration file
GitHub Actions and Go
Storing secrets in GitHub
The final version of the configuration file
Versioning utilities
Exercises
Summary
Additional resources
Telling a UNIX System What to Do
stdin, stdout, and stderr
UNIX processes
Handling UNIX signals
Handling two signals
File I/O
The io.Reader and io.Writer interfaces
Using and misusing io.Reader and io.Writer
Buffered and unbuffered file I/O
Reading text files
Reading a text file line by line
Reading a text file word by word
Reading a text file character by character
Reading from /dev/random
Reading a specific amount of data from a file
Writing to a file
Working with JSON
Using Marshal() and Unmarshal()
Structures and JSON
Reading and writing JSON data as streams
Pretty printing JSON records
Working with XML
Converting JSON to XML and vice versa
Working with YAML
The viper package
Using command-line flags
Reading JSON configuration files
The cobra package
A utility with three commands
Adding command-line flags
Creating command aliases
Creating subcommands
Finding cycles in a UNIX file system
New to Go 1.16
Embedding files
ReadDir and DirEntry
The io/fs package
Updating the phone book application
Using cobra
Storing and loading JSON data
Implementing the delete command
Implementing the insert command
Implementing the list command
Implementing the search command
Exercises
Summary
Additional resources
Go Concurrency
Processes, threads, and goroutines
The Go scheduler
The GOMAXPROCS environment variable
Concurrency and parallelism
Goroutines
Creating a goroutine
Creating multiple goroutines
Waiting for your goroutines to finish
What if the number of Add() and Done() calls differ?
Creating multiple files with goroutines
Channels
Writing to and reading from a channel
Receiving from a closed channel
Channels as function parameters
Race conditions
The Go race detector
The select keyword
Timing out a goroutine
Timing out a goroutine – inside main()
Timing out a goroutine – outside main()
Go channels revisited
Buffered channels
nil channels
Worker pools
Signal channels
Specifying the order of execution for your goroutines
Shared memory and shared variables
The sync.Mutex type
What happens if you forget to unlock a mutex?
The sync.RWMutex type
The atomic package
Sharing memory using goroutines
Closured variables and the go statement
The context package
Using context as a key/value store
The semaphore package
Exercises
Summary
Additional resources
Building Web Services
The net/http package
The http.Response type
The http.Request type
The http.Transport type
Creating a web server
Updating the phone book application
Defining the API
Implementing the handlers
Exposing metrics to Prometheus
The runtime/metrics package
Exposing metrics
Creating a Docker image for a Go server
Exposing the desired metrics
Reading metrics
Putting the metrics in Prometheus
Visualizing Prometheus metrics in Grafana
Developing web clients
Using http.NewRequest() to improve the client
Creating a client for the phone book service
Creating file servers
Downloading the contents of the phone book application
Timing out HTTP connections
Using SetDeadline()
Setting the timeout period on the client side
Setting the timeout period on the server side
Exercises
Summary
Additional resources
Working with TCP/IP and WebSocket
TCP/IP
The nc(1) command-line utility
The net package
Developing a TCP client
Developing a TCP client with net.Dial()
Developing a TCP client that uses net.DialTCP()
Developing a TCP server
Developing a TCP server with net.Listen()
Developing a TCP server that uses net.ListenTCP()
Developing a UDP client
Developing a UDP server
Developing concurrent TCP servers
Working with UNIX domain sockets
A UNIX domain socket server
A UNIX domain socket client
Creating a WebSocket server
The implementation of the server
Using websocat
Using JavaScript
Creating a WebSocket client
Exercises
Summary
Additional resources
Working with REST APIs
An introduction to REST
Developing RESTful servers and clients
A RESTful server
A RESTful client
Creating a functional RESTful server
The REST API
Using gorilla/mux
The use of subrouters
Working with the database
Testing the restdb package
Implementing the RESTful server
Testing the RESTful server
Testing GET handlers
Testing POST handlers
Testing the PUT handler
Testing the DELETE handler
Creating a RESTful client
Creating the structure of the command-line client
Implementing the RESTful client commands
Using the RESTful client
Working with multiple REST API versions
Uploading and downloading binary files
Using Swagger for REST API documentation
Documenting the REST API
Generating the documentation file
Serving the documentation file
Exercises
Summary
Additional resources
Code Testing and Profiling
Optimizing code
Benchmarking code
Rewriting the main() function for better testing
Benchmarking buffered writing and reading
The benchstat utility
Wrongly defined benchmark functions
Profiling code
Profiling a command-line application
Profiling an HTTP server
The web interface of the Go profiler
The go tool trace utility
Tracing a web server from a client
Visiting all routes of a web server
Testing Go code
Writing tests for ./ch03/intRE.go
The TempDir function
The Cleanup() function
The testing/quick package
Timing out tests
Testing code coverage
Finding unreachable Go code
Testing an HTTP server with a database backend
Fuzzing
Cross-compilation
Using go:generate
Creating example functions
Exercises
Summary
Additional resources
Working with gRPC
Introduction to gRPC
Protocol buffers
Defining an interface definition language file
Developing a gRPC server
Developing a gRPC client
Testing the gRPC server with the client
Exercises
Summary
Additional resources
Go Generics
Introducing generics
Constraints
Creating constraints
Defining new data types with generics
Using generics in Go structures
Interfaces versus generics
Reflection versus generics
Exercises
Summary
Additional resources
Appendix A – Go Garbage Collector
Heap and stack
Garbage collection
The tricolor algorithm
More about the operation of the Go garbage collector
Maps, slices, and the Go garbage collector
Using a slice
Using a map with pointers
Using a map without pointers
Splitting the map
Comparing the performance of the presented techniques
Additional resources