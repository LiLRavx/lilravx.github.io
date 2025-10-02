---
layout: post
title:  Volatiltiy - imageinfo Plugin
date: 2025-10-03 00:12:00 +0300
categories: [Blue Team, Volatility] # Add your new categories here
tags: [Volatiltiy2, Volatiltiy3, imageinfo, DFIR]
image: "photos/Volatiltiy/bg.jpg"
---

Instead of struggling for hours with the plugin **`imageinfo`**to identify the image profile, especially when dealing with images exceeding 50GB that take 2+ hours, we can utilize **Volatility3** plugins and leverage their output for **Volatility2, yeah !!** 

Essentially, Windows stores comprehensive information in registry hives. In this article, we’ll focus on the **Software hive**, particularly on the current build number and other relevant information.

**Volatility3** can extract Software hive information using only the **“windows.registry”** Plugin, bypassing the need for the imageinfo plugin. Thus, we can take advantage of this plugin to read the registry hive.

This method isn’t common, so if I see you using it, I’m pretty sure you’ve read this article! xD

The following example involves an image from a CTF, but this method can be applied to any memory image you desire! I’ve been using this approach for two years now, and it has never let me down. 

1. we need to  read all the information stored in the current version of the Software hive, specifically related to the system’s general information, by using the following command in volatility 3:

```jsx
vol3 -f memdump.mem windows.registry.printkey.PrintKey — key “Microsoft\Windows NT\CurrentVersion”
```

![](/photos/Volatiltiy/1.jpg)

1. After running the command, what we need to focus on are:
    1. ProductName: Windows 10 Pro
    2. CurrentBuildNumber: 19043
        
        ![](/photos/Volatiltiy/2.jpg)
        
2. now we need to use volatiltiy 2 to get the excat or closest profile for the memory by using this command:

```jsx
vol.py -info | grep Win10
```

![](/photos/Volatiltiy/3.jpg)

Our Profile number on volatiltiy 3 is 19043 , from this step we can chose the same number if we can’t find the same number we can choose the closest one to the current number, in our case 19041

![](/photos/Volatiltiy/4.jpg)

> Note(1): Sometimes you may obtain the exact number depending on the Windows version, while other times you might receive a number higher than yours. It’s generally better to select a number that is equal to or higher than yours.
> 

> ***Note(2): For Windows service pack numbers, you can obtain them using the same method. The more accurate the number, the faster and error-free the output you’ll receive.***
>