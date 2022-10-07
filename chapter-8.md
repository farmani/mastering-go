# Building Web Services

## The net/http package

### The http.Response type

The http.Response structure embodies the response from an HTTP request—both http.Client and http.Transport return http.Response values once the response headers have been received.

```go
type Response struct {
    Status     string // e.g. "200 OK"
    StatusCode int    // e.g. 200
    Proto      string // e.g. "HTTP/1.0"
    ProtoMajor int    // e.g. 1
    ProtoMinor int    // e.g. 0
    Header Header
    Body io.ReadCloser 
    ContentLength int64
    TransferEncoding []string
    Close bool
    Uncompressed bool
    Trailer Header 
    Request *Request
    TLS *tls.ConnectionState
}
```
You do not have to use all the structure fields, but it is good to know that they exist. However, some of them, such as Status, StatusCode, and Body, are more important than others. The Go source file, as well as the output of go doc http.Response, contains more information about the purpose of each field

### The http.Request type

```go
type Request struct {
    Method string
    URL *url.URL
    Proto  string
    ProtoMajor int
    ProtoMinor int
    Header Header
    Body io.ReadCloser
    GetBody func() (io.ReadCloser, error)
    ContentLength int64
    TransferEncoding []string
    Close bool
    Host string
    Form url.Values
    PostForm url.Values
    MultipartForm *multipart.Form
    Trailer Header
    RemoteAddr string
    RequestURI string
    TLS *tls.ConnectionState
    Cancel <-chan struct{}
    Response *Response
}
```
The Body field holds the body of the request. After reading the body of a request, you are allowed to call GetBody(), which returns a new copy of the body—this is optional.

### The http.Transport type
The definition of http.Transport, which gives you more control over your HTTP connections, is fairly long and complex:

```go
type Transport struct {
    Proxy func(*Request) (*url.URL, error)
    DialContext func(ctx context.Context, network, addr string) (net.Conn, error)
    Dial func(network, addr string) (net.Conn, error)
    DialTLSContext func(ctx context.Context, network, addr string) (net.Conn, error)
    DialTLS func(network, addr string) (net.Conn, error)
    TLSClientConfig *tls.Config
    TLSHandshakeTimeout time.Duration
    DisableKeepAlives bool
    DisableCompression bool
    MaxIdleConns int
    MaxIdleConnsPerHost int
    MaxConnsPerHost int
    IdleConnTimeout time.Duration
    ResponseHeaderTimeout time.Duration
    ExpectContinueTimeout time.Duration
    TLSNextProto map[string]func(authority string, c *tls.Conn) RoundTripper
    ProxyConnectHeader Header
    GetProxyConnectHeader func(ctx context.Context, proxyURL *url.URL, target string) (Header, error)
    MaxResponseHeaderBytes int64
    WriteBufferSize int
    ReadBufferSize int
    ForceAttemptHTTP2 bool
}
```
Note that http.Transport is pretty low-level, whereas http.Client, which is also used in this chapter, implements a high-level HTTP client—each http.Client contains a Transport field. If its value is nil, then DefaultTransport is used. You do not need to use http.Transport in all of your programs and you are not required to deal with all of its fields each time you use it. If you want to learn more about DefaultTransport, type go doc http.DefaultTransport.

## Creating a web server

You might ask why the presented web server uses HTTP instead of secure HTTP (HTTPS). The answer to this question is simple: most Go web servers are deployed as Docker images and are hidden behind web servers such as Caddy and Nginx that provide the secure HTTP operation part using the appropriate security credentials. It does not make any sense to use the secure HTTP protocol along with the required security credentials without knowing how and under which domain name the application is going to be deployed. This is a common practice in microservices as well as regular web applications that are deployed in Docker images.
The net/http package offers functions and data types that allow you to develop powerful web servers and clients. The http.Set() and http.Get() methods can be used to make HTTP and HTTPS requests, whereas http.ListenAndServe() is used for creating web servers given the user-specified handler function or functions that handle incoming requests.
The simplest way to define the supported endpoints, as well as the handler function that responds to each client request, is with the use of http.HandleFunc(), which can be called multiple times

```go
package main
import (
    "fmt"
    "net/http"
    "os"
    "time"
)
func myHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Serving: %s\n", r.URL.Path)
    fmt.Printf("Served: %s\n", r.Host)
}
func timeHandler(w http.ResponseWriter, r *http.Request) {
    t := time.Now().Format(time.RFC1123)
    Body := "The current time is:"
    fmt.Fprintf(w, "<h1 align=\"center\">%s</h1>", Body)
    fmt.Fprintf(w, "<h2 align=\"center\">%s</h2>\n", t)
    fmt.Fprintf(w, "Serving: %s\n", r.URL.Path)
    fmt.Printf("Served time for: %s\n", r.Host)
}
func main() {
	PORT := ":8001"
	arguments := os.Args
	if len(arguments) != 1 {
		PORT = ":" + arguments[1]
	}
	fmt.Println("Using port number: ", PORT)
	http.HandleFunc("/time", timeHandler)
	http.HandleFunc("/", myHandler)
	err := http.ListenAndServe(PORT, nil)
	if err != nil {
		fmt.Println(err)
		return
	}
}
```

