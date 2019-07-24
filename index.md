---
layout: default
published: true
---
<title>Rucker :: Security Researcher</title>

# $ cat about.txt
{:id="about"}

Hi, I am an information security researcher. Here is collected and structured useful information for common use and education.

Welcome and let's hack all the things :)

# $ cat articles.txt
{:id="articles"}

<ul>
{% for post in site.categories.articles %}
  {% for uefi site.tags.uefi %}
<li><a href="{{ post.url }}" title="{{ post.description }}">{{ post.title }}</a></li>

{% endfor %}
</ul>

# $ cat cheat_sheets.txt
{:id="cheatsheets"}

<ul>
{% for cheatsheets in site.categories.cheatsheets %}
<li><a href="{{ cheatsheets.url }}" title="{{ cheatsheets.description }}">{{ cheatsheets.title }}</a></li>
 
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

Telegram:

> @ru_cker

GitHub:

> [https://github.com/Ruuucker](https://github.com/Ruuucker)
