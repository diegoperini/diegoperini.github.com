---
layout: page
title: Home
tagline:
---
{% include JB/setup %}

Welcome to my personal blog! You can find my latest posts below. Enjoy!

### Latest Posts

<ul class="posts">
    {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
        {{ post.content | strip_html | truncatewords:25 }}<br>
            <a href="{{ post.url }}">Read more...</a><br><br>
    {% endfor %}
</ul>
