# Working with TCP/IP and WebSocket
## TCP/IP
TCP/IP is a family of protocols that help the internet operate. Its name comes from its two most well-known protocols: TCP and IP. TCP stands for Transmission Control Protocol. TCP software transmits data between machines using segments, which are also called TCP packets. The main characteristic of TCP is that it is a reliable protocol, which means that it makes sure that a packet was delivered without requiring any extra code from the programmer. If there is no proof of packet delivery, TCP resends that particular packet. Among other things, TCP packets can be used for establishing connections, transferring data, sending acknowledgments, and closing connections.

> When creating TCP/IP server processes, remember that port numbers 0-1024 have restricted access and can only be used by the root user, which means that you need administrative privileges to use a port in that range. Running a process with root privileges is a security risk and must be avoided

The main problem with IPv4 is that it is about to run out of available IP addresses, which is the main reason for creating the IPv6 protocol. This happened because an IPv4 address is represented using 32 bits only, which allows a total number of 232 (4,294,967,296) different IP addresses. On the other hand, IPv6 uses 128 bits to define each one of its addresses. The format of an IPv4 address is 10.20.32.245 (four parts separated by dots), while the format of an IPv6 address is 3fce:1706:4523:3:150:f8ff:fe21:56cf (eight parts separated by colons).

UDP (User Datagram Protocol) is based on IP, which means that it is also unreliable. UDP is simpler than TCP, mainly because UDP is not reliable by design. As a result, UDP messages can be lost, duplicated, or arrive out of order. Furthermore, packets can arrive faster than the recipient can process them. So, UDP is used when speed is more important than reliability

### The nc(1) command-line utility

The nc(1) utility, which is also called netcat(1), comes in very handy when you want to test TCP/IP servers and clients. Actually, nc(1) is a utility for everything that involves TCP and UDP as well as IPv4 and IPv6, including opening TCP connections, sending and receiving UDP messages, and acting as a TCP server

You can use nc(1) as a client for a TCP service that runs on a machine with the 10.10.1.123 IP address and listens to port number 1234, as follows:

```shell
$ nc 10.10.1.123 1234
```

The -l option tells netcat(1) to act as a server, which means that netcat(1) starts listening for incoming connections at the given port number. By default, nc(1) uses the TCP protocol. However, if you execute nc(1) with the -u flag, then nc(1) uses the UDP protocol, either as a client or as a server. Finally, the -v and -vv options tell netcat(1) to generate verbose output, which can come in handy when you want to troubleshoot network connections.

## The net package
The net.Dial() function is used for connecting to a remote server. The first parameter of the net.Dial() function defines the network protocol that is going to be used, while the second parameter defines the server address, which must also include the port number. Valid values for the first parameter are tcp, tcp4 (IPv4-only), tcp6 (IPv6-only), udp, udp4 (IPv4-only), udp6 (IPv6-only), ip, ip4 (IPv4-only), ip6 (IPv6-only), unix (UNIX sockets), unixgram, and unixpacket. On the other hand, valid values for net.Listen() are tcp, tcp4, tcp6, unix, and unixpacket

## Developing a TCP client
## Developing a TCP client with net.Dial()

```go
package main
import (
    "bufio"
    "fmt"
    "net"
    "os"
    "strings"
)
func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		fmt.Println("Please provide host:port.")
		return
	}
	connect := arguments[1]
	c, err := net.Dial("tcp", connect)
	if err != nil {
		fmt.Println(err)
		return
	}
	for {
		reader := bufio.NewReader(os.Stdin)
		fmt.Print(">> ")
		text, _ := reader.ReadString('\n')
		fmt.Fprintf(c, text+"\n")
		message, _ := bufio.NewReader(c).ReadString('\n')
		fmt.Print("->: " + message)
		if strings.TrimSpace(string(text)) == "STOP" {
			fmt.Println("TCP client exiting...")
			return
		}
	}
}
```
With the connection details, we call net.Dial()—its first parameter is the protocol we want to use, which in this case is tcp, and its second parameter is the connection details. A successful net.Dial() call returns an open connection (a net.Conn interface), which is a generic stream-oriented network connection

