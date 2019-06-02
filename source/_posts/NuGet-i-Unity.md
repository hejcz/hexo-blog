---
title: Unity i NuGet - mortal enemies
date: 2018-03-17 22:34:22
tags:
- Unity
- NuGet
---
Core of my Msc project is training various machine learning models in environment created in Unity editor. I use [ML-Agent](https://github.com/Unity-Technologies/ml-agents) library simplyfing passing data between running game and TensorFlow python scripts.

It uses [Json.NET](https://www.newtonsoft.com/json) under the hood. I downloaded Json.NET with NuGet and compile errors in IDE disapeared. I switched back to Unity but it was still complaining about missing dependency. I couldn't get rid of it so I came back to IDE. Project's references were missing dependency once again.

After some research I found in documentation that Unity generates solution and projects automatically overriding information about dependencies added with NuGet. I was lucky finding [Using NuGet in Visual Studio Tools for Unity](http://www.damirscorner.com/blog/posts/20150523-UsingNuGetInVisualStudioToolsForUnity.html) article. I manually copied Json.NET dependency to `Assets` directory. There is some mechanism I'm not aware of responsible for detecting assembly and automatically adding it to generated solution.
