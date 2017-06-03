# Packages

declare package in start of source file

```kotlin
package com.coder
fun function(){
  
}
class class1{
  
}
```

## Imports

We can import either a single name, e.g.

```kotlin
import foo.Bar // Bar is now accessible without qualification
```

or all the accessible contents of a scope (package, class, object etc):

```kotlin
import foo.* // everything in 'foo' becomes accessible
```

import keyword can also use to import other declarations:

- top-level functions and properties;
- functions and properties declared in [object declarations](https://kotlinlang.org/docs/reference/object-declarations.html#object-declarations);
- [enum constants](https://kotlinlang.org/docs/reference/enum-classes.html)

