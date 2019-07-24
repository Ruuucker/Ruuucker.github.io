---
published: false
---
# Summary
{:.no_toc}

* toc
{:toc}

# NVRAM Variables Explained

NVRAM is non-volatile RAM that is used in EFI to store variables that need to persist between boots. Many of these NVRAM variables are architecturally defined, and setting invalid options to NVRAM could cause a machine to not be able to boot.

During the startup process, multiple drivers and applications can rely on NVRAM values to help them do their jobs. Below is a diagram from the UEFI 2.56 specification that shows this happening.

![1]({{ site.baseurl }}/assets/img/posts/nvram-startup.png){:class="imghalf"}

Because the bootloader and other drivers are configured to load information from NVRAM, if we can write to some of these NVRAM variables, we will have control over parts of how the system boots.

NVRAM variables are a combination of a GUID of the variable owner, a name, and a value. An example of a few of these values, as displayed by the UEFI shell command dmpstore, is below:

~~~
Variable NV+RT+BS 'Efi:Boot0080' DataSize = 68
  00000000: 01 00 00 00 50 00 4D 00-61 00 63 00 20 00 4F 00  *....P.M.a.c. .O.*
  00000010: 53 00 20 00 58 00 00 00-02 01 0C 00 D0 41 03 0A  *S. .X........A..*
  00000020: 00 00 00 00 01 01 06 00-05 1C 01 01 06 00 00 00  *................*
  00000030: 03 12 0A 00 00 00 00 00-00 00 04 01 2A 00 03 00  *............*...*
  00000040: 00 00 B8 A0 0C 0E 00 00-00 00 20 5F 13 00 00 00  *.......... _....*
  00000050: 00 00 65 5A 48 A0 E7 20-4D 4A B3 DE DB 1C 77 3D  *..eZH.. MJ....w=*
  00000060: 81 87 02 02 7F FF 04 00-                         *........*
Variable NV+RT+BS '7C436110-AB2A-4BBB-A880-FE41995C9F82:efi-boot-device-data' DataSize = 50
  00000000: 02 01 0C 00 D0 41 03 0A-00 00 00 00 01 01 06 00  *.....A..........*
  00000010: 05 1C 01 01 06 00 00 00-03 12 0A 00 00 00 00 00  *................*
  00000020: 00 00 04 01 2A 00 03 00-00 00 B8 A0 0C 0E 00 00  *....*...........*
  00000030: 00 00 20 5F 13 00 00 00-00 00 65 5A 48 A0 E7 20  *.. _......eZH.. *
  00000040: 4D 4A B3 DE DB 1C 77 3D-81 87 02 02 7F FF 04 00  *MJ....w=........*
  ~~~
  
  
  # Important NVRAM Variables and GUIDs.
  
Below are some important NVRAM variables to be aware of. There is technically no guarantee that these variables will be present on any machine, they are either defined by the specification or known to be used by specific vendors.

This list is not comprehensive: items will be added/taken away as things change or we discover new things that we care about.

~~~
GUID 	Variable Name 	

General Description
8BE4DF61-93CA-11D2-AA0D-00E098032B8C (Efi) 	BootOrder 	

An in-order array of 16-bit integers that refer to boot options. The system will

attempt to boot from each of the Boot#### devices in the order that they are

listed in this variable. Once a boot option is successfully loaded, the system

does not continue to try to load any subsequent boot options.
8BE4DF61-93CA-11D2-AA0D-00E098032B8C (Efi) 	Boot#### 	

One particular device that can be booted. #### is a four-digit hex number.

This variable is of the type EFI_LOAD_OPTION, found in section 3.1.3 of the

UEFI 2.5 spec.
8BE4DF61-93CA-11D2-AA0D-00E098032B8C (Efi) 	DriverOrder 	

An in-order array of 16-bit integers of drivers to be loaded. Similar to BootOrder,

this variable lists the drivers that are to be loaded on boot in order. However,

unlike BootOrder, all drivers will be loaded, not just the first successful driver.
 8BE4DF61-93CA-11D2-AA0D-00E098032B8C (Efi) 	Driver#### 	

A driver that is to be loaded on boot, if selected in DriverOrder.

#### is a four-digit hex number. This variable is of the type EFI_LOAD_OPTION,

found in section 3.1.3 of the UEFI 2.5 spec.
4D1ED05-38C7-4A6A-9CC6-4BCCA8B38C14 	EnableDriverOrder 	

Apple specific, causes loader to use the DriverOrder and Driver#### variables.

Consumed after use. Uses DriverOrder if EnableDriverOrder==0x31
7C436110-AB2A-4BBB-A880-FE41995C9F82 	csr-active-config 	

Apple specific, configuration for El Capitan's SIP.

A value of 0x10 means that it is enabled, 0x77 means that it is completely disabled.
~~~