sends a message back to the client using the w http.ResponseWriter, which is also an interface that implements io.Writer and is used for sending the server response
The / path matches every URL not matched by other handlers.
As there is no hostname given in the PORT string, the web server is going to listen to all available network interfaces. The port number and the hostname should be separated with a colon (:), which should be there even if there is no hostname—in that case the server listens to all available network interfaces and, therefore, all supported hostnames. This is the reason that the value of PORT is :8001 instead of just 8001.

Part of the net/http package is the ServeMux type (go doc http.ServeMux), which is an HTTP request multiplexer that provides a slightly different way of defining handler functions and endpoints than the default one

if we do not create and configure our own ServeMux variable, then http.HandleFunc() uses DefaultServeMux, which is the default ServeMux. So, in this case we are going to implement the web service using the default Go router—this is the reason that the second parameter of http.ListenAndServe() is nil.

## Updating the phone book application
### Defining the API
configure our own http.NewServeMux() variable. This changes the way we provide handler functions: a handler function with the func(http.ResponseWriter, *http.Request) signature has to be converted into an http.HandlerFunc type and be used by the ServeMux type and its own Handle() method. Therefore, when using a different ServeMux than the default one, we should do that conversion explicitly by calling http.HandlerFunc(), which makes the http.HandlerFunc type act as an adapter that allows the use of ordinary functions as HTTP handlers provided that they have the required signature. This is not a problem when using the default Go router (DefaultServeMux) because the http.HandleFunc() function does that conversion automatically and internally

To make things clearer, the http.HandlerFunc type has support for a method named HandlerFunc()—both the type and method are defined in the http package. The similarly named the http.HandleFunc() function (without an r) is used with the default Go router.

As an example, for the /time endpoint and the timeHandler() handler function, you should call mux.Handle() as mux.Handle("/time", http.HandlerFunc(timeHandler)).

### Implementing the handler
Usually, handlers are put in a separate package, but for reasons of simplicity, we decided to put handlers in a separate file within the same package named handlers.go

```go
package main
import (
    "fmt"
    "log"
    "net/http"
    "strings"
)
const PORT = ":1234"
func defaultHandler(w http.ResponseWriter, r *http.Request) {
    log.Println("Serving:", r.URL.Path, "from", r.Host)
    w.WriteHeader(http.StatusOK)
    Body := "Thanks for visiting!\n"
    fmt.Fprintf(w, "%s", Body)
}
func deleteHandler(w http.ResponseWriter, r *http.Request) {
    // Get telephone
    paramStr := strings.Split(r.URL.Path, "/")
    
    fmt.Println("Path:", paramStr)
    if len(paramStr) < 3 {
        w.WriteHeader(http.StatusNotFound)
        fmt.Fprintln(w, "Not found: "+r.URL.Path)
        return
    }
    log.Println("Serving:", r.URL.Path, "from", r.Host)
    telephone := paramStr[2]
    err := deleteEntry(telephone)
    if err != nil {
        fmt.Println(err)
        Body := err.Error() + "\n"
        w.WriteHeader(http.StatusNotFound)
        fmt.Fprintf(w, "%s", Body)
        return
    }
    Body := telephone + " deleted!\n"
    w.WriteHeader(http.StatusOK)
    fmt.Fprintf(w, "%s", Body)
}

func listHandler(w http.ResponseWriter, r *http.Request) {
    log.Println("Serving:", r.URL.Path, "from", r.Host)
    w.WriteHeader(http.StatusOK)
    Body := list()
    fmt.Fprintf(w, "%s", Body)
}
func statusHandler(w http.ResponseWriter, r *http.Request) {
    log.Println("Serving:", r.URL.Path, "from", r.Host)
    w.WriteHeader(http.StatusOK)
    Body := fmt.Sprintf("Total entries: %d\n", len(data))
    fmt.Fprintf(w, "%s", Body)
}

func insertHandler(w http.ResponseWriter, r *http.Request) {
    // Split URL
    paramStr := strings.Split(r.URL.Path, "/")
    fmt.Println("Path:", paramStr)
    if len(paramStr) < 5 {
        w.WriteHeader(http.StatusNotFound)
        fmt.Fprintln(w, "Not enough arguments: "+r.URL.Path)
        return
    }
    name := paramStr[2]
    surname := paramStr[3]
    tel := paramStr[4]
    t := strings.ReplaceAll(tel, "-", "")
    if !matchTel(t) {
        fmt.Println("Not a valid telephone number:", tel)
        return
    }

    temp := &Entry{Name: name, Surname: surname, Tel: t}
    err := insert(temp)
    
    if err != nil {
        w.WriteHeader(http.StatusNotModified)
        Body := "Failed to add record\n"
        fmt.Fprintf(w, "%s", Body)
    } else {
        log.Println("Serving:", r.URL.Path, "from", r.Host)
        Body := "New record added successfully\n"
        w.WriteHeader(http.StatusOK)
        fmt.Fprintf(w, "%s", Body)
    }
    log.Println("Serving:", r.URL.Path, "from", r.Host)
}

func searchHandler(w http.ResponseWriter, r *http.Request) {
    // Get Search value from URL
    paramStr := strings.Split(r.URL.Path, "/")
    fmt.Println("Path:", paramStr)
    if len(paramStr) < 3 {
        w.WriteHeader(http.StatusNotFound)
        fmt.Fprintln(w, "Not found: "+r.URL.Path)
        return
    }
    var Body string
    telephone := paramStr[2]
    t := search(telephone)
    if t == nil {
        w.WriteHeader(http.StatusNotFound)
        Body = "Could not be found: " + telephone + "\n"
    } else {
        w.WriteHeader(http.StatusOK)
        Body = t.Name + " " + t.Surname + " " + t.Tel + "\n"
    }
    fmt.Println("Serving:", r.URL.Path, "from", r.Host)
    fmt.Fprintf(w, "%s", Body)
}

func main() {
    err := readCSVFile(CSVFILE)
    if err != nil {
        fmt.Println(err)
        return
    }
    err = createIndex()
    if err != nil {
        fmt.Println("Cannot create index.")
        return
    }
    mux := http.NewServeMux()
    s := &http.Server{
        Addr:         PORT,
        Handler:      mux,
        IdleTimeout:  10 * time.Second,
        ReadTimeout:  time.Second,
        WriteTimeout: time.Second,
    }
    mux.Handle("/list", http.HandlerFunc(listHandler))
    mux.Handle("/insert/", http.HandlerFunc(insertHandler))
    mux.Handle("/insert", http.HandlerFunc(insertHandler))
    mux.Handle("/search", http.HandlerFunc(searchHandler))
    mux.Handle("/search/", http.HandlerFunc(searchHandler))
    mux.Handle("/delete/", http.HandlerFunc(deleteHandler))
    mux.Handle("/status", http.HandlerFunc(statusHandler))
    mux.Handle("/", http.HandlerFunc(defaultHandler))
    fmt.Println("Ready to serve at", PORT)
    err = s.ListenAndServe()
    if err != nil {
        fmt.Println(err)
        return
    }
}

```
If we do not have enough parameters, we should send an error message back to the client with the desired HTTP code, which in this case is http.StatusNotFound
The WriteHeader() method sends back a header with the provided status code before writing the body of the response
The list() helper function that is used in the /list path cannot fail. Therefore, http.StatusOK is always returned when serving /list. However, sometimes the return value of list() can be empty.