The last part of the TCP client keeps reading user input until the word STOP is given as input—in this case, the client waits for the server response before terminating after STOP because this is how the for loop is constructed. This mainly happens because the server might have a useful answer for us, and we do not want to miss that. All given user input is sent (written) to the open TCP connection using fmt.Fprintf(), whereas bufio.NewReader() is used for reading data from the TCP connection, just like you would do with a regular file.

## Developing a TCP client that uses net.DialTCP()
The difference lies in the Go functions that are being used for establishing the TCP connection, which are net.DialTCP() and net.ResolveTCPAddr(), and not in the functionality of the client

```go
package main
import (
    "bufio"
    "fmt"
    "net"
    "os"
    "strings"
)
func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		fmt.Println("Please provide a server:port string!")
		return
	}
	connect := arguments[1]
	tcpAddr, err := net.ResolveTCPAddr("tcp4", connect)
	if err != nil {
		fmt.Println("ResolveTCPAddr:", err)
		return
	}
	conn, err := net.DialTCP("tcp4", nil, tcpAddr)
	if err != nil {
		fmt.Println("DialTCP:", err)
		return
	}
	for {
		reader := bufio.NewReader(os.Stdin)
		fmt.Print(">> ")
		text, _ := reader.ReadString('\n')
		fmt.Fprintf(conn, text+"\n")
		message, _ := bufio.NewReader(conn).ReadString('\n')
		fmt.Print("->: " + message)
		if strings.TrimSpace(string(text)) == "STOP" {
			fmt.Println("TCP client exiting...")
			conn.Close()
			return
		}
	}
}
```

Although we are working with TCP/IP connections, we need packages such as bufio because UNIX treats network connections as files, so we are basically working with I/O operations over networks

The net.ResolveTCPAddr() function is specific to TCP connections, hence its name, and resolves the given address to a *net.TCPAddr value, which is a structure that represents the address of a TCP endpoint

With the TCP endpoint at hand, we call net.DialTCP() to connect to the server

## Developing a TCP server
## Developing a TCP server with net.Listen()
The net.Listen() function listens for connections, whereas the net.Accept() method waits for the next connection and returns a generic Conn variable with the client information.

```go
package main
import (
    "bufio"
    "fmt"
    "net"
    "os"
    "strings"
    "time"
)
func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		fmt.Println("Please provide port number")
		return
	}
	PORT := ":" + arguments[1]
	l, err := net.Listen("tcp", PORT)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer l.Close()
	c, err := l.Accept()
	if err != nil {
		fmt.Println(err)
		return
	}
	for {
		netData, err := bufio.NewReader(c).ReadString('\n')
		if err != nil {
			fmt.Println(err)
			return
		}
		if strings.TrimSpace(string(netData)) == "STOP" {
			fmt.Println("Exiting TCP server!")
			return
		}
		fmt.Print("-> ", string(netData))
		t := time.Now()
		myTime := t.Format(time.RFC3339) + "\n"
		c.Write([]byte(myTime))
	}
}
```
The net.Listen() function listens for connections and is what makes that particular program a server process. If the second parameter of net.Listen() contains a port number without an IP address or a hostname, net.Listen() listens to all available IP addresses of the local system, which is the case here

We just call Accept() and wait for a client connection—Accept() blocks until a connection comes. There is something unusual with this particular TCP server: it can only serve the first TCP client that is going to connect to it because the Accept() call is outside of the for loop and therefore is called only once. Each individual client should be specified by a different Accept() call

This endless for loop keeps interacting with the same TCP client until the word STOP is sent from the client. As it happened with the TCP clients, bufio.NewReader() is used for reading data from the network connection, whereas Write() is used for sending data to the TCP client.

In order to exit nc(1), we need to press Ctrl + D, which is EOF (End Of File) in UNIX.

## Developing a TCP server that uses net.ListenTCP()
Echo service TCP server sends back to the client the data that was received by the client.

