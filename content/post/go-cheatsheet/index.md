---
title: Go cheatsheet
description: Condensed summary of Go's syntax, type system and peculiarities
slug: go-cheatsheet
date: 2024-08-31 00:00:00+0000
categories:
    - Go
---

## Types

### Booleans

- zero initialized when declared without a value
- cannot be modified by functions, unless when using a pointer

```go
var zeroInitializedBool bool // false
var valueBool bool = true
```

### Integers

- zero initialized when declared without a value
- cannot be modified by functions, unless when using a pointer

```go
var zeroInitializedInt int // 0
var valueInt int = 5

// Other integer types have a minimum and maximum value
var min8, max8 int8 = -128, 127
var min16, max16 int16 = -32768, 32767
var min32, max32 int32 = -2147483648, 2147483647
var min64, max64 int64 = -9223372036854775808, 9223372036854775807

var min_u8, max_u8 uint8 = 0, 255
var min_u16, max_u16 uint16 = 0, 65535
var min_u32, max_u32 uint32 = 0, 4294967295
var min_u64, max_u64 uint64 = 0, 18446744073709551615
```

### Decimals (float)

- zero initialized when declared without a value
- cannot be modified by functions, unless when using a pointer

```go
var max_f32 float32 = 3.40282346638528859811704183484516925440e+38
var smallest_f32 float32 =  1.401298464324817070923729583289916131280e-45

var max_f64 float64 = 1.797693134862315708145274237317043567981e+308
var smallest_f64 float64 = 4.940656458412465441765687928682213723651e-324
```

### Runes

- zero initialized when declared without a value
- cannot be modified by functions, unless when using a pointer

```go
var r1 rune = 'A'

// a line break
var line_break rune = '\n'

// a unicode checkmark symbol ✓
var checkmark rune = '\u2713' 
```

### Pointers

```go
var value int = 5
var pointer *int = &value
println(value == *pointer)
```

### Arrays

- zero initialized when declared without a value
- cannot be modified by functions, unless when using a pointer

```go
var arrayOfZeros [5]int
var array = [5]int{0, 1, 2, 10: 10, 11}
// note the colon : syntax to initialize the element at index 10

nElementsCopied := copy(dst, src)
length := len(array)

firstElement = array[0]
lastElement = array[len(array)-1]
fullSlice = array[:]
subSlice = array[3:5]
```

### Slices

Slices are dynamically allocated, variable-length arrays.

- They are default initialized to `nil`.
- Their values can be modified in function, with caveats: Their length, capacity and memory location cannot be modified in functions because they are passed by value. However, the value of elements can be modidied by functions since they are accessed through the pointer.

Their implementation can be illustrated as follows:

```go
struct slice {
    length int
    capacity int
    memory *T
}
```

Slices Cheatsheet:

```go
var nilSlice []int  // == nil
var emptySlice = []int{}  // length=0, capacity=0
var sliceWithValues = []int{
    0, 1, 2, 3,
    10: 10,
}
var sliceWithLength = make([]int, length)
var sliceWithLengthAndCapacity = make([]int, length, capacity)

firstElement = slice[0]

length := len(slice)
slice = append(slice, element1, element2)
slice = append(slice, ...otherSlice)
yesno = slices.Equal(slice1, slice2)
yesno = slice1 == nil
nElementsCopied := copy(dst, src)
```

### Maps

Implemented with pointers:
- They are default initialized to `nil`,
- Their values can be modified in function, with caveats.

```go
var nilMap map[string]int // == nil
var emptyMap = map[string]int{}
var mapWithValues = map[string]int{
    "https://takia.dev": 100,
}
var mapWithLength = make(map[string]int, length)

// read
v, ok := emptyMap["hello"]
if !ok {
    // ...
}

if v, ok := emptyMap["hello"]; ok {
    // use v here
}

delete(mapWithValues, "key")
maps.Equal(map1, map2)
```

## Control Structures

### If statements

