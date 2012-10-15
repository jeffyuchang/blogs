--- 
layout: post
title: Getting/Using services deployed in JBoss AS 5.x
published: true
tags: 
- Java
- JBoss
- RiftSaw
type: post
status: publish
---
JBoss AS Version 5 and above used the [JBoss MicroContainer](http://jboss.org/jbossmc) project to accomplish the service management/dependency injection. If we want to write a service that uses a JBoss Server service, like the 'HAPartitionService' for example, the below shows several ways that can be done.

#### 1. Using JBoss MC's xml syntax#####
This is the most common and recommended way to do. Below is the [JBoss RiftSaw](http://www.jboss.org/riftsaw) Clustering service that uses the 'HAPartitonService' from JBoss AS Cluster.

{% highlight xml %}
   <bean name="RiftSawClusteringService" class="org.jboss.soa.bpel.clustering.JBossClusteringService">
   	 <property name="haPartition"><inject bean="HAPartition" /></property>
   	 <property name="bpelEngineName">bpel/Engine</property>
   	 <depends>BPELEngine</depends>
   </bean>
{% endhighlight %}


See Ales [Advanced Dependency Injections article](http://java.dzone.com/articles/a-look-inside-jboss-microconta-0)  for more about MC's xml syntax.


#### 2. Using JBoss MC's service programmatically ####

This way actually is the main topic for this blog entry. If you are a [Spring framework](http://www.springframework.org) user, you would know you can use BeanFactory or ApplicationContext to get the service bean. Then in JBoss, you would wonder, whats the equivalent way to do this?

1) org.jboss.kernel.Kernel, this is the service that has the KernelController and ControllerContext that we need for getting the service. so firstly, you can define a service in -jboss-bean.xml that injects the Kernel service, like the following:

{% highlight xml %}
<!--    Locate the single instance of the kernel    -->
<bean name="org.jboss.soa.bpel.runtime.util:service=KernelLocator"
     class="org.jboss.soa.bpel.runtime.integration.KernelLocator">
    <property name="kernel">
      <inject bean="jboss.kernel:service=Kernel" />
    </property>
</bean>
{% endhighlight %}


We've defined above class as following:

{% highlight java %}
package org.jboss.soa.bpel.runtime.integration;
public class KernelLocator{
  private static Kernel kernel;
  public static Kernel getKernel()  {    return KernelLocator.kernel;  }
  public void setKernel(Kernel kernel)  {    KernelLocator.kernel = kernel;  }
}
{% endhighlight %}


2) Use the KernelController and ControllerContext to get the service that we defined in -jboss-bean.xml.
Below is the code to actually obtain the started service out of JBoss AS.

{% highlight java %}
public class KernelAwareSPIFactory{
   @SuppressWarnings("unchecked")
   public <T> T getKernelProvidedSPI(String beanName, Class<T> spiArtifact)   {
      KernelController controller = KernelLocator.getKernel().getController();
      ControllerContext ctx = controller.getInstalledContext(beanName);
      return (T)ctx.getTarget();   }
}
{% endhighlight %}


In the end, let's take an example from [RiftSaw code base](http://jboss.org/riftsaw) to show how it was used.


In RiftSaw, we've defined a ServerConfig interface.

{% highlight java %}
public interface ServerConfig{
  /** The default bean name */
  String BEAN_NAME = "org.jboss.soa.bpel.runtime.util:service=ServerConfig";

  String getImplementationTitle();

  String getImplementationVersion();

  File getServerTempDir();

  File getServerDataDir();

  String getWebServiceHost();

  int getWebServicePort();

  .....
}
{% endhighlight %}


And then we've had the ServerConfigImpl class.

{% highlight java %}
public class ServerConfigImpl implements ServerConfig{  ....}
{% endhighlight %}


With this implementation, we've defined the ServerConfig service in the -jboss-bean.xml as following.

{% highlight xml %}
<!--       ServerConfig    -->
 <bean name="org.jboss.soa.bpel.runtime.util:service=ServerConfig"
        class="org.jboss.soa.bpel.runtime.integration.ServerConfigImpl">
    <property name="mbeanServer">
     <inject bean="JMXKernel" property="mbeanServer"/>
    </property>
    <property name="webServiceHost">${jboss.bind.address}</property>
  </bean>
{% endhighlight %}


Now, finally let's see how we get this ServerConfig Service in our code.

{% highlight java %}
ServerConfig = new KernelAwareSPIFactory()
               .getKernelProvidedSPI("org.jboss.soa.bpel.runtime.util:service=ServerConfig", ServerConfig.class
{% endhighlight %}


And then, here you go, you've got the ServerConfig service that is in the JBoss AS.


Hope this can help people who are doing integration with JBoss AS a little bit.

PS: Thanks [Glen](http://www.jroller.com/gmazza) for pointing out some grammar errors that I did earlier. As what we used to say "your patch has been applied, thanks a lot. ;)"
