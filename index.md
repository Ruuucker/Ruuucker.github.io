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
{% for uefi in site.tags.uefi %}

   <li><a href="#changing-dns" id="markdown-toc-changing-dns">Changing DNS</a>      
          <li><a href="#first-step-creating-the-server" id="markdown-toc-first-step-creating-the-server">First Step: Creating the server</a></li>
          <li><a href="#second-step-creating-a-video-page" id="markdown-toc-second-step-creating-a-video-page">Second Step: Creating a video page</a></li>
          <li><a href="#third-informing-the-necessary-referer" id="markdown-toc-third-informing-the-necessary-referer">Third: Informing the necessary referer</a></li>
    
     
<li><a href="{{ uefi.url }}" title="{{ uefi.description }}">{{ uefi.title }}</a></li>
{% endfor %}  

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
