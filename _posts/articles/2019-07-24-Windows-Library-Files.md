---
published: false
layout: post
categories: articles
tags:
  - windows
---
# Summary
{:.no_toc}

* toc
{:toc}

# Using Library Files To Aid In Execution

Library files were introduce in Windows 7 and are a way of viewing the contents of multpile directories in a single view. For example the "Pictures" library actually includes the locations C:\Users\User\Pictures and C:\Users\Public\Pictures. The way Microsoft implements this is by creating .library-ms files at a location inside of AppData. The library-ms files are XML and follow a schema. Most of Microsofts .-ms file types are XML. The schema is easily found online, the description of the elements, however, is fairly vague. Below is an example of a typical library-ms file. 

![1]({{ site.baseurl }}/assets/img/posts/Windows-Library-Files/image2011.png){:class="imghalf"}

This example only has one location, but the specification says we can have as many as we want, All the libraries Microsoft provides have specific owners with serialized locations. Meaning you can't move it from machine to machine or even user for user. However, all the elements that tie it to a certain machine/user/location, are optional. Thanks, Obama. The parts we're interested in are the SearchConenectionDescription.

![1]({{ site.baseurl }}/assets/img/posts/Windows-Library-Files/image2012.png){:class="imghalf"}
and the templateInfo element.
![1]({{ site.baseurl }}/assets/img/posts/Windows-Library-Files/image2013.png){:class="imghalf"}

The SearchConnectionDescription allows you to choose what paths should be included when conglomerting the files in the library view. In this section we'll just cover the local capabilities of the library files. So, this is useful in many cases. In aiding execution, you can modify the url element of the search connector description path. Pointing the path to a non-existent sub-folder of a junction folder will cause the rendering of the junction folder. This is useful in triggering execution on a link file. This also causes exececution of COM objects (see the article CLSIDs and Junction Folders).

 

The templateInfo section allows the user to choose the view explorer will give to the folders/files. In cases where you are using icon rendering (like desktop.ini files), you can change the explorer view from details to icon view.

 

As a tradecraft aspect, you can choose to give the library an icon (folder icon, Recycle Bin icon, etc) by changing the {iconReference} element.