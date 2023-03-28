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
