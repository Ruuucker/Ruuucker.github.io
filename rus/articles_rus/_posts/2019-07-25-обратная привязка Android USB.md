---
title: Обратный модем Android USB
layout: post
categories: articles
tags:
  - articles_rus
  - usb
published: true
---
# Оглавление
{:.no_toc}

* toc
{:toc}

# Введение

Это руководство описывает метод получения Интернет соединения от ПК к смартфону/планшету на Android с помощью USB-кабеля. Эта техника называется обратным USB модемом. Она отличается от USB-модема, с помощью которого ты обычно проводишь Интернет с Android на ПК. Есть много причин, почему эта техника нужна:

- Нет мобильной/Wi-Fi сети или на компьютере нет адаптера Wi-Fi
- Нельзя использовать Wi-Fi точку доступа для этих целей, например, компания не позволяет создавать на работе точки подключения.
- Нет желания тратить деньги на мобильный тариф и хочется более стабильное и быстрое соединение чем Wi-Fi
- Не хочешь чтобы Android слишком сильно нагревался из-за Wi-Fi
- Хочешь чтобы Android заряжался во время использования
...


Этот метод работает по принципу разделения доступа в Интернет. Если твоё Android-устройство способно подключаться к ПК с помощью USB в качестве модема, тогда будет работать и обратный USB модем.

Преимущества:

- Не требуется дополнительное ПО на ПК и не остается следов. Представьте когда во время работы нужно установить на компьютер какое-то дрянное приложение чтобы установить соединение для этой задачи.

- Работает с Windows, Linux и Mac OS X

- Ты знаешь что делаешь. Кто знает что "легкие-в-использовании" приложения делают если им не удалось помочь с этой задачей?


Недостатки:

- Нужно будет ввести некоторые комманды в Android.

Если готов, давай начнем!

# Зависимости:

- Рутированный Android девайс с возможностью "USB модема". Проверь настройки Android - Беспроводные & Сеть - Модем & Точка доступа. Многие стандартные ROM (Read only Memory/Пямять только для чтения) отключают эту возможность. Это нужно как то включить (рутировать твой Android и использовать приложение для включения возможности записи или перемещения ROM). ROM от CyanogenMod всегда имеет эту возможность.
Важно помнить, что когда ты подключаешь свой Android и включаешь "USB-модем", он выглядит как сетевой USB-адаптер, а не запоминающее устройство или мультимедийное устройство.

- ПК с работающим интернет соединением.
- USB кабель для подключения Android к ПК.
- Эмулятор терминала на твоем Android. Если ты не хочешь вводить команды на сенсорном экране с помощью эмулятора терминала, ты можешь использовать клавиатуру ПК для ввода команд с помощью "adb shell". adb - это часть Android SDK которая доступна для скачивания из Google. Для использования adb, тебе нужно включить "USB дебагинг" на твоём Android.
- Опционально, BusyBox на твоём Android.

## Шаг 1: 

- Подключи Android к ПК через USB-кабель и включи "USB-модем". Ты по-прежнему можешь включить эту опцию, даже если 3G/Wi-Fi на твоём Android выключен.

- Если используется Linux (Ubuntu), то не нужно ничего устанавливать. Апплет NetworkManager попытается установить соединение на новом обнаруженном устройстве проводной сети.

- Если Windows, то ОС автоматически выполнит поиск Windows Update и установит драйвер для тебя. Ты можешь пропустить поиск Windows Update и установить вручную уже включенный драйвер от Microsoft.

- В окне «Установка драйвера» нажми «Обзор моего компьютера», затем «Позвольте мне выбрать ...», выбери «Сетевые адаптеры», сними флажок «Показать совместимое оборудование», посмотри на «Microsoft Corporation» в левом столбце и выбери «Удаленное устройство совместимое с NDIS» в правом столбце. Ты можешь установить или обновить драйвер из диспетчера устройств в Windows.


- Если используется Mac, установи драйвер HoRNDIS. Ты будешь уведомлен о новом сетевом интерфейсе. Нажми «Настройки сети» в диалоговом окне чтобы добавить его в список известных интерфейсов. Затем «Применить»

- Если используется Linux без графического интерфейса или NetworkManager, запусти эти команды как root (или используй sudo):

~~~
ifconfig usb0 10.42.0.1 netmask 255.255.255.0
~~~

(Предполагается что нет другого сетевого адаптера USB, в противном случае, твой Android может быть usb1, usb2...)


	echo 1 > /proc/sys/net/ipv4/ip_forward
    
Команда для sudo будет:

~~~
sudo 'echo 1 > /proc/sys/net/ipv4/ip_forward'

iptables -t nat -F
iptables -t nat -A POSTROUTING -j MASQUERADE
~~~

## Шаг 2:

