---
layout: default
title: "첫 페이지"
---
# 다시 시작하는 블로그!
직접 하나씩 만들어갑니다.

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%Y-%m-%d" }}
    </li>
  {% endfor %}
</ul>