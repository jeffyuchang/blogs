--- 
layout: post
title: Building a component in SwitchYard
tags: 
- JBoss
- SwitchYard
- ESB
- Java
---
As I said in the Introduction to SwitchYard entry, SwitchYard is a lightweight ESB, which means it should have some out-of-box services/gateways that you can use.

##### Project Layout #####
SwitchYard project hosts its source project in the [github](http://github.com), and the url is [http://github.com/jboss-switchyard](http://github.com/jboss-switchyard), so you'll need to have the Git installed for hacking the code and create the pull request.

Below is the project list, all of them are self-descriptive:

- parent: This project only has a pom file that defines all the dependencies' version, this will be shared for all other subprojects, and all version will be used should only be defined here.
- core: 
- components:
- quickstarts:
- release:

Each project has its own ReadMe file and you can know more information.

##### Building a service component #####






