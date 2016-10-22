---
layout: post
category: Rust
title: String in Rust
comments: true
tags: Rust
---

Recently, I'm working on [TiKV](https://github.com/pingcap/tikv), a distributed KV database based on Rust. I found that encode/decode is an important issue when resolving stuff related to string. Here are some notes about `str` and `String` in Rust.

# Data Representation
As we all know, all data is stored as 0/1 for a bit in either memory or disk. To represent numbers, it can be transferred into binary format plus some mechanism called
twos-complement notation. But wait, how can we know some bits like `01100001` represented 'a'. Communities have tried to come into a contract which is a bijection between one or more bytes(even arbitrary bits) and characters. Then ASCII comes out. But it only resolve the English characters. Other languages like Chinese and Japanese also have this issue. Then [Unicode](https://en.wikipedia.org/wiki/Unicode) was invented to represent all possible characters on the earth. In a word,
> All characters are stored as bytes in memory, whatever how they encoded.

# strings in Rust
Anyone who learns Rust even just for five minutes knows that there are two 'strings' in Rust, `str` and `String`. It may confused a lot of new comers, included me. But it exactly reflects how subtle the Rust's memory management is. Let's dive into them.

## Definition
`str`, also called `string slice`, which is a sequence of valid UTF-8 bytes. You can imagine it like an array of bytes, but not one byte for on character, since it's UTF-8 encoded. `str` is not useful in Rust. But `&str` is spreaded in Rust code. There are two types of `&str`, `&'static str` and `&'a str`. The first one can be determined when compiling. So it can be stored in the program and loaded when used. The later one can only be determined on runtime, and is dynamic sized. So actually `&'a str` is a pointer to a [DST](http://smallcultfollowing.com/babysteps/blog/2014/01/05/dst-take-5/) and some usable runtime information.

`String` is exactly what we usually think the way `String` implemented in languages. Actually it's just [a vector of u8](https://github.com/rust-lang/rust/blob/master/src/libcollections/string.rs#L262), heap allocated, which can be expanded automatically. You can also find the implementation of String [here](https://github.com/rust-lang/rust/blob/master/src/libcollections/string.rs).

## The rust way
Since we have the two options of strings. What's the best choice of the two options when writing code?

### In function
I do think `&str` is preferred when the string will not be modified in place. For example, there are no operations like replace some characters in it with others. Besides, `&str` is preferred when defining functions, since there are ways to convert `String` to `&str`.

#### Deref coercion
[Rust doc](https://doc.rust-lang.org/std/string/struct.String.html#deref) points out a way to convert `String` to `&str`. You can imagine `&` as a special operator here. It accepts a String as an argument and then generates a `&str` with the same content.

#### Into conversion
By `Deref` coercion, we still need to explicitly call `&` on a String. Is there any way we can define a function that accepts both `&str` or `String`.

Of course. [https://doc.rust-lang.org/std/convert/trait.Into.htmlInto]() trait will help you. If `Into<U> for T` is implemented, it means that there is a function `into` for T to be converted into `U`. Here is an example function that accepts both `String` and `&str`, since `Into<String> for &'a str` is implemented by `From<'a str> for String`.

{% highlight rust %}
fn print_name<S: Into<String>>(name: S) {
  println!("My name is {}", name.into());
}
{% endhighlight %}

But here comes other two issues.
- It requires more memory than `&str`
- `name` in the function cannot be treated as `String`. You must call `into()` to convert it into String explicitly.

But of course, it simplify the interface of functions.

### In struct
There are also two options for a string in a struct. I'm also not sure which is better. It will be depended on the situation.

- Do you need to use the string member outside the struct? If so, `String`, otherwise `&str`. It can also be interpreted as `Do you need the ownership` of the member?
- Consider the size of the type. If it's large, then passing it as a reference will be much cheaper.
