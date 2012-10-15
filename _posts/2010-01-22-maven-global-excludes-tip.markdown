--- 
layout: post
title: Maven Global Excludes tip.
published: true
tags: 
- Maven
type: post
status: publish
---
If you work on a project that has ton of jar dependencies, and you want to exclude the 'xerces' (for example), you might end up using the exclusion multiple times, as a lot of jars are depending on the 'xerces'. In this case, you can use [following way to achieve the global exclude](http://jlorenzen.blogspot.com/2009/06/maven-global-excludes.html) in maven.
