# Functions

## Declaring and Calling Functions

```go
func div(numerator int, denomerator int) int {
    return numerator / denominator
}
```

When you have multiple input parameters of the same type, you can write your input parameters like this:

```go
func div(numerator, denominator int) int {}
```

### Simulating Name and Optional Parameters

Go doesn't have named and optional input parameters. If you want to emulate named and optional parameters, define a
struct that has fields that match the desired parameters.

```go
type User struct {
    FirstName string
    LastName string
    Age int
}

func create(user User) error {
    // do something here
}

func main() {
    create(User {
        LastName: "Patel",
        Age: 50,
    })
}
```

In practice, not having named and optional parameters isn’t a limitation. A function shouldn’t have more than a few
parameters, and named and optional parameters are mostly useful when a function has many inputs. If you find yourself in
that situation, your function is quite possibly too complicated.

### Variadic Input Parameters and Slices

Go supports **variadic parameters**. The variadic parameter must be the last (or only) parameter in the input parameter
list. You indicate it with three dots
(...) before the type. The variable that’s created within the function is a slice of the specified type.

```go
func add(base int, numbers ...int) []int {
    out := make([]int, 0, len(numbers))
    for _, v := range numbers {
        out = append(out, base+v)
    }
    return out
}
```

And now we’ll call it a few different ways:

```go
func main() {
    add(3)
    add(3, 2)
    add(3, 2, 4, 6, 8)
    a := []int{4, 3}
    add(3, a...)
    add(3, []int{1, 2, 3, 4, 5}...)
}
```

As you can see, you can supply however many values you want for the variadic parameter, or no values at all. Since the
variadic parameter is converted to a slice, you can supply a slice as the input. However, you must put three dots (...)
after the variable or slice literal.

### Multiple Return Values

```go
func div(numerator int, denominator int) (int, int, error) {
    if denominator == 0 {
        return 0, 0, errors.New("cannot divide by zero")
    }
    return numerator / denominator, numerator % denominator, nil
}
```

If the function completes successfully, we return nil for the error’s value. By convention, the error is always the
last (or only) value returned from a function.

Calling our updated function looks like this:

```go
func main() {
    quotient, remainder, err := div(5, 2)
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
}
```

### Multiple Return Values Are Multiple Values

If you are familiar with Python, you might think that multiple return values are like Python functions returning a tuple
that’s optionally destructured if the tuple’s values are assigned to multiple variables. Example 5-2 shows some sample
code run in the Python interpreter.

```python
def div(n,d):
    if d == 0:
        raise Exception("cannot divide by zero")
    return n / d, n % d
  
>>> v = div(5,2)
>>> v
(2.5, 1)

>>> quotient, remainder = div(5,2)
>>> quotient
2.5
>>> remainder
1
```

That’s not how Go works. You must assign each value returned from a function. If you try to assign multiple return
values to one variable, you get a compile-time error.

### Ignoring Returned Values

If a function returns multiple values, but you don’t need to read one or more of the values, assign the unused values to
the name _. For example, if we weren't going to read remainder, we would write the assignment as:

```go
quotient, _, error := div(5, 2)

// Implicitly ignore all of the return values
div(5,2)
```

### Named Return Values

You must surround named return values with parentheses, even if there is only a single return value.

```go
func div(n int, d int) (quotient int, remainder int, err error) {
    if d == 0 {
        err = errors.New("cannot divide by zero")
        return quotient, remainder, err
    }
    quotient, remainder = n/d, n%denominator
    return quotient, remainder, err
}
```

Named return values are initialized to their zero values when created.

> If you only want to name some of the return values, you can do so
> by using _ as the name for any return values you want to remain
> nameless.

While some developers like to use named return parameters because they provide additional documentation, they do have
some potential corner cases:

- Shadowing
- No guarantee to be returned by function