```go
package main
import (
    "fmt"
    "net"
    "os"
    "strings"
)
func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		fmt.Println("Please provide a port number!")
		return
	}
	SERVER := "localhost" + ":" + arguments[1]
	s, err := net.ResolveTCPAddr("tcp", SERVER)
	if err != nil {
		fmt.Println(err)
		return
	}
	l, err := net.ListenTCP("tcp", s)
	if err != nil {
		fmt.Println(err)
		return
	}
	buffer := make([]byte, 1024)
	conn, err := l.Accept()
	if err != nil {
		fmt.Println(err)
		return
	}
	for {
		n, err := conn.Read(buffer)
		if err != nil {
			fmt.Println(err)
			return
		}
		if strings.TrimSpace(string(buffer[0:n])) == "STOP" {
			fmt.Println("Exiting TCP server!")
			conn.Close()
			return
		}
		fmt.Print("> ", string(buffer[0:n-1]), "\n")
		_, err = conn.Write(buffer)
		if err != nil {
			fmt.Println(err)
			return
		}
	}
}
```
As before, due to the place where Accept() is called, this particular implementation can work with a single client only. This is used for reasons of simplicity. The concurrent TCP server that is developed later on in this chapter puts the Accept() call inside the endless for loop.

You need to use strings.TrimSpace() in order to remove any space characters from your input and compare the result with STOP, which has a special meaning in this implementation. When the STOP keyword is received from the client, the server closes the connection using the Close() method

## Developing a UDP client
```go
package main
import (
    "bufio"
    "fmt"
    "net"
    "os"
    "strings"
)
func main() {
    arguments := os.Args
    if len(arguments) == 1 {
        fmt.Println("Please provide a host:port string")
        return
    }
    CONNECT := arguments[1]
	s, err := net.ResolveUDPAddr("udp4", CONNECT)
	c, err := net.DialUDP("udp4", nil, s)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("The UDP server is %s\n", c.RemoteAddr().String())
	defer c.Close()
	for {
		reader := bufio.NewReader(os.Stdin)
		fmt.Print(">> ")
		text, _ := reader.ReadString('\n')
		data := []byte(text + "\n")
		_, err = c.Write(data)
		if strings.TrimSpace(string(data)) == "STOP" {
			fmt.Println("Exiting UDP client!")
			return
		}
		if err != nil {
			fmt.Println(err)
			return
		}
		buffer := make([]byte, 1024)
		n, _, err := c.ReadFromUDP(buffer)
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Printf("Reply: %s\n", string(buffer[0:n]))
	}
}
```
The previous two lines declare that we are using UDP and that we want to connect to the UDP server that is specified by the return value of net.ResolveUDPAddr(). The actual connection is initiated using net.DialUDP().

This part of the program finds the details of the UDP server by calling the RemoteAddr() method

Data is read from the UDP connection using the ReadFromUDP() method.

## Developing a UDP server

```go
package main
import (
    "fmt"
    "math/rand"
    "net"
    "os"
    "strconv"
    "strings"
    "time"
)
func random(min, max int) int {
	return rand.Intn(max-min) + min
}
func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		fmt.Println("Please provide a port number!")
		return
	}
	PORT := ":" + arguments[1]
	s, err := net.ResolveUDPAddr("udp4", PORT)
	if err != nil {
		fmt.Println(err)
		return
	}
	connection, err := net.ListenUDP("udp4", s)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer connection.Close()
	buffer := make([]byte, 1024)

	rand.Seed(time.Now().Unix())
	for {
		n, addr, err := connection.ReadFromUDP(buffer)
		fmt.Print("-> ", string(buffer[0:n-1]))
		if strings.TrimSpace(string(buffer[0:n])) == "STOP" {
			fmt.Println("Exiting UDP server!")
			return
		}
		data := []byte(strconv.Itoa(random(1, 1001)))
		fmt.Printf("data: %s\n", string(data))
		_, err = connection.WriteToUDP(data, addr)
		if err != nil {
			fmt.Println(err)
			return
		}
	}
}
```
The net.ResolveUDPAddr() function creates a UDP endpoint that is going to be used to create the server

