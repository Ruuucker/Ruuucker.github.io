---
title: ExitBootServices Hooking
layout: post
tags:
  - uefi_rus
published: true
---

# Оглавление
{:.no_toc}

* toc
{:toc}

# Так значит, хочешь хукнуть ExitBootServices?

Если ты здесь, это потому что ты хочешь узнать больше о ExitBootServices и, вероятно, хочешь подключить его чтобы вы могли что-то делать с ОС. Вот некоторая базовая информация если тебе нужно освежить память о EFI.
 
 

При загрузке код расположенный на прошивке контролирует систему. Это то что определяет происходящее во время загрузки. Как правило, приложение UEFI запускаемое из прошивки является загрузчиком ОС. Основная цель загрузчика - инициализировать все что нужно операционной системе, загрузить ядро в память и передать управление самому ядру. Однако загрузчик - это просто еще одно приложение UEFI: он может использовать только память выделенную прошивкой и может получать доступ только к службам и протоколам UEFI которые предоставляет прошивка. За всю стадию UEFI загрузки, вплоть до DXE фазы, отвечает прошивка. Однако в конце концов все службы загрузки UEFI должны быть завершены, а ОС должна получить контроль. Это выполняется с помощью загрузочного сервиса UEFI ExitBootServices.

Когда ExitBootServices вызывается загрузчиком ОС, прошивка передает управление системой загрузчику. Вся память службы загрузки освобождается, все службы загрузки завершаются и загрузчик ОС может передать управление системой к ОС. На данный момент доступны только сервисы времени выполнения предоставляемые прошивкой.

# Actually Hooking ExitBootServices

When ExitBootServices is called, the DXE phase is about to end. The firmware has done all that it needs to do from UEFI to set up the system for the OS, and the OS itself has already been loaded into memory. You can be creative with what you could do to the kernel that is just sitting there in memory, not protected by anything.

Because the ExitBootServices service can be found by getting its pointer from the global EFI_BOOT_SERVICES table, hooking the ExitBootServices call is trivial. From within a UEFI driver, you store the original pointer and then replace the table's pointer with one to your hook function. From there, you let your driver run and wait for ExitBootServices to be called by the OS loader, and your hook code will run just before the OS loader gets control. When you're running in UEFI, that EFI_BOOT_SERVICES table isn't protected by anything, so you can just write directly to it.

At this point, you can do whatever you want. UEFI boot services are still running (because they will be terminated when the real ExitBootServices is called), and the OS is sitting there. However, if you mess with memory (especially if you allocate new buffers), you will need to do some cleanup before you can return successfully to the original ExitBootServices.

The second parameter to ExitBootServices is a UINTN MapKey. This value identifies the current memory map of the system, and is changed every time something in the memory map changes. In order for ExitBootServices to do its job properly, it MUST have the current memory map. If it does not, it will do weird things, like cause the OS to not be able to identify the startup disk or not be able to load altogether.

In order to make sure your ExitBootServices call goes correctly, you will need to call GetMemoryMap first. Ironically, calling GetMemoryMap will require you to allocate memory for the map itself, which, in turn, will change the memory map.

You can deal with this issue by looping your calls – allocating space for the map, then calling GetMemoryMap again. Eventually, you will have allocated enough space for the (again updated) map before you make the GetMemoryMap call, and you'll get the up-to-date map.

Once you have the map, you can simply call the original ExitBootServices function and be on your merry way.

# Example Code

Below is the code you'll need to do basic ExitBootServices hooking. Actually compiling this into an EFI executable isn't covered in this tutorial, but you will need to run this from a UEFI driver. In this example, your driver's entry point is HookDriverMain.

~~~
extern EFI_BOOT_SERVICES *gBS;
EFI_EXIT_BOOT_SERVICES     gOrigExitBootServices;



EFI_STATUS
EFIAPI
ExitBootServicesHook(IN EFI_HANDLE ImageHandle, IN UINTN MapKey){

	/* <hook related fun> */
	/* Do fun hook-related stuff here */
	/* </hook-related fun> */

	/* Fix the pointer in the boot services table */
	/* If you don't do this, sometimes your hook method will be called repeatedly, which you don't want */
    gBS->ExitBootServices = gOrigExitBootServices;

    /* Get the memory map */
    UINTN MemoryMapSize;
    EFI_MEMORY_DESCRIPTOR *MemoryMap;
    UINTN LocalMapKey;
    UINTN DescriptorSize;
    UINT32 DescriptorVersion;
    MemoryMap = NULL;
    MemoryMapSize = 0;
    
	
    do {  
        Status = gBS->GetMemoryMap(&MemoryMapSize, MemoryMap, &LocalMapKey, &DescriptorSize,&DescriptorVersion);
        if (Status == EFI_BUFFER_TOO_SMALL){
            MemoryMap = AllocatePool(MemoryMapSize + 1);
            Status = gBS->GetMemoryMap(&MemoryMapSize, MemoryMap, &LocalMapKey, &DescriptorSize,&DescriptorVersion);      
        } else {
            /* Status is likely success - let the while() statement check success */
        }
        DbgPrint(L"This time through the memory map loop, status = %r\n",Status);
    
    } while (Status != EFI_SUCCESS);

    return gOrigExitBootServices(ImageHandle,LocalMapKey);

}
EFI_STATUS
EFIAPI
HookDriverMain(IN EFI_HANDLE ImageHandle, IN EFI_SYSTEM_TABLE *SystemTable){

    /* Store off the original pointer and replace it with your own */
    gOrigExitBootServices = gBS->ExitBootServices;
    gBS->ExitBootServices = ExitBootServicesHook;

	/* It's hooked! Return EFI_SUCCESS so your driver stays in memory */
    return EFI_SUCCESS;
}
~~~