we store the parameters of the HTTP server in the http.Server structure and use our own http.NewServeMux() instead of the default one.

Note that /search and /search/ are both handled by the same handler function even though /search is going to fail as it does not include the required argument. On the other hand, /delete/ is handled in a special way

As we are using http.NewServeMux() and not the default Go router, we need to use http.HandlerFunc() when defining the handler functions

The presented message was generated by the Go router and tells us that we should try /delete/ instead as /delete was moved permanently. This is the kind of message that we get by not specifically defining both /delete and /delete/ in the routes.

interact with it using multiple HTTP requests as the http package uses multiple goroutines for interacting with clients—in practice, this means that the phone book application runs concurrently!

## Exposing metrics to Prometheus

Prometheus accepts many types of data. The list of supported data types for metrics is the following:
* Counter: This is a cumulative value that is used for representing increasing counters—the value of a counter can stay the same, go up, or be reset to zero but cannot decrease. Counters are usually used for representing cumulative values such as the number of requests served so far, the total number of errors, etc.
* Gauge: This is a single numerical value that is allowed to increase or decrease. Gauges are usually used for representing values that can go up or down such as the number of requests, time durations, etc.
* Histogram: A histogram is used for sampling observations and creating counts and buckets. Histograms are usually used for counting request durations, response times, etc.
* Summary: A summary is like a histogram but can also calculate quantiles over sliding windows that work with times.
Both histograms and summaries are useful and handy for performing statistical calculations and properties. Usually, a counter or a gauge is all that you need for storing your system metrics.

We begin by explaining the use of the runtime/metrics package, which provides Go runtime-related metrics.

The runtime/metrics package makes metrics exported by the Go runtime available to the developer. Each metric name is specified by a path. As an example, the number of live goroutines is accessed as /sched/goroutines:goroutines. However, if you want to collect all available metrics, you should use metrics.All()—this saves you from having to write lots of code in order to collect all metrics manually

Metrics are saved using the metrics.Sample data type. The definition of the metrics.Sample data structure is as follows:

### The runtime/metrics package


```go
type Sample struct {
    Name string
    Value Value
}
```

```go
package main
import (
    "fmt"
    "runtime/metrics"
    "sync"
    "time"
)
func main() {
    const nGo = "/sched/goroutines:goroutines
	// A slice for getting metric samples
	getMetric := make([]metrics.Sample, 1)
	getMetric[0].Name = nGo

	var wg sync.WaitGroup
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			time.Sleep(4 * time.Second)
		}()

		// Get actual data
		metrics.Read(getMetric)
		if getMetric[0].Value.Kind() == metrics.KindBad {
			fmt.Printf("metric %q no longer supported\n", nGo)
		}
		mVal := getMetric[0].Value.Uint64()
		fmt.Printf("Number of goroutines: %d\n", mVal)
	}
	wg.Wait()
	metrics.Read(getMetric)
	mVal := getMetric[0].Value.Uint64()
	fmt.Printf("Before exiting: %d\n", mVal)
}
```
The metrics.Read() function collects the desired metrics based on the data in the getMetric slice