The net.ListenUDP("udp4", s) function call makes this process a server for the udp4 protocol using the details specified by its second parameter

The buffer variable stores a byte slice and is used to read data from the UDP connection

The ReadFromUDP() and WriteToUDP() methods are used to read data from a UDP connection and write data to a UDP connection, respectively. Additionally, due to the way UDP operates, the UDP server can serve multiple clients

Excerpt From
Mastering Go
Mihalis Tsoukalos
This material may be protected by copyright.

## Developing concurrent TCP servers

```go
package main
import (
    "bufio"
    "fmt"
    "net"
    "os"
    "strconv"
    "strings"
)
var count = 0
func handleConnection(c net.Conn) {
    fmt.Print(".")
	for {
		netData, err := bufio.NewReader(c).ReadString('\n')
		if err != nil {
			fmt.Println(err)
			return
		}
		temp := strings.TrimSpace(string(netData))
		if temp == "STOP" {
			break
		}
		fmt.Println(temp)
		counter := "Client number: " + strconv.Itoa(count) + "\n"
		c.Write([]byte(string(counter)))
	}
	c.Close()
}
func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		fmt.Println("Please provide a port number!")
		return
	}
	PORT := ":" + arguments[1]
	l, err := net.Listen("tcp4", PORT)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer l.Close()
	for {
		c, err := l.Accept()
		if err != nil {
			fmt.Println(err)
			return
		}
		go handleConnection(c)
		count++
	}
}
```
Each time a new client connects to the server, the count variable is increased. Each TCP client is served by a separate goroutine that executes the handleConnection() function. This frees the server process and allows it to accept new connections. Put simply, while multiple TCP clients are served, the TCP server is free to interact with more TCP clients. As before, new TCP clients are connected using the Accept() function.

# Working with UNIX domain sockets
A UNIX Domain Socket or Inter-Process Communication (IPC) socket is a data communications endpoint that allows you to exchange data between processes that run on the same machine. You might ask, why use UNIX domain sockets instead of TCP/IP connections for processes that exchange data on the same machine? First, because UNIX domain sockets are faster than TCP/IP connections and second, because UNIX domain sockets require fewer resources than TCP/IP connections. So, you can use UNIX domain sockets when both the clients and the server are on the same machine

## A UNIX domain socket server
This section illustrates how to develop a UNIX domain socket server. Although we do not have to deal with TCP ports and network connections, the code presented is very similar to the code of the TCP server as found in tcpS.go and concTCP.go

```go
package main
import (
    "fmt"
    "net"
    "os"
)
func echo(c net.Conn) {
	for {
		buf := make([]byte, 128)
		n, err := c.Read(buf)
		if err != nil {
			fmt.Println("Read:", err)
			return
		}
		data := buf[0:n]
		fmt.Print("Server got: ", string(data))
		_, err = c.Write(data)
		if err != nil {
			fmt.Println("Write:", err)
			return
		}
	}
}
func main() {
	if len(os.Args) == 1 {
		fmt.Println("Need socket path")
		return
	}
	socketPath := os.Args[1]
	_, err := os.Stat(socketPath)
	if err == nil {
		fmt.Println("Deleting existing", socketPath)
		err := os.Remove(socketPath)
		if err != nil {
			fmt.Println(err)
			return
		}
	}
	l, err := net.Listen("unix", socketPath)
	if err != nil {
		fmt.Println("listen error:", err)
		return
	}
	for {
		fd, err := l.Accept()
		if err != nil {
			fmt.Println("Accept error:", err)
			return
		}
		go echo(fd)
	}
}
```
The buf[0:n] notation makes sure that we are going to send back the same amount of data that was read even if the size of the buffer is bigger.

This function serves all client connections—as you are going to see in a while, it is executed as a goroutine, which is the main reason that it does not return any values.
You cannot tell whether this function serves TCP/IP connections or UNIX socket domain connections, which mainly happens because UNIX treats all connections as files.

If the socket file already exists, you should delete it before the program continues—net.Listen() creates that file again.

