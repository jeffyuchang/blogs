--- 
layout: post
title: Fedora environment setting.
published: true

tags: 
- Fedora
- OS
type: post
status: publish
---
I still recalled that when I used the Ubuntu, I put all of my environment variables, like JAVA_HOME et al in the .bash_profile file, and if I want to make it effect, I am running the:

> source .bash_profile

But in Fedora, I couldn't make it effect unless I do the logout andlogin, and then searched on the google, it turns out the people would do as following:

put all of environment setting in the .bashrc file, and then if you update it, try to

> source .bash_profile

It will make all of updates effective.

I tried it, it works as it was told. So, now I changed my habit, put the environment setting to the .bashrc file.
