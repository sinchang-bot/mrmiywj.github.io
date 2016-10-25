---
layout: page
title: Home
tagline: Supporting tagline
---
{% include JB/setup %}

Hi,

I'm Ivan Yang, an undergraduate student in Shanghai Jiao Tong University(SJTU),
    majoring in computer science.

- Open source developer [@github](https://github.com/mrmiywj)
- Rust/C/Python/Ruby/Scala/Haskell
- `Hard-core` programming, like system programming/database/programming languages
- Programming language theory/Distributed system/Database/Deep learning
- Active developer in Rust community. Contributor of [Rust](https://github.com/rust-lang/rust)/[Servo](https://github.com/servo/servo)/[TiKV](https://github.com/pingcap/tikv)
- Tabletop game lover
- Single now
- Prepare to apply for master programs in the U.S.

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