After reading the desired metric, we convert it into a numeric value (unsigned int64 here) in order to use it in our program

The last lines of the code verify that after all goroutines have finished, the value of the metric is going to be 1, which is the goroutine used for running the main() function


### Exposing metrics

```go
package main
import (
    "fmt"
    "net/http"
    "math/rand"
    "time"
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var PORT = ":1234"
var counter = prometheus.NewCounter(
	prometheus.CounterOpts{
		Namespace: "mtsouk",
		Name:      "my_counter",
		Help:      "This is my counter",
	})
var gauge = prometheus.NewGauge(prometheus.GaugeOpts{
Namespace: "mtsouk",
Name:      "my_gauge",
Help:      "This is my gauge",
})

var histogram = prometheus.NewHistogram(
	prometheus.HistogramOpts{
		Namespace: "mtsouk",
		Name:      "my_histogram",
		Help:      "This is my histogram",
	})

var summary = prometheus.NewSummary(
	prometheus.SummaryOpts{
		Namespace: "mtsouk",
		Name:      "my_summary",
		Help:      "This is my summary",
	})

func main() {
	rand.Seed(time.Now().Unix())
	prometheus.MustRegister(counter)
	prometheus.MustRegister(gauge)
	prometheus.MustRegister(histogram)
	prometheus.MustRegister(summary)

	go func() {
		for {
			counter.Add(rand.Float64() * 5)
			gauge.Add(rand.Float64()*15 - 5)
			histogram.Observe(rand.Float64() * 10)
			summary.Observe(rand.Float64() * 10)
			time.Sleep(2 * time.Second)
		}
	}()
	http.Handle("/metrics", promhttp.Handler())
	fmt.Println("Listening to port", PORT)
	fmt.Println(http.ListenAndServe(PORT, nil))
}
```
This is how we define a new counter variable and specify the desired options. The Namespace field is very important as it allows you to group metrics in sets.
This is how we define a new gauge variable and specify the desired options.
This is how we define a new histogram variable and specify the desired options.
This is how we define a new summary variable and specify the desired options. However, as you are going to see, defining a metric variable is not enough. You also need to register it.
In these four statements, you register the four metric variables. Now Prometheus knows about them.
This goroutine runs for as long as the web server runs with the help of the endless for loop.
in this case we are using the `promhttp.Handler()` handler function that comes with the `github.com/prometheus/client_golang/prometheus/promhttp` package
Note that the metrics are found under the /metrics path—Prometheus knows how to find that.
So, the metrics are there and ready to be pulled by Prometheus—in practice, this means that every production Go application can export metrics that can be used for measuring its performance and discovering its bottlenecks

#### Creating a Docker image for a Go server
Docker images can be put in docker-compose.yml files and can be deployed using Kubernetes. The same is not true about Go binaries.
You usually start with a base Docker image that already includes Go and you create the desired binary in there. The key point here is that samplePro.go uses an external package that should be downloaded in the Docker image before building the executable binary.
The process must start with go mod init and go mod tidy

```dockerfile
# WITH Go Modules
FROM golang:alpine AS builder
RUN apk update && apk add --no-cache git
RUN mkdir $GOPATH/src/server
ADD ./samplePro.go $GOPATH/src/server
WORKDIR $GOPATH/src/server
RUN go mod init
RUN go mod tidy
RUN go mod download
RUN mkdir /pro
RUN go build -o /pro/server samplePro.go
FROM alpine:latest
RUN mkdir /pro
COPY --from=builder /pro/server /pro/server
EXPOSE 1234
WORKDIR /pro
CMD ["/pro/server"]
```
Once the Docker image has been created, there is no difference in the way you should use it in a docker-compose.yml file

```yaml
goapp:
    image: goapp
    container_name: goapp-int
    restart: always
    ports:
      - 1234:1234
    networks:
      - monitoring
```
The name of the Docker image is goapp whereas the internal name of the container would be goapp-int. So, if a different container from the monitoring network wants to access that container, it should use the goapp-int hostname. Last, the only open port is port number 1234

#### Exposing the desired metrics
In our case we use /sched/goroutines:goroutines and /memory/classes/total:bytes. You already know about the former, which is the total number of goroutines. The latter metric is the amount of memory mapped by the Go runtime into the current process as read-write.

> As the presented code uses an external package, it should be put inside ~/go/src and Go modules should be enabled using go mod init

