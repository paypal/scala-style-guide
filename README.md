PayPal Scala Style Guidelines
=================

This repository contains style guidelines for writing Scala code at PayPal. Here are our goals for style
guides in this repository:

1. Our style guidelines should be clear.
2. We all should agree on our style guidelines.
3. We should change this guideline as we learn and as Scala evolves.
4. Keep the [scalastyle-config.xml](https://github.com/paypal/scala-style-guide/blob/develop/scalastyle-config.xml) in our projects in line with this guide as much as possible.

This repository follows git-flow. If you have a new style guideline, open a pull request against `develop`. We'll
discuss in the PR comments. The official guidelines live in [master](https://github.com/paypal/scala-style-guide/blob/master/README.md)

# Whitespace

* Use two spaces for indentation.
* Add a newline to the end of every file.

# Line Length

* Use a maximum line length of 160 characters.

# Names

## Functions, Classes and Variables

Use camel case for all function, class and variable names. Classes start with an upper case letter, and values and
functions start with a lower case. Here are some examples:

```scala
class MyClass {
  val myValue = 1
  def myFunction: Int = myValue
}
```
### Acronyms

Use camel case for acroynms, for example, `HttpUtil`. For consistency, this includes class names like `Ttl` and `Json` as well as common terms such as `Db` and `Io`. Follow normal camel-casing rules.

```scala
val httpIoJsonToXmlParser = new HttpIoJsonToXmlParser()
```

## Package Objects

Package names should be all lowercase with no underscores.

File names for package objects must match the most specific name in the package.
For example, for the `com.paypal.mypackage` package object, the file
should be named `mypackage.scala` and the file should be structured like this:


```scala
package com.paypal

package object mypackage {
  //...
}
```

# Classes

## Public Methods

Limit the number of public methods in your class to 30. You can make an exception to this if everything in the class is an extremely simple definition of a new class.

# Throwables

1. Prefer non-case classes when defining exceptions and errors unless you plan on pattern matching on the exception.
2. If providing a cause is desirable and not mandatory then define it as an option. Note, the use of ```.orNull```.

```scala
class CrazyException(msg: String, cause: Option[Throwable] = None) extends Exception(msg, cause.orNull)
class SuperCrazyError(msg: String, cause: Option[Throwable] = None) extends Error(msg, cause.orNull)
```

# Imports

## Ordering

IntelliJ's code style configuration allows for automatic grouping of imports by namespace.
We use the following ordering, which you should add to your IntelliJ configuration
(found in Preferences > Code Style > Scala):

```
java
___blank line___
scala
___blank line___
akka
spray
all other imports
___blank line___
com.ebay
com.paypal
```

You should also regularly run IntelliJ's Optimize Imports (Edit > Optimize Imports) against
your code before merging with `develop`, to maintain import cleanliness.

## Location

Most `import`s in a file should go at the top, right below the package name. The
only time you should break that rule is if you import some or all names from
an `object` from inside a `class`. For example:

```scala
package mypackage

import a.b.c.d
import d.c.b.a

class MyClass {
  import Utils._
  import MyClassCompanion.A
}

object MyClassCompanion {
  case class A()
}

object Utils {
  case class Util1()
  case class Util2()
}
```

## Predef

Never `import` anything inside of Scala's
['Predef'](http://www.scala-lang.org/api/2.10.3/index.html#scala.Predef$)
object. It is automatically imported for you, so it's redundant to manually
`import`. IntelliJ will often try to import it, so be aware.

# Values

Use `val` by default. `var`s should be limited to local variables or `private` class variables. Their usage
should be well documented, including any considerations that developers must make with respect to concurrency.

One common usage of `var`s will be inside of Akka `Actor`s. Here's an example of that usage:

```scala
class MyActor extends Actor {
  //This is local state to the actor that represents the current sum of all integers it has received.
  //It must be marked private and never accessed outside of the processing of a single message, to ensure
  //that it's concurrency safe. Additionally, don't acquire a lock before you access this variable.
  //Doing so might block message processing unnecessarily. If you follow all of these rules related to concurrency,
  //you shouldn't need a lock anyway.
  private var sum: Int = 0

  override def receive: Receive {
    case i: Int => sum = sum + 1
  }
}
```

## Modifiers

Optional variable modifiers should be declared in the following order: `override access (private|protected) final implicit lazy`.

For example,

```scala
private implicit lazy val someSetting = ...
```

# Functions

Rules to follow for all functions:

* Always put a space after `:` characters in function signatures.
* Always put a space after `,` in function signatures.

## Public Functions
All public functions and methods, including those inside `object`s must have:

* A return type
* Scaladoc including a function overview, information on parameters, and information on the return value. See the [Scaladoc](#scaladoc-comments-and-annotations) section for more details.

Here is a complete example of a function inside of an `object`.

```scala
object MyObject {
  /**
   * returns the static integer 123
   * @return the number 123
   */
  def myFunction: Int = {
    123
  }
}
```

### Parameter Lists

If a function has a parameter list with fewer than 70 characters, put all parameters on the same line:

```scala
def add(a: Int, b: Int): Int = {
  ...
}
```

If a function has long or complex parameter lists, follow these rules:

1. Put the first parameter on the same line as the function name.
2. Put the rest of the parameters each on a new line, aligned with the first parameter.
3. If the function has multiple parameter lists, align the opening parenthesis with the previous one and align
parameters the same as #2.

Example:

```scala
def lotsOfParams(aReallyLongParameterNameOne: Int,
                 aReallyLongParameterNameTwo: Int,
                 aReallyLongParameterNameThree: Int,
                 aReallyLongParameterNameFour: Int)
                (implicit adder: Adder,
                 reader: Reader): Int = {
  ...
}
```

For function names over 30 characters, try to shorten the name. If you can't,
start the parameter list on the next line and indent everything 2 spaces:

```scala
def aVeryLongMethodThatShouldHaveAShorterNameIfPossible(
  aParam: Int,
  anotherParam: Int,
  aThirdParam: Int)
 (implicit iParam: Foo,
  bar: Bar): String = {
  ...
}
```

In all cases, the function's return type still has to be written directly
following the last closing parenthesis.

#### Calling Functions

When calling functions with numerous arguments, place the first parameter on the
same line as the function and align the remaining parameters with the first:

```scala
fooBar(someVeryLongFieldName,
       andAnotherVeryLongFieldName,
       "this is a string",
       3.1415)
```

For functions with very long names, start the parameter list on the second line
and indent by 2 spaces:

```scala
aMuchLongerMethodNameThatShouldProbablyBeRefactored(
  aParam,
  anotherParam,
  aThirdParam)
```

It's your choice whether to place closing parenthesis directly following the
last parameter or on a new line (in "dangling" style).

You can do this:

```scala
aLongMethodNameThatReturnsAFuture(
  aParam,
  anotherParam,
  aThirdParam
).map { res =>
  ...
}
```

Or this:

```scala
aLongMethodNameThatReturnsAFuture(
  aParam,
  anotherParam,
  aThirdParam).map { res =>
  ...
}
```

## Anonymous functions

Anonymous functions start on the same line as preceding code. Declarations
start with ` { ` (note the space before and after the `{`). Arguments are then
listed on the same line. A few more notes:

* Do not use braces after the argument list, just start the function body on the next line.
* Argument types are not necessary. You should use them if it makes the code clearer though.

Here's a complete example example:

```scala
Option(123).map { number =>
  println(s"the number plus one is: ${number + 1}")
}
```

### Exceptions to the Rule

Use parentheses and an underscore for anonymous functions that are:

* single binary operations
* a single method invocation on the input
* two or fewer unary methods on the input

Examples:

```scala
val list = List("list", "of", "strings")
list.filter(_.length > 2)
list.filter(_.contains("i"))
```

## Passing named functions

If the function takes a single argument, then arguments and underscores should be omitted.

For example:
```scala
Option(123).map(println)
```

is preferred over

```scala
Option(123).map(println(_))
```

## Calling functions

Prefer dot notation except for these specific scenarios:
1. Specs2 matchers
2. Akka ```pipeTo```, ```!```, ```?```

For example:
```scala
Option(someInt).map(println)
```

is preferred over

```scala
Option(someInt) map println
```

##### Rule Exceptions

Java <=> Scala interoperation on primitives: Scala and Java booleans, for example, are not directly compatible, and
require an implicit conversion between one another. Normally this is handled automagically by the Scala compiler, but
the following code would reveal a compiler error:

```scala
Option(true).foreach(someMethodThatTakesAJavaBoolean)
```

A named parameter is *not* necessary to achieve this. An underscore should be used to resolve the implicit conversion.
To avoid confusion, please also add a note that a Scala <=> Java conversion is taking place:

```scala
Option(true).foreach(someMethodThatTakesAJavaBoolean(_))  // lol java
```

# Logic Flows

In general, logic that handles a choice between two or more outcomes should prefer to use `match`.

## Match Statements

When you `match` on any type, follow these rules:

1. Pattern matching should be exhaustive and explicitly handle the failure/default case rather than relying on a runtime `MatchError`. (This is specific to match blocks and not case statements in partial functions.) Case classes used in pattern matching should extend a common `sealed trait` so that compiler warnings will be generated for inexhaustive matching.
2. Indent all `case` statements at the same level, and put the `=>` one space to
  the right of the closing `)`
3. Short single line expressions should be on the same line as the `case`
4. Long single line and multi-line expressions should be on the line below the case, indented one level from the `case`.
5. Do not add extra newlines in between each case statement.
6. Filters on case statements should be on the same line if doing so will not make the line excessively long.

Here's a complete example:

```scala
Option(123) match {
  case Some(i: Int) if i > 0 =>
    val intermediate = doWorkOn(i + 1)
    doMoreWorkOn(intermediate)
  case _ => 123
}
```

## Option

Flows with `Option` values should be constructed using the `match` keyword as follows.

```scala
def stuff(i: Int): Int = { ... }

Option(123) match {
  case None => 0
  case Some(number) => stuff(number)
}
```

The `.fold` operator should generally not be used. Simple, single-line patterns are acceptable for `.fold`, such as:

```scala
def stuff(i: Int): Int = { ... }

Option(123).fold(0)(stuff)
```

Similarly, simple patterns are acceptable for `.map` with `.getOrElse`, such as:

```scala
def stuff(i: Int): Int = { ... }

Option(123).map(stuff).getOrElse(0)
```

You should enforce expected type signatures, as `match` does not guarantee consistent types between match outcomes
(e.g. the `None` case could return `Int`, while the `Some` case could return `String`).

When creating `Option`s, use
[`.opt`](https://github.com/paypal/cascade/blob/develop/common/src/main/scala/com/paypal/cascade/common/option/option.scala#L61).
If the value is a constant, use
[`.some`](https://github.com/paypal/cascade/blob/develop/common/src/main/scala/com/paypal/cascade/common/option/option.scala#L55)
instead.

```scala
val doThisForConstants = "hello".some
val notThisForConstants = "goodbye".opt

val doThisForEverythingElse = foo().opt
val notThisForEverythingElse = bar().some
```

## For Comprehension

For comprehensions should generally not be wrapped in parentheses in order to recover, flatMap, etc.
Instead, separate the for comprehension into its own variable and perform additional operations on that.

```scala
val userAddress = for {
  address <- loadAddress()
} yield address

userAddress.recover {
  // recover from loadAddress exception
}.flatMap {
  // pull info from address, etc.
}
```

In addition, if performing additional operations on the result of the yield as part of the result of the for comprehension itself,
code should be separated by braces and begin on a newline. For example:

```scala
for {
  address <- loadAddress() // returns Option
} yield {
  address match {
    case Some(s) => ...
    case None => ...
  }
}
```

# Akka

## Ask and Tell

Prefer ```!``` and ```?``` over ```.tell``` and ```.ask```.

```scala
someActor1 ! msg
someActor2 ? msg
```

is preferred over

```scala
someActor1.tell(msg)
someActor2.ask(msg)
```

# Tests

Write your tests using [Specs2](http://etorreborre.github.io/specs2/). Each
specification is a single `class` that extends `Specification`. Put each
specification into its own file. Inside each specification, follow these rules:

- create a single `trait` inside your `Specification` class that extends
[`CommonImmutableSpecificationContext`](https://github.com/paypal/cascade/blob/develop/common/src/test/scala/com/paypal/cascade/common/tests/util/CommonImmutableSpecificationContext.scala)
(from [Cascade](https://github.com/paypal/cascade).) Most people name this
trait `Context`.
- Group tests into `class`es, `case class`es or `object`s at your discretion.
- All of your grouped test `class`es, `case class`es and `objects`s should go
inside your specification class and should extend your `Context` trait
- All methods inside your test classes should be wrapped with `apply`.
When you do so, `CommonImmutableSpecificationContext` will execute `before` and
`after` hooks automatically.
- Your tests execute concurrently by default. Only change them to execute
sequentially for a good reason, and document that reason in comments.

Here's an example of the above rules in code:

```scala
class MyTest extends Specification { override def is = s2"""
  add(1, 2) should equal 3 ${Add().equals3()}
  """

  trait Context extends CommonImmutableSpecificationContext

  case class Add() extends Context {
    def equals3 = apply {
      add(1, 2) must beEqualTo(3)
    }
  }
}
```

# Scaladoc, Comments, and Annotations

All classes, objects, traits, and methods should be documented. Generally, follow the documentation guidelines
provided by the Scala Documentation Style Guide on [ScalaDoc](http://docs.scala-lang.org/style/scaladoc.html).

Rules:

* Classes that are instantiated via methods in a companion object **must** include ScalaDoc documentation with a code example.
* Abstract classes *should* be documented with an example of their intended implementation.
* Implicit wrapper classes **must** include ScalaDoc documentation with a code example.
* Public, protected, and package-private methods **must** include ScalaDoc documentation.
* Private methods *should* be documented, however it is left to the discretion of the developer as to the level of documentation.
* All methods **must** include `@throws` annotations if they throw an exception in their normal operation.

Use your best judgment otherwise, and err toward more documentation rather than less.

# Further Reading

Several of these recommendations are adapted from the Scala Documentation Style Guide,
including [Declarations](http://docs.scala-lang.org/style/declarations.html) and
[Indentation](http://docs.scala-lang.org/style/indentation.html). The guidelines presented
in this document should supersede all other style recommendations.
