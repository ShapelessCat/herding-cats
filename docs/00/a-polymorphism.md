---
out: polymorphism.html
---

What is polymorphism?
---------------------

### Parametric polymorphism

Nick says:

> In this function `head`, it takes a list of `A`'s, and returns an `A`. And it doesn't matter what the `A` is: It could be `Int`s, `String`s, `Orange`s, `Car`s, whatever. Any `A` would work, and the function is defined for every `A` that there can be.

```scala mdoc:reset
def head[A](xs: List[A]): A = xs(0)
head(1 :: 2 :: Nil)

case class Car(make: String)
head(Car("Civic") :: Car("CR-V") :: Nil)
```

[Haskell wiki](http://www.haskell.org/haskellwiki/Polymorphism) says:

> Parametric polymorphism refers to when the type of a value contains one or more (unconstrained) __type variables__, so that the value may adopt any type that results from substituting those variables with concrete types.

### Subtype polymorphism

Let's think of a function `plus` that can add two values of type `A`:

```scala mdoc
def plus[A](a1: A, a2: A): A = ???
```

Depending on the type `A`, we need to provide different definition for what it means to add them.
One way to achieve this is through subtyping.

```scala mdoc
trait PlusIntf[A] {
  def plus(a2: A): A
}

def plusBySubtype[A <: PlusIntf[A]](a1: A, a2: A): A = a1.plus(a2)
```

We can at least provide different definitions of `plus` for `A`.
But, this is not flexible since trait `Plus` needs to be mixed in at the time of defining the datatype.
So it can't work for `Int` and `String`.

### Ad-hoc polymorphism

The third approach in Scala is to provide an implicit conversion or implicit parameters for the trait.

```scala mdoc
trait CanPlus[A] {
  def plus(a1: A, a2: A): A
}

def plus[A: CanPlus](a1: A, a2: A): A = implicitly[CanPlus[A]].plus(a1, a2)
```

This is truely ad-hoc in the sense that

1. we can provide separate function definitions for different types of `A`
2. we can provide function definitions to types (like `Int`) without access to its source code
3. the function definitions can be enabled or disabled in different scopes

The last point makes Scala's ad-hoc polymorphism more powerful than that of Haskell.
More on this topic can be found at [Debasish Ghosh @debasishg](https://twitter.com/debasishg)'s [Scala Implicits : Type Classes Here I Come](http://debasishg.blogspot.com/2010/06/scala-implicits-type-classes-here-i.html).

Let's look into `plus` function in more detail.