```go
a := 5
if a == 5 {
    // ...
} else if a > 5 {
    // ...
} else {
    // ...
}
```

If statements can use if-local variables as follows:

```go
if n := rand.Intn(10); n < 5 {
    // n exists in this block
} else {
    // n exists in this block too
}
// n does not exist outside the if block
```

### Switch statements

Classic switch:
```go
switch value {
case 0, 1, 2:
    // ...
default:
    // ...
}
```

Empty switch:
```go
value := 0

switch {
case value < 5:
    // ...
case value > 5:
    // ...
default:
    // ...
}
```

Switch-local variables:

```go
switch value := 5; {
case value < 5:
// ...
}

switch value := 5; value {
case 0, 1, 2:
// ...
}
```

### For loops

Classic C-style for loop:
```go
for i := 0; i < 10; i++ {
}
```

Condition-only for loop:
```go
i := 0
for i < 10 {
    i += 1
}
```

Sequence iteration:
```go
s := []string{"a", "b", "c"}
for index, value := range s {
    println(index, value)
}
```

Map iteration:
```go
m := map[string]string{"key1": "value1"}
for key, value := range m {
    println(key, value)
}
```

String iteration (as array of `rune`):
```go
str := "Hello 😊"
for index, rune := range str {
    // for iterates over runes, not bytes
    println(index, rune)
}
```

Labeled for loops allow using `break` or `continue` on an outer loop from an inner loop:
```go
// labeled for loop
outer:
for i_outer := range []int{1, 2, 3, 4} {
    for i_inner := range []int{1, 2, 3} {
        unused(i_outer, i_inner)
        continue outer
    }
}
```

### Goto statements

Goto statements can be use to skip over blocks of code. Although it should only be sparingly used, it can sometimes come handy for error handling. Use of `defer` is preferred when possible.

```go
jumpOver := true

if jumpOver {
    goto skip
}
println("This statement will be skipped")
skip:
// ...
```

### Functions

```go

func simpleFunction(arg1 int, arg2 int) int {
    return arg1 + arg2
}

func genericFunction[T any](arg1 T) T {
    return arg1
}

func variadicFunction(vals ...int) int {
    result := 0
    for _, v := range vals {
        result += v
    }
    return result
}

func multipleReturnValues() (int, error) {
    return 0, nil
}

func namedReturnValues() (retA int, err error) {
    // defer registers statements to be executed at exit
    // named return values allows defer to modify them
    defer func() {
        if err != nil {
            retA = 0
            // rollback code
        }
    }()

    return 1, nil
}
```

Functions can be stored in variables:

```go
// Function variables
var myfunc func(int, int) int = simpleFunction

// Function types
type FuncType func(int, int) int
var myfunc2 FuncType = simpleFunction
```

Anonymous Function can access outer variables:
```go

func f() {
    outerVariable := 5
    setValue := func(arg int) {
        outerVariable = arg
    }
    setValue(7)
    println(outerVariable)
}
```

## User-Defined types

### User-Defined types

User-Defined type can be declared as follows:

```go
type NewType ExistingType
```

There is no implicit cast between user-defined types. The code belows shows how this can be exploited to ensure that every string has been escaped before it is displayed on a webpage:

```go
type EscapedString string

func safeEscape(s string) EscapedString {
    escaped := s //html.EscapeString(s)
    return EscapedString(escaped)
}

func displayOnPage(s EscapedString) {
    // ...
}
```

### Structs

Structs are passed by value, they cannot be modified by functions unless when using a pointer. Struct fields are default initialized to their respective zero values.

```go
type person struct {
    name string
    age  int
}

var structOfZeros person // == {"", 0}
var julien = person{"Julien", 30}
var alisha = person{name: "Alisha"}
alisha.age = 30
```

### Methods & Member functions

Methods can be defined on user-defined types as follows:

