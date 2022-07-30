# Go Packages and Functions
Go packages, which are Go's way of organizing, delivering, and using code. The most common component of packages is functions, which are pretty flexible and powerful and are used for data processing and manipulation. Go also supports modules, which are packages with version numbers.

Regarding the visibility of package elements, Go follows a simple rule that states that functions, variables, data types, structure fields, and so forth that begin with an uppercase letter are public, whereas functions, variables, types, and so on that begin with a lowercase letter are private. 
The same rule applies not only to the name of a struct variable but to the fields of a struct variable—in practice, this means that you can have a struct variable with both private and public fields. However, this rule does not affect package names, which are allowed to begin with either uppercase or lowercase letters.

## Go packages
Everything in Go is delivered in the form of packages. A Go package is a Go source file that begins with the package keyword, followed by the name of the package.

> Note that packages can have structure. For example, the net package has several subdirectories, named http, mail, rpc, smtp, textproto, and url, which should be imported as net/http, net/mail, net/rpc, net/smtp, net/textproto, and net/url, respectively.
    
There are external packages that can be imported using their full address and that should be downloaded on the local machine, before their first use.
Note that apart from the main package, Go packages are not autonomous programs and cannot be compiled into executable files on their own.



### Downloading Go packages
The go get command for downloading the cobra package is as follows:

```shell
$ go get github.com/spf13/cobra
$ tree ~/go -L 3
```
> The x path, which is displayed last, is used by the Go team.

Note that you can download the package without using https:// in its address. The results can be found inside the ~/go directory—the full path is ~/go/src/github.com/spf13/cobra. As the cobra package comes with a binary file that helps you structure and create command-line utilities, you can find that binary file inside ~/go/bin as cobra.

Basically, there are three main directories under ~/go with the following properties:

      The bin directory: This is where binary tools are placed.
      The pkg directory: This is where reusable packages are put. The darwin_amd64 directory, which can be found on macOS machines only, contains compiled versions of the installed packages. On a Linux machine, you can find a linux_amd64 directory instead of darwin_amd64.
      The src directory: This is where the source code of the packages is located. The underlying structure is based on the URL of the package you are looking for. So, the URL for the github.com/spf13/viper package is ~/go/src/github.com/spf13/viper. If a package is downloaded as a module, then it will be located under ~/go/pkg/mod.

> Starting with Go 1.16, go install is the recommended way of building and installing packages in module mode. The use of go get is deprecated, but this chapter uses go get because it's commonly used online and is worth knowing about. However, most of the chapters in this book use go mod init and go mod tidy for downloading external dependencies for your own source files.

If you want to upgrade an existing package, you should execute go get with the -u option. Additionally, if you want to see what is happening behind the scenes, add the -v option to the go get command


## Functions
The main elements of packages are functions,

> Type methods and functions are implemented in the same way and sometimes, the terms functions and type methods are used interchangeably.

functions must be as independent from each other as possible and must do one job (and only one job) well. So, if you find yourself writing functions that do multiple things, you might want to consider replacing them with multiple functions instead.
All function definitions begin with the func keyword, followed by the function's signature and its implementation, and that functions accept none, one, or more arguments and return none, one, or more values back. The single-most popular Go function is main(), which is used in every executable Go program—the main() function accepts no parameters and returns nothing, but it is the starting point of every Go program. Additionally, when the main() function ends, the entire program ends as well.


### Anonymous functions
Anonymous functions can be defined inline without the need for a name, and they are usually used for implementing things that require a small amount of code. In Go, a function can return an anonymous function or take an anonymous function as one of its arguments. Additionally, anonymous functions can be attached to Go variables. Note that anonymous functions are called lambdas in functional programming terminology. Similar to that, a closure is a specific type of anonymous function that carries or closes over variables that are in the same lexical scope as the anonymous function that was defined.
It is considered a good practice for anonymous functions to have a small implementation and a local focus. If an anonymous function does not have a local focus, then you might need to consider making it a regular function. When an anonymous function is suitable for a job, it is extremely convenient and makes your life easier; just do not use too many anonymous functions in your programs without having a good reason to

### Functions that return multiple values
functions can return multiple distinct values, which saves you from having to create a dedicated structure for returning and receiving multiple values from a function. However, if you have a function that returns more than 3 values, you should reconsider that decision and maybe redesign it to use a single structure or slice for grouping and returning the desired values as a single entity—this makes handling the returned values simpler and easier.

```go
package main
import "fmt"
func doubleSquare(x int) (int, int) {
    return x * 2, x * x
}
```
This function returns two int values, without the need for having separate variables to keep them—the returned values are created on the fly. Note the compulsory use of parentheses when a function returns more than one value.
The only difference between an anonymous function and a regular one is that the name of the anonymous function is func() and that there is no func keyword.

### The return values of a function can be named
Go allows you to name the return values of a Go function. Additionally, when such a function has a return statement without any arguments, the function automatically returns the current value of each named return value, in the order in which they were declared in the function signature.