```go
package main
import (
    "log"
    "math/rand"
    "net/http"
    "runtime"
    "runtime/metrics"
    "time"
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var PORT = ":1234"
var n_goroutines = prometheus.NewGauge(
	prometheus.GaugeOpts{
		Namespace: "packt",
		Name:      "n_goroutines",
		Help:      "Number of goroutines"})
var n_memory = prometheus.NewGauge(
	prometheus.GaugeOpts{
		Namespace: "packt",
		Name:      "n_memory",
		Help:      "Memory usage"})

func main() {
	rand.Seed(time.Now().Unix())
	prometheus.MustRegister(n_goroutines)
	prometheus.MustRegister(n_memory)
	const nGo = "/sched/goroutines:goroutines"
	const nMem = "/memory/classes/heap/free:bytes
	getMetric := make([]metrics.Sample, 2)
	getMetric[0].Name = nGo
	getMetric[1].Name = nMem
	http.Handle("/metrics", promhttp.Handler())
	go func() {
		for {
			for i := 1; i < 4; i++ {
				go func() {
					_ = make([]int, 1000000)
					time.Sleep(time.Duration(rand.Intn(10)) * time.Second)
				}()
			}
			runtime.GC()
			metrics.Read(getMetric)
			goVal := getMetric[0].Value.Uint64()
			memVal := getMetric[1].Value.Uint64()
			time.Sleep(time.Duration(rand.Intn(15)) * time.Second)
			n_goroutines.Set(float64(goVal))
			n_memory.Set(float64(memVal))
		}
	}()
	log.Println("Listening to port", PORT)
	log.Println(http.ListenAndServe(PORT, nil))
}
```
The first external package is the Go client library for Prometheus and the second package is for using the default handler function (promhttp.Handler()).
Note that such a program should definitely have at least two goroutines: one for running the HTTP server and another one for collecting the metrics. Usually, the HTTP server is on the goroutine that runs the main() function and the metric collection happens in a user-defined goroutine.

The runtime.GC() function tells the Go garbage collector to run and is called for changing the /memory/classes/heap/free:bytes metric. The two Set() calls update the values of the metrics

## Reading metrics

```dockerfile
RUN apk update && apk add --no-cache git
RUN mkdir $GOPATH/src/server
ADD ./prometheus.go $GOPATH/src/server
WORKDIR $GOPATH/src/server
RUN go mod init
RUN go mod tidy
RUN go mod download
RUN mkdir /pro
RUN go build -o /pro/server prometheus.go
FROM alpine:latest
RUN mkdir /pro
COPY --from=builder /pro/server /pro/server
EXPOSE 1234
WORKDIR /pro
CMD ["/pro/server"]
```
This is the name of the base Docker image that is used for building the binary. golang:alpine always contains the latest Go version as long as you update it regularly.

As golang:alpine does not come with git, we need to install it manually.
Building the desired Docker image, which is going to be named goapp, is as simple as running the next command:

```shell
$ docker build -f Dockerfile -t goapp .
```
## Putting the metrics in Prometheus

```yaml
# prometheus.yml
scrape_configs:
  - job_name: GoServer
    scrape_interval: 5s
    static_configs:
       - targets: ['goapp:1234']
```
We tell Prometheus to connect to a host named goapp using port number 1234. Prometheus pulls data every 5 seconds, according to the value of the scrape_interval field. You should put prometheus.yml in the prometheus directory, which should be in the same directory as the docker-compose.yml file that is presented next.

```yaml
version: "3"
services:
  goapp:
    image: goapp
    container_name: goapp
    restart: always
    ports:
      - 1234:1234
    networks:
      - monitoring
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: always
    user: "0"
    volumes:
      - ./prometheus/:/etc/prometheus/
      - ./prometheus_data/:/prometheus/
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    ports:
      - 9090:9090
    networks:
      - monitoring
  grafana:
    image: grafana/grafana
    container_name: grafana
    depends_on:
      - prometheus
    restart: always
    user: "0"
    ports:
      - 3000:3000
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=helloThere
volumes:
  grafana_data: {}
  prometheus_data: {}
networks:
  monitoring:
    driver: bridge
```
This is how you pass your own copy of prometheus.yml to the Docker image to be used by Prometheus. So, ./prometheus/prometheus.yml from the local machine can be accessed as /etc/prometheus/prometheus.yml from within the Docker image.

The Docker image used is called prom/prometheus:latest and the internal name of it is prometheus. Prometheus listens to port number 9090.

Last, we present the Grafana part. Grafana listens to port number 3000.

The preceding two lines in combination with the two volumes fields allow both Grafana and Prometheus to save their data locally so that data is not lost each time you restart the Docker images

Internally, all three containers are known by the value of their container_name field. However, externally, you can connect to the open ports from your local machine as http://localhost:port or from another machine using http://hostname:port—the second way is not very secure and should be blocked by a firewall. Lastly, you need to run docker-compose up and you are done! The Go application begins exposing data and Prometheus begins collecting it.

## Developing web clients

```go
package main
import (
    "fmt"
    "io"
    "net/http"
    "os"
    "path/filepath"
)
func main() {
    if len(os.Args) != 2 {
        fmt.Printf("Usage: %s URL\n", filepath.Base(os.Args[0]))
        return
    }
	URL := os.Args[1]
	data, err := http.Get(URL)
	if err != nil {
		fmt.Println(err)
		return
	}
	_, err = io.Copy(os.Stdout, data.Body)
	if err != nil {
		fmt.Println(err)
		return
	}
	data.Body.Close()
}
```
The filepath.Base() function returns the last element of a path. When given os.Args[0] as its parameter, it returns the name of the executable binary file

