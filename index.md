---
layout: page
title: Home
tagline: Supporting tagline
---
{% include JB/setup %}

Hi,

I'm Ivan Yang, an undergraduate student in Shanghai Jiao Tong University(SJTU), majoring at computer science.

I'm an active developer on [github](https://github.com/mrmiywj). I love C/Python/Scala/Haskell. I'm interested in `hard-core` programming, like server programming and system programming.

My research interest focuses on programming language theory and distributed system.

I dislike data science. I insist computer science is related to maths and system, not data.

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



