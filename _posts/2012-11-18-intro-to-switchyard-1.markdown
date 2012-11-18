--- 
layout: post
title: Introduction to SwitchYard - (Part I)
tags: 
- JBoss
- Java
- ESB
- SwitchYard
---

[SwitchYard](http://www.jboss.org/switchyard) released its [0.6 final version](https://community.jboss.org/en/switchyard/blog/2012/11/12/switchyard-06-final-released) last week, I think this is a good time to write some entries on exploring the SwitchYard. By the way, you don't need to be afraid by it is only 0.6 release, it started from 0.1, and typically in each Final version, it had Beta or CR releases, so it is quite stable today.

#### SwitchYard overview ####

###### What is SwitchYard ######

Simply put, it is the JBoss next generation Enterprise Service Bus (ESB). Offically, following one is used in the SwitchYard website.
> SwitchYard is a lightwegiht service delivery framework provding full lifecycle support for developing, deploying and managing service-oriented applications.

So its aim is not only just trying to build another ESB, but also created its various tools, like Eclipse based editor, GWT based console etc, and emphasis the easy testing as well. So it wants to enable users integration work easy.

###### What is in SwitchYard ######
I think the following picture[^picture_note] illustrates the components that we have in the SwitchYard.

![SwitchYard Architecture](../../../../images/switchyard-architecture.png)

As the above picture shows, I'll briefly talk about parts of it, which will be needed for the simple example later.

*Configuration Model*: Every switchyard application needs to have the switchyard application descriptor:

- switchyard.xml: This file defines the service implementations, gateways, transformations, validations and deployment dependencies. Note: The elements in SwitchYard.xml conform to [OASIS Service Component Architecture Assembly Model Specification](http://docs.oasis-open.org/opencsa/sca-assembly/sca-assembly-spec-v1.1.html). (detailed documentation can be found at [here](https://docs.jboss.org/author/display/SWITCHYARD/Configuration)).

*Implementation Commponent(Service Implementations)*: These components are available in the SwitchYard, which means you can use them in your switchyard application. As of 0.6 release, following is the service implementation list(detailed documentation can be found at [here](https://docs.jboss.org/author/display/SWITCHYARD/Service+Implementations)):

- Bean Services: This is based on the [Weld](http://seamframework.org/Weld).
- Camel Services: This is based on the [Apache Camel](http://camel.apache.org).
- BPM Services: This is based on [jBPM5](http://www.jboss.org/jbpm).
- Rules Services: This is based on [Drools](http://www.jboss.org/drools).
- BPEL Services: This is based on [RiftSaw](http://www.jboss.org/riftsaw)

*Gateway Component(Gateway Bindings)*: As of 0.6 release, following is the gateway bindings list that you used in switchyard out of box(detailed documentation can be found at [here](https://docs.jboss.org/author/display/SWITCHYARD/Gateway+Bindings)).

- Camel Bindings: Camel has its own binding components, switchyard has integrated some of bindings, namely: File Binding, JMS, SEDA, SQL, Timer Binding etc.
- SOAP Binding
- HortnetQ Bindings
- JCA Bindings
- RETEasy Bindings
- HTTP Bindings

*Transformers(Transformation)* Transformer's primary duty is to tranform the data betweent differnt data format as needed. As of 0.6 release, it has following transformers (detailed documentation can be found at [here](https://docs.jboss.org/author/display/SWITCHYARD/Transformation)):

- Java Transformer
- Smooks Transformer
- JSON Transformer
- XSLT Transformer
- JAXB Transformer

So, enough of concepts here, next, we are going to build our first switchyard application with the knowlege that we had.

#### Installation ####
Before we try anything interesting, we need to install the SwitchYard and its various tools. Going to the [SwitchYard downloads page](http://www.jboss.org/switchyard/downloads), I downloaded the following artifacts.

- switchyard-as7-0.6.0.Final.zip: this one contains both the switchyard runtime and the JBoss AS7.
- switchyard-site-assembly-0.6.0.Final.zip: this one is offline package for the eclipse based switchyard editor.
- switchyard-tools-0.6.0.Final.zip: this one contains the switchyard web based console, forge plugin etc.



#### Footnotes ####
[^picture_note]: I copied this picture from one of [Keith's](https://community.jboss.org/people/kcbabo) presentation when he did the SwitchYard workshop :).

#### Reference ####
1. [An (Impatient Newbie User’s) Introduction to Switchyard](https://community.jboss.org/en/switchyard/blog/2012/06/22/an-impatient-newbie-user-s-introduction-to-switchyard) By [Len Dimaggio](http://swqetesting.blogspot.com.au/)
