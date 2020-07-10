---
layout: default
published: true
---
<title>Rucker :: Security Researcher</title>

# $ whoami
{:id="about"}

Привет, я делаю в инфобез и это мой блог. <br>

# $ cat статьи.txt
{:id="articles"}

<ul>
  <li>EFI</li>
<ul>
{% for uefi_rus in site.tags.uefi_rus %}
     
<li><a href="{{ uefi_rus.url }}" title="{{ uefi_rus.description }}">{{ uefi_rus.title }}</a></li>
{% endfor %}  
</ul>
<br>

{% for articles_rus in site.tags.articles_rus %}
<li><a href="{{ articles_rus.url }}" title="{{ articles_rus.description }}">{{ articles_rus.title }}</a></li>

{% endfor %}
</ul>

# $ cat чит_щит.txt
{:id="cheatsheets"}

<ul>
{% for cheatsheets in site.categories.cheatsheets_rus %}
<li><a href="{{ cheatsheets.url }}" title="{{ cheatsheets.description }}">{{ cheatsheets.title }}</a></li>
 
{% endfor %}
</ul>

# $ cat арсенал.txt
{:id="tools"}

<ul>
{% for tool in site.categories.tools_rus %}
<li><a href="{{ tool.link }}">{{ tool.title }}</a> - {{ tool.description }}</li>
{% endfor %}
</ul>

# $ cat ссылки.txt
{:id="contact"}
<!--
Telegram:

> @ru_cker
-->
GitHub:

> [https://github.com/Ruuucker](https://github.com/Ruuucker)
