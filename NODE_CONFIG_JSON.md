<p align="center"> 
<img src='./images/banner.jpg'>
</p>

# Understanding the Node configuration JSON file.

A typical node configuration file would look something like this :

```json
{
  "capabilities":
  [
    {
      "browserName": "firefox",
      "maxInstances": 5,
      "seleniumProtocol": "WebDriver"
    },
    {
      "browserName": "chrome",
      "maxInstances": 5,
      "seleniumProtocol": "WebDriver"
    },
    {
      "browserName": "internet explorer",
      "maxInstances": 1,
      "seleniumProtocol": "WebDriver"
    }
  ],
  "proxy": "org.openqa.grid.selenium.proxy.DefaultRemoteProxy",
  "maxSession": 5,
  "port": 5555,
  "register": true,
  "registerCycle": 5000,
  "hub": "http://localhost:4444",
  "nodeStatusCheckTimeout": 5000,
  "nodePolling": 5000,
  "role": "node",
  "unregisterIfStillDownAfter": 60000,
  "downPollingLimit": 2,
  "debug": false,
  "servlets" : [],
  "withoutServlets": [],
  "custom": {}
}
```

So lets look at some of the salient configurations in this JSON file:

1. **capabilities** : A typical capability comprises of two most components viz.,
  * **browserName** : Represents the browser flavor. Can be one of the following:
        * **firefox** *for Firefox*
        * **chrome** *for Google chrome*
        * **internet explorer** *for Internet explorer*
        * **MicrosoftEdge** *for Microsoft Edge*
        * **operablink** *for Opera browser*
        * **safari** *for Safari browser*
        * **phantomjs** *for PhantomJS browser*
        * **htmlunit** *for HTMLUnit*
  * **maxInstances** : On a per browser basis, represents how many concurrent instances can be spun off.
  * **seleniumProtocol** : Its better to leave this as is or even skip it (Default is always **WebDriver**). The other value that can be specified is **Selenium**. This value represents if an incoming test is working with the Selenium #1 protocol or the Selenium2 protocol. Now that SeleniumRC is deprecated, its better to leave this value to its default state.

  *Tip:* The configuration parameter `maxSession` has the final say in determining how many tests can be concurrently executed on a given node. So going by the sample JSON from above, even though the node is capable of running **5 firefox based tests**, **5 chrome based tests** and **1 Internet Explorer based test**, at any given point only 5 tests across all these 3 browser flavors can run.
  
2. **proxy** : Accepts a fully qualified class name. The class has to be an implementation of the core selenium interface `org.openqa.grid.internal.RemoteProxy`. A Proxy can be visualised as an object that lives in the Hub's memory and represents one remote node. For more details around what is a Proxy and how can we plug-in a customization around this, refer [**_here_**](./CUSTOMIZE_GRID.md#proxy)

3. **maxSession** : Lets you decide how many concurrent tests can run at any given point in time in a node.

4. **port** : Lets you specify the port on which the node is to be listening on.

5. **register** and **registerCycle** : When **register** is set to `true` (which is the default value), the node periodically tries to register itself every **n** milliseconds (the value that you specified via **registerCycle**)

6. **nodeStatusCheckTimeout**, **nodePolling**, **unregisterIfStillDownAfter** and **downPollingLimit** : All these 4 parameters are related to each other in terms of functionality. **nodePolling** a value expressed in milliseconds, decides how often the Hub will try to check if the node is still working or not. If a particular node is down after **n** millseconds (value specified via **unregisterIfStillDownAfter**) and after the hub has attempted **x** number of times (value specified via **downPollingLimit**), the hub removes of the faulty proxy object from its registry and prevents tests being routed to the problematic node. The value specified in milliseconds for **nodeStatusCheckTimeout** , determines what should be the connection timeout and what should be the socket timeout.

Quoting the definition of both these timeouts from [**_Baeldung's tutorial_**](http://www.baeldung.com/httpclient-timeout) :

> * Connection Timeout (http.connection.timeout) – the time to establish the connection with the remote host.
> * Socket Timeout (http.socket.timeout) – the time waiting for data – after the connection was established; maximum time of inactivity between two data packets.

Incase you would like to know what else can go within this JSON configuration file, take a look [**_here_**](https://github.com/SeleniumHQ/selenium/blob/master/java/server/src/org/openqa/grid/common/defaults/DefaultNodeWebDriver.json)

