---
layout: post
category: Scala
title: Type class in Scala
comments: true
tags: Scala FP
---

Type class is an important functional programming engineering idea. It's designed to support [Ad hoc polymorphism](https://en.wikipedia.org/wiki/Ad_hoc_polymorphism). It's first introduced in Haskell. It's easy to support it in Scala.

Let me give you a example implements type class `Show`.

First of all, define a `Show` trait.

{% highlight scala %}
trait Show[A]{
  def show(s: A): String
}
{% endhighlight %}

Then, for example, we want to log the class information which implements `Show`.

{% highlight scala %}
def log[A](a: A)(implicit s: Show[A]) = println(s.show(a))
{% endhighlight %}

Now, for each class we want to support the function, just define an implicit object extending `Show`.

{% highlight scala %}
case class Person(name: String)
implicit val personShow = new Show[Person]{
  def show(p: Person) = p.name
}
{% endhighlight %}

Done! You can just use `log(p)` to print p's info.

However, for some types, provding a `Show` instance depends on some other types' `Show` instances.

For instance, we could implement `Show` for `Option`:
{% highlight scala %}
case class Person(name: String)
implicit val optionShow[A](implicit sa: Show[A]) = new Show[Option[A]]{
  def show(oa: Option[A]) = a match{
    case None => "None"
    case Some(a) => "Some(" + sa.show(a) + ")"
  }
}
{% endhighlight %}

Now, you can log Option classes.
{% highlight scala %}
scala> log(Option(Person("Ivan")))
Some(Ivan)
{% endhighlight %}

Elegant!