---
layout: default
published: true
---
<title>Rucker :: Security Researcher</title>

# $ cat об_этом_всём.txt
{:id="about"}

Привет, я делаю в инфобез и это мой блог. <br>
Тут собрана и структирирована полезная информация для всеобщего пользования и образования.

# $ cat статьи.txt
{:id="articles"}

<ul>
{% for post in site.categories.articles_rus %}
<li><a href="{{ post.url }}" title="{{ post.description }}">{{ post.title }}</a></li>

{% endfor %}
</ul>

# $ cat чит_таблицы.txt
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

Telegram:

> @ru_cker

GitHub:

> [https://github.com/Ruuucker](https://github.com/Ruuucker)
