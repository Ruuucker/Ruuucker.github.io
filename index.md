---
layout: default
published: true
---
<title>Rucker :: Security Researcher</title>

<--dfdf-->

# $ whoami
{:id="about"}

{% if page.url contains 'ru'  %}
Привет, я делаю в инфобез и это мой блог.
{% else %}
Hi, I do some info security shit and this is my blog.
{% endif %}

# $ cat articles.txt
{:id="articles"}

<ul>
  <li>EFI</li>
<ul>

{%  for uefi in site.tags.uefi  %}
    {% if page.url contains 'ru'  %}
        {% if uefi.ru == true %}
             <li><a href="{{ uefi.url }}" title="{{ uefi.description }}">{{ uefi.title }}</a></li>
        {% endif %}
    {% else %}
        {% if uefi.ru != true %}
            <li><a href="{{ uefi.url }}" title="{{ uefi.description }}">{{ uefi.title }}</a></li>
        {% endif %}
    {% endif %}
{% endfor %}
  
</ul>
</ul>

<ul>
  <li>Windows</li>
<ul>

{%  for windows in site.tags.windows  %}
    {% if page.url contains 'ru'  %}
        {% if windows.ru == true %}
             <li><a href="{{ windows.url }}" title="{{ windows.description }}">{{ windows.title }}</a></li>
        {% endif %}
    {% else %}
        {% if windows.ru != true %}
            <li><a href="{{ windows.url }}" title="{{ windows.description }}">{{ windows.title }}</a></li>
        {% endif %}
    {% endif %}
{% endfor %}
  
</ul>
<br>

{% for common in site.tags.common %}
    {% if page.url contains 'ru'  %}
        {% if common.ru == true %}
             <li><a href="{{ common.url }}" title="{{ common.description }}">{{ common.title }}</a></li>
        {% endif %}
    {% else %}
        {% if common.ru != true %}
            <li><a href="{{ common.url }}" title="{{ common.description }}">{{ common.title }}</a></li>
        {% endif %}
    {% endif %}
{% endfor %}

</ul>
<!--
//# $ cat methodolodys.txt
//{:id="methods"}

//Soon...
-->
# $ cat cheat_sheets.txt
{:id="cheatsheets"}

<ul>
  
{% for cheatsheets in site.categories.cheatsheets %}
    {% if page.url contains 'ru'  %}
        {% if cheatsheets.ru == true %}
              <li><a href="{{ cheatsheets.url }}" title="{{ cheatsheets.description }}">{{ cheatsheets.title }}</a></li>
        {% endif %}
    {% else %}
        {% if cheatsheets.ru != true %}
              <li><a href="{{ cheatsheets.url }}" title="{{ cheatsheets.description }}">{{ cheatsheets.title }}</a></li>
        {% endif %}
    {% endif %}
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
<!--
Telegram:

> @
-->
GitHub: > [https://github.com/Ruuucker](https://github.com/Ruuucker)
