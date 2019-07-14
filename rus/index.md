---
layout: default
published: true
theme: Нaчaло
---
<title>Rucker :: Security Researcher</title>

# $ cat об_этом_всём.txt
{:id="about"}

Привет, я исследователь в области информационной безопасности. <br>
Тут собрана и структирирована полезная информация для всеобщего пользования и образования.

# $ cat статьи.txt
{:id="articles"}






<ul id="markdown-toc">
  <li>{{site.theme}}</a>    <ul>
      <li><a href="#motivation" id="markdown-toc-motivation">Motivation</a></li>
      <li><a href="#the-issue" id="markdown-toc-the-issue">The Issue</a></li>
    </ul>
  </li>
  <li><a href="#theory" id="markdown-toc-theory">Theory</a></li>
  <li><a href="#practice" id="markdown-toc-practice">Practice</a>    <ul>
      <li><a href="#changing-dns" id="markdown-toc-changing-dns">Changing DNS</a>        <ul>
          <li><a href="#first-step-creating-the-server" id="markdown-toc-first-step-creating-the-server">First Step: Creating the server</a></li>
          <li><a href="#second-step-creating-a-video-page" id="markdown-toc-second-step-creating-a-video-page">Second Step: Creating a video page</a></li>
          <li><a href="#third-informing-the-necessary-referer" id="markdown-toc-third-informing-the-necessary-referer">Third: Informing the necessary referer</a></li>
        </ul>
      </li>
    </ul>
  </li>
  <li><a href="#another-methods" id="markdown-toc-another-methods">Another methods</a>    <ul>
      <li><a href="#modifying-the-http-request-through-burpsuite" id="markdown-toc-modifying-the-http-request-through-burpsuite">Modifying the HTTP request through BurpSuite</a></li>
      <li><a href="#make-the-request-through-curl--wget" id="markdown-toc-make-the-request-through-curl--wget">Make the request through cURL &amp; wget</a></li>
      <li><a href="#building-the-request-with-ncat-from-the-scratch" id="markdown-toc-building-the-request-with-ncat-from-the-scratch">Building the request with ncat from the scratch</a></li>
    </ul>
  </li>
  <li><a href="#conclusion" id="markdown-toc-conclusion">Conclusion</a></li>
  <li><a href="#referencies" id="markdown-toc-referencies">Referencies</a></li>
</ul>










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
