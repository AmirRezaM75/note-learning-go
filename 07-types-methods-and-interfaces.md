# Types, Methods, and Interfaces

## Types in Go

Go allows you to attach methods to types. It also has type abstraction, allowing you to write code that invokes methods
without explicitly specifying the implementation.

```go
type Person struct {
    FirstName string
    LastName string
    Age int
}
```

This should be read as declaring a _user-defined type_ with the name Person to have the _underlying type_ of the struct
literal that follows.

An _abstract type_ is one that specifies what a type should do, but not how it is done. A _concrete type_ specifies what
and how.

## Methods

```go
func (p Person) String() string {
    return fmt.Sprintf("%s %s, age %d", p.FirstName, p.LastName, p.Age)
}

// Method invocation:
output := p.String()
```

By convention, the _receiver_ name is a short abbreviation of the type’s name, usually its first letter. It is
nonidiomatic to use this or self.

Be aware that methods must be declared in the same package as their associated type; Go doesn't allow you to add methods
to types you don’t control. While you can define a method in a different file within the same package as the type
declaration, it is best to keep your type definition and its associated methods together so that it’s easy to follow the
implementation.

### Pointer Receivers and Value Receivers

- If your method modifies the receiver, you must use a pointer receiver.
- If your method needs to handle `nil` instances (see
  [“Code Your Methods for nil Instances”](#code-your-methods-for-nil-instances)), then it must use a pointer receiver.
- If your method doesn't modify the receiver, you can use a value receiver.

When a type has any pointer receiver methods, a common practice is to be consistent and use pointer receivers for all
methods, even the ones that don’t modify the receiver.

```go
type Counter struct {
  total int
}
func (c *Counter) Increment() {
  c.total++
}
func (c Counter) String() string {
  return fmt.Sprintf("total: %d", c.total)
}

var c Counter
fmt.Println(c.String())
c.Increment()
fmt.Println(c.String())
```

You should see the following output:

```
total: 0
total: 1
```

When you use a pointer receiver with a local variable that’s a value type, Go automatically converts it to a pointer
type. In this case, `c.Increment()` is converted to `(&c).Increment()`.

However, be aware that the rules for passing values to functions still apply. If you pass a value type to a function and
call a pointer receiver method on the passed value, you are invoking the method on a copy.

```go
func update(c *Counter) {
  c.Increment()
  fmt.Println(c.String())
}

func main() {
  var c Counter
  update(&c)
}
```

The parameter in `update` is of type `*Counter`, which is a pointer instance. Go considers both pointer and value
receiver methods to be in the _method set_ for a pointer instance. For a value instance, only the value receiver methods
are in the method set.

### Code Your Methods for nil Instances

What happens when you call a method on a nil instance?

```go
type IntTree struct {
  val int
  left, right *IntTree
}

func (it *IntTree) Insert(val int) *IntTree {
  if it == nil {
    return &IntTree{val: val}
  }
  if val < it.val {
    it.left = it.left.Insert(val)
  } else if val > it.val {
    it.right = it.right.Insert(val)
  }
  return it
}
```

If it’s a method with a value receiver, you’ll get a panic, as there is no value being pointed to by the pointer. If
it’s a method with a pointer receiver, it can work if the method is written to handle the possibility of a nil instance.

Pointer receivers work just like pointer function parameters; it’s a copy of the pointer that’s passed into the method.
Just like nil parameters passed to functions, if you change the copy of the pointer, you haven’t changed the original.
This means you can’t write a pointer receiver method that handles nil and makes the original pointer non-nil.

### Methods Are Functions Too

```go
type Adder struct {
  start int
}

func (a Adder) AddTo(val int) int {
  return a.start + val
}
```

We can also assign the method to a variable or pass it to a parameter of type `func(int)int`. This is called a _method
value_:

```go
myAdder := Adder{start: 10}
f1 := myAdder.AddTo
fmt.Println(f1(10)) // prints 20
```

You can also create a function from the type itself. This is called a _method expression_:

```go
f2 := Adder.AddTo
fmt.Println(f2(myAdder, 15)) // prints 25
```

### Type Declarations Aren’t Inheritance

You can also declare a user-defined type based on another user-defined type:

```go
type Score int
type HighScore Score
```

It looks a bit like inheritance, but it isn’t. You can’t assign an instance of type HighScore to a variable of type
Score or vice versa without a type conversion. Furthermore, any methods defined on Score aren’t defined on HighScore:

```go
// assigning literals and constants compatible with the underlying type is valid
var i int = 300
var s Score = 100
var hs HighScore = 200
hs = s // compilation error!
s = i // compilation error!
s = Score(i) // ok
hs = HighScore(s) // ok
```

### Types Are Executable Documentation

Types are documentation. They make code clearer by providing a name for a concept and describing the kind of data that
is expected.

### iota Is for Enumerations—Sometimes

Go doesn't have an enumeration type. Instead, it has `iota`, which lets you assign an increasing value to a set of
constants.

When using `iota`, the best practice is to first define a type based on `int` that will represent all of the valid
values:

```go
type MailCategory int
```

Next, use a const block to define a set of values for your type:

```go
const (
  Uncategorized MailCategory = iota // 0
  Personal
  Spam
  Social
  Advertisements
)
```

When a new const block is created, `iota` is set back to 0.

> Using iota with constants is fragile when you care about the value. You don’t want a future maintainer to insert a new
> constant in the middle of the list and break your code.

It may be necessary to skip a value. If so, you can use the _ (underscore) operator:

```go
const (
  _ MailCategory = iota
  Uncategorized // 1
  _
  Personal // 3
)
```

`Iota` can be used to quickly create the correct values by using the bit shift operator.

```go
const (
  read   = 1 << iota // 00000001 = 1
  write              // 00000010 = 2
  remove             // 00000100 = 4
  
  // admin will have all of the permissions
  admin = read | write | remove
)
```

## Use Embedding for Composition

The software engineering advice “Favor object composition over class inheritance”

```go
type Employee struct {
  Name string
  ID   string
}
func (e Employee) Description() string {
  return fmt.Sprintf("%s (%s)", e.Name, e.ID)
}
type Manager struct {
  Employee
  Reports []Employee
}
```

Note that Manager contains a field of type Employee, but no name is assigned to that field. This makes Employee an
_embedded field_. Any fields or methods declared on an embedded field are _promoted_ to the containing struct and can be
invoked directly on it.

```go
m := Manager{
    Employee: Employee{
        Name: "Bob Bobson",
        ID:   "12345",
    },
    Reports: []Employee{},
}
fmt.Println(m.ID)	// prints 12345
fmt.Println(m.Description()) // prints Bob Bobson (12345)
```

If the containing struct has fields or methods with the same name as an embedded field, you need to use the embedded
field’s type to refer to the obscured fields or methods.

## A Quick Lesson on Interfaces

```go
type Stringer interface {
  String() string
}
```

Interfaces are usually named with “er” endings. Like `fmt.Stringer`, `io.Reader`, `io.Closer`, `io.ReadCloser`
, `json.Marshaler`, and `http.Handler`.

## Interfaces Are Type-Safe Duck Typing

What makes Go’s interfaces special is that they are implemented _implicitly_. A concrete type does not declare that it
implements an interface. If the method set for a concrete type contains all of the methods in the method set for an
interface, the concrete type implements the interface.

```go
type LogicProvider struct {}

func (lp LogicProvider) Process(data string) string {
  // business logic
}

type Logic interface {
  Process(data string) string
}

type Client struct{
  L Logic
}

func(c Client) Program() {
  // get data from somewhere
  c.L.Process(data)
}

func main() {
  c := Client{
    L: LogicProvider{},
  }
  c.Program()
}
```

## Embedding and Interfaces

Just like you can embed a type in a struct, you can also embed an interface in an interface.

```go
type Reader interface {
  Read(p []byte) (n int, err error)
}

type Closer interface {
  Close() error
}

type ReadCloser interface {
  Reader
  Closer
}
```

## Accept Interfaces, Return Structs

Your code should “Accept interfaces, return structs.”

If you create an API that returns interfaces, you are losing one of the main advantages of implicit interfaces:
decoupling. You want to limit the third-party interfaces that your client code depends on because your code is now
permanently dependent on the module that contains those interfaces, as well as any dependencies of that module, and so
on.

To avoid the coupling, you’d have to write another interface and do a type conversion from one to the other. While
depending on concrete instances can lead to dependencies, using a
[dependency injection](#implicit-interfaces-make-dependency-injection-easier) layer in your application limits the
effect.

Another reason to avoid returning interfaces is versioning. If a concrete type is returned, new methods and fields can
be added without breaking existing code. The same is not true for an interface. Adding a new method to an interface
means that you need to update all existing implementations of the interface, or your code breaks.

Rather than writing a single factory function that returns different instances behind an interface based on input
parameters, try to write separate factory functions for each concrete type. In some situations (such as a parser that
can return one or more different kinds of tokens), it’s unavoidable and you have no choice but to return an interface.

Errors are an exception to this rule.

As we discussed in
[“Reducing the Garbage Collector’s Workload”](./06-pointers.md#reducing-the-garbage-collectors-workload), reducing heap
allocations improves performance by reducing the amount of work for the garbage collector. Returning a struct avoids a
heap allocation, which is good. However, when invoking a function with parameters of interface types, a heap allocation
occurs for each of the interface parameters.

## Interfaces and nil

The zero value for interface is `nil`

In the Go runtime, interfaces are implemented as a pair of pointers, one to the underlying type and one to the
underlying value.

| Type | Value |
|------|-------|

In order for an interface to be considered `nil` both the type and the value must be `nil`

```go
var s *string
fmt.Println(s == nil) // prints true
var i interface{}
fmt.Println(i == nil) // prints true
i = s
fmt.Println(i == nil) // prints false
```

| Type    | Value |
|---------|-------|
| *string | nil   |

What `nil` indicates for an interface is whether or not you can invoke methods on it. As we covered in
[Code Your Methods for nil Instances](#code-your-methods-for-nil-instances), you can invoke methods on nil concrete
instances, so it makes sense that you can invoke methods on an interface variable that was assigned a nil concrete
instance. If an interface is nil, invoking any methods on it triggers a panic. If an interface is non-nil, you can
invoke methods on it. (But note that if the value is nil and the methods of the assigned type don’t properly handle nil,
you could still trigger a panic.)

Since an interface instance with a non-nil type is not equal to nil, it is not straightforward to tell whether or not
the value associated with the interface is nil when the type is non-nil. You must use reflection (which we’ll discuss in
[“Use Reflection to Check If an Interface’s Value Is nil”](./14-reflect-unsafe-and-cgo.md#use-reflection-to-check-if-an-interfaces-value-is-nil))
to find out.

## The Empty Interface Says Nothing

Go uses `interface{}` to represent that a variable could store a value of any type.

One common use of the empty interface is as a placeholder for data of uncertain schema that’s read from an external
source, like a JSON file:

```go
// one set of braces for the interface{} type,
// the other to instantiate an instance of the map
data := map[string]interface{}{}
contents, err := ioutil.ReadFile("testdata/sample.json")
if err != nil {
  return err
}
defer contents.Close()
json.Unmarshal(contents, &data)
// the contents are now in the data map
```

## Type Assertions and Type Switches

A _type assertion_ names the concrete type that implemented the interface, or names another interface that is also
implemented by the concrete type underlying the interface.

```go
type MyInt int
func main() {
  var i interface{}
  var mine MyInt = 20
  i = mine
  i2 := i.(MyInt)
  fmt.Println(i2 + 1)
}
```

In the preceding code, the variable `i2` is of type `MyInt`.

You might wonder what happens if a type assertion is wrong. In that case, your code panics.

```go
i2 := i.(string)
fmt.Println(i2)
```

Running this code produces the following panic:

```
panic: interface conversion: interface {} is main.MyInt, not string
```

Go is very careful about concrete types. Even if two types share an underlying type, a type assertion must match the
type of the underlying value.

```go
i2 := i.(int)
fmt.Println(i2 + 1) // panic
```

Obviously, crashing is not desired behavior. We avoid this by using the comma ok idiom.

```go
i2, ok := i.(int)
if !ok {
  return fmt.Errorf("unexpected type for %v",i)
}
fmt.Println(i2 + 1)
```

> A type assertion is very different from a type conversion. Type conversions can be applied to both concrete types and
> interfaces and are checked at compilation time. Type assertions can only be applied to interface types and are checked
> at runtime. Because they are checked at runtime, they can fail. Conversions change, assertions reveal.

When an interface could be one of multiple possible types, use a _type switch_ instead:

```go
func doSomething(i interface{}) {
  switch j := i.(type) {
  case nil:
    // i is nil, type of j is interface{}
  case int:
    // j is of type int
  case MyInt:
    // j is of type MyInt
  case io.Reader:
    // j is of type io.Reader
  case string:
    // j is a string
  case bool, rune:
    // i is either a bool or rune, so j is of type interface{}
    // If you list more than one type on a case, the new variable is of type interface{}.
  default:
    // no idea what i is, so j is of type interface{}
  }
}
```

Since the purpose of a type switch is to derive a new variable from an existing one, it is idiomatic to assign the
variable being switched on to a variable of the same name (`i := i.(type)`), making this one of the few places where
shadowing is a good idea.

If you don’t know the underlying type, you need to use [reflection](./14-reflect-unsafe-and-cgo.md).

## Use Type Assertions and Type Switches Sparingly

TODO: Could not understand :)

## Function Types Are a Bridge to Interfaces

The most common usage is for HTTP handlers. An HTTP handler processes an HTTP server request. It’s defined by an
interface:

```go
type Handler interface {
  ServeHTTP(http.ResponseWriter, *http.Request)
}
```

By using a type conversion to `http.HandlerFunc`, any function that has the signature
`func(http.ResponseWriter,*http.Request)` can be used as an `http.Handler`:

```go
type HandlerFunc func(http.ResponseWriter, *http.Request)
func (f HandlerFunc) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  f(w, r)
}
```

## Implicit Interfaces Make Dependency Injection Easier

## Wire

Those who feel like writing dependency injection code by hand is too much work, can use
[Wire](https://github.com/google/wire), a dependency injection helper written by Google.

## Go Isn’t Particularly Object-Oriented (and That’s Great)

Go clearly isn’t a strictly procedural language. At the same time, Go’s lack of method overriding, inheritance means
that it is not a particularly object-oriented language, either. Go has function types and closures, but it isn’t a
functional language, either. If you attempt to shoehorn Go into one of these categories, the result is nonidiomatic
code. If you had to label Go’s style, the best word to use is _practical_. It borrows concepts from many places with the
overriding goal of creating a language that is simple, readable, and maintainable by large teams for many years.