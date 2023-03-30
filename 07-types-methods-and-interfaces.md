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