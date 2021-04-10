---
out: Coproduct.html
---

  [@milessabin]: https://twitter.com/milessabin
  [scala-union-types]: http://www.chuusai.com/2011/06/09/scala-union-types-curry-howard/
  [alacarte]: http://www.cs.ru.nl/~W.Swierstra/Publications/DataTypesALaCarte.pdf
  [@wouterswierstra]: https://twitter.com/wouterswierstra

### Coproduct

One of the well known duals is *coproduct*, which is the dual of product. Prefixing with "co-" is the convention to name duals.

Here's the definition of products again:

> **Definition 2.15.** In any category **C**, a product diagram for the objects A and B consists of an object P and arrows p<sub>1</sub> and p<sub>2</sub><br>
> ![product diagram](files/day17-product-diagram.png)<br>
> satisfying the following UMP:
>
> Given any diagram of the form<br>
> ![product definition](files/day17-product-definition.png)<br>
> there exists a unique u: X => P, making the diagram<br>
> ![product of objects](files/day17-product-of-objects.png)<br>
> commute, that is, such that x<sub>1</sub> = p<sub>1</sub> u and x<sub>2</sub> = p<sub>2</sub> u.

Flip the arrows around, and we get a coproduct diagram:<br>
![coproducts](files/day17-coproducts.png)

Since coproducts are unique up to isomorphism, we can denote the coproduct as *A + B*, and *[f, g]* for the arrow *u: A + B => X*.

> The "coprojections" *i<sub>1</sub>: A => A + B* and *i<sub>2</sub>: B => A + B* are usually called *injections*, even though they need not be "injective" in any sense.

Similar to the way products related to product type encoded as `scala.Product`, coproducts relate to the notion of sum type, or disjoint union type.

#### Algebraic datatype

First way to encode *A + B* might be using sealed trait and case classes.

```scala mdoc
sealed trait XList[A]

object XList {
  case class XNil[A]() extends XList[A]
  case class XCons[A](head: A, rest: XList[A]) extends XList[A]
}

XList.XCons(1, XList.XNil[Int])
```

#### Either datatype as coproduct

If we squint `Either` can be considered a union type. We can define a type alias called `|:` for `Either` as follows:

```scala mdoc
type |:[+A1, +A2] = Either[A1, A2]
```

Because Scala allows infix syntax for type constructors, we can write `Either[String, Int]` as `String |: Int`.

```scala mdoc
val x: String |: Int = Right(1)
```

Thus far I've only used normal Scala features only. Cats provides a typeclass called `cats.Inject` that represents injections *i<sub>1</sub>: A => A + B* and *i<sub>2</sub>: B => A + B*. You can use it to build up a coproduct without worrying about Left or Right.

```scala mdoc
import cats._, cats.data._, cats.syntax.all._

val a = Inject[String, String |: Int].inj("a")

val one = Inject[Int, String |: Int].inj(1)
```

To retrieve the value back you can call `prj`:

```scala mdoc
Inject[String, String |: Int].prj(a)

Inject[String, String |: Int].prj(one)
```

We can also make it look nice by using `apply` and `unapply`:

```scala mdoc
lazy val StringInj = Inject[String, String |: Int]

lazy val IntInj = Inject[Int, String |: Int]

val b = StringInj("b")

val two = IntInj(2)

two match {
  case StringInj(x) => x
  case IntInj(x)    => x.show + "!"
}
```

The reason I put colon in `|:` is to make it right-associative. This matters when you expand to three types:

```scala mdoc
val three = Inject[Int, String |: Int |: Boolean].inj(3)
```

The return type is `String |: (Int |: Boolean)`.

#### Curry-Howard encoding

An interesting read on this topic is [Miles Sabin (@milessabin)][@milessabin]'s [Unboxed union types in Scala via the Curry-Howard isomorphism][scala-union-types].

#### Shapeless.Coproduct

See also [Coproducts and discriminated unions](https://github.com/milessabin/shapeless/wiki/Feature-overview:-shapeless-2.0.0#coproducts-and-discriminated-unions) in Shapeless.

#### EitherK datatype

There's a datatype in Cats called `EitherK[F[_], G[_], A]`, which is an either on type constructor.

In [Data types à la carte][alacarte] [Wouter Swierstra (@wouterswierstra)][@wouterswierstra] describes how this could be used to solve the so-called Expression Problem.

That's it for today.