```go
func minMax(x, y int) (min, max int) {
    if x > y {
        min = y
        max = x
        return min, max
	}
    min = x
    max = y
    return
}
```
both min and max are defined in the function signature and not in the function body.
This return statement is equivalent to return min, max, which is based on the function signature and the use of named return values.

### Functions that accept other functions as parameters
The signature of sort.Slice() is func Slice(slice interface{}, less func(i, j int) bool). This means the following:

      The sort.Slice() function does not return any data.
      The sort.Slice() function requires two arguments, a slice of type interface{} and another function—the slice variable is modified inside sort.Slice().
      The function parameter of sort.Slice() is named less and should have the func(i, j int) bool signature—there is no need for you to name the anonymous function. The name less is required because all function parameters should have a name.
      The i and j parameters of less are indexes of the slice parameter.

>You are not obliged to use an anonymous function in either sort.Slice() or sort.SliceIsSorted(). You can define a regular function with the required signature and use that. However, using an anonymous function is more convenient.

```go
package main
import (
    "fmt"
    "sort"
)
type Grades struct {
    Name    string
    Surname string
    Grade   int
}
func main() {
    data := []Grades{{"J.", "Lewis", 10}, {"M.", "Tsoukalos", 7},
        {"D.", "Tsoukalos", 8}, {"J.", "Lewis", 9}}
    isSorted := sort.SliceIsSorted(data, func(i, j int) bool {
        return data[i].Grade < data[j].Grade
    })
    if isSorted {
        fmt.Println("It is sorted!")
    } else {
        fmt.Println("It is NOT sorted!")
    }
    sort.Slice(data, func(i, j int) bool { return data[i].Grade < data[j].Grade })
    fmt.Println("By Grade:", data)
}
```

### Functions can return other functions

functions can also return anonymous functions, which can be handy when the returned function is not always the same but depends on the function's input or other external parameters.

```go
package main
import "fmt"
func funRet(i int) func(int) int {
    if i < 0 {
        return func(k int) int {
            k = -k
            return k + k
        }
    }
    return func(k int) int {
        return k * k
    }
}
```
The signature of funRet() declares that the function returns another function with the func(int) int signature.

```go
func main() {
    n := 10
	i := funRet(n)
    j := funRet(-4)
	fmt.Printf("%T\n", i)
    fmt.Printf("%T %v\n", j, j)
    fmt.Println("j", j, j(-5))
    // Same input parameter but DIFFERENT
    // anonymous functions assigned to i and j
    fmt.Println(i(10))
    fmt.Println(j(10))
}
```
The first statement prints (printf) the signature of the function whereas the second statement prints the function signature and its memory address. The last statement also returns the memory address of j, because j is a pointer to the anonymous function and the value of j(-5).
This makes Go a functional programming language, albeit not a pure one, and allows Go to benefit from the functional programming paradigm.
### Variadic functions
Variadic functions are functions that can accept a variable number of parameters
most functions found in the fmt package are variadic.
    Variadic functions use the pack operator, which consists of a ..., followed by a data type. So, for a variadic function to accept a variable number of int values, the pack operator should be ...int.
    The pack operator can only be used once in any given function.
    The variable that holds the pack operation is a slice and, therefore, is accessed as a slice inside the variadic function.
    The variable name that is related to the pack operator is always last in the list of function parameters.
    When calling a variadic function, you should put a list of values separated by , in the place of the variable with the pack operator or a slice with the unpack operator.

This list contains all the rules that you need to know in order to define and use variadic functions.
The pack operator can also be used with an empty interface. In fact, most functions in the fmt package use ...interface{} to accept a variable number of arguments of all data types.
the two data types ([]string and []interface{}) do not have the same representations in memory—this applies to all data types. In practice, this means that you cannot write os.Args... to pass each individual value of the os.Args slice to a variadic function.

On the other hand, if you just use os.Args, it will work, but this passes the entire slice as a single entity instead of its individual values! This means that the everything(os.Args, os.Args) statement works but does not do what you want.
The solution to this problem is converting the slice of strings—or any other slice—into a slice of interface{}. One way to do that is by using the code that follows:

```go
empty := make([]interface{}, len(os.Args[1:]))
for i, v := range os.Args {
    empty[i] = v
}
```
Now, you are allowed to use empty... as an argument to the variadic function. This is the only subtle point related to variadic functions and the pack operator.

> As there is no standard library function to perform that conversion for you, you have to write your own code. Note that the conversion takes time because the code must visit all slice elements. The more elements the slice has, the more time the conversion will take. This topic is also discussed at https://github.com/golang/go/wiki/InterfaceSlice.

you usually use a slice variable with the unpack operator.

```go
s := []float64{1.1, 2.12, 3.14}
sum = addFloats("Adding numbers...", s...)
```
Variadic functions come in very handy when you want to have an unknown number of parameters in a function.

