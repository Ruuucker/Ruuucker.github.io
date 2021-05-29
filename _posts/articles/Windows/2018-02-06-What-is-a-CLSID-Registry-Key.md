---
published: true
layout: post
tags:
  - windows
title: What is a CLSID Registry Key
---
# Summary
{:.no_toc}

* toc
{:toc}

# Introduction

The CLSID or Class Identifier is a string of alphanumeric (both numbers and alphabet characters) symbols that are used to represent a specific instance of a Component Object Model or COM-based program. It allows operating systems and software, particularly for Windows, to detect and access software components without identifying them by their names. Although Microsoft has phased out usage of COM in favor of the .NET infrastructure, COM remains in usage as an important component for many commonly-used programs and has no plans of being discontinued.

Examples of objects that use COM and a corresponding CLSID include ActiveX, the My Computer directory and the Windows Start Menu. A typical CLSID in your Windows Registry can look like this:

    {48E7CAAB-B918-4E58-A94D-505519C795DC}

Your most likely encounter with the CLSID is when a website asks you to update ActiveX or another program. Your browser detects the version of your software by checking its CLSID, and transmits this information to the website without compromising your computer.

However, fake media updates often are used to distribute malicious software and other PC threats, and you should avoid downloading updates from websites that you don't trust implicitly.

# When a Good CLSID Entry Goes Bad

If the CLSID is corrupted, your PC may experience problems related to the program the CLSID is linked to. One common problem is a damage CLSID resulting in software being unable to verify its own version and update itself. As an easy-to-implement solution, uninstalling and reinstalling your software usually remedies this issue.

The most common issue related to a Registry CLSID entry is a program's failure to delete its CLSID from the Registry when the rest of the program is uninstalled. Although this is a poor programming practice that clutters up the PC's Registry with meaningless text entries, an unused CLSID entry isn't likely to harm your computer. However, some Registry cleaners and other system maintenance programs specialize in removing this CLSID-based 'junk.' In very extreme circumstances, such as with a computer with low system resources, a Registry with too many unused CLSID entries may cause performance issues.

If you're interested in correcting CLSID Registry entries manually, a high level of caution should be used. Changes to your Registry can damage your operating system in many ways, most notably by causing it to fail to recognize critical components and programs. Regardless of whether or not you're interested in making changes to your computer's CLSID entries, having a backup Windows Registry through a system restore point or another method is recommended in all cases.

# The Vanishing CLSID

Although the CLSID ordinarily is a permanent text entry in your Registry – at least until you uninstall the program that it's linked to - temporary folders and files also may display CLSID entries in their names. This is often caused by program installers that decompress files to use for installation before removing them. Most such files and folders should delete themselves automatically after installation has completed. In cases of poor coding or interrupted installation, you may need to delete these objects yourself, although they shouldn't damage your computer.

Not all CLSID-using programs are forced to write their CLSID entries into your Windows Registry. RegFree or Registration-Free COM components are capable of storing their CLSID entries in their own EXE files or in separate XML files. This has certain advantages, such as allowing a program to be installed several times as several different versions. However, RegFree COM support is more limited and sometimes (in cases of system-wide programs like DirectX) wholly unavailable.

# The Difference Between CLSID's COM and the Rest of the COM Universe

The COM interface with the CLSID is a Component Object Model, an interfacing method that uses the object-oriented programming philosophy (or OOP). It doesn't have a direct relationship with the web domain suffix .COM, which signifies a top level 'commercial' domain.

Likewise, CLSID's COM components aren't related to .COM files, which is a subtype of 
executable or EXE file. Although some Windows components and other programs use .COM, this outdated file format requires MS-DOS emulation that isn't included (by default) on 64-bit Windows OSes.

# CLSID's Place in the Malware Industry

CLSID entries may be used to run harmful programs, as well as safe ones. Rootkits, trojans, malicious Browser Helper Objects and other types of malware all may make use of the CLSID system to launch themselves automatically or when certain conditions are triggered. The majority of competent anti-malware programs will detect and delete malicious CLSID entries along with the malware that's associated with them. However, like normal CLSID entries, undeleted CLSID malware entries for programs that have been removed aren't capable of causing damage to your computer.

Malware programs also have been known to use CLSID entries to make calls to other programs (such as Internet Explorer). These programs may or may not display visible indications of being open, although, in most cases, you should be able to detect the open program's memory process via Task Manager and similar utilities. Such attacks can be used to conduct various online attacks without the PC user's knowledge. While knowledge of CLSID is unnecessary for casual PC usage, a working awareness of its capabilities and limitations can help resolve software and Registry-related errors with a minimum of frustration.