In the previous two statements we get the URL and get its data using http.Get(), which returns an *http.Response and an error variable

The io.Copy() function reads from the data.Body reader, which contains the body of the server response, and writes the data to os.Stdout. As os.Stdout is always open, you do not need to open it for writing. Therefore, all data is written to standard output, which is usually the terminal window.

Last, we close the data.Body reader to make the work of the garbage collection easier.

### Using http.NewRequest() to improve the client
```go
package main
// For the import block go to the book GitHub repository
func main() {
	if len(os.Args) != 2 {
		fmt.Printf("Usage: %s URL\n", filepath.Base(os.Args[0]))
		return
	}

	URL, err := url.Parse(os.Args[1])
	if err != nil {
		fmt.Println("Error in parsing:", err)
		return
	}
	c := &http.Client{
		Timeout: 15 * time.Second,
	}
	request, err := http.NewRequest(http.MethodGet, URL.String(), nil)
	if err != nil {
		fmt.Println("Get:", err)
		return
	}
	httpData, err := c.Do(request)
	if err != nil {
		fmt.Println("Error in Do():", err)
		return
	}
	fmt.Println("Status code:", httpData.Status)
	
	header, _ := httputil.DumpResponse(httpData, false)
	fmt.Print(string(header))
	
	contentType := httpData.Header.Get("Content-Type")
	characterSet := strings.SplitAfter(contentType, "charset=")
	if len(characterSet) > 1 {
		fmt.Println("Character Set:", characterSet[1])
	}

	if httpData.ContentLength == -1 {
		fmt.Println("ContentLength is unknown!")
	} else {
		fmt.Println("ContentLength:", httpData.ContentLength)
	}

	length := 0
	var buffer [1024]byte
	r := httpData.Body
	for {
		n, err := r.Read(buffer[0:])
		if err != nil {
			fmt.Println(err)
			break
		}
		length = length + n
	}
	fmt.Println("Calculated response data length:", length)
}
```
The url.Parse() function parses a string into a URL structure. This means that if the given argument is not a valid URL, url.Parse() is going to notice

The http.NewRequest() function returns an http.Request object given a method, a URL, and an optional body. The http.MethodGet parameter defines that we want to retrieve the data using a GET HTTP method whereas URL.String() returns the string value of an http.URL variable.

The http.Do() function sends an HTTP request (http.Request) using an http.Client and gets an http.Response. So, http.Do() does the job of http.Get() in a more detailed way.

The httputil.DumpResponse() function is used here to get the response from the server and is mainly used for debugging purposes. The second argument of httputil.DumpResponse() is a Boolean value that specifies whether the function is going to include the body or not in its output—in our case it is set to false, which excludes the response body from the output and only prints the header. If you want to do the same on the server side, you should use httputil.DumpRequest()

In the last part of the program, we use a technique for discovering the size of the server HTTP response on our own. If we wanted to display the HTML output on our screen, we could have printed the contents of the r buffer variable

### Creating a client for the phone book service

```shell
$ cd ~/go/src/github.com/mactsouk
$ git clone git@github.com:mactsouk/phone-cli.git
$ cd phone-cli
$ ~/go/bin/cobra init --pkg-name github.com/mactsouk/phone-cli
$ go mod init
$ go mod tidy
$ go mod download
$ ~/go/bin/cobra add search
$ ~/go/bin/cobra add insert
$ ~/go/bin/cobra add delete
$ ~/go/bin/cobra add status
$ ~/go/bin/cobra add list
```

```go
//root.go
func init() {
    rootCmd.PersistentFlags().StringP("server", "S", "localhost", "Server")
    rootCmd.PersistentFlags().StringP("port", "P", "1234", "Port number")
    viper.BindPFlag("server", rootCmd.PersistentFlags().Lookup("server"))
    viper.BindPFlag("port", rootCmd.PersistentFlags().Lookup("port"))
}
```

```go
//status.go
SERVER := viper.GetString("server")
PORT := viper.GetString("port")
// Create request
URL := "http://" + SERVER + ":" + PORT + "/status
data, err := http.Get(URL)
if err != nil {
    fmt.Println(err)
    return
}
// Check HTTP Status Code
if data.StatusCode != http.StatusOK {
    fmt.Println("Status code:", data.StatusCode)
    return
}

// Read data
responseData, err := io.ReadAll(data.Body)
if err != nil {
    fmt.Println(err)
    return
}
fmt.Print(string(responseData))

```

```go
//list.go
URL := "http://" + SERVER + ":" + PORT + "/list

```

