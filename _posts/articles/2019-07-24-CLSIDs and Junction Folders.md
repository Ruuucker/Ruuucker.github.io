---
published: false
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

![1]({{ site.baseurl }}/assets/img/posts/image2014-10-6.png){:class="imghalf"}

Name the folder My Control Panel.{ED7BA470-8E54-465E-825C-99712043E01C} for a junction to Control Panel (All Tasks).

 	| Syntax | Description |
| ----------- | ----------- |
| Header | Title |
| Paragraph | Text | 