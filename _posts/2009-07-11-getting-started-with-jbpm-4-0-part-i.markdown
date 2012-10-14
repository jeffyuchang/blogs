--- 
layout: post
title: Getting Started with jBPM 4.0 - (Part I )
published: true
tags: 
- BPM
- JBoss
- jBPM
type: post
status: publish
---
[jBPM 4.0 was out](http://processdevelopments.blogspot.com/2009/07/jbpm-40-is-out.html) yesterday, so I played it with a simple example, also include installing the eclipse plugin, and see the jbpm console, which I haven't tried them out before, to get a feeling about them. Below are the steps that I did, hopefully you find it helpful for you to get started.

##### Install jBPM and JBoss #####

* Download the [jbpm 4.0 distribution](http://sourceforge.net/projects/jbpm/files/a%29%20jBPM%204/jbpm-4.0.zip/download).
* Extract it into your local directory, here mine is: /local/deploy
* Go to jbpm-4.0/jboss.
* Added the build.properties into ${user.home}/.jbpm4 directory if you want to update the default properties in the ant script. (below is what I did, I used the jboss-5.0.1.GA, and I already had the zip ball in the /local/softwares directory)

> database=hsqldbjboss.version=5.0.1.GAjbpm.parent.dir=/local/deployjboss.distro.dir=/local/softwares

You can refer to [this wiki entry](http://www.jboss.org/community/wiki/jBPM4BuildingfromSource) for detail.

5. run following command to install jboss and jbpm.
> ant install.jbossant install.jbpm.into.jboss

Now, you go to the jbpm-4.0/jboss-5.0.1.GA, run:
> bin/run.sh -c default.

It should start the jbpm server properly, later then we need to populate the schema and initialized sqls.

##### Import jBPM db schema and identity sql #####

1. Go to jBPM-4.0/db, run following command to populate the schema and sql.
> ant create.jbpm.schemaant load.example.identities

Now, you can go to http://localhost:8080/jbpm-console, use the 'alex/password' as the username and password combination, you should be able to log in the jbpm console.

##### Install the jBPM eclipse plugin #####

I won't described the steps for its installation, as it was very well documented in the [user guide](http://docs.jboss.com/jbpm/v4.0/userguide/html_single/#graphicalprocessdesigner).
*Note*: it was based on the eclipse 3.5 jee distribution.

In the next entry, we will try to build a simple process by using the Eclipse GUI.