- Если ты используешь Linux, нажми на апплет NetworkManager в правом верхнем углу экрана и выберите «Редактировать соединения...». На вкладке «Проводной» выбери новое установленное соединение (будь осторожен, не LAN Ethernet соединение) и нажми «Изменить...». На вкладке «Настройки IPv4» выбери «Доступ к другим компьютерам» в качестве метода. Нажми «Сохранить». NetworkManager восстановит соединение и назначит твоему ПК IP-адрес в этом сетевом соединении USB, по умолчанию: 10.42.0.1. Оставь подключение к Интернету (проводное или беспроводное) без изменений.

- Если ты используешь Windows, открой «Сетевые подключения» на панели управления. Это несколько отличается от настройки в Linux. Щелкни правой кнопкой мыши на твоё подключение к Интернету. Я предполагаю, что ты используешь десктоп без адаптера Wi-Fi, поэтому щелкни правой кнопкой мыши по LAN-соединению Ethernet с Интернетом и выбери «Свойства». На вкладке «Общий доступ» (или «Дополнительно» для Windows XP) нажми «Разрешить другим пользователям сети подключаться через...», затем выбери USB-соединение в раскрывающемся списке ниже и нажми ОК. Windows автоматически настроит сетевое USB-соединение и назначит ему IP-адрес, по умолчанию для Windows 7: 192.168.137.1, для Windows 10: 192.168.0.1. Ты можете видеть, что твоё интернет-соединение теперь «Общее», а USB-соединение теперь «Неопознанная сеть».

- Если ты используешь Mac, откройте Системные настройки - Сеть. Если ты установил HoRNDIS, то увидишь новый сетевой интерфейс соответствующий твоему USB-соединению. При включенном «Использование DHCP» в качестве конфигурации Ipv4, он может быть уже подключен. Вернись в Системные настройки, нажми «Общий доступ». Выбери «Общий Интернет». Выбери подключение к Интернету (Ethernet или Airport...) в разделе «Поделиться своим подключением из» и выбери интерфейс USB-подключения в разделе «К компьютерам, использующим». Mac назначит твоему USB-интерфейсу IP-адрес, по умолчанию: 192.168.2.1.

- Если ты используешь Linux без графического интерфейса или NetworkManager, ты уже выполнил все настройки ПК на шаге 1.

Настройка твоего ПК завершена !

## Шаг 3:
Open Terminal Emulator on your Android. Type:

	su
    
Type the following command in Terminal Emulator, the same for all PC operating systems:

	netcfg rndis0 dhcp
    
The name for usb interface inside Android may vary. It is usually rndis0 or usb0. Type:

	busybox ifconfig
    
to identify the name.
Use OLD instructions below when automatical dhcp method does not work.
[OLD]Type these following commands in Terminal Emulator:
For Linux PC:

~~~
ifconfig rndis0 10.42.0.2 netmask 255.255.255.0
route add default gw 10.42.0.1 dev rndis0
~~~

If route fails, try:

	busybox route add default gw 10.42.0.1 dev rndis0
    
For Windows PC, use the same above commands, replace 10.42.0.2 by 192.168.137.2 (192.168.0.2 for Windows XP), replace 10.42.0.1 by 192.168.137.1 (192.168.0.1 for Windows XP)
For Mac PC, replace 10.42.0.2 by 192.168.2.2, replace 10.42.0.1 by 192.168.2.1
Now you can close Terminal Emulator and start the browser for Internet.

Some applications (download in Google Play, GMail, Facebook...) don't recognize Internet connection. You can try this way (WARNING: NOT TESTED):
- Enable temporarily 3G connection on your Android
- Type:

~~~
ifconfig rmnet0 0.0.0.0
~~~

The name for 3G interface inside Android may vary: ppp0, rmnet0... Type:

	busybox ifconfig
    
to identify the name.
before ifconfig rmnet0 ... above.
This will make applications see your Internet connection via USB as 3G!

USB tethering settings on Android will be reverted automatically when you unplug USB cable. To revert back settings on PC, uncheck "Allow other network users to connect through..." on Windows, "Internet sharing" on Mac, change from "Shared to other computers" back to "Automatically (DHCP)", or simply delete USB connection from NetworkManager on Linux.

# Simplified instructions (Mac OSX):

  > Mac: Install HorNDIS package.<br>
   Device: Turn on USB tethering under Settings > More Settings > Tethering and portable hotspot > USB tethering<br>
   Plug in USB from device to computer.<br>
   Mac: Connect device under System Preferences > Network > SAMSUNG_Android <br>
   Mac: Add Internet Sharing under System Preferences > Sharing > Service > Internet Sharing<br>
   Device: Root your device<br>
   Device: "netcfg rndis0 dhcp"<br>
   Device: "netcfg" to see the ip address of rndis0<br>
   Mac: "ifconfig" to see the ip address of your computer; use this ip address as mc_creator's url input
