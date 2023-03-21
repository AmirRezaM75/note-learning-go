# Pointers

## A Quick Pointer Primer

Every variable is stored in one or more contiguous memory locations, called _addresses_.

```go
var x int32 = 10
var y bool = true
```

![Figure 6-1. Storing two variables in memory](images/figure6-1.png)

You only need a bit to represent true or false, but the smallest amount of memory that can be independently addressed is
a byte.

A pointer is simply a variable whose contents are the address where another variable is stored.

```go
var x int32 = 10
var y bool = true
pointerX := &x
pointerY := &y
var pointerZ *string
```

![Figure 6-2. Storing pointers in memory](images/figure6-2.png)

While different types of variables can take up different numbers of memory locations, every pointer, no matter what type
it is pointing to, is always the same size.

The zero value for a pointer is `nil`.

Slices, maps, functions, channels and interfaces are implemented with pointers.

_pointer arithmetic_, are not allowed in Go.

The `&` is the _address_ operator.

The `*` is the _indirection_ operator. It precedes a variable of pointer type and returns the pointed-to value. This is
called _dereferencing_:

```go
x := 10
pointerX := &x
fmt.Println(pointerX) // prints a memory address
fmt.Println(*pointerX) // prints 10
z := 5 + *pointerX
fmt.Println(z) // prints 15
```

Your program will panic if you attempt to dereference a `nil` pointer:

```go
var x *int
fmt.Println(x == nil) // prints true
fmt.Println(*x) // panics
```

A _pointer type_ is a type that represents a pointer.

```go
x := 0
var pointerToX *int
pointerToX = &x
```

The built-in function `new` creates a pointer variable. It returns a pointer to a zero value instance of the provided
type:

```go
var pointer = new(int)
fmt.Println(pointer == nil) // prints false
fmt.Println(*pointer) // prints 0
```

The `new` function is rarely used.

You can’t use an & before a primitive literal (numbers, booleans, and strings) or a constant because they don’t have
memory addresses; they exist only at compile time.

If you have a struct with a field of a pointer to a primitive type, you can’t assign a literal directly to the field:

```go
type person struct {
    FirstName string
    MiddleName *string
    LastName string
}
p := person{
    FirstName: "Pat",
    MiddleName: "Perry", // This line won't compile
    LastName: "Peterson",
}
```

Compiling this code returns the error:

```
cannot use "Perry" (type string) as type *string in field value
```

If you try to put an & before "Perry", you’ll get the error message:

```
cannot take the address of "Perry"
```

There are two ways around this problem. The first is to introduce a variable to hold the constant value. The second way
is to write a helper function that takes in a boolean, numeric, or string type and returns a pointer to that type:

```go
func stringp(s string) *string {
    return &s
}
```

With that function, you can now write:

```go
p := person{
    FirstName: "Pat",
    MiddleName: stringp("Perry"), // This works
    LastName: "Peterson",
}
```

Why does this work? When we pass a constant to a function, the constant is copied to a parameter, which is a variable.
Since it’s a variable, it has an address in memory. The function then returns the variable’s memory address.