---
published: false
title: Bypassing Windows User Account Control (UAC) and ways of mitigation
layout: post
categories: articles
tags:
  - windows
---
# Summary
{:.no_toc}

* toc
{:toc}


# Introduction

Securing machines from abuse and compromise in a corporate environment has always been an ongoing process. Providing admin rights to users has always been abused as users have ended up installing unapproved software, change configurations, etc. Not giving local admin rights and they claim they can’t do their work. If malware happens to compromise the machine with full admin rights then you are most likely looking at reimaging the machine.

User Account Control (UAC) gives us the ability to run in standard user rights instead of full administrator rights. So even if your standard user account is in the local admin group damage is limited, i.e. installing services, drivers, writing to secure locations, etc. are denied. To carry out these actions users would need to interact with the desktop such us right click and run as administrator or accept the UAC elevation prompt. UAC was introduced from Windows Vista onwards and contains a number of technologies that include file system and registry virtualization, the Protected Administrator (PA) account, UAC elevation prompts and Windows Integrity levels.

UAC works by adjusting the permission level of our user account, so programs actions are carried out as a standard user even if we have local admin rights on the computer. When changes are going to be made that require administrator-level permission UAC notifies us. If we have local admin rights then we can click yes to continue otherwise we would be prompted to enter an administrator password. These would however depend on what policies have been defined in your environment.

This blog post shows how easily UAC elevation prompts could be bypassed and what actions could be taken to mitigate this threat.

![1]({{ site.baseurl }}/assets/img/posts/bypass uac/regedituac.png){:class="imghalf"}

# Bypassing UAC

Exploiting UAC is a trivial process. There are two stages needed to be taken to achieve bypass to elevate from standard user rights to administrator user rights. These steps have widely been published so it’s nothing new though stage 2 documents some more DLL hijacking vulnerabilities.

   >Writing to a secure location
    Exploiting DLL hijacking vulnerability

In order for our bypass to be successful to start off with we need

   >A medium integrity process
    A standard user in an administrators group
    Windows executable must be signed by Microsoft code signing certificate
    Windows executable must be located in a secure directory
    Windows executable also must specify the auto Elevate property in their manifest

# Writing to a secure location
There are a couple of ways we can write to a secure location.

   >Using the IFileOperation COM Object
    Using Windows Update Standalone Installer (wusa.exe)

IFileOperation COM Object
The IFileOperation COM object has a method that we can use to copy files to our secure location as the operation will auto-elevate and able to do a privilege copy. To exploit we can in inject our malicious DLL in a medium integrity process to carry out the operation. Since the COM object is set to auto-elevate the injected process does not need to be marked for auto-elevation in its manifest.

On windows 7 injected processes that have copied successfully are