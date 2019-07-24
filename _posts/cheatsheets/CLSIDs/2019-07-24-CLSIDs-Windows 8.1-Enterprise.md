---
published: false
title: CLSIDs Windows 8.1 Enterprise
layout: post
categories: cheatsheets
tags:
  - clsid
---
# This is a baseline for Windows 8.1 Enterprise x64 with Office 2013.


| COM Name           | CLSID         | InProcServer32|
| :------------------| :-----------  | :-----------  |
| SOFTWARE\Classes\CLSID  | | | This has not always been created in HKEY_CURRENT_USER |
| SOFTWARE\Classes\CLSID\<CLSID>  | | | Create your own CLSID, not one that is already being used on the system |
| SOFTWARE\Classes\CLSID\<CLSID>  |(Default)| Name of Class |This is optional. Many classes have names |  	 
| SOFTWARE\Classes\CLSID\<CLSID>\InProcServer32                                   
|
| SOFTWARE\Classes\CLSID\<CLSID>\InProcServer32 | (Default) | *{Path To Dll}* | This is the path to the dll. The dll must match the architecture of the OS. |
| SOFTWARE\Classes\CLSID\<CLSID>\InProcServer32 | ThreadingModel  | Apartment | REG_SZ |
| SOFTWARE\Classes\CLSID\<CLSID>\InProcServer32 | LoadWithoutCOM  |   | REG_SZ, but leave as empty string |
| SOFTWARE\Classes\CLSID\<CLSID>\ShellFolder  |	  	  	 
| SOFTWARE\Classes\CLSID\<CLSID>\ShellFolder  | HideOnDesktop | | REG_SZ, but leave as empty string |
| SOFTWARE\Classes\CLSID\<CLSID>\ShellFolder  | Attributes  | 0xf090013d (4035969341) | REG_DWORD |