What makes this a UNIX domain socket server is the use of net.Listen() with the "unix" parameter. In this case, we need to provide net.Listen() with the path of the socket file.

## A UNIX domain socket client

```go
package main
import (
    "bufio"
    "fmt"
    "net"
    "os"
    "strings"
    "time"
)
func main() {
    if len(os.Args) == 1 {
        fmt.Println("Need socket path")
        return
    }
    socketPath := os.Args[1]
	c, err := net.Dial("unix", socketPath)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer c.Close()
	for {
		reader := bufio.NewReader(os.Stdin)
		fmt.Print(">> ")
		text, _ := reader.ReadString('\n')
		_, err = c.Write([]byte(text))
		if err != nil {
			fmt.Println("Write:", err)
			break
		}
		buf := make([]byte, 256)
		n, err := c.Read(buf[:])
		if err != nil {
			fmt.Println(err, n)
			return
		}
		fmt.Print("Read:", string(buf[0:n]))
		if strings.TrimSpace(string(text)) == "STOP" {
			fmt.Println("Exiting UNIX domain socket client!")
			return
		}
		time.Sleep(5 * time.Second)
	}
}
```

This is the part where we get from the user the socket file that is going to be used—the socket file should already exist and be handled by the UNIX domain socket server.

The net.Dial() function is used for connecting to the socket.

Generally speaking, it is always good to have a way of gracefully exiting such a utility

The time.Sleep() call is used to delay the for loop and emulate the operation of a real program

# Creating a WebSocket server
The WebSocket protocol is a computer communications protocol that provides full-duplex (transmission of data in two directions simultaneously) communication channels over a single TCP connection

according to its documentation, golang.org/x/net/websocket lacks some features and it is advised that you use https://godoc.org/github.com/gorilla/websocket, the one used here, or https://godoc.org/nhooyr.io/websocket instead.

The advantages of the WebSocket protocol include the following:

- A WebSocket connection is a full-duplex, bidirectional communications channel. This means that a server does not need to wait to read from a client to send data to the client and vice versa.
- WebSocket connections are raw TCP sockets, which means that they do not have the overhead required to establish an HTTP connection.
- WebSocket connections can also be used for sending HTTP data. However, plain HTTP connections cannot work as WebSocket connections.
- WebSocket connections live until they are killed, so there is no need to reopen them all the time.
- WebSocket connections can be used for real-time web applications.
- Data can be sent from the server to the client at any time, without the client even requesting it.
- WebSocket is part of the HTML5 specification, which means that it is supported by all modern web browsers.

Before showing the server implementation, it would be good for you to know that the websocket.Upgrader method of the gorilla/websocket package upgrades an HTTP server connection to the WebSocket protocol and allows you to define the parameters of the upgrade. After that, your HTTP connection is a WebSocket connection, which means that you will not be allowed to execute statements that work with the HTTP protocol.

## The implementation of the server

```go
package main
import (
    "fmt"
    "log"
    "net/http"
    "os"
    "time"
    "github.com/gorilla/websocket"
)
var PORT = ":1234"
var upgrader = websocket.Upgrader{
	ReadBufferSize:  1024,
	WriteBufferSize: 1024,
	CheckOrigin: func(r *http.Request) bool {
		return true
	},
}
func rootHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Welcome!\n")
	fmt.Fprintf(w, "Please use /ws for WebSocket!")
}
func wsHandler(w http.ResponseWriter, r *http.Request) {
	log.Println("Connection from:", r.Host)
	ws, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println("upgrader.Upgrade:", err)
		return
	}
	defer ws.Close()
	for {
		mt, message, err := ws.ReadMessage()
		if err != nil {
			log.Println("From", r.Host, "read", err)
			break
		}
		log.Print("Received: ", string(message))
		err = ws.WriteMessage(mt, message)
		if err != nil {
			log.Println("WriteMessage:", err)
			break
		}
	}
}
func main() {
	arguments := os.Args
	if len(arguments) != 1 {
		PORT = ":" + arguments[1]
	}
	mux := http.NewServeMux()
	s := &http.Server{
		Addr:         PORT,
		Handler:      mux,
		IdleTimeout:  10 * time.Second,
		ReadTimeout:  time.Second,
		WriteTimeout: time.Second,
	}
	mux.Handle("/", http.HandlerFunc(rootHandler))
	mux.Handle("/ws", http.HandlerFunc(wsHandler))
	log.Println("Listening to TCP Port", PORT)
	err := s.ListenAndServe()
	if err != nil {
		log.Println(err)
		return
	}
}

```
This is where the parameters of websocket.Upgrader are defined

