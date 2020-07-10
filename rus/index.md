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

# $ cat методологии.txt
{:id="methods"}

<ul>
  <li>Взломать Windows</li>
<ul>
{% for win_hacks_rus in site.tags.win_hacks_rus %}
     
<li><a href="{{ uefi_rus.url }}" title="{{ uefi_rus.description }}">{{ uefi_rus.title }}</a></li>
{% endfor %}  
</ul>
<br>
</ul>

<ul>
  <li>Взломать Linux</li>
<ul>
{% for linux_hacks_rus in site.tags.linux_hacks_rus %}
     
<li><a href="{{ uefi_rus.url }}" title="{{ uefi_rus.description }}">{{ uefi_rus.title }}</a></li>
{% endfor %}  
</ul>
<br>
</ul>

<ul>
  <li>Взломать Android</li>
<ul>
{% for android_hacks_rus in site.tags.android_hacks_rus %}
     
<li><a href="{{ uefi_rus.url }}" title="{{ uefi_rus.description }}">{{ uefi_rus.title }}</a></li>
{% endfor %}  
</ul>
<br>
</ul>

<ul>
  <li>Взломать сеть</li>
<ul>
{% for net_hacks_rus in site.tags.net_hacks_rus %}
     
<li><a href="{{ uefi_rus.url }}" title="{{ uefi_rus.description }}">{{ uefi_rus.title }}</a></li>
{% endfor %}  
</ul>
<br>
</ul>

<ul>
  <li>Взломать вообще всё</li>
<ul>
{% for hack_all_rus in site.tags.hack_all_rus %}
     
<li><a href="{{ uefi_rus.url }}" title="{{ uefi_rus.description }}">{{ uefi_rus.title }}</a></li>
{% endfor %}  
</ul>
<br>
</ul>

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
