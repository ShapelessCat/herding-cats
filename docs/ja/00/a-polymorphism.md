---
out: polymorphism.html
---

多相性って何?
-----------

### パラメータ多相 (parametric polymorphism)

Nick さん曰く:

> この関数 `head` は `A` のリストを取って `A` を返します。`A` が何であるかはかまいません。`Int` でもいいし、`String` でもいいし、`Orange` でも `Car` でもいいです。どの `A` でも動作し、存在可能な全ての `A` に対してこの関数は定義されています。

```console:new
scala> def head[A](xs: List[A]): A = xs(0)
scala> head(1 :: 2 :: Nil)
scala> case class Car(make: String)
scala> head(Car("Civic") :: Car("CR-V") :: Nil)
```

[Haskell wiki](http://www.haskell.org/haskellwiki/Polymorphism) 曰く:

> パラメータ多相 (parametric polymorphism) とは、ある値の型が 1つもしくは複数の (制限の無い) **型変数**を含むことを指し、その値は型変数を具象型によって置換することによって得られる型ならどれでも採用することができる。

### 派生型による多態 (subtype polymorphism)

ここで、型 `A` の 2つの値を足す `plus` という関数を考える:

```scala
scala> def plus[A](a1: A, a2: A): A = ???
plus: [A](a1: A, a2: A)A
```

型 `A` によって、足すことの定義を別々に提供する必要がある。これを実現する方法の一つが派生型 (subtyping) だ。

```console
scala> trait PlusIntf[A] {
         def plus(a2: A): A
       }
scala> def plusBySubtype[A <: PlusIntf[A]](a1: A, a2: A): A = a1.plus(a2)
```

これで `A` の型によって異なる `plus` の定義を提供できるようにはなった。しかし、この方法はデータ型の定義時に `Plus` を mixin する必要があるため柔軟性に欠ける。例えば、`Int` や `String` には使うことができない。

### アドホック多相

Scala における3つ目の方法は trait への暗黙の変換か暗黙のパラメータ (implicit parameter) を使うことだ。

```console
scala> trait CanPlus[A] {
         def plus(a1: A, a2: A): A
       }
scala> def plus[A: CanPlus](a1: A, a2: A): A = implicitly[CanPlus[A]].plus(a1, a2)
```

これは以下の意味においてまさにアドホックだと言える

1. 異なる `A` の型に対して別の関数定義を提供することができる
2. (`Int` のような) 型に対してソースコードへのアクセスが無くても関数定義を提供することができる
3. 異なるスコープにおいて関数定義を有効化したり無効化したりできる

この最後の点によって Scala のアドホック多相性は Haskell のそれよりもより強力なものだと言える。このトピックに関しては [Debasish Ghosh さん (@debasishg)](https://twitter.com/debasishg) の[Scala Implicits: 型クラス、襲来](http://eed3si9n.com/ja/scala-implicits-type-classes)参照。

この `plus` 関数をより詳しく見ていこう。
