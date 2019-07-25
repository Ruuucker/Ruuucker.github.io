---
layout: default
published: true
---
<title>Rucker :: Security Researcher</title>

# $ cat about.txt
{:id="about"}

Hi, Im Rucker and I do some info security. Here is collected and structured useful information for common use and education.

Welcome and let's hack all the things :)

# $ cat articles.txt
{:id="articles"}

<ul>
  <li>EFI</li>
<ul>
{% for uefi in site.tags.uefi %}
     
<li><a href="{{ uefi.url }}" title="{{ uefi.description }}">{{ uefi.title }}</a></li>
{% endfor %}  
</ul>
 <br>
{% for post in site.categories.articles %}
  
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
