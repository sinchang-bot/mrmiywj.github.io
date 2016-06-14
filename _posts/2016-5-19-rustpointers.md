---
layout: post
category: Rust
title: Various guarantees in Rust
comments: true
tags: Rust
---

Since I promised [@arrowrowe](http://arrowrowe.me/) to write a series of posts about introduction Rust to Chinese university students, though I'm busy with finals and TOEFL, I still planned to write some posts about my learning experience of Rust language.

When I was a new comer to Rust, everything was easy for me to understand except so many guarantees in Rust, like `Rc`, `Cell` and so on. I do believe these notions are not easy to handle for the guys from either C-like languages like C/C++, Java, Python or functional languages. I hope this post will help you organize all these notions.

# Ownership, reference and borrowing

This is easy to understand. Since Rust have the property called [Variable bindings](https://doc.rust-lang.org/book/variable-bindings.html), variables have the `Ownership` of what they are bound to. Then, you are allowed to give the ownership out to other variables or pass it as an argument. Besides, the ownership can also be borrowed out. If you follow the rules Rust provided, congrats! Your code will be safe due to the guarantees of Rust compiler. The borrow checker will help you handle all the things!

{% highlight rust %}
fn f(x: &mut usize) {
    *x = 2
}
fn main() {
    let mut x = 1;
    f(&mut x);
    println!("{}", x);
}
{% endhighlight %}

# Box

Since `Box` is easy to understand, which is just allocating memory chunks on heap, it still stunned me when I fisrt met it. Why `Box` here?

In short, when you want to call a `new` in C++, you should try to use `Box` here.

More precisely, `Box` is useful in recursive data structs. IMP, `Box` is needed since Rust language wants to have the same powerful expreesion with functional languages like Scala and OCaml as well as the same memory control ability with C/C++.

Let's recall how to write a linked list in Haskell and C++.

{% highlight haskell %}
data MyList a = Cons a (MyList a)
              | MyNil deriving (Show, Eq)
{% endhighlight %}

{% highlight C++ %}
class List {
    int data;
    List* next;
}
{% endhighlight %}

Take a look at the difference between them. Since Haskell compiler helps us manage the memory, we do not need to point out that the next list is actually a pointer to a memory chunk. But it's different in C++. Rust combines these two.

{% highlight rust %}
pub enum List {
    Emtpy,
    Elem(isize, Box<List>),
}
{% endhighlight %}

So remember, `Box` is only needed in recursive data structs!

# Cell

Now consider another question, when you want to construct a binary tree structure. A little more, you want your nodes has a pointer to its parents. Here comes a issue, actually you cannot hold a pointer in a struct in Rust! No such pointer!

So we need `Rc`, which means `Reference Counting`. `Reference Counting` is a garbage collection algorithm. It will maintain a counter in this variable, recording the number of pointers pointing to this value. Let's see how to 


> To be continued
