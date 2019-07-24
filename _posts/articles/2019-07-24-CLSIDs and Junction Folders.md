---
published: true
layout: post
categories: articles
tags:
  - windows
title: CLSIDs and Junction Folders
---
# Summary
{:.no_toc}

* toc
{:toc}

# Junction Folders

A junction folder in Windows is a method in which the user can cause a redirection to another folder, usually another known folder. A Junction Folder is considered a hard link. There are a couple of different ways to create junction folders. Registry modification can cause a folder junction point. Another is a desktop.ini entry inside the folder. The third, and probably most useful to us is the method that is a naming convention only. The method is simply to create the folder as such. MyFolder.{CLSID of folder you want to junction to}. Once you have named the folder as such, double clicking the folder will navigate you to the known folder represented to by the CLSID. Microsoft gives you many examples. This way you can easily create a folder that junctions to "Control Panel (All Tasks)" and call it My Control Panel. You can also junction to Pictures, Documents, Internet Explorer, My Computer. All of these special folders have been assigned unique clsids. See CLSIDs (Class IDs) for availble clsids that come default on Windows. Below is example of a junction folder to Control Panel (All Tasks).

![1]({{ site.baseurl }}/assets/img/posts/CLSIDs and Junction Folders/image2014-10-6.png){:class="imgsquire"}


Name the folder My Control Panel.{ED7BA470-8E54-465E-825C-99712043E01C} for a junction to Control Panel (All Tasks).

![1]({{ site.baseurl }}/assets/img/posts/CLSIDs and Junction Folders/image2014-10-68.png){:class="imgsquire"}

![1]({{ site.baseurl }}/assets/img/posts/CLSIDs and Junction Folders/image2014-10-64.png){:class="imgsquire"}



Windows is nice enough to hide the CLSID for you.

![1]({{ site.baseurl }}/assets/img/posts/CLSIDs and Junction Folders/image2014-10-1.png){:class="imghalf"}

The junction works!
<br>

So what. 

Well, what happens when the CLSID isn't of the expected folders? What if it's not a folder at all? That's where it gets a little interesting.

# CLSIDs and COM Objects

So, using CreateGuid I generated a new guid (highly unlikey to have already been used - {24138469-5DDA-479D-A150-3695B9365DC0}).  

![1]({{ site.baseurl }}/assets/img/posts/CLSIDs and Junction Folders/image2014-10-2.png){:class="imgsquire"}

add the CLSID and we get this.



![1]({{ site.baseurl }}/assets/img/posts/CLSIDs and Junction Folders/image2014-10-3.png){:class="imgsquire"}
The CLSID doesn't become hidden if there is nothing to correlate it to. When you nvigate into the folder it appears as a normal folder. However, if you watch Procmon you see this:
![1]({{ site.baseurl }}/assets/img/posts/CLSIDs and Junction Folders/image2014-10-7.png){:class="img"}


From watching Procmon, you can see that navigating inside the folder causes explorer to look for this CLSID in HKEY_CURRENT_USER. By default, the large majority of CLSIIDs are stored under HKEY_LOCAL_MACHINE, yet CLSIDs under HKEY_CURRENT_USER can stll be used. So, now let's set up our key.
<br><br>
Before:<br>
![1]({{ site.baseurl }}/assets/img/posts/CLSIDs and Junction Folders/image2014-10-9.png){:class="imgsquire"}

Entries Added:
Default of the {24138469-5DDA-479D-A150-3695B9365DC0} key is the name of the COM object:
![1]({{ site.baseurl }}/assets/img/posts/CLSIDs and Junction Folders/image2014-10-11.png){:class="imghalf"}

Values of the InProcServer32 key:
![1]({{ site.baseurl }}/assets/img/posts/CLSIDs and Junction Folders/image2014-12.png){:class="imghalf"}

Now when we navigate inside of our junction folder, we see:
Values of the InProcServer32 key:
![1]({{ site.baseurl }}/assets/img/posts/CLSIDs and Junction Folders/image2014-13.png){:class="imghalf"}
Execution is gained as verclsid.exe calls process attach on our dll. Nice.

# User-level Persistence
Unfortunately, this is probably a better persistence mechanism than execution vector. Mostly, because it would require both a registry write and navigation into the junction folder. When looking at it as a persistence mechanism, all we have to do is set the registry keys, and create the junction folder in a location that explorer (or some other user process) will navigate into/under on startup. Preferrably we would like explorer to navigate into the junction folder. We could do this with Library Files on Windows 7+ (see Windows Library Files (.library-ms) ). For wider coverage using the same technique, I ended up looking into folders navigated for the "start menu". They get loaded at the start, they already exist, they exist in AppData, and they are most often viewed through the start menu. 

So, navigate into %appdata%\Microsoft\Windows\Start Menu and you should see directories under the start menu, all should get navigated on login. However, you have to be careful when modifying any of these folders as they are often combined with folders in another location for the start menu and it could be alerting if there are two folders named Accessories in the targets start menu. If desired, you can also create a new folder vs using an already existing one. Anyways, so now takes something like Accessories.















| Key           | Name          | Value         | Notes         |
| :---------------------------------- | :-----------  | :-----------  | :-----------  |
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
  
  
  
