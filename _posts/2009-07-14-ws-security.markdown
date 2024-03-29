--- 
layout: post
title: '浅谈ws-security: 加密,签名,证书等'

published: true
tags: 
- Chinese
- Java
- Security
- WS-Security
type: post
status: publish
---
我们这个贴子所谈的,可以说是一个普遍的Security问题,不过因为我在做SOA的东西,所以就想从web service security的角度来探讨一下.必须声明的是,我并不是在解读 [ws-security](http://www.oasis-open.org/committees/tc_home.php?wg_abbrev=wss) 规范, 从某种意义上来说,是在学习Java Security.

从web service的应用场景来说, 要从endpoint A 发送 SOAP message 到 endpoint B. 一般情况下, 如果我们要保证这个soap message信息的安全,主要会从以下三个方面来考虑,阐述.

* 保密性(Confidentiality)
* 完整性(Integrity)
* 真实性(Authenticity)

我们接下来挨个来看下,是怎么满足这三个需求的,以及在JDK中,分别提供了什么样的API来于之对应.

##### 保密性(Confidentiality)#####

从大的分类来看,我们有两大种加密技术,分别是对称加密(symmetric encryption)和非对称加密(asymmetric encryption).

##### 对称加密(symmetric encryption)#####
对称加密是一个比较早的一种加密方式.比如说Alice要发送一个消息跟Bob,那么他们之前就应该说定了一个secret key,然后Alice把信息用这个secret key进行加密. 当信息到达Bob的时候, Bob再利用这个secret key来把它解密.
这种加密方式,可以是对每个bit进行加密,也可以是对block(chunk of bit, 比如64-bit)进行加密. 如果在是选择block加密的话,就得有一个补足(Padding)的概念,就比如说不够64-bit,你用0或者其他的来补足成一个block.
在Java中, Cipher类是负责加密,解密的.在对称加密中,需要以下三个属性.
1) 加密模式, 比如(ECB -Encryption Code Book, CBC, CFB, OFB, PCBC)
2) 加密算法, 比如(DES- Data Encryption Standard, TripleDES, AES, RC2, RC4, RC5, Blowfish, PBE)
3) 补足方式, 比如(No padding, PKCS5, OAEP, SSL3)

我们下面这个例子就是使用Java的Cipher类来加密.

{% highlight java %}
KeyGenerator keygen = KeyGenerator.getInstance("DES");
keygen.init(56);
SecretKey key = keygen.generateKey();


