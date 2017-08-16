<p align="center"> 
<img src='./images/banner.jpg'>
</p>
*Image Courtesy*: [Boloji.com](http://www.boloji.com/index.cfm?md=Content&sd=Articles&ArticleID=1452)

# Building your own proxy

Before we get into learning how to build our own proxy, lets start off with the basics. 

## What is a Proxy in the Selenium World ?

A *Proxy* is basically an object that represents a Selenium node. 

## Why do we need a Proxy object ?

The Selenium Grid eco-system contains atleast two JVMs. One JVM is for the **Hub** and the other JVMs are for each of the **Node**. So if the Selenium Grid infrastructure contains for e.g., a hub and 2 nodes, there are totally 3 JVMs involved here. In order for the Selenium Grid eco-system to function, the JVM that runs the Hub, needs to talk to the other JVMs that run as the nodes. In order for these JVMs to talk to each other, Selenium uses `HTTP` way of talking. The other way of interacting would be to use [**_RMI_**](https://en.wikibooks.org/wiki/Java_Programming/Remote_Method_Invocation) calls. The Proxy contains vital information (the most important among them being the **IP Address** and the **port** of the actual node). The hub uses this proxy object to actually interact with the remote node.

## How is a Proxy object created ?

When a selenium node is spun off, as part of the node coming up, it sends something called as a *Registration Request* to the Hub. This is the Selenium node's way of telling the hub *Hello there Selenium Hub, I am a new node and these are my capabilities. So any test from the user that matches these capabilities, you can forward them my way.This is my IP and my port so that you can forward tests to me*. The *Registration Request* comes in as a JSON PayLoad to the Hub. The Hub receives the JSON PayLoad and creates an Object that captures all the information it received from the selenium node. This Object is what is called as the **RemoteProxy**. 

_For all you design pattern enthusiasts_ : This is yet another example of the [**_Proxy Pattern_**](www.journaldev.com/1572/proxy-design-pattern).

## Steps to build your own proxy.

There are two ways in which one can go about building your own proxy.

* Extend the class `org.openqa.grid.selenium.proxy.DefaultRemoteProxy` (or)
* Implement the interface `org.openqa.grid.internal.RemoteProxy`.

Since `org.openqa.grid.selenium.proxy.DefaultRemoteProxy` already provides a lot of the basic infrastructure, its advised that one extends this class when one is building one's own proxy.

Now within your custom proxy implementation, you can basically plug-in whatever logic you would like to have. For e.g., lets say we are building our custom proxy by extending `org.openqa.grid.selenium.proxy.DefaultRemoteProxy`, the following are good possibilities :

* Override `beforeSession()` if you would like to do some adhoc setup right before a new session request is created.
* Override `getNewSession()` if you would like to intercept new session requests and do something with the new session (maybe even decline new sessions based on some conditions). One example for this would be, when you want to create a new session if and only if, you are trying to build a proxy that counts the number of test sessions and new session requests are to be blocked if the count has attained the max new session count.
* Override `afterSession()` if you would like to do some adhoc cleanup right after a session is cleaned up.
* Override `beforeRelease()` if you would like to perform some clean-up right before a test session is cleaned up due to a timeout. Things such as cleanup an Ondemand Selenium node container, can be handled here.
* Override `beforeCommand()` if you would like to do something right before every action is done. For e.g., you may want to build a request and response logger at the hub level. 
* Override `afterCommand()` if you would like to do something right after every action is done. For e.g., you may want to build a request and response logger at the hub level. 

***Tip:*** If you are trying to build something around `beforeCommand()` in terms of a logger, then remember to wrap the `javax.servlet.http.HttpServletRequest` with a customized `javax.servlet.http.HttpServletRequestWrapper` so that the stream can be read multiple times, else the request will never be forwarded to the actual node. See [**_here_**](http://mpas.github.io/post/2015/06/10/20150610_httpservletwrapper-3.1/) for a sample.

## Wiring in your custom Proxy.

Now that we have built our custom proxy, here's what needs to be done to hook in this proxy.

### On the node side 

Refer your custom proxy via one of the ways : 
1. Using the configuration parameter `-proxy`. Lets say your custom proxy is called `rationale.emotions.proxy.SimpleProxy`, then the command to start the selenium node would be : 

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-proxy rationale.emotions.proxy.SimpleProxy
```

2. Via the node configuration JSON file. So a simplified Node configuration JSON file could look like below:

```json
{
  "capabilities":
  [
    {
      "browserName": "firefox",
      "maxInstances": 5,
      "seleniumProtocol": "WebDriver"
    }
  ],
  "proxy": "rationale.emotions.proxy.SimpleProxy"
}
```

and now the command to start the selenium node would be :

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-nodeConfig node.json
```

To learn more about how to configure the node make sure you read [**_here_**](./NODE_CONFIG.md) and [**_here_**](./NODE_CONFIG_JSON.md)

### On the Hub side 
You need to build a jar out of your code that contains your custom proxy and then start the hub using it. For the sake of e.g., lets say our custom proxy jar name is called `simple-proxy.jar`, then the command to start the hub would be :

```
java -cp simple-proxy.jar:selenium-server-standalone-3.4.0.jar \
org.openqa.grid.selenium.GridLauncherV3 -role hub
```

If you are on Windows environment, then the command would be :

```
java -cp simple-proxy.jar;selenium-server-standalone-3.4.0.jar \
org.openqa.grid.selenium.GridLauncherV3 -role hub
```

***Tip***: 

* Notice how we are using `;` as a separator between the two jar names. On Windows `;` is a valid path separator and on non windows, its `:`. 
* Since we are using `java -cp` we have to specify the class name that contains the `main()` method. And yes you guessed it right. `org.openqa.grid.selenium.GridLauncherV3` contains the `main()` method. 

## When does one build our own proxy.

The use-cases are end-less, but here are the ones that I am aware of:

* **A Self healing grid**, wherein each node counts the number of tests that it has served and once the max count has been achieved, new sessions are denied. Once all the tests have run to completion, then a shutdown request is triggered from the proxy to the actual node. See [**_here_**](https://github.com/paypal/SeLion/tree/develop/server/src/main/java/com/paypal/selion/proxy)
* **An on-demand grid**, which is powered by an underlying system such as Docker. Here there are no actual no nodes, but only a ghost proxy exists which acts as a bridge between the Hub and the Docker Daemon. See [**_here_**](https://github.com/RationaleEmotions/just-ask/blob/master/src/main/java/com/rationaleemotions/proxy/GhostProxy.java)
* **A video recording grid**, wherein for every test, automatically video is recorded as well. See [**_here_**](https://github.com/aimmac23/selenium-video-node/tree/master/src/main/java/com/aimmac23/hub/proxy)
