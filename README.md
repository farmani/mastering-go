Mastering Go
==========

# Contents

- [Chapter 1: A Quick Introduction to Go](chapter-1.md#chapter-1-a-quick-introduction-to-go)
    * [Introducing Go](chapter-1.md#introducing-go)
        + [The advantages of Go](chapter-1.md#the-advantages-of-go)
        + [The go doc and godoc utilities](chapter-1.md#the-go-doc-and-godoc-utilities)
    * [Hello World!](chapter-1.md#hello-world)
        + [Introducing functions](chapter-1.md#introducing-functions)
            + [Introducing packages](chapter-1.md#introducing-packages)
    * [Running Go code](chapter-1.md#running-go-code)
    * [Important characteristics of Go](chapter-1.md#important-characteristics-of-go)
        + [Defining and using variables](chapter-1.md#defining-and-using-variables)
        + [Printing variables](chapter-1.md#printing-variables)
    * [Controlling program flow](chapter-1.md#controlling-program-flow)
    * [Iterating with for loops and range](chapter-1.md#iterating-with-for-loops-and-range)
    * [Getting user input](chapter-1.md#getting-user-input)
        + [Reading from standard input](chapter-1.md#reading-from-standard-input)
        + [Working with command-line arguments](chapter-1.md#working-with-command-line-arguments)
    * [Using error variables to differentiate between input types](chapter-1.md#using-error-variables-to-differentiate-between-input-types)
    * [Understanding the Go concurrency model](chapter-1.md#understanding-the-go-concurrency-model)
    * [Developing the which(1) utility in Go](chapter-1.md#developing-the-which1-utility-in-go)
    * [Logging information](chapter-1.md#logging-information)
        + [log.Fatal() and log.Panic()](chapter-1.md#logfatal-and-logpanic)
        + [Writing to a custom log file](chapter-1.md#writing-to-a-custom-log-file)
        + [Printing line numbers in log entries](chapter-1.md#printing-line-numbers-in-log-entries)
    * [Overview of Go generics](chapter-1.md#overview-of-go-generics)
    * [Developing a basic phone book application](chapter-1.md#developing-a-basic-phone-book-application)
    * [Exercises](chapter-1.md#exercises)
- [Chapter 2: Basic Go Data Types](chapter-2.md#chapter-2-basic-go-data-types)
    * [The error data type](chapter-2.md#the-error-data-type)
    * [Numeric data types](chapter-2.md#numeric-data-types)
    * [Non-numeric data types](chapter-2.md#non-numeric-data-types)
        + [Strings, Characters, and Runes](chapter-2.md#strings-characters-and-runes)
        + [Times and dates](chapter-2.md#times-and-dates)
    * [Go constants](chapter-2.md#go-constants)
        + [The constant generator iota](chapter-2.md#the-constant-generator-iota)
    * [Grouping similar data](chapter-2.md#grouping-similar-data)
        + [Arrays](chapter-2.md#arrays)
        + [Slices](chapter-2.md#slices)
    * [Pointers](chapter-2.md#pointers)
    * [Generating random numbers](chapter-2.md#generating-random-numbers)
    * [Exercises](chapter-2.md#exercises)
- [Chapter 3: Composite Data Types](chapter-3.md#chapter-3-composite-data-types)
    * [Maps](chapter-3.md#maps)
        + [Storing to a nil map](chapter-3.md#storing-to-a-nil-map)
        + [Iterating over maps](chapter-3.md#iterating-over-maps)
    * [Structures](chapter-3.md#structures)
        + [Defining new structures](chapter-3.md#defining-new-structures)
        + [Using the new keyword](chapter-3.md#using-the-new-keyword)
        + [Slices of structures](chapter-3.md#slices-of-structures)
    * [Regular expressions and pattern matching](chapter-3.md#regular-expressions-and-pattern-matching)
        + [About Go regular expressions](chapter-3.md#about-go-regular-expressions)
        + [Matching names and surnames](chapter-3.md#matching-names-and-surnames)
        + [Matching integers](chapter-3.md#matching-integers)
        + [Matching the fields of a record](chapter-3.md#matching-the-fields-of-a-record)
        + [Working with CSV files](chapter-3.md#working-with-csv-files)
        + [Adding an index](chapter-3.md#adding-an-index)
        + [The improved version of the phone book application](chapter-3.md#the-improved-version-of-the-phone-book-application)
    * [Exercises](chapter-3.md#exercises)
    * [Additional resources](chapter-3.md#additional-resources)
- [Reflection and Interfaces](chapter-4.md#reflection-and-interfaces)
    * [Reflection](chapter-4.md#reflection)
        + [Learning the internal structure of a Go structure](chapter-4.md#learning-the-internal-structure-of-a-go-structure)
        + [Changing structure values using reflection](chapter-4.md#changing-structure-values-using-reflection)
        + [The three disadvantages of reflection](chapter-4.md#the-three-disadvantages-of-reflection)
    * [Type methods](chapter-4.md#type-methods)
        + [Creating type methods](chapter-4.md#creating-type-methods)
        + [Using type methods](chapter-4.md#using-type-methods)
        * [Interfaces](chapter-4.md#interfaces)
        + [The sort.Interface interface](chapter-4.md#the-sortinterface-interface)
        + [The empty interface](chapter-4.md#the-empty-interface)
        + [Type assertions and type switches](chapter-4.md#type-assertions-and-type-switches)
        + [The map[string]interface{} map](chapter-4.md#the-mapstringinterface-map)
        + [The error data type](chapter-4.md#the-error-data-type)
        + [Writing your own interfaces](chapter-4.md#writing-your-own-interfaces)
    * [Working with two different CSV file formats](chapter-4.md#working-with-two-different-csv-file-formats)
    * [Object-oriented programming in Go](chapter-4.md#object-oriented-programming-in-go)
    * [Updating the phone book application](chapter-4.md#updating-the-phone-book-application)
    * [Exercises](chapter-4.md#exercises)
    * [Additional resources](chapter-4.md#additional-resources)
- [Go Packages and Functions](chapter-5.md#go-packages-and-functions)
    * [Go packages](chapter-5.md#go-packages)
        + [Downloading Go packages](chapter-5.md#downloading-go-packages)
    * [Functions](chapter-5.md#functions)
        + [Anonymous functions](chapter-5.md#anonymous-functions)
        + [Functions that return multiple values](chapter-5.md#functions-that-return-multiple-values)
        + [The return values of a function can be named](chapter-5.md#the-return-values-of-a-function-can-be-named)
        + [Functions that accept other functions as parameters](chapter-5.md#functions-that-accept-other-functions-as-parameters)
        + [Functions can return other functions](chapter-5.md#functions-can-return-other-functions)
        + [Variadic functions](chapter-5.md#variadic-functions)
        + [The defer keyword](chapter-5.md#the-defer-keyword)
    * [Developing your own packages](chapter-5.md#developing-your-own-packages)
        + [The init() function](chapter-5.md#the-init---function)
        + [Order of execution](chapter-5.md#order-of-execution)
    * [A package for working with a database](chapter-5.md#a-package-for-working-with-a-database)
        + [Getting to know your database](chapter-5.md#getting-to-know-your-database)
        + [The design of the Go package](chapter-5.md#the-design-of-the-go-package)
        + [The implementation of the Go package](chapter-5.md#the-implementation-of-the-go-package)
        + [Testing the Go package](chapter-5.md#testing-the-go-package)
    * [Modules](chapter-5.md#modules)
    * [Creating better packages](chapter-5.md#creating-better-packages)
    * [Generating documentation](chapter-5.md#generating-documentation)
    * [GitLab Runners and Go](chapter-5.md#gitlab-runners-and-go)
        + [The initial version of the configuration file](chapter-5.md#the-initial-version-of-the-configuration-file)
        + [The final version of the configuration file gitlab pipeline](chapter-5.md#the-final-version-of-the-configuration-file-gitlab-pipeline)
    * [GitHub Actions and Go](chapter-5.md#github-actions-and-go)
        + [Storing secrets in GitHub](chapter-5.md#storing-secrets-in-github)
        + [The final version of the configuration file github action](chapter-5.md#the-final-version-of-the-configuration-file-github-action)
    * [Versioning utilities](chapter-5.md#versioning-utilities)
    * [Exercises](chapter-5.md#exercises)
    * [Additional resources](chapter-5.md#additional-resources)