Cipher cipher = Cipher.getInstance("DES/ECB/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, key);

byte[] cipherText = cipher.doFinal("This is clear text".getBytes("UTF-8"));

cipher.init(Cipher.DECRYPT_MODE, key);
byte[] clearText = cipher.doFinal(cipherText);
{% endhighlight %}


先是生成一个56的DES的加密钥匙, 然后采用DES算法, ECB加密模式, PKCS5Padding补足方式来加密.


##### 非对称加密(Asymmetric Encryption) #####
对称加密存在的一个主要问题在于怎么安全的配发这个密钥给Alice和Bob呢?所以,我们后来引入了这个非对称加密,可以说现在这个是一个很普遍的加密方法.
非对称加密主要是他有两把钥匙, 公钥(Public Key)和私钥(Private Key),这两把钥匙是配对的. 一样的情况, Alice想发送信息给Bob, 那么流程就如下:
> Alice使用Bob的公钥加密信息 -> 加密过的信息 -> Bob用他自己的私钥解密.

这种情况下,大家只需要把公钥放在一个信任的机构(CA,例如Verisign),把私钥保留在自己手上,那么就解决了对称加密中密钥的配发问题.当然了,这个非对称加密的伟大之处还在于如果你没有私钥,至少在你的有生之年是无法破解的.
非对称加密的算法主要有两种: RSA 和 Diffie-Hellman. RSA是最为广泛使用的一种加密算法.
相对应的,我们就得使用keypair来加密解密,代码如下:

{% highlight java %}
KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA");
keyGen.initialize(1024);
KeyPair keypair = keyGen.generateKeyPair();


Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
cipher.init(Cipher.ENCRYPT_MODE, keypair.getPublic());

byte[] cipherText = cipher.doFinal("This is clear text".getBytes("UTF-8"));

cipher.init(Cipher.DECRYPT_MODE, keypair.getPrivate());
byte[] clearText = cipher.doFinal(cipherText);
{% endhighlight %}


不同的是,我们用公钥加密,然后用私钥来解密. 当然了,我们也可以用私钥加密,然后公钥来解密,这在我们后面的数字签名中会用到.


##### 完整性(Integrity)#####

我们可以用加密的手段对信息进行了加密,那么保证这个信息的明文不会被其他人看到,但是我们怎么来保证说这个发过来的信息就是完整的呢?
那么我们这里就因为一个叫信息摘要(Message Digest)的概念,所谓的信息摘要就是说用一个摘要算法,把信息生成一串字符,可以算是信息的足迹(fingerprint of the message). 这种算法是一个单方向的,也就是说,从生成的字符想倒推到原来的这个信息,那几乎是不可能的.这个算法的另外一个特点是,只要你对信息做一个小小的改动,那么生成的字符串差别就很明显.
常见的消息摘要(Message-digest)算法有: MD2, MD5 (128位的算法), SHA-1 (160位的算法)
在Java中,MessageDigest类来负责信息摘要,使用大致如下:

{% highlight java %}
MessageDigest messageDigest = MessageDigest.getInstance("MD5");
messageDigest.update("TestMessage".getBytes("UTF-8"));
System.out.println(new String(messageDigest.digest(), "UTF-8"));
{% endhighlight %}


##### 真实性(Authenticity)#####

上面我们解决了保密性和完整性,那我们怎么来保证Bob收到的这个信息就是Alice发的吗?有可能是Eva借用Alice的名字,拿到Bob的公钥发的.
基于这个考虑,我们就引入了一个数字签名(Digital Signature)的概念.数字签名就是在你发送信息的时候,用你的私钥进行加密,实际应用中,会对信息摘要进行数字签名,那么当Bob收到这个被数字签名过的信息摘要时,用Alice的公钥去解密,就可以确定这个消息是来自于Alice.
数字签名的算法有两种: RSA, DSA(Digital Signature Algorithm). 注意, RSA算法,既可用于加密,也可以用于数字签名. 但是DSA只可用于数字签名.
JDK支持以下的组合, MD2/RSA, MD5/RSA, SHA1/DSA, SHA1/RSA.
下面我们直接看JDK中Siganiture类来签名的使用.

{% highlight java %}
KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA");
keyGen.initialize(1024);
KeyPair keypair = keyGen.generateKeyPair();


Signature sig = Signature.getInstance("MD5WithRSA");
sig.initSign(keypair.getPrivate());
sig.update("This is message".getBytes("UTF-8"));

byte[] signature = sig.sign();

//Verify the signature
sig.initVerify(keypair.getPublic());
sig.update("This is message".getBytes("UTF-8"));
if (sig.verify(signature)) {
  System.out.println("Signature verified.");
}
{% endhighlight %}


到目前为止,我们介绍了加密,信息完整,数字签名这三个概念,应该说可以是完成了,你都懂了.

但是,等等,你是不是在经常听到说证书,X.509, keytool, keystore等等的概念呢?
下面,我们再来依次看看这几个概念.

##### 证书(certificates) and X.509 #####

之前我们说,在这种非对称加密算法中,我们一般会把公钥放在一个另外一个机构,这个机构专门负责来保管你的公钥,而且这个机构还负责核实你的真实性. 那么他一旦核实你后,就会创建一个东西,这个东西包含了你的公钥,你的个人信息(Identity),然后再用这个机构的私钥进行数字签名. 我们管这个东西就叫数字证书. 我们管这样的机构叫做CA( Certificate Authority)
X.509是一种存储证书的标准,我们通常直接叫X.509证书.

##### keytool, keystore#####

keystore是用来存放钥匙(包括公钥,私钥),证书的容器. keystore是一个以.keystore为后缀名的文件.在keystore里面的钥匙和证书可以有名字,叫做alias. 而且他们各自可以有密码保护.

JDK自带的keytool是用来创建keystore以及key的工具,下面我们来看几个常用的命令.

1) 创建keys (注意不能换行).

>keytool -genkey -alias serverkey -keypass serverpass -keyalg RSA -sigalg SHA1withRSA -keystore server.keystore -storepass nosecret
>keytool -genkey -alias clientkey -keypass clientpass -keyalg RSA -sigalg SHA1withRSA -keystore client.keystore -storepass nosecret


2)导出证书

> keytool -export -alias serverkey -keystore server.keystore -storepass nosecret -file servercert.cer

3)导入证书

> keytool -import -alias serverkey -keystore client.keystore -storepass nosecret -file servercert.cer


##### keystore, truststore的区别##### 

在跟web service 打交道,或者做测试的时候,你会经常听到trust store这个词,他其实就是一个CA,他是专门存放public key的; keystore是既存放public key,也存放private key的. [这个贴子上](http://www.jboss.org/community/index.html?module=bb&amp;op=viewtopic&amp;t=94406), Jason详细的解释了在ws security中,keystore和truststore的配置问题.

#### References ####
* [Java security Part 1, Crypto basics](http://www.ibm.com/developerworks/edu/j-dw-javasec1-i.html)
* [Understanding WS-Security](http://www.ibm.com/developerworks/edu/ws-dw-ws-understand-web-services4.html)
* [WS-Security signing and encryption](http://www.ibm.com/developerworks/webservices/library/j-jws5/index.html)
* [Keytool documentation](http://java.sun.com/j2se/1.5.0/docs/tooldocs/solaris/keytool.html)
