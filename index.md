---
layout: default
published: true
---
<title>Rucker :: Security Researcher</title>

<!-- I'm soming out the socket -->
<!-- Nothing you can do can stop it -->
<!-- I'm on your lap and in your pocket -->
<!-- How you gonna shoot me down -->
<!-- When I guide the rocket? -->

{% if page.url contains 'ru'  %}
### Война в Украине не значит ничего кроме маниакального безумия Путина и не приведёт ни к чему кроме человеческий страданий. Украинцы - герои.
{% else %}
### The war in Ukraine means nothing but Putin's madness and will result in nothing but human suffering. Ukrainians are heroes.
{% endif %}  

# $ whoami
{:id="about"}

{% if page.url contains 'ru'  %}
Привет, я делаю в инфобез и это мой блог.
{% else %}
Hi, I do some info security stuff and this is my blog.
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

<!--
# $ cat tools.txt
{:id="tools"}

<ul>
{% for tool in site.categories.tools %}
<li><a href="{{ tool.link }}">{{ tool.title }}</a> - {{ tool.description }}</li>
{% endfor %}
</ul>
-->

# $ cat contact.txt
{:id="contact"}
Telegram: > [https://t.me/CyberRucker](https://t.me/CyberRucker) <br />
