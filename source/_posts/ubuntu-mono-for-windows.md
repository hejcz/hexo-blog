---
title: Ubuntu Mono for Windows
date: 2018-03-21 22:49:06
tags:
- windows
- intellij idea
---

Recently I had to install Intellij Idea on my Windows machine. I don't really like programming on Windows because every single time I have problems installing my favorite [Ubuntu Mono](https://fonts.google.com/specimen/Ubuntu+Mono) font. It looks perfect on Linux but Windows version of Intellij renders it much worse. Dot above i and j letters is a joke and all of the letters are thinner.

Problems with fonts rendering in various JetBrains IDEs are a subject of many bugs reported over last 11 years. Recommendations differs between fonts and operating systems. When I had this problem o Linux 1 year ago I used trick with [switching IDE JDK](https://gist.github.com/CrazyCoder/0d9e54f450000d3fb6edcbda6d9788be). I couldn't find this version of JDK for Windows though.

Then I found some solution with [font modification](https://superuser.com/questions/614960/how-to-fix-font-anti-aliasing-in-intellij-idea-when-using-high-dpi/1114515). I added necessary flags to `.vmoptions` and removed hints from font. IDE displayed it with lowered baseline. It completely missed cursor position so it was in the middle of two consucutive lines in IDE's editor. I deleted font with Control Panel but IDE were still detecting it.  Powershell to the rescue.

```powershell
remove-item C:\Windows\Fonts\Ubuntu*  -force
```

I have installed a clean font once again. This time baseline was ok but letters were still not perfect. I removed flags from `.vmoptions` and deleted hints from this fresh Ubuntu Mono font what finally solved the problem. I [uploaded](https://drive.google.com/open?id=1a8l6Y_CAjf4HhZMS9PaLAQOIDQG37Ged) font to Google Drive to not go through this hell anymore.

**Update 6.5.2018**

The better approach than manipulating font is helping Windows to render font correctly. Following [article](https://www.askvg.com/how-to-get-mac-os-or-linux-style-font-smoothing-in-windows/) was very helpful. [Gdipp](https://code.google.com/archive/p/gdipp/) project solved my problem.