```go
//delete.go
SERVER := viper.GetString("server")
PORT := viper.GetString("port")
number, _ := cmd.Flags().GetString("tel")
if number == "" {
    fmt.Println("Number is empty!")
    return
}
// Create request
URL := "http://" + SERVER + ":" + PORT + "/delete/" + number

// Send request to server
data, err := http.Get(URL)
if err != nil {
    fmt.Println(err)
    return
}

// Check HTTP Status Code
if data.StatusCode != http.StatusOK {
    fmt.Println("Status code:", data.StatusCode)
    return
}

// Read data
responseData, err := io.ReadAll(data.Body)
if err != nil {
    fmt.Println(err)
    return
}
fmt.Print(string(responseData))

func init() {
    rootCmd.AddCommand(deleteCmd)
    deleteCmd.Flags().StringP("tel", "t", "", "Telephone number to delete")
}

```

```go
//search.go
URL := "http://" + SERVER + ":" + PORT + "/search/" + number
```

```go
//insert.go
func init() {
    rootCmd.AddCommand(insertCmd)
    insertCmd.Flags().StringP("name", "n", "", "Name value")
    insertCmd.Flags().StringP("surname", "s", "", "Surname value")
    insertCmd.Flags().StringP("tel", "t", "", "Telephone value")
}
SERVER := viper.GetString("server")
PORT := viper.GetString("port")

number, _ := cmd.Flags().GetString("tel")
if number == "" {
    fmt.Println("Number is empty!")
    return
}
name, _ := cmd.Flags().GetString("name")
if number == "" {
    fmt.Println("Name is empty!")
    return
}
surname, _ := cmd.Flags().GetString("surname")
if number == "" {
    fmt.Println("Surname is empty!")
    return
}
URL := "http://" + SERVER + ":" + PORT + "/insert/"
URL = URL + "/" + name + "/" + surname + "/" + number

data, err := http.Get(URL)
if err != nil {
    fmt.Println("**", err)
    return
}
if data.StatusCode != http.StatusOK {
    fmt.Println("Status code:", data.StatusCode)
    return
}
responseData, err := io.ReadAll(data.Body)
if err != nil {
    fmt.Println("*", err)
    return
}
fmt.Print(string(responseData))
```

So, we define two global parameters named server and port, which are the hostname and the port number, respectively. Both parameters have an alias and both parameters are handled by viper.

The init() function of delete.go contains the definition of the local tel command-line parameter:

These three parameters are needed for getting the required user input. Note that the alias for surname is a lowercase s whereas the alias for server, which is defined in root.go, is an uppercase S. Commands and their aliases are user-defined—use common sense when selecting command names and aliases.

## Creating file servers
Although a file server is not a web server per se, it is closely connected to web services because it is being implemented using similar Go packages. Additionally, file servers are frequently used for supporting the functionality of web servers and web services.

Go offers the http.FileServer() handler for doing so, as well as http.ServeFile(). The biggest difference between these two is that http.FileServer() is an http.Handler whereas http.ServeFile() is not. Additionally, http.ServeFile() is better at serving single files whereas http.FileServer() is better at serving entire directory trees

```go
package main
import (
    "fmt"
    "log"
    "net/http
)
var PORT = ":8765"
func defaultHandler(w http.ResponseWriter, r *http.Request) {
    log.Println("Serving:", r.URL.Path, "from", r.Host)
    w.WriteHeader(http.StatusOK)
    Body := "Thanks for visiting!\n"
    fmt.Fprintf(w, "%s", Body)
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", defaultHandler)
    fileServer := http.FileServer(http.Dir("/tmp/"))
    mux.Handle("/static/", http.StripPrefix("/static", fileServer))
    fmt.Println("Starting server on:", PORT)
    err := http.ListenAndServe(PORT, mux)
    fmt.Println(err)
}
```
mux.Handle() registers the file server as the handler for all URL paths that begin with /static/. However, when a match is found, we strip the /static/ prefix before the file server tries to serve such a request because /static/ is not part of the location where the actual files are located. As far as Go is concerned, http.FileServer() is just another handler

### Downloading the contents of the phone book application
The code creates a temporary file with a different filename for each request with the contents of the phone book application

The temporary path is created using os.CreateTemp() based on a given pattern and by adding a random string to the end. If the pattern includes an *, then the randomly generated string replaces the last *. The exact place where the file is created depends on the operating system being used.

We do not want to end up having lots of temporary files, so we delete the file when the handler function returns


The time.Sleep() call delays the deletion of the temporary file for 30 seconds—you can define any delay period you like.
As far as the main() function is concerned, getFileHandler() is a regular handler function used in a mux.HandleFunc("/getContents/", getFileHandler) statement. Therefore, each time there is a client request for /getContents/, the contents of a file are returned to the HTTP client.


```go
func getFileHandler(w http.ResponseWriter, r *http.Request) {
    var tempFileName string
    // Create temporary file name
    f, err := os.CreateTemp("", "data*.txt")
    tempFileName = f.Name()
    // Remove the file
    defer os.Remove(tempFileName)
    // Save data to it
    err = saveCSVFile(tempFileName)
    if err != nil {
    fmt.Println(err)
    w.WriteHeader(http.StatusNotFound)
    fmt.Fprintln(w, "Cannot create: "+tempFileName)
        return
    }
    fmt.Println("Serving ", tempFileName)
    http.ServeFile(w, r, tempFileName)
    // 30 seconds to get the file
    time.Sleep(30 * time.Second)
}
```
## Timing out HTTP connections
### Using SetDeadline()
The SetDeadline() function is used by net to set the read and write deadlines of network connections. Due to the way the SetDeadline() function works, you need to call SetDeadline() before any read or write operation. Keep in mind that Go uses deadlines to implement timeouts, so you do not need to reset the timeout every time your application receives or sends any data