### The defer keyword
The defer keyword postpones the execution of a function until the surrounding function returns.
Usually, defer is used in file I/O operations to keep the function call that closes an opened file close to the call that opened it, so that you do not have to remember to close a file that you have opened just before the function exits.
It is very important to remember that deferred functions are executed in last in, first out (LIFO) order after the surrounding function has been returned. Putting it simply, this means that if you defer function f1() first, function f2() second, and function f3() third in the same surrounding function, then when the surrounding function is about to return, function f3() will be executed first, function f2() will be executed second, and function f1() will be the last one to get executed.

```go
package main
import (
    "fmt"
)
func d1() {
    for i := 3; i > 0; i-- {
        defer fmt.Print(i, " ")
    }
}
func d2() {
	for i := 3; i > 0; i-- {
		defer func() {
			fmt.Print(i, " ")
		}()
	}
	fmt.Println()
}
func d3() {
	for i := 3; i > 0; i-- {
		defer func(n int) {
			fmt.Print(n, " ")
		}(i)
	}
}
func main() {
	d1()
	d2()
	fmt.Println()
	d3()
	fmt.Println()
}
```
In d2(), defer is attached to an anonymous function that does not accept any parameters. In practice, this means that the anonymous function should get the value of i on its own—this is dangerous because the current value of i depends on when the anonymous function is executed.

> The anonymous function is a closure, and that is why it has access to variables that would normally be out of scope.

In this case, the current value of i is passed to the anonymous function as a parameter that initializes the n function parameter. This means that there are no ambiguities about the value that i has.

After that, it should be clear that the best approach to using defer is the third one, which is exhibited in the d3() function, because you intentionally pass the desired variable in the anonymous function in an easy-to-read way.



## Developing your own packages
it is a best practice to use lowercase package names

Compiling a Go package can be done manually, if the package exists on the local machine, but it is also done automatically after you download the package from the internet, so there is no need to worry about it. Additionally, if the package you are downloading contains any errors, you will learn about them at downloading time.