```go
type person struct {
    name string
    age  int
}

// value-receiver methods cannot modify the instance
func (p person) String() string {
    return fmt.Sprintf("%s (age:%d)", p.name, p.age)
}

// pointer-receiver methods can modify the instance
func (p *person) GrowOld() {
    // handle case where p == nil!
    p.age += 1
}
```
Go will automatically reference or dereference values and pointer to match the value-receiver method or pointer-receiver method. But remember that value-receiver methods work on a copy of the object and therefore should not attempt to modify its member values.

```go
p1 := person{"Julien", 30}
p1.String()
p1.GrowOld() // auto reference

p2 = &p1
p2.String() // auto dereference
p2.GrowOld()
```

### Type Embeddings

Type embeddings embed one `InnerType` inside an `OuterType` and make methods defined on `InnerType` available on instances of `OuterType`:

For instance, the given `Inner` type:
```go
type Inner struct {
    innerField int
}

func (i Inner) ShowInnerField() {
    println(i.innerField)
}
```

Can be embedded inside an `Outer` type as follows:

```go
type Outer struct {
    Inner
    outerField int
}

o := Outer{
    Inner: Inner{
        innerField: 5,
    },
    outerField: 10,
}
```

And the `Inner` method can be called on the `Outer` object:
```go
o.ShowInnerField()
```

Embedding merge the method set of `Inner` and `Outer` making it possible to implement interfaces more easily. But beware that:

**Embeddings are not inheritance**:
- they do not provide implicit slicing or type cast from derived type to parent type
- they do not provide dynamic dispatch and method resolution

### Interfaces

Interfaces specify what the caller code needs, they define the set of method than an object is expected to offer. Interfaces are similar to python's Protocols: unlike in Java they are implemented implicitly.

```go
type Provider interface {
    ProviderID() string
}
```

Any object with a `Read() string` method automatically implements the `Reader` interface. 

```go
type MyProvider struct {
    name string
}
func (p MyProvider) ProviderID() string {
    return p.name
}
```

The following function requests a `Provider` instance and can be used with our `MyProvider` object implicitely:

```go
func useProvider(p Provider) {
	// ...
}

var p = MyProvider{"name"}
useProvider(p)
```

#### Interface Embeddings
Interfaces can be merged together using interface embeddings:

```go
type Reader interface{}
type Writer interface{}

type ReadWriter interface {
    Reader
    Writer
}
```

#### nil Interfaces

Under the hood, interface are implemented with two pointers: one for the instance and one for the instance type:

```go
type Interface[T any] struct {
    type *Type[T]
    value *T
}
```

An interface is considered `nil` if it has not been assigned a type. In the code below, even though `nilInstance` is set to the `nil` pointer, the `nilInterface` object is assigned the `Object` type and therefore does not evaluate to `nil`:

```go

type Interface interface {
	Method()
}

type Object struct{}
func (o Object) Method() {
}

var nilInstance *O
var notNilInterface I = nilInstance

println("Instance is nil:", nilInstance == nil) // true
println("Interface is nil:", notNilInterface == nil) // false
```

### Enums

Go uses `iota`, a special counter whose value increases for each constant in a const block. It can be used to assign increasing values to consecutive const values. Note that it is possible to use `_` to disable the default value.

```go
type EnumType int
const (
    EnumDefaultValue EnumType = iota
    EnumValue1
    EnumValue2
    EnumValue3
)
```

### Generics

A function or type is said generic when it accepts type arguments. Type arguments are passed between square brackets `[]`. In the code below, the generic type argument `T` implements the `comparable` constraint. Any interface can be used as a type constraint.

```go
type Set[T comparable] struct {
	storage map[T]bool
}

func (s *Set[T]) add(value T) {
	if s.storage == nil {
		s.storage = map[T]bool{}
	}
	s.storage[value] = true
}
func (s *Set[T]) remove(value T) {
	delete(s.storage, value)
}
func (s Set[T]) contains(value T) bool {
	return s.storage[value]
}

var s = Set[int]{}
s.add(5)
println("Set contains 5?", s.contains(5))
s.remove(5)
```