```go
var timeout = time.Duration(time.Second)
func Timeout(network, host string) (net.Conn, error) {
    conn, err := net.DialTimeout(network, host, timeout)
    if err != nil {
        return nil, err
    }
    conn.SetDeadline(time.Now().Add(timeout))
    return conn, nil
}

t := http.Transport{
    Dial: Timeout,
}
client := http.Client{
    Transport: &t,
}
```

So, http.Transport uses Timeout() in the Dial field and http.Client uses http.Transport. When you call the client.Get() method with the desired URL, which is not shown here, Timeout is automatically being used because of the http.Transport definition. So, if the Timeout function returns before the server response is received, we have a timeout.

### Setting the timeout period on the client side
This section presents a technique for timing out network connections that take too long to finish on the client side. So, if the client does not receive a response from the server in the desired time, it closes the connection.

```go
package main
// For the import block go to the book code repository
var myUrl string
var delay int = 5
var wg sync.WaitGroup
type myData struct {
    r   *http.Response
    err error
}

func connect(c context.Context) error {
    defer wg.Done()
    data := make(chan myData, 1)
    tr := &http.Transport{}
    httpClient := &http.Client{Transport: tr}
    req, _ := http.NewRequest("GET", myUrl, nil)
    
    go func() {
        response, err := httpClient.Do(req)
        if err != nil {
            fmt.Println(err)
            data <- myData{nil, err}
            return
        } else {
            pack := myData{response, err}
            data <- pack
        }
    }()

    select {
        case <-c.Done():
            tr.CancelRequest(req)
            <-data
            fmt.Println("The request was canceled!")
            return c.Err()
        
        case ok := <-data:
            err := ok.err
            resp := ok.r
            if err != nil {
            fmt.Println("Error select:", err)
            return err
            }
            defer resp.Body.Close()
            realHTTPData, err := io.ReadAll(resp.Body)
            if err != nil {
                fmt.Println("Error select:", err)
                return err
            }
        fmt.Printf("Server Response: %s\n", realHTTPData)
    }
    return nil
}
func main() {
    if len(os.Args) == 1 {
        fmt.Println("Need a URL and a delay!")
        return
    }
    myUrl = os.Args[1]
    if len(os.Args) == 3 {
        t, err := strconv.Atoi(os.Args[2])
        if err != nil {
            fmt.Println(err)
            return
        }
        delay = t
    }
    fmt.Println("Delay:", delay)
    c := context.Background()
    c, cancel := context.WithTimeout(c, time.Duration(delay)*time.Second)
    defer cancel()
    fmt.Printf("Connecting to %s \n", myUrl)
    wg.Add(1)
    go connect(c)
    wg.Wait()
    fmt.Println("Exiting...")
}
```
The code that this select block executes is based on whether the context is going to time out or not. If the context times out first, then the client connection is canceled using tr.CancelRequest(req)
The timeout period is defined by the context.WithTimeout() method. It is considered a good practice to use context.Background() in the main() function or the init() function of a package or in tests.

### Setting the timeout period on the server side

```go
func main() {
    PORT := ":8001"
    arguments := os.Args
    if len(arguments) != 1 {
        PORT = ":" + arguments[1]
    }
    fmt.Println("Using port number: ", PORT)
    m := http.NewServeMux()
    srv := &http.Server{
        Addr:         PORT,
        Handler:      m,
        ReadTimeout:  3 * time.Second,
        WriteTimeout: 3 * time.Second,
    }
    m.HandleFunc("/time", timeHandler)
    m.HandleFunc("/", myHandler)
    err := srv.ListenAndServe()
    if err != nil {
        fmt.Println(err)
        return
    }
}
```
The value of the ReadTimeout field specifies the maximum duration allowed to read the entire client request, including the body, whereas the value of the WriteTimeout field specifies the maximum time duration before timing out the sending of the client response
## Exercises
Put all handlers from www-phone.go in a different Go package and modify www-phone.go accordingly. You need a different repository for storing the new package.
Modify wwwClient.go to save the HTML output to an external file.
Include the functionality of getEntries.go in the phone book application.
Implement a simple version of ab(1) using goroutines and channels. ab(1) is an Apache HTTP server benchmarking tool.
## Additional resources
* Caddy server: [https://caddyserver.com/](https://caddyserver.com/)
* Nginx server: [https://nginx.org/en/](https://caddyserver.com/)
* Histograms in Prometheus: [https://prometheus.io/docs/practices/histograms/](https://caddyserver.com/)
* The net/http package: [https://golang.org/pkg/net/http/](https://caddyserver.com/)
* Official Docker Go images: [https://hub.docker.com/_/golang/](https://caddyserver.com/)