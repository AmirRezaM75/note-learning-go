# Primitive Types and Declarations

## Built-in Types

### The Zero Value

Go assigns a default zero value to any variable that is declared but not assigned a value.

### Literals

A literal in go refers to writing out a number, character and string.

There are five kinds of literals:

1. [Integer](#integer)
2. [Float](#floating-point)
3. [Rune](#rune)
4. [String](#string)

#### Integer

Integer literals are sequences of numbers; they are normally base ten, but different prefixes are used to indicate other
bases:

- `0b` for binary (base two)
- `0o` for octal (base eight), A leading 0 with no letter after it is another way to represent an octal literal. Octal
  representations are rare, mostly used to represent POSIX permission flag values (such as 0o777 for rwxrwxrwx).
- `0x` for hexadecimal (base sixteen)

You can use either or upper- or lowercase letters for the prefix.

To make it easier to read longer integer literals, Go allows you to put underscores in the middle of your literal. This
allows you to, for example, group by thousands in base ten (1_234).

#### Floating point

They can also have an exponent specified with the letter `e` (`p` for hexadecimal) and a positive or negative number (
such as 6.03e2 = 603).

#### Rune

They represent characters and are surrounded by single quotes. Unlike many other languages, in Go single quotes and
double quotes are not interchangeable. Rune literals can be written as:

- Single Unicode characters ('a')
- 8-bit octal numbers ('\141')
- 8-bit hexadecimal numbers ('\x61')
- 16-bit hexadecimal numbers ('\u0061')
- 32-bit Unicode numbers ('\U00000061')

#### String

1. Interpreted string literal

Use double quotes to create an _interpreted string literal_

2. Raw string literal

If you need to include backslashes, double quotes, or newlines in your string, use a _raw string literal_. These are
delimited with backquotes (\`) and can contain any literal character except a backquote.

\`Greetings and "Salutations"\` vs "Greetings and \"Salutations\""

As we’ll see in [“Explicit Type Conversion”](#explicit-type-conversion) you can’t even add two integer variables
together if they are declared to be of different sizes. However, Go lets you use an integer literal in floating point
expressions

```go
fmt.Println(2 + 3.2) // 5.2
```

or even assign an integer literal to a floating point variable.

```go
s := 3.2
s = 2
fmt.Println(s) // 2
```

This is because literals in Go are untyped; they can interact with any variable that’s compatible with the literal.

### Booleans

The zero value for a bool is false.

```go
var flag bool
```

### Numeric Types

#### Integer Types

The zero value for all the integer types is 0.

| Type name | Value range                                 |
|-----------|---------------------------------------------|
| int8      | –128 to 127                                 |
| int16     | –32768 to 32767                             |
| int32     | –2147483648 to 2147483647                   |
| int64     | –9223372036854775808 to 9223372036854775807 |
| uint8     | 0 to 255                                    |
| uint16    | 0 to 65536                                  |
| uint32    | 0 to 4294967295                             |
| uint64    | 0 to 18446744073709551615                   |

A `byte` is an alias for `uint8`.

On a 32-bit CPU, `int` is a 32-bit signed integer like an int32. On **most** 64-bit CPUs, int is a 64-bit signed
integer, just like an int64.

There are some uncommon 64-bit CPU architectures that use a 32-bit signed integer for the int type. Go supports three of
them: amd64p32, mips64p32, and mips64p32le.

`uint` follows the same rules as int, only it is unsigned.

Integer literals default to being of int type.

There are two other special names for integer types, `rune` and `uintptr`. We looked at rune literals earlier and
discuss the rune type in [“A Taste of Strings and Runes”](#a-taste-of-strings-and-runes) and `uintptr` in Chapter 14.

If you are writing a library function that should work with any integer type, write a pair of functions, one
with `int64` for the parameters and variables and the other with `uint64`.

> The reason why int64 and uint64 are the idiomatic choice in this situation is that Go doesn't have [function overloading](https://geeksforgeeks.org/function-overloading-c/). Without this feature, you’d need to write many functions with slightly different names to implement your algorithm. Using int64 and uint64 means that you can write the code once and let your callers use type conversions to pass values in and convert data that’s returned.

The result of an integer division is an integer; if you want to get a floating point result, you need to use a type
conversion to make your integers into floating point numbers.

> Integer division in Go follows truncation toward zero

| Symbol | Operator            | Supported Types                           |
|--------|---------------------|-------------------------------------------|
| +      | sum                 | integers, floats, complex values, strings |
| -      | difference          | integers, floats, complex values          |
| *      | product             | integers, floats, complex values          |
| /      | quotient            | integers, floats, complex values          |
| %      | remainder           | integers                                  |
| &      | bitwise AND         | integers                                  |
| &#124; | bitwise OR          | integers                                  |
| ^      | bitwise XOR         | integers                                  |
| &^     | bit clear (AND NOT) | integers                                  |
| <<     | left shift          | integers                                  |
| \>>    | right shift         | integers                                  |

#### Floating Point Types

| Type name | Largest absolute value                         | Smallest (nonzero) absolute value              |
|-----------|------------------------------------------------|------------------------------------------------|
| float32   | 3.40282346638528859811704183484516925440e+38   | 1.401298464324817070923729583289916131280e-45  |
| float64   | 1.797693134862315708145274237317043567981e+308 | 4.940656458412465441765687928682213723651e-324 |

The zero value for the floating point types is 0.

Floating point literals have a default type of float64.

Go stores floating point numbers using a specification called [IEEE 754](https://www.youtube.com/watch?v=RuKkePyo9zk).
For example, if you store the number –3.1415 in a float64, the 64-bit representation in memory looks like:

1100000000001001001000011100101011000000100000110001001001101111

which is exactly equal to –3.14150000000000018118839761883.

> A floating point number cannot represent a decimal value exactly. Do not use them to represent money or any other value that must have an exact decimal representation!

Dividing a nonzero floating point variable by 0 returns `+Inf` or `-Inf` (positive or negative infinity), depending on
the sign of the number. Dividing a floating point variable set to 0 by 0 returns `NaN` (Not a Number).

While Go lets you use == and != to compare floats, don’t do it. Due to the inexact nature of floats, two floating point
values might not be equal when you think they should be. Instead, define a maximum allowed variance and see if the
difference between two floats is less than that. This value (sometimes called _epsilon_) depends on what your accuracy
needs are.

### A Taste of Strings and Runes

The zero value for a string is an empty string.

Strings in Go are immutable.

The `rune` type represents a single code point and is an alias for the `int32` type.

A rune literal's default type is a rune and a string literal's default type is a string.

### Explicit Type Conversion

Most languages that have multiple numeric types, automatically convert from one to another when needed. This is called
_automatic type promotion_. Go doesn't support that. You must use a _type conversion_.

```go
var x int = 10
var y float64 = 30.2
var z float64 = float64(x) + y
var d int = x + int(y)
```

Since all type conversions in Go are explicit, you can't treat another Go type as a boolean. In many languages nonzero
number or a nonempty string can be interpreted as a boolean `true`. The rules for "truthy" values vary from language to
language and can be confusing.

No other type can be converted to a bool, implicitly or explicitly. You must use one of the comparison operators.

## var Versus :=

```go
var x int = 10
```

If the type on the righthand side of the = is the expected type of your variable, you can leave off the type from the
left side of the = . Since the default type of an integer literal is `int`, we can rewrite it as follows:

```go
var x = 10
```

Go also supports a short declaration format. When you are within a function, you can use the `:=` operator to replace a
var declaration that uses _type inference_.

```go
x := 10
```

Conversely, if you want to declare a variable and assign it the zero value:

```go
var x int
```

You can declare multiple variables at once with `var`, and they can be of the same type:

```go
var x, y int = 10, 20
```

all zero values of the same type:

```go
var x, y int
```

or of different types:

```go
var x, y = 10, "hello"
// or
x, y := 10, "hello"
```

While var and := allow you to declare multiple variables on the same line, only use this style when assigning multiple
values returned from a [function](05-functions.md) or the comma ok idiom on page 54.

If you are declaring multiple variables at once, you can wrap them in a declaration list:

```go
var (
  x     int
  y         = 20
  z     int = 30
  d, e      = 40, "hello"
  f, g string
)
```

The `:=` operator can do one trick that you cannot do with `var`: it allows you to assign values to existing variables,
too. As long as there is one new variable on the lefthand side of the :=

```go
x := 10
x, y := 30, "hello"
```

If you are declaring a variable at package level, you must use `var` because `:=` is not legal outside of functions.

There are some situations within functions where you should avoid `:=` :

- When initializing a variable to its zero value, use `var x int`. This makes it clear that the zero value is intended.

- When assigning an untyped constant or a literal to a variable and the default type for the constant or literal isn’t
  the type you want for the variable. Favor `var x byte = 20` over `x := byte(20)`.

- Because := allows you to assign to both new and existing variables, it sometimes creates new variables when you think
  you are reusing existing ones (see “Shadowing Variables” on page 62 for details). In those situations, explicitly
  declare all of your new variables with var to make it clear which variables are new, and then use the assignment
  operator to assign values to both new and old variables.

> Avoid declaring variables outside of functions (package level variables) because they complicate data flow analysis. Also the Go compiler won’t stop you from creating [unread package-level variables](#unused-variables).

## Using const

```go
const x int64 = 10

const (
  idKey   = "id"
  nameKey = "name"
)

const z = 20 * 10

func main() {
  const y = "hello"
  fmt.Println(x)
  fmt.Println(y)
  x = x + 1
  y = "bye"
  fmt.Println(x)
  fmt.Println(y)
}
```

If you try to run this code, compilations fails with the following error messages:

```shell
./const.go:20:4: cannot assign to x
./const.go:21:4: cannot assign to y
```

As you see, you declare a constant at the package level or within a function. Just like `var`, you can (and should)
declare a group of related constants within a set of parentheses.

> Constants in Go are a way to give names to literals. There is no way in Go to declare that a variable is immutable.

## Typed and Untyped Constants

If you are giving a name to a mathematical constant that could be used with multiple numeric types, then keep the
constant untyped. In general, leaving a constant untyped gives you more flexibility. We’ll use typed constants when we
look at creating enumerations with `iota` in “iota Is for Enumerations—Sometimes” on page 137.

Here’s what an untyped constant declaration looks like:

```go
const x = 10
```

All of the following assignments are legal:

```go
var y int = x
var z float64 = x
var d byte = x
```

Here’s what a typed constant declaration looks like:

```go
const typedX int = 10
```

This constant can only be assigned directly to an int . Assigning it to any other type produces a compile-time error
like this:

`cannot use typedX (type int) as type float64 in assignment`

## Unused Variables

Another Go requirement is that every declared local variable must be read. It is a compile-time error to declare a local
variable and to not read its value.

Perhaps surprisingly, the Go compiler allows you to create unread constants with `const`. This is because constants in
Go are calculated at compile time and cannot have any side effects. This makes them easy to eliminate: if a constant
isn’t used, it is simply not included in the compiled binary.

## Naming Variables and Constants

Like most languages, Go requires identifier names to start with a letter or underscore, and the name can contain
numbers, underscores, and letters. Go’s definition of “letter” and “number” is a bit broader than many languages. Any
Unicode character that is considered a letter or digit is allowed.

```go
_0 := 0_0
_𝟙 := 20
π := 3
ａ := "hello" // Unicode U+FF41
```

These names are confusing or difficult to type on many keyboards.

Idiomatic Go doesn't use snake case (names like index_counter). Instead, Go uses camel case (names like indexCounter)
when an identifier name consists of multiple words.

In many languages, constants are always written in all uppercase letters, with words separated by underscores (names
like INDEX_COUNTER). Go does not follow this pattern. This is because Go uses the case of the first letter in the name
of a package-level declaration to determine if the item is accessible outside the package. We will revisit this when we
talk about packages in Chapter 9.
