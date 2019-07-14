---
layout: default
published: true
---
<title>Rucker :: Security Researcher</title>

# $ cat об_этом_всём.txt
{:id="about"}

Привет, я исследователь в области информационной безопасности. <br>
Тут собрана и структирирована полезная информация для всеобщего пользования и образования.

# $ cat статьи.txt
{:id="articles"}


<ul>
{% for post in site.categories.articles_rus %}
<li>{{ post.title }} :: <a href="{{ post.url }}" title="{{ post.description }}">en</a> :: <a href="{{ post.pt }}" title="{{ post.description_pt }}">pt_br</a></li>

{% endfor %}
</ul>

# $ cat чит-таблицы.txt
{:id="cheatsheets"}

<ul>
{% for cheatsheets in site.categories.cheatsheets_rus %}
<li><a href="{{ cheatsheets.url }}" title="{{ cheatsheets.description }}">{{ cheatsheets.title }}</a></li>
 
{% endfor %}
</ul>

# $ cat тулзы.txt
{:id="tools"}

<ul>
{% for tool in site.categories.tools_rus %}
<li><a href="{{ tool.link }}">{{ tool.title }}</a> - {{ tool.description }}</li>
{% endfor %}
</ul>

# $ cat контакт.txt
{:id="contact"}

E-mail:

> lampiaosec[at]riseup[dot]net

Facebook:

> [https://facebook.me/lampiaosec](https://fb.me/lampiaosec)

GitHub:

> [https://github.com/lampiaosec](https://github.com/lampiaosec)

IRC:

> \#lampiaosec at OFTC