There is one situation where named return parameters are essential; [defer](#defer)

### Blank Returns — Never Use These!

If you have [named return values](#named-return-values), you can just write return without specifying the values that
are returned. This returns the last values assigned to the named return values

```go
func div(n, d int) (quotient int, remainder int, err error) {
    if d == 0 {
        err = errors.New("cannot divide by zero")
        return
    }
    quotient, remainder = n/d, n%d
    return
}
```

Never use a blank (sometimes called _naked_) return. It can make it very confusing to figure out what value is actually
returned

## Functions Are Values

The type of a function is built out of the keyword `func` and the types of the parameters and return values. This
combination is called the _signature_ of the function. Any function that has the exact same number and types of
parameters and return values meets the type signature.

```go
func add(i int, j int) int { return i + j }
func sub(i int, j int) int { return i - j }
func mul(i int, j int) int { return i * j }
func div(i int, j int) int { return i / j }


var opMap = map[string]func(int, int) int{
    "+": add,
    "-": sub,
    "*": mul,
    "/": div,
}
```

let’s try out our calculator with a few expressions:

```go
func main() {
    expressions := [][]string{
        []string{"2", "+", "3"},
    }

    for _, expression := range expressions {
        p1, err := strconv.Atoi(expression[0])
        if err != nil {
            fmt.Println(err)
            continue
        }
      
        p2, err := strconv.Atoi(expression[2])
        if err != nil {
            fmt.Println(err)
            continue
        }
      
        op := expression[1]
        opFunc, ok := opMap[op]
        if !ok {
            fmt.Println("unsupported operator:", op)
            continue
        }
      
        // Calling a function in a variable
        // looks just like calling a function directly.
        result := opFunc(p1, p2)
        fmt.Println(result)
    }
}
```

We’re using the `strconv.Atoi` function in the standard library to convert a string to an int.

### Function Type Declarations

```go
type opFuncType func(int, int) int
```

We can then rewrite the opMap declaration to look like this:

```go
var opMap = map[string]opFuncType {
    // same as before
}
```

What’s the advantage of declaring a function type?

- Documentation: It’s useful to give something a name if you are going to refer to it multiple times.
- “Function Types Are a Bridge to Interfaces” on page 154.

### Anonymous Functions

Not only can you assign functions to variables, you can also define new functions within a function and assign them to
variables. These inner functions are anonymous functions; they don’t have a name. You don’t have to assign them to a
variable, either. You can write them inline and call them immediately.

```go
func main() {
    for i := 0; i < 5; i++ {
        func(j int) {
            fmt.Println("printing", j)
        }(i)
    }
}
```

There are two situations where declaring anonymous functions without assigning them to variables is useful:

- [defer](#defer) statements
- Launching goroutines (page 203).

## Closures

[Closure](https://www.practical-go-lessons.com/chap-24-anonymous-functions-and-closures#closures) is a computer science
word that means that anonymous functions declared inside of functions are able to access and modify variables declared
in the outer function.

### Passing Functions as Parameters

Passing functions as parameters to other functions is often useful for performing different operations on the same kind
of data. Let’s see how we use closures to sort the same data different ways.

```go
// sort package
func Slice(slice interface{}, less func(i, j int) bool)

sort.Slice(people, func(i int, j int) bool {
    return people[i].LastName < people[j].LastName
})
```

In computer science terms, people is _captured_ by the closure.

### Returning Functions from Functions

```go
func makeMult(base int) func(int) int {
    return func(factor int) int {
        return base * factor
    }
}

func main() {
    twoBase := makeMult(2)
    fmt.Println(twoBase(3)) // 6
}
```

If you spend any time with programmers who use functional programming languages like Haskell, you might hear the term
_higher-order functions_. That’s a very fancy way to say that a function has a function for an input parameter or a
return value.

## defer

Programs often create temporary resources, like files or network connections, that need to be cleaned up. This cleanup
has to happen, no matter how many exit points a function has, or whether a function completed successfully or not. We
use the `defer` keyword, followed by a function or method call.

```go
func main() {
    if len(os.Args) < 2 {
        // print a message and exit the program
        log.Fatal("no file specified")
    }
    f, err := os.Open(os.Args[1])
    if err != nil {
        log.Fatal(err)
    }

    defer f.Close()
    data := make([]byte, 2048)
    for {
        count, err := f.Read(data)
        os.Stdout.Write(data[:count])
        if err != nil {
            if err != io.EOF {
                log.Fatal(err)
            }
            break
        }
    }
}
```

`os.Args`: a slice that contains the name of the program launched and the arguments passed to it.

We need to close the file after we use it, no matter how we exit the function. Normally, a function call runs
immediately, but defer delays the invocation until the surrounding function exits.

You can defer multiple closures in a Go function. They run in last-in-first-out order; the last defer registered runs
first.

The code within `defer` closures runs after the return statement. As I mentioned, you can supply a function with input
parameters to a `defer`. Just as defer doesn't run immediately, any variables passed into a deferred closure aren’t
evaluated until the closure runs.

```go
func example() {
    defer func() int {
        return 2 // there's no way to read this value
    }()
}
```

You might be wondering if there’s a way for a deferred function to examine or modify the return values of its
surrounding function. There is, and it’s the best reason to use
[named return values](#named-return-values).

```go
func insert(ctx context.Context, db *sql.DB, name string) (err error) {
    tx, err := db.BeginTx(ctx, nil)
        if err != nil {
        return err
    }
    
    defer func() {
        if err == nil {
            err = tx.Commit()
        }
        
        if err != nil {
            tx.Rollback()
        }
    }()
    
    _, err = tx.ExecContext(ctx, "INSERT INTO users (name) values $1", name)
    if err != nil {
        return err
    }
    
    // use tx to do more database inserts here
    
    return nil
}
```

A common pattern in Go is for a function that allocates a resource to also return a closure that cleans up the resource.

```go
func getFile(name string) (*os.File, func(), error) {
    file, err := os.Open(name)
    
    if err != nil {
        return nil, nil, err
    }
    
    return file, func() {
        file.Close()
    }, nil
}
```

Now in main, we use our getFile function:

```go
f, closer, err := getFile(os.Args[1])
if err != nil {
    log.Fatal(err)
}
defer closer()
```

## Go Is Call By Value

It means that when you supply a variable for a parameter to a function, Go always makes a copy of the value of the
variable.

```go
type person struct {
    age int
    name string
}
```

Next, we write a function that takes in an int, a string, and a person, and modifies their values:

```go
func modifier(i int, s string, p person) {
    i = i * 2
    s = "Goodbye"
    p.name = "Bob"
}
```

We then call this function from main and see if the modifications stick:

```go
func main() {
    p := person{}
    i := 2
    s := "Hello"
    modifier(i, s, p)
    fmt.Println(i, s, p) // 2 Hello {0 }
}
```

The function won’t change the values of the parameters passed into it.

The behavior is a little different for maps and slices.

```go
func mapModifier(m map[int]string) {
    m[2] = "hello"
    m[3] = "goodbye"
    delete(m, 1)
}

func sliceModifier(s []int) {
    for k, v := range s {
        s[k] = v * 2
    }
    s = append(s, 10)
}
```

We then call these functions from main:

```go
func main() {
    m := map[int]string{
        1: "first",
        2: "second",
    }
    mapModifier(m)
    fmt.Println(m) // map[2:hello 3:goodbye]
    
    s := []int{1, 2, 3}
    sliceModifier(s)
    fmt.Println(s) // [2 4 6]
}
```

For the map, it’s easy to explain what happens: any changes made to a map parameter are reflected in the variable passed
into the function. For a slice, it’s more complicated. You can modify any element in the slice, but you can’t lengthen
the slice. This is true for maps and slices that are passed directly into functions as well as map and slice fields in
structs.

> Every type in Go is a value type. It’s just that sometimes the value is a pointer.


Call by value is one reason why Go’s limited support for constants is only a minor handicap. Since variables are passed
by value, you can be sure that calling a function doesn't modify the variable whose value was passed in (unless the
variable is a slice or map).

In general, this is a good thing. It makes it easier to understand the flow of data through your program when functions
don’t modify their input parameters and instead return newly computed values.

While this approach is easy to understand, there are cases where you need to pass something mutable to a function. What
do you do then? That’s when you need a pointer.