> The main reason for compiling Go packages on your own is to check for syntax or other kinds of errors in your code. Additionally, you can build Go packages as plugins (https://golang.org/pkg/plugin/) or shared libraries.

### The init() function
Each Go package can optionally have a private function named init() that is automatically executed at the beginning of execution time—init() runs when the package is initialized at the beginning of program execution. The init() function has the following characteristics:

    init() takes no arguments.
    init() returns no values.
    The init() function is optional.
    The init() function is called implicitly by Go.
    You can have an init() function in the main package. In that case, init() is executed before the main() function. In fact, all init() functions are always executed prior to the main() function.
    A source file can contain multiple init() functions—these are executed in the order of declaration.
    The init() function or functions of a package are executed only once, even if the package is imported multiple times.
    Go packages can contain multiple files. Each source file can contain one or more init() functions.

The fact that the init() function is a private function by design means that it cannot be called from outside the package in which it is contained. Additionally, as the user of a package has no control over the init() function, you should think carefully before using an init() function in public packages or changing any global state in init().

There are some exceptions where the use of init() makes sense:

      For initializing network connections that might take time prior to the execution of package functions or methods.
      For initializing connections to one or more servers prior to the execution of package functions or methods.
      For creating required files and directories.
      For checking whether required resources are available or not.

### Order of execution
if a main package imports package A and package A depends on package B, then the following will take place:

      The process starts with main package.
      The main package imports package A.
      Package A imports package B.
      The global variables, if any, in package B are initialized.
      The init() function or functions of package B, if they exist, run. This is the first init() function that gets executed.
      The global variables, if any, in package A are initialized.
      The init() function or functions of package A, if there are any, run.
      The global variables in the main package are initialized.
      The init() function or functions of main package, if they exist, run.
      The main() function of the main package begins execution.
    
    
> Notice that if the main package imports package B on its own, nothing is going to happen because everything related to package B is triggered by package A. This is because package A imports package B first.


## A package for working with a database
When interacting with specific schemas and tables in your application, you usually create separate packages with all the database-related functions—this also applies to NoSQL databases.
Go offers a generic package (https://golang.org/pkg/database/sql/) for working with databases. However, each database requires a specific package that acts as the driver and allows Go to connect and work with this specific database.
The steps for creating the desired Go package are as follows:

      Downloading the necessary external Go packages for working with PostgreSQL.
      Creating package files.
      Developing the required functions.
      Using the Go package for developing utilities.
      Using CI/CD tools for automation (this is optional).

### Getting to know your database

There are two main Go packages for connecting to PostgreSQL—we are going to use the github.com/lib/pq package

> There is another Go package for working with PostgreSQL called jackc/pgx that can be found at https://github.com/JackC/pgx.

```yaml
version: '3'
services:
  postgres:
    image: postgres
    container_name: postgres
    environment:
      - POSTGRES_USER=mtsouk
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=master
    volumes:
      - ./postgres:/var/lib/postgresql/data/
    networks:
      - psql
    ports:
      - "5432:5432"
volumes:
  postgres:
networks:
  psql:
    driver: bridge
```
The default port number the PostgreSQL server listens to is 5432. As we connect to that PostgreSQL server from the same machine, the hostname that is going to be used is localhost or, if you prefer an IP address, 127.0.0.1. If you are using a different PostgreSQL server, then you should change the connection details in the code that follows accordingly.

> In PostgreSQL, a schema is a namespace that contains named database objects such as tables, views, and indexes. PostgreSQL automatically creates a schema called public for every new database.

```go
package main
import (
    "database/sql"
    "fmt"
    "os"
    "strconv"
    _ "github.com/lib/pq"
)
func main() {
	arguments := os.Args
	if len(arguments) != 6 {
		fmt.Println("Please provide: hostname port username password db")
		return
	}
	host := arguments[1]
	p := arguments[2]
	user := arguments[3]
	pass := arguments[4]
	database := arguments[5]

    // Port number SHOULD BE an integer
    port, err := strconv.Atoi(p)
	if err != nil {
		fmt.Println("Not a valid port number:", err)
		return
	}
	// connection string
	conn := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable", host, port, user, pass, database)

    // open PostgreSQL database
    db, err := sql.Open("postgres", conn)
	if err != nil {
		fmt.Println("Open():", err)
		return
	}
	defer db.Close()

    // Get all databases
    rows, err := db.Query(`SELECT "datname" FROM "pg_database"
    WHERE datistemplate = false`)
	if err != nil {
		fmt.Println("Query", err)
		return
	}
	for rows.Next() {
		var name string
		err = rows.Scan(&name)
		if err != nil {
			fmt.Println("Scan", err)
			return
		}
		fmt.Println("*", name)
	}
	defer rows.Close()
	// Get all tables from __current__ database
	query := `SELECT table_name FROM information_schema.tables WHERE 
        table_schema = 'public' ORDER BY table_name`
	rows, err = db.Query(query)
	if err != nil {
		fmt.Println("Query", err)
		return
	}
	// This is how you process the rows that are returned from SELECT
	for rows.Next() {
		var name string
		err = rows.Scan(&name)
		if err != nil {
			fmt.Println("Scan", err)
			return
		}
		fmt.Println("+T", name)
	}
	defer rows.Close()
}
```
The lib/pq package, which is the interface to the PostgreSQL database, is not used directly by the code. Therefore, you need to import the lib/pq package with _ in order to prevent the Go compiler from creating an error message related to importing a package and not "using" it.
This kind of import is usually because the imported package has side effects, such as registering itself as the database handler for the sql package:
In order to execute a SELECT query, you need to create it first. As the presented SELECT query contains no parameters, which means that it does not change based on variables, you can pass it to the Query() function and execute it. The live outcome of the SELECT query is kept in the rows variable, which is a cursor. You do not get all the results from the database, as a query might return millions of records, but you get them one by one—this is the point of using a cursor.
The previous code shows how to process the results of a SELECT query, which can be from nothing to lots of rows. As the rows variable is a cursor, you advance from row to row by calling Next(). After that, you need to assign the values returned from the SELECT query into Go variables, in order to use them. This happens with a call to Scan(), which requires pointer parameters. If the SELECT query returns multiple values, you need to put multiple parameters in Scan(). Lastly, you must call Close() with defer for the rows variable in order to close the statement and free various types of used resources.

### The design of the Go package
Remember that when working with a specific database and schema, you need to "include" the schema information in your Go code
The command-line utility for working with Postgres is called psql.
```shell
$ psql -h localhost -p 5432 -U mtsouk master < create_tables.sql
```
### The implementation of the Go package
The first element that you need in your Go package is one or more structures that can hold data from the database tables. Most of the times, you need as many structures as there are database tables
As the package communicates with PostgreSQL, we import the github.com/lib/pq package and we use _ in front of the package's path. As we discussed earlier, this happens because the imported package is registering itself as the database handler for the sql package, but it is not being directly used in the code. It is only being used through the sql package.

```go
package post05
import (
    "database/sql"
    "errors"
    "fmt"
    "strings"
    _ "github.com/lib/pq"
)
// Connection details
// should be accessible from outside the package, which means that their first letter should be in uppercase
var (
	Hostname = ""
	Port     = 2345
	Username = ""
	Password = ""
	Database = ""
)

type Userdata struct {
	ID          int
	Username    string
	Name        string
	Surname     string
	Description string
}
func openConnection() (*sql.DB, error) {
	// connection string
	conn := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable", Hostname, Port, Username, Password, Database)
	// open database
	db, err := sql.Open("postgres", conn)
	if err != nil {
		return nil, err
	}
	return db, nil
}
// The function returns the User ID of the username
// -1 if the user does not exist
func exists(username string) int {
	username = strings.ToLower(username)
	db, err := openConnection()
	if err != nil {
		fmt.Println(err)
		return -1
	}
	defer db.Close()
	userID := -1
	// This is where we define the query that shows whether the provided username exists in the database or not.
	statement := fmt.Sprintf(`SELECT "id" FROM "users" where username = '%s'`, username)
	rows, err := db.Query(statement)
	for rows.Next() {
		var id int
		// If the rows.Scan(&id) call is executed without any errors, then we know that a result has been returned, which is the desired user ID
		err = rows.Scan(&id)
		if err != nil {
			fmt.Println("Scan", err)
			return -1
		}
		userID = id
	}
	defer rows.Close()
	return userID
}
// AddUser adds a new user to the database
// Returns new User ID
// -1 if there was an error
func AddUser(d Userdata) int {
	d.Username = strings.ToLower(d.Username)
	db, err := openConnection()
	if err != nil {
		fmt.Println(err)
		return -1
	}
	defer db.Close()
	userID := exists(d.Username)
	if userID != -1 {
		fmt.Println("User already exists:", Username)
		return -1
	}
	insertStatement := `insert into "users" ("username") values ($1)`
	// This is how we construct a query that accepts parameters. The presented query requires one value that is named $1.
	_, err = db.Exec(insertStatement, d.Username)
	if err != nil {
		fmt.Println(err)
		return -1
	}
	userID = exists(d.Username)
	if userID == -1 {
		return userID
	}
	insertStatement = `insert into "userdata" ("userid", "name", "surname", "description") values ($1, $2, $3, $4)`
	_, err = db.Exec(insertStatement, userID, d.Name, d.Surname, d.Description)
	if err != nil {
		fmt.Println("db.Exec()", err)
		return -1
	}
	return userID
}
// DeleteUser deletes an existing user
func DeleteUser(id int) error {
	db, err := openConnection()
	if err != nil {
		return err
	}
	defer db.Close()
	// Does the ID exist?
	statement := fmt.Sprintf(`SELECT "username" FROM "users" where id = %d`, id)
	rows, err := db.Query(statement)
	var username string
	for rows.Next() {
		err = rows.Scan(&username)
		if err != nil {
			return err
		}
	}
	defer rows.Close()
	if exists(username) != id {
		return fmt.Errorf("User with ID %d does not exist", id)
	}
	// Delete from Userdata
	deleteStatement := `delete from "userdata" where userid=$1`
	_, err = db.Exec(deleteStatement, id)
	if err != nil {
		return err
	}
	// Delete from Users
	deleteStatement = `delete from "users" where id=$1`
	_, err = db.Exec(deleteStatement, id)
	if err != nil {
		return err
	}
	return nil
}
func ListUsers() ([]Userdata, error) {
    Data := []Userdata{}
    db, err := openConnection()
    if err != nil {
        return Data, err
    }
    defer db.Close()
	rows, err := db.Query(`SELECT  
        "id","username","name","surname","description"
        FROM "users","userdata"
        WHERE users.id = userdata.userid`)
	if err != nil {
		return Data, err
	}
	for rows.Next() {
		var id int
		var username string
		var name string
		var surname string
		var description string
		err = rows.Scan(&id, &username, &name, &surname, &description)
		temp := Userdata{ID: id, Username: username, Name: name, Surname: surname, Description: description}
		Data = append(Data, temp)
		if err != nil {
			return Data, err
		}
	}
	defer rows.Close()
	return Data, nil
}
// UpdateUser is for updating an existing user
func UpdateUser(d Userdata) error {
	db, err := openConnection()
	if err != nil {
		return err
	}
	defer db.Close()
	userID := exists(d.Username)
	if userID == -1 {
		return errors.New("User does not exist")
	}

    d.ID = userID
	updateStatement := `update "userdata" set "name"=$1, "surname"=$2, "description"=$3 where "userid"=$4`
	_, err = db.Exec(updateStatement, d.Name, d.Surname, d.Description, d.ID)
	if err != nil {
		return err
	}
	return nil
}
```
> During development, I include many fmt.Println() statements in the package code for debugging purposes. However, I have removed most of them in the final version of the Go package and replaced them with error values. These error values are passed to the program that uses the functionality of the package, which is responsible for deciding what to do with the error messages and error conditions. You can also use logging for this—the output can go to standard output or even /dev/null when not needed.

### Testing the Go package
> you should not forget to download the latest version of that external package using go get or go get -u.

## Modules
A Go module is like a Go package with a version—however, Go modules can consist of multiple packages. Go uses semantic versioning for versioning modules. This means that versions begin with the letter v, followed by the major.minor.patch version numbers. Therefore, you can have versions such as v1.0.0, v1.0.5, and v2.0.2. The v1, v2, and v3 parts signify the major version of a Go package that is usually not backward compatible. This means that if your Go program works with v1, it will not necessarily work with v2 or v3—it might work, but you cannot count on it. The second number in a version is about features. Usually, v1.1.0 has more features than v1.0.2 or v1.0.0, while being compatible with all older versions. Lastly, the third number is just about bug fixes without having any new features. Note that semantic versioning is also used for Go versions.
If you want to learn more about modules, visit and read https://blog.golang.org/using-go-modules, which has five parts, as well as https://golang.org/doc/modules/developing. Just remember that a Go module is similar but not identical to a regular Go package with a version, and that a module can consist of multiple packages.

## Creating better packages
Here are several good rules to follow to create high-class Go packages:

      The first unofficial rule of a successful package is that its elements must be connected in some way. Thus, you can create a package for supporting cars, but it would not be a good idea to create a single package for supporting cars and bicycles and airplanes. Put simply, it is better to split the functionality of a package unnecessarily into multiple packages than to add too much functionality to a single Go package.
      A second practical rule is that you should use your own packages first for a reasonable amount of time before giving them to the public. This helps you discover silly bugs and make sure that your packages operate as expected. After that, give them to some fellow developers for additional testing before making them publicly available. Additionally, you should always write tests for any package you intend others to use.
      Next, make sure your package has a clear and useful API so that any consumer can be productive with it quickly.
      Try and limit the public API of your packages to only what is absolutely necessary. Additionally, give your functions descriptive but not very long names.
      Interfaces, and in future Go versions, generics, can improve the usefulness of your functions, so when you think it is appropriate, use an interface instead of a single type as a function parameter or return type.
      When updating one of your packages, try not to break things and create incompatibilities with older versions unless it is absolutely necessary.
      When developing a new Go package, try to use multiple files in order to group similar tasks or concepts.
      Do not create a package that already exists from scratch. Make changes to the existing package and maybe create your own version of it.
      Nobody wants a Go package that prints logging information on the screen. It would be more professional to have a flag for turning on logging when needed. The Go code of your packages should be in harmony with the Go code of your programs. This means that if you look at a program that uses your packages and your function names stand out in the code in a bad way, it would be better to change the names of your functions. As the name of a package is used almost everywhere, try to use concise and expressive package names.
      It is more convenient if you put new Go type definitions near where they are used the first time because nobody, including yourself, wants to search source files for definitions of new data types.
      Try to create test files for your packages, because packages with test files are considered more professional than ones without them; small details make all the difference and give people confidence that you are a serious developer! Notice that writing tests for your packages is not optional and that you should avoid using packages that do not include tests. You will learn more about testing in Chapter 11, Code Testing and Profiling.

that the actual Go code in a package should be bug-free, the next most important element of a successful package is its documentation, as well as some code examples that clarify its use and showcase the idiosyncrasies of the functions of the package.

## Generating documentation
Go follows a simple rule regarding documentation: in order to document a function, a method, a variable, or even the package itself, you can write comments, as usual, that should be located directly before the element you want to document, without any empty lines in between. You can use one or more single-line comments, which are lines beginning with //, or block comments, which begin with /* and end with */—everything in-between is considered a comment.

> It is highly recommended that each Go package you create has a block comment preceding the package declaration that introduces developers to the package, and also explains what the package does.

```go
/*
The package works on 2 tables on a PostgreSQL data base server.
The names of the tables are:
    * Users
    * Userdata
The definitions of the tables in the PostgreSQL server are:
    CREATE TABLE Users (
        ID SERIAL,
        Username VARCHAR(100) PRIMARY KEY
    );
    CREATE TABLE Userdata (
        UserID Int NOT NULL,
        Name VARCHAR(100),
        Surname VARCHAR(100),
        Description VARCHAR(200)
    );
    This is rendered as code
This is not rendered as code
*/
package document
// BUG(1): Function ListUsers() not working as expected
// BUG(2): Function AddUser() is too slow
import (
    "database/sql"
    "fmt"
    "strings"
)
/*
This block of global variables holds the connection details to the Postgres server
    Hostname: is the IP or the hostname of the server
    Port: is the TCP port the DB server listens to
    Username: is the username of the database user
    Password: is the password of the database user
    Database: is the name of the Database in PostgreSQL
*/
var (
	Hostname = ""
	Port     = 2345
	Username = ""
	Password = ""
	Database = ""
)
// The Userdata structure is for holding full user data
// from the Userdata table and the Username from the
// Users table
type Userdata struct {
	ID          int
	Username    string
	Name        string
	Surname     string
	Description string
}

// openConnection() is for opening the Postgres connection
// in order to be used by the other functions of the package.
func openConnection() (*sql.DB, error) {
	var db *sql.DB
	return db, nil
}

// The function returns the User ID of the username
// -1 if the user does not exist
func exists(username string) int {
	fmt.Println("Searching user", username)
	return 0
}

// AddUser adds a new user to the database
//
// Returns new User ID
// -1 if there was an error
func AddUser(d Userdata) int {
	d.Username = strings.ToLower(d.Username)
	return -1
}
/*
   DeleteUser deletes an existing user if the user exists.
   It requires the User ID of the user to be deleted.
*/
func DeleteUser(id int) error {
	fmt.Println(id)
	return nil
}

// ListUsers lists all users in the database
// and returns a slice of Userdata.
func ListUsers() ([]Userdata, error) {
	// Data holds the records returned by the SQL query
	Data := []Userdata{}
	return Data, nil
}

// UpdateUser is for updating an existing user
// given a Userdata structure.
// The user ID of the user to be updated is found
// inside the function.
func UpdateUser(d Userdata) error {
	fmt.Println(d)
	return nil
}
```
This is the first block of documentation that is located right before the name of the package. This is the appropriate place to document the functionality of the package, as well as other essential information.
Other information that you can put at the beginning of a package is the author, the license, and the version of the package.
If a line in a block comment begins with a tab, then it is rendered differently in the graphical output, which is good for differentiating between various kinds of information in the documentation:
The BUG keyword is special when writing documentation. Go knows that bugs are part of the code and therefore should be documented as well. You can write any message you want after a BUG keyword, and you can place them anywhere you want—preferably close to the bugs they describe.
The github.com/lib/pq package was removed from the import block to make the file size smaller.
The previous code shows a way of documenting lots of variables at once—in this case, global variables. The good thing with this way is that you do not have to put a comment before each global variable and make the code less readable.
When documenting a function, it is good to begin the first line of the comments with the function name.
In this case, we will explain the return values of the exists() function as they have a special meaning.
When you request the documentation of the Userdata structure, Go automatically presents the functions that use Userdata as input or output, or both.
There are two ways to see the documentation of the package. The first one involves using go get, which also means creating a GitHub repository of the package, as we did with post05. However, as this is for testing purposes, we are going to do things the easy way: we are going to copy it in ~/go/src and access it from there. As the package is called document, we are going to create a directory with the same name inside ~/go/src. After that, we are going to copy document.go in ~/go/src/document and we are done—for more complex packages, the process is going to be more complex as well. In such cases, it would be better to go get the package from its repository.
Either way, the go doc command is going to work just fine with the document package:

```shell
$ go doc document
package document // import "document"
The package works on 2 tables on a PostgreSQL data base server.
The names of the tables are:
    * Users
    * Userdata
The definitions of the tables in the PostgreSQL server are:
        CREATE TABLE Users (
            ID SERIAL,
            Username VARCHAR(100) PRIMARY KEY
        );
        CREATE TABLE Userdata (
            UserID Int NOT NULL,
            Name VARCHAR(100),
            Surname VARCHAR(100),
            Description VARCHAR(200)
        );
        This is rendered as code
        This is not rendered as code
var Hostname = "" ...
func AddUser(d Userdata) int
func DeleteUser(id int) error
func UpdateUser(d Userdata) error
type Userdata struct{ ... }
    func ListUsers() ([]Userdata, error)
BUG: Function ListUsers() not working as expected
BUG: Function AddUser() is too slow
```

```shell
$ go doc document ListUsers
package document // import "document"
func ListUsers() ([]Userdata, error)
    ListUsers lists all users in the database and returns a slice of Userdata.
```
> Go automatically puts the bugs at the end of the text and graphical output.

In my personal opinion, rendering the documentation is much better when using the graphical interface, making it better when you do not know what you are looking for. On the other hand, using go doc from the command line is much faster and allows you to process the output using traditional UNIX command-line tools.


## GitLab Runners and Go
### The initial version of the configuration file

The name of the configuration file is .gitlab-ci.yml and is a YAML file that should be located in the root directory of the GitLab repository. This initial version of the .gitlab-ci.yml configuration file compiles hw.go and creates a binary file, which is executed in a different stage than the one it was created in.


```shell
$ cat .gitlab-ci.yml
image: golang:1.15.7
stages:
    - download
    - execute
compile:
    stage: download
    script:
        - echo "Getting System Info"
        - uname -a
        - mkdir bin
        - go version
        - go build -o ./bin/hw hw.go
    artifacts:
        paths:
            - bin/
execute:
    stage: execute
    script:
        - echo "Executing Hello World!"
        - ls -l bin
        - ./bin/hw
```
The important thing about the previous configuration file is that we are using an image that comes with Go already installed, which saves us from having to install it from scratch and allows us to specify the Go version we want to use.

### The final version of the configuration file gitlab pipeline

```yaml
image: golang:1.15.7
stages:
    - download
    - execute
compile:
    stage: download
    script:
        - echo "Compiling usePost05.go"
        - mkdir bin
        - go get -v -d ./...
        # The go get -v -d ./... command is the Go way of downloading all the package dependencies of a project. After that, you are free to build your project and generate your executable file:
        - go build -o ./bin/usePost05 usePost05.go
    artifacts:
        paths:
            - bin/

    The bin directory, along with its contents, will be available to the execute state:
    execute:
    stage: execute
    script:
        - echo "Executing usePost05"
        - ls -l bin
        - ./bin/usePost05
```
As we do not have a PostgreSQL instance available, we cannot try interacting with PostgreSQL, but we can execute usePost05.go and see the values of the Hostname and Port global variables.

## GitHub Actions and Go
In order to set up GitHub Actions, we need to create a directory named .github and then create another directory named workflows in it. The .github/workflows directory contains YAML files with the pipeline configuration.

### Storing secrets in GitHub
In your GitHub repository, go to the Settings tab and select Secrets from the left column. You will see your existing secrets, if any, and an Add new secret link, which you need to click on. Do this process twice to store your Docker Hub username and password.

### The final version of the configuration file github action
The final version of the configuration file compiles the Go code, puts it in a Docker image, as described by Dockerfile, connects with Docker Hub using the specified credentials, and pushes the Docker image to Docker Hub using the provided data. This is a very common way of automation when creating Docker images. The contents of go.yml is as follows:

```yaml
name: Go + PostgreSQL
on: [push]

    This line in the configuration file specifies that this pipeline is triggered on push operations only.
    jobs:
  build:
    runs-on: ubuntu-18.04
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2
    - uses: actions/setup-go@v2
      with:
        stable: 'false'
        go-version: '1.15.7'
    - name: Publish Docker Image
      env:
         USERNAME: ${{ secrets.USERNAME }}
         PASSWORD: ${{ secrets.PASSWORD }}
         IMAGE_NAME: gopost
      run: |
        docker images
        docker build -t "$IMAGE_NAME" .
        docker images
        echo "$PASSWORD" | docker login --username "$USERNAME" --password-stdin
        docker tag "${IMAGE_NAME}" "$USERNAME/${IMAGE_NAME}:latest"
        docker push "$USERNAME/${IMAGE_NAME}:latest"
        echo "* Running Docker Image"
        docker run ${IMAGE_NAME}:latest

```
This time, most of the work is performed by the docker build command because the Go executable is built inside a Docker image.
## Versioning utilities
One of the most difficult tasks is to automatically and uniquely version command-line utilities, especially when using a CI/CD system.
> You can apply the same technique to GitLab—just search for the available GitLab variables and values and choose one that fits your needs.

What we need to do is tell the Go linker that we are going to define the value of the VERSION variable. This happens with the help of the -ldflags flag, which stands for linker flags—this passes values to the cmd/link package, which allows us to change values in imported packages at build time. The -X value that is used requires a key/value pair, where the key is a variable name and the value is the value that we want to set for that key. In our case, the key has the main.Variable form because we change the value of a variable in the main package. As the name of the variable in gitVersion.go is VERSION, the key is main.VERSION.
But first, we need to decide on the GitHub value that we are going to use as the version string. The git rev-list HEAD command returns a full list of commits for the current repository from the latest to the oldest. We only need the last one—the most recent—which we can get using git rev-list -1 HEAD or git rev-list HEAD | head -1. So, we need to assign that value to an environment variable and pass that environment variable to the Go compiler. As this value changes each time you make a commit and you always want to have the latest value, you should reevaluate it each time you execute go build—this will be shown in a while.
In order to provide gitVersion.go with the value of the desired environment variable, we should execute it as follows:

```go
package main
import (
    "fmt"
    "os"
)
// VERSION is the variable that is going to be set at runtime using the Go linker.
var VERSION string
func main() {
    if len(os.Args) == 2 {
        if os.Args[1] == "version" {
            fmt.Println("Version:", VERSION)
        }
    }
}
```

```shell
$ export VERSION=$(git rev-list -1 HEAD)
$ go build -ldflags "-X main.VERSION=$VERSION" gitVersion.go
```

## Exercises
Can you write a function that sorts three int values? Try to write two versions of the function: one with named returned values and another without named return values. Which one do you think is better?
Rewrite the getSchema.go utility so that it works with the jackc/pgx package.
Rewrite the getSchema.go utility so that it works with MySQL databases.
Use GitLab CI/CD to push Docker images to Docker Hub.

## Additional resources
New module changes in Go 1.16: [https://blog.golang.org/go116-module-changes](https://blog.golang.org/go116-module-changes)
How do you structure your Go apps? Talk by Kat Zien from GopherCon UK 2018: [https://www.youtube.com/watch?v=1rxDzs0zgcE](https://www.youtube.com/watch?v=1rxDzs0zgcE)
PostgreSQL: [https://www.postgresql.org/](https://www.postgresql.org/)
PostgreSQL Go package: [https://github.com/lib/pq](https://github.com/lib/pq)
PostgreSQL Go package: [https://github.com/jackc/pgx](https://github.com/jackc/pgx)
HashiCorp Vault: [https://www.vaultproject.io/](https://www.vaultproject.io/)
The documentation of database/sql: [https://golang.org/pkg/database/sql/](https://golang.org/pkg/database/sql/)
You can learn more about GitHub Actions environment variables at [https://docs.github.com/en/actions/reference/environment-variables](https://docs.github.com/en/actions/reference/environment-variables)
GitLab CI/CD variables: [https://docs.gitlab.com/ee/ci/variables/](https://docs.gitlab.com/ee/ci/variables/)
The documentation of the cmd/link package: [https://golang.org/cmd/link/](https://golang.org/cmd/link/)
golang.org moving to go.dev: [https://go.dev/blog/tidy-web](https://go.dev/blog/tidy-web)