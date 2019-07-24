---
published: true
layout: post
categories: articles
tags:
  - uefi
---
# Summary
{:.no_toc}

* toc
{:toc}

# So, You Want to Hook ExitBootServices?

If you're here, it's because you want to know more about ExitBootServices, and probably want to hook it so you can do things to the OS. Here's some basic information for you in case you need your memory refreshed about EFI things.

 

On boot, the code located on the firmware has control of the system. It is what determines what happens during a boot. Typically, the UEFI application run from the firmware is an OS Loader. The loader's main purpose is to initialize everything that the OS needs to run, load the kernel into memory, and pass control to the kernel itself. However, the loader is just another UEFI Application: it can only use the memory it has been allocated by the firmware and can only access the UEFI services and protocols that the firmware provides. All the way through to the end of the DXE phase of a UEFI boot, the firmware is in charge. However, eventually, all of the UEFI boot services need to end, and the OS needs to take control. This is accomplished using the UEFI boot service ExitBootServices.

When ExitBootServices is called by the OS loader, the firmware gives control of the system to the loader. All of the boot service memory is reclaimed, the boot services are all terminated, and the OS loader can hand over control of the system to the OS. At this point, only the runtime services provided by the firmware are still accessible.

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