This is a regular HTTP handler function

A WebSocket server application calls the Upgrader.Upgrade method in order to get a WebSocket connection from an HTTP request handler. Following a successful call to Upgrader.Upgrade, the server begins working with the WebSocket connection and the WebSocket client

The for loop in wsHandler() handles all incoming messages for /ws—you can use any technique you want. Additionally, in the presented implementation, only the client is allowed to close an existing WebSocket connection unless there is a network issue, or the server process is killed.

Last, remember that in a WebSocket connection, you cannot use fmt.Fprintf() statements for sending data to the WebSocket client—if you use any of these, or any other call that can implement the same functionality, the WebSocket connection fails and you are not going to be able to send or receive any data. Therefore, the only way to send and receive data in a WebSocket connection implemented with gorilla/websocket is through WriteMessage() and ReadMessage() calls, respectively
as this is a server process, using log.Println() is a much better choice than fmt.Println() because logging information is sent to files that can be examined at a later time. However, during development, you might prefer fmt.Println() calls and avoid writing to your log files because you can see your data on screen immediately without having to look elsewhere.

## Using websocat
websocat is a command-line utility that can help you test WebSocket connections

```shell
$ websocat -v ws://localhost:1234/ws
```

## Using JavaScript

```html
<!DOCTYPE html>
<meta charset="utf-8">
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Testing a WebSocket Server</title>
  </head>
  <body>
    <h2>Hello There!</h2>
    <script>
        let ws = new WebSocket("ws://localhost:1234/ws");
        console.log("Trying to connect to server.");
        ws.onopen = () => {
            console.log("Connected!");
            ws.send("Hello From the Client!")
        };
        ws.onmessage = function(event) {
            console.log(`[message] Data received from server: ${event.data}`);
            ws.close(1000, "Work complete");
        };
        ws.onclose = event => {
            if (event.wasClean) {
                console.log(`[close] Connection closed cleanly, code=${event.code} reason=${event.reason}`);
            }
            console.log("Socket Closed Connection: ", event);
        };
        ws.onerror = error => {
            console.log("Socket Error: ", error);
        };
    </script>
  </body>
</html>
```
The ws.onopen event is used for making sure that the WebSocket connection is open, whereas the send() method is used for sending messages to the WebSocket server.
The onmessage event is triggered each time the WebSocket server sends a new message—however, in our case, the connection is closed as soon as the first message from the server is received
Lastly, the close() JavaScript method is used for closing a WebSocket connection—in our case, the close() call is included in the onmessage event. Calling close() triggers the onclose event, which contains the code that follows

## Creating a WebSocket client
the gorilla/websocket package is going to help us develop the WebSocket client.

