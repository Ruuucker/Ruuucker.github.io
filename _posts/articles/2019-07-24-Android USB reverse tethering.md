---
published: false
layout: post
categories: articles
tags:
  - usb
title: Android USB reverse tethering
---
# Summary
{:.no_toc}

* toc
{:toc}

# Introduction

This guide is intended to help you share Internet connection from your PC to your Android smartphone/tablet via USB cable. This technique is called USB reverse tethering. It is different from USB tethering in which you share Internet from your Android to your PC. There are many reasons why you need this technique working:

- You don't have mobile/wifi network and your PC doesn't have any wifi adapter
- You can't do tethering over wifi, for example, your company doesn't let you make any hotspot at work
- You don't want to spend money for a mobile plan, and you want a more stable and faster Internet connection than wifi
- You don't want your PC and your Android to become too hot because of wifi
- You want your Android charged while in use
...


You have searched and found some applications/tools/solutions, you followed exactly their instructions but finally you were frustrated because they didn't work, here I come for a new method. 

This method works following Internet connection sharing basics. If your Android device is able to do USB tethering, you will be able to do USB reserve tethering with this method!
Advantages:
- No additional software on your PC. Leave no trace on your PC. Imagine when you are at work and you have to install some crappy "toy" application on your PC to estiblish a tunnel connection for this task.
- Works with Windows, Linux and Mac OS X
- You know what you do. Who knows what the "easy-to-use" tools do when they fail to help you?
Disadvantages:
- You have to type some commands on your Android.
If you are ready, let's start!

# Requirements:

- Rooted Android device with "USB tethering" capability. Check in Android Settings - Wireless & networks - Tethering & portable hotspot. Many stock ROMs disable this capability. You must enable it somehow (root your Android and use an application to enable or replace the stock ROM). CyanogenMod ROMs always have this capability. The important thing to remember is when you connect your Android and enable "USB tethering", it appears as a USB network adapter, not a mass storage or media device.
- PC with a working Internet connection.
- USB cable to connect your Android to your PC.
- Terminal Emulator on your Android. If you don't want to type commands on your touchscreen with Terminal Emulator, you can use your PC keyboard to enter commands with "adb shell". adb is a part of Android SDK which is available for download from Google. To use adb, you need to enable "USB debugging" on your Android.
- Optional, BusyBox on your Android.

Step 1: Connect your Android to PC by USB cable and enable "USB tethering". You are still allowed to enable this option even when your 3g/wifi on your Android is off.
- If you are using Linux (Ubuntu), you don't need to install anything. NetworkManager applet will try to establish a connection on the new detected wired network device.
- If you are using Windows, Windows will automatically search Windows Update and install driver for you. You can skip Windows Update search and install manually an already included driver from Microsoft. In Install Driver window, click Browse My Computer, then Let me pick..., select Network Adapters, uncheck Show Compatible Hardware, look at "Microsoft Corporation" at the left column, and choose Remote NDIS Compatible Device from the right column. You can install or update a driver from Device Manager in Windows.
- If you are using Mac, install driver HoRNDIS. You will be notified about a new network interface. Click "Network Preferences" in the dialog to add it to known interfaces list. Then "Apply".
- If you are using Linux without GUI or NetworkManager, run these commands as root (or use sudo):

Code:

	ifconfig usb0 10.42.0.1 netmask 255.255.255.0
    


`test`

