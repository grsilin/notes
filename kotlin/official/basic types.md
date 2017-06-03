# Basic Types

variable types of kotlin

## Numbers

| Type   | Bit Width |
| ------ | --------- |
| Double | 64        |
| Float  | 32        |
| Long   | 64        |
| Int    | 32        |
| Short  | 16        |
| Byte   | 8         |

## use underscores

add underscores to make value more readable:

```kotlin
val phoneNumber = 150_6529_2356
```

## Representation

numbers will be boxed when it is  nullable,it preserves equality,but not same identity:

```kotlin
val a: Int = 1000;
println(a === a)
val boxedA: Int? = a
val anotherBoxedA = a
println(boxedA === anotherBoxedA)	// it prints false
println(boxedA==anotherBoxedA)	// true
println(boxedA === a)	//false
println(boxedA == a)	//true
```

##  Explicit Conversions

small types are not implicitly  converted to big types:

```kotlin
    val b: Byte = 5;
    val i:Int = b.toInt()
    println(b.equals(i).toString())
```

every numbers types supports such conversions:

```kotlin
toByte()	//convert to Byte
toShort()	//to Short
toInt()		//to Int
toLong()	//to Long
toFloat()	//to Float
toDouble()	//to Double
toChar()	//to Char
```

## Bitwise Operations

it can be called in infix:

```kotlin
shl()	//signed shift left
shr()	//signed shift right
ushl()	//unsigned shift left
ushr()	//unsigned shift right
and()	//bitwise and
or() 	//bitwise or
xor()	//bitwise xor
inv()	//bitwise inversion
```

## Character

characters literal go in single quotes like `1`,special characters can be escaped using a backslash,like,`/t`,`\b`,`\n`, `\r`, `\'`, `\"`, `\\` and `\$`.	Any character can be encode by Unicode escape sequence syntax:`\uFF00`.

characters can not treat as number:

```kotlin
//explicit conversions to number
'1'.toInt()
```

## Boolean

build-in operations of boolean:

```kotlin
||		//lazy disjunction
&&		//lazy conjunction
!		//negation
```

## Arrays

```kotlin
//create an Array<String> with values["0","1","4","9","16"]
val asc = Array(5,{ i -> (i*i).toString() })
```

use`[]` stands for member functions `get()`,`set()`.

Kotlin also has specialized classes to represent arrays of primitive types without boxing overhead: `ByteArray`, `ShortArray`, `IntArray` and so on.

```kotlin
al x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

## Strings

get element character of string by indexing operation:`s[i]`

```kotlin
val str="123456789"
for (c in str){
	println(c)
}
```

 raw string:delimited by a triple quote (`"""`),contains no escaping and can contain new lines and other characters.

```kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

String Templates: A template expression starts with a dollar sign ($) and consists of either a simple name

```kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // evaluates to "abc.length is 3"
```