```go
package main
import (
    "bufio"
    "fmt"
    "log"
    "net/url"
    "os"
    "os/signal"
    "syscall"
    "time"
    "github.com/gorilla/websocket"
)
var SERVER = ""
var PATH = ""
var TIMESWAIT = 0
var TIMESWAITMAX = 5
var in = bufio.NewReader(os.Stdin)
func getInput(input chan string) {
	result, err := in.ReadString('\n')
	if err != nil {
		log.Println(err)
		return
	}
	input <- result
}
func main() {
	arguments := os.Args
	if len(arguments) != 3 {
		fmt.Println("Need SERVER + PATH!")
		return
	}
	SERVER = arguments[1]
	PATH = arguments[2]
	fmt.Println("Connecting to:", SERVER, "at", PATH)
	interrupt := make(chan os.Signal, 1)
	signal.Notify(interrupt, os.Interrupt)
	input := make(chan string, 1)
	go getInput(input)
	URL := url.URL{Scheme: "ws", Host: SERVER, Path: PATH}
	c, _, err := websocket.DefaultDialer.Dial(URL.String(), nil)
	if err != nil {
		log.Println("Error:", err)
		return
	}
	defer c.Close()
	done := make(chan struct{})
	go func() {
		defer close(done)
		for {
			_, message, err := c.ReadMessage()
			if err != nil {
				log.Println("ReadMessage() error:", err)
				return
			}
			log.Printf("Received: %s", message)
		}
	}()
	for {
		select {
		case <-time.After(4 * time.Second):
			log.Println("Please give me input!", TIMESWAIT)
			TIMESWAIT++
			if TIMESWAIT > TIMESWAITMAX {
				syscall.Kill(syscall.Getpid(), syscall.SIGINT)
			}
        case <-done:
			return
		case t := <-input:
			err := c.WriteMessage(websocket.TextMessage, []byte(t))
			if err != nil {
				log.Println("Write error:", err)
				return
			}
			TIMESWAIT = 0
			go getInput(input)
		case <-interrupt:
			log.Println("Caught interrupt signal - quitting!")
			err := c.WriteMessage(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseNormalClosure, ""))
			if err != nil {
                log.Println("Write close error:", err)
                return
            }
			select {
                case <-done:
                case <-time.After(2 * time.Second):
			}
			return
		}
	}
}
```
The in variable is just a shortcut for bufio.NewReader(os.Stdin)

The WebSocket client handles UNIX interrupts with the help of the interrupt channel. When the appropriate signal is caught (syscall.SIGINT), the WebSocket connection with the server is closed with the help of the websocket.CloseMessage message. This is how professional tools work!
Another goroutine, which this time is implemented using an anonymous Go function, is responsible for reading data from the WebSocket connection using the ReadMessage() method

The syscall.Kill(syscall.Getpid(), syscall.SIGINT) statement sends the interrupt signal to the program using Go code. According to the logic of client.go, the interrupt signal makes the program close the WebSocket connection with the server and terminate its execution. This only happens if the current number of timeout periods is bigger than a predefined global value

If you get user input, the current number of the timeout periods (TIMESWAIT) is reset and new input is read.

Just before we close the client connection, we send websocket.CloseMessage to the server in order to do the closing the right way
WebSocket gives you an alternative way of creating services. As a rule of thumb, WebSocket is better when we want to exchange lots of data, and we want the connection to remain open all the time and exchange data in full-duplex. However, if you are not sure about what to use, begin with a TCP/IP service and see how it goes before upgrading it to the WebSocket protocol

# Exercises
- Develop a concurrent TCP server that generates random numbers in a predefined range.
- Develop a concurrent TCP server that generates random numbers in a range that is given by the TCP client. This can be used as a way of randomly picking values from a set.
  Add UNIX signal processing to the concurrent TCP server developed in this chapter to gracefully stop the server process when a given signal is received.
- Develop a UNIX domain socket server that generates random numbers. After that, program a client for that server. 
- Develop a WebSocket server that creates a variable number of random integers that are sent to the client. The number of random integers is specified by the client at the initial client message.

Additional resources
* The WebSocket protocol: [https://tools.ietf.org/rfc/rfc6455.txt](https://tools.ietf.org/rfc/rfc6455.txt)
* Wikipedia WebSocket: [https://en.wikipedia.org/wiki/WebSocket](https://tools.ietf.org/rfc/rfc6455.txt)
* Gorilla WebSocket package: [https://github.com/gorilla/websocket](https://tools.ietf.org/rfc/rfc6455.txt)
* Gorilla WebSocket docs: [https://www.gorillatoolkit.org/pkg/websocket](https://tools.ietf.org/rfc/rfc6455.txt)
* The websocket package: [https://pkg.go.dev/golang.org/x/net/websocket](https://tools.ietf.org/rfc/rfc6455.txt)