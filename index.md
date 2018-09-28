---
layout: page
title: Home
tagline: Supporting tagline
---
{% include JB/setup %}

Hi,

I'm Ivan Yang, a second-year master student in Carnegie Mellon University(CMU),
    majoring in computer science.

- Open source developer [@github](https://github.com/mrmiywj)
- Rust/C/Python/Ruby/Scala/Haskell
- `Hard-core` programming, like system programming/database/data processing/ streaming
- Programming language theory/Distributed system/Database/Parallel System
- Active developer in Rust community. Contributor of [Rust](https://github.com/rust-lang/rust)/[Servo](https://github.com/servo/servo)/[TiKV](https://github.com/pingcap/tikv)
- Board/ video game lover. I love Nintendo.
- Software Engineer Intern at LinkedIn in Summer 2018.

You are welcome to contact me via:

Email: jsyangwenjie#gmail.com (replace # with @ please)

Github: [@mrmiywj](https://github.com/mrmiywj)

Twitter: [@jsyangwenjie](https://twitter.com/jsyangwenjie)

Zhihu: [@Ivan Yang](https://www.zhihu.com/people/ivanyang-36-58)

You can find my resume [here]({{BASE_PATH}}/resume.pdf).

> Recent Post:

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
