---
title: Перехват ExitBootServices
layout: post
tags:
  - uefi
published: true
ru: true
---

# Оглавление
{:.no_toc}

* toc
{:toc}

# Так значит, хочешь перехватить ExitBootServices?

Если ты здесь, это потому что ты хочешь узнать больше о ExitBootServices и, вероятно, хочешь использовать это чтобы что-то делать с ОС. Вот некоторая базовая информация если тебе нужно освежить память о EFI.
 
 

При загрузке, код расположенный на прошивке контролирует систему. Это то что определяет происходящее во время загрузки. Как правило, приложение UEFI запускаемое из прошивки является загрузчиком ОС. Основная цель загрузчика - инициализировать все что нужно операционной системе, загрузить ядро в память и передать управление самому ядру. Однако загрузчик - это просто еще одно приложение UEFI: он может использовать только память выделенную прошивкой и может получать доступ только к службам и протоколам UEFI которые предоставляет прошивка. За всю стадию UEFI загрузки, вплоть до DXE фазы, отвечает прошивка. Однако в конце концов все службы загрузки UEFI должны быть завершены, а ОС должна получить контроль. Это выполняется с помощью загрузочного сервиса UEFI ExitBootServices.

Когда ExitBootServices вызывается загрузчиком ОС, прошивка передает управление системому загрузчику. Вся память службы загрузки освобождается, все службы загрузки завершаются и загрузчик ОС может передать управление машиной к ОС. На данный момент доступны только сервисы времени выполнения предоставляемые прошивкой.

# Перехватываем ExitBootServices

Когда вызывается ExitBootServices, DXE фаза подходит к концу. Прошивка сделала все что нужно от UEFI для настройки системы под ОС, а сама ОС уже загружена в память. Мы можем проявить творческий подход к тому, что можно сделать с ядром которое просто находится в памяти, и ничем не защищено.


Поскольку сервис ExitBootServices можно найти получив его указатель из глобальной таблицы EFI_BOOT_SERVICES, перехват вызова ExitBootServices тривиален. Внутри драйвера UEFI вы сохраняете исходный указатель, а затем заменяете указатель таблицы на один из наших функции-перехватчиков. Оттуда можно позволять нашему драйверу работать, и ждать пока загрузчик ОС вызовет ExitBootServices, и ваш код перехвата будет запущен непосредственно перед тем как загрузчик ОС получит контроль. Когда вы работаете в UEFI, эта таблица EFI_BOOT_SERVICES ничем не защищена, поэтому вы можете просто писать прямо в нее.

На данный момент можно делать все что угодно. Сервисы загрузки UEFI все еще работают (потому что они будут прерваны когда произойдет вызов реальных ExitBootServices) и ОС находится в памяти. Однако, если вы возитесь с памятью (особенно если вы выделяете новые буферы), то тебе нужно будет выполнить некоторую очистку, прежде чем ты сможешь успешно вернуться к исходным ExitBootServices.


Второй параметр ExitBootServices - это UINTN MapKey. Это значение идентифицирует текущую карту памяти системы и изменяется каждый раз когда что-то на карте памяти изменяется. Чтобы ExitBootServices правильно выполнял свою работу, он ДОЛЖЕН иметь текущую карту памяти. Если этого не произойдет, это приведет к странным вещам, например, к тому что ОС не сможет определить загрузочный диск или вообще не сможет загрузиться.


Чтобы убедиться, что ваш вызов ExitBootServices проходит правильно, нужно сначала вызвать GetMemoryMap. По иронии судьбы, вызов GetMemoryMap потребует от вас выделения памяти для самой карты, что в свою очередь изменит карту памяти.


Можно решить эту проблему зацикливая свои вызовы - выделяя место для карты, а затем снова вызывая GetMemoryMap. В конце концов, будет выделено достаточно места для (снова обновленной) карты, и после этого сделать вызов GetMemoryMap и получить обновленную карту.

Получив карту, можно просто вызвать оригинальную функцию ExitBootServices и отправиться в путь.

# Пример кода

Ниже приведен код, который понадобится для базового перехвата ExitBootServices. На самом деле компиляция этого в исполняемый файл EFI не рассматривается в этом руководстве, но тебе нужно будет запустить его из драйвера UEFI. В этом примере точкой входа драйвера является HookDriverMain.


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
