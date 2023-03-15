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

As we’ll see in “Explicit Type Conversion” on page 26 you can’t even add two integer variables together if they are
declared to be of different sizes. However, Go lets you use an integer literal in floating point expressions

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
discuss the rune type in “A Taste of Strings and Runes” on page 26 and `uintptr` in Chapter 14.

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