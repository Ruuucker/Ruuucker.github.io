---
published: true
layout: post
tags:
  - uefi
title: EDK2 Compiler Information and CI Concerns
---
# Summary
{:.no_toc}

* toc
{:toc}

# Purpose

This page will discuss locations of compiler flags within the TianoCore EDK2 (UDK 2014) build environment and CI concerns that need to be taken into account when building EFI binaries.

If you have not built your environment, you can find the page that discusses how to do that (for Linux) here: [Setting Up a Linux Build Environment for EFI]({{ site.baseurl }}/articles/Setting-Up-a-Linux-Build-Environment-for-EFI/ "Setting Up a Linux Build Environment for EFI").


# Compiler Flags

The sets of compiler flags that will be used by the build script are stored by default in ~/src/edk2/Conf/tools_def.txt. Within this document, you can find the specific flags for the compiler that you are using.

These flags change based on which compiler and which version of that compiler are being used.

As you walk through the file, you will see that it first defines paths to each of the compilers that are to be used, and then goes into the compiler flags to use. The compiler will look to the TOOL_CHAIN_TAG value that you set in target.txt to tell it which compiler you are using.

Conceivably, you could add EDK support to a new compiler (eg: a new version of GCC or VS) by defining it in tools_def.txt and then setting it as your target architecture in target.txt.

The general structure of a set of compiler flags is: <Target_Toolchain_Arch_CommandType_Attribute> Some of these values can be left out, as needed.

For example, flags to be used for 64-bit applications compiled using the GCC 4.9 toolchain are defined under GCC49_X64_CC_FLAGS, whereas flags for use in general GCC compilations are located in GCC_ALL_CC_FLAGS.

# CI Concern - File Paths in Binaries

Below is a snippet of output from the strings command, run on the HelloWorld.efi binary created in the [Setting Up a Linux Build Environment for EFI]({{ site.baseurl }}/articles/Setting-Up-a-Linux-Build-Environment-for-EFI/ "Setting Up a Linux Build Environment for EFI") for EFI tutorial.

~~~
$ strings -a HelloWorld.efi
[...]
/home/User/src/edk2/MdePkg/Library/BasePrintLib/PrintLibInternal.c
Width < 38
(Flags & ~(0x01 | 0x08 | 0x20 | 0x80)) == 0
((Flags & 0x08) == 0) || ((Flags & 0x80) == 0)
StrSize ((CHAR16 *) Format) != 0
AsciiStrSize (Format) != 0
<null string>
<null guid>
%08x-%04x-%04x-%02x%02x-%02x%02x%02x%02x%02x%02x
<null time>
%02d/%02d/%04d  %02d:%02d
%08X
(((Flags & 0x00000040) == 0)) || (StrSize ((CHAR16 *) OriginalBuffer) != 0)
(((Flags & 0x00000040) != 0)) || (AsciiStrSize (OriginalBuffer) != 0)
Divisor != 0
/home/User/src/edk2/MdePkg/Library/BaseLib/DivU64x32Remainder.c
NB10
/home/User/src/edk2/Build/DuetPkgX64/DEBUG_GCC49/X64/HelloWorld/HelloWorld/DEBUG/HelloWorld.dll
[...]
~~~

One of the first things you may notice is that absolute file paths are included in these binaries. For obvious reasons, this is something you need to be aware of, as your username, the package name, and the binary name are included in the binary.

Furthermore, these strings cannot (so far, with my experimentation) be stripped out of the binary using compiler flags. If you change the GCC compiler flags from -g to -s, the strings remain. If you change the objcopy flags to include --strip-all, the binary will hang when you run it.

However, you can manually NULL out the bytes that should have stored the path string (or use a python script to do it.) Unlike what I previously believed, if you simply NULL out the buffer where the string would have been stored, the program will execute normally. I would presume that you can put random junk there too and it will work.
