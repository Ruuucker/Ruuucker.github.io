---
layout: default
published: true
---
<title>Rucker :: Security Researcher</title>

# $ cat about.txt
{:id="about"}

Hi, I am an information security researcher. Here is collected and structured useful information for general use and education.

Welcome and let's hack all the things :)

# $ cat articles.txt
{:id="articles"}

<ul>
{% for post in site.categories.articles %}
<li>{{ page.title }}</li>
<li>{{ post.title }} :: <a href="{{ post.url }}" title="{{ post.description }}">en</a> :: <a href="{{ post.pt }}" title="{{ post.description_pt }}">pt_br</a></li>

{% endfor %}
</ul>

# $ cat cheat_sheets.txt
{:id="cheatsheet"}

<ul>
{% for member in site.categories.team reversed %}
<li id="{{ member.title }}">{{ member.title }}
<ul>
<li>{{ member.mail }}</li>
<li><a href="https://github.com/{{ member.github }}">https://github.com/{{ member.github }}</a></li>
<li><a href="{{ member.site }}">{{ member.site }}</a></li>
</ul>
</li>
{% endfor %}
</ul>

# $ cat tools.txt
{:id="tools"}

<ul>
{% for tool in site.categories.tools %}
<li><a href="{{ tool.link }}">{{ tool.title }}</a> - {{ tool.description }}</li>
{% endfor %}
</ul>

# $ cat contact.txt
{:id="contact"}

E-mail:

> lampiaosec[at]riseup[dot]net

Facebook:

> [https://facebook.me/lampiaosec](https://fb.me/lampiaosec)

GitHub:

> [https://github.com/lampiaosec](https://github.com/lampiaosec)

IRC:

> \#lampiaosec at OFTC
