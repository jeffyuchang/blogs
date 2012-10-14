--- 
layout: post
title: "Exploring ODE Part IV: BpelServer API"
published: true
tags: 
- BPM
- Java
- ODE
- RiftSaw
---
If you look at the ODE source code, the BpelServer API is a very important one. In this entry, we will look at the class that how we use the BpelServer, for the ODE, lets look at the [ODEServer](http://svn.apache.org/repos/asf/ode/trunk/axis2/src/main/java/org/apache/ode/axis2/ODEServer.java) source code.

{% highlight java %}
__log.debug("Initializing transaction manager");
initTxMgr();
__log.debug("Creating data source.");
initDataSource();
__log.debug("Starting DAO.");
initDAO();
EndpointReferenceContextImpl eprContext = new EndpointReferenceContextImpl(this);
__log.debug("Initializing BPEL process store.");
initProcessStore(eprContext);
__log.debug("Initializing BPEL server.");
initBpelServer(eprContext);
 __log.debug("Initializing HTTP connection manager");
 initHttpConnectionManager();

// Register BPEL event listeners configured in axis2.properties file.
registerEventListeners();
registerMexInterceptors();
registerContextInterceptors(); 
...
{% endhighlight %}

As above code shown:

* Initialized the TransactionManager, which will be used in the datasource and scheduler service.
* Created the data source.
* Created the DAOConnectionFactory, the extension API is BpelDAOConnectionFactory.
* Created the EndpointReferenceContext, which takes care of resolving EndpointReference.
* Created the Process Store, it takes care of process deploying, undeploying, list etc.
* Initializing the BpelServer, includes following actions: setMessageExchangeContext, setDaoConnectionFactory, setBindingContext, the BindingContext and the MessageExchangeContext are the extension points for communicating with partner services.
* Register the EventListners, MexInterceptors etc, these are all the APIs in BpelServer.


You also could see how we initialize the BpelServer in the [MockBpelServer](http://svn.apache.org/repos/asf/ode/trunk/engine/src/test/java/org/apache/ode/bpel/runtime/MockBpelServer.java) class. In the [Riftsaw project](http://www.jboss.org/riftsaw),it is in the [BpelEngineImpl](http://anonsvn.jboss.org/repos/riftsaw/trunk/runtime/engine/src/main/java/org/jboss/soa/bpel/runtime/engine/ode/BPELEngineImpl.java) class. In the Riftsaw project, we are adding another implementation for the ProcessStore that leverages the JBoss Application Server's Deployer mechanism, also we adding another implementation for the BindingContext that use the JAXWS based approach, which used the JBossWS to accomplish.
