<p align="center"> 
<img src='./images/banner.jpg'>
</p>
*Image Courtesy*: [Boloji.com](http://www.boloji.com/index.cfm?md=Content&sd=Articles&ArticleID=1452)

[**<<Back Home**](./README.md)

# Understanding the Hub parameters

## Configuration parameters that can be set at the Hub.

The **Selenium Grid** has many a configuration parameters that would help you to tweak its behaviour to suite your needs.

The good part of all this is that you don't need to look for documentation else where. Its embedded as part of the selenium standalone uber jar.

To see the documentation related to the Selenium Hub, run :

```
java -jar selenium-server-standalone-3.4.0.jar -role hub -help
```

See [**_here_**](./HUB_CONFIG_DOC.md) for the documentation of the Hub.

Lets see some of the most commonly used configuration parameters applicable for the Hub.

1. [Explicitly specifying the host for the hub](#specifyhost)
2. [Having the hub listen on a different port](#specifyport)
3. [Specifying the Hub configuration via a JSON file](#jsonconfig)
4. [Disabling default servlets](#disableservlets)
5. [Capturing the Hub logs in a file](#logfile)

### Explicitly specifying the host for the hub. <a name="specifyhost"></a>
The selenium uber jar by default always uses the first **Non Loopback ip4** that it finds.

Quoting the definition of _Loopback address_ from [**_Webopedia_**](http://www.webopedia.com/TERM/L/loopback_address.html)

> Loopback address is a special IP number (127.0.0.1) that is designated for the software loopback interface of a machine. The loopback interface has no hardware associated with it, and it is not physically connected to a network. The loopback interface allows IT professionals to test IP software without worrying about broken or corrupted drivers or hardware.

Sometimes you may want to override this and provide the hostname to be used. You can do that as below :

```
java -jar selenium-server-standalone-3.4.0.jar -role hub \
-host 192.168.1.2 -port 8080
```

Since we provided a different host value, we would need to use the same value when spinning off a node. So the command would look like this :

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-hub http://192.168.1.2:8080/grid/register
```

### Having the hub listen on a different port. <a name='specifyport'></a>
The Selenium Hub by default always listens on port `4444`.In order to have the hub use a different port (say for e.g., `8080`) spawn the Hub as shown below:

```
java -jar selenium-server-standalone-3.4.0.jar -role hub -port 8080
```

### Specifying the Hub configuration via a JSON file. <a name='jsonconfig'></a>
Incase we feel that the command to start the hub is getting long, we can wrap up the configuration for the Hub via a JSON file as well. 

So if we were to specify the host and the port in the JSON file, it could look like below :

```json
{
    "host": "192.168.1.2",
    "port": 8080
}
``` 

Here's the command : 

```
java -jar selenium-server-standalone-3.4.0.jar -role hub \
-hubConfig hubconfig.json
```

Incase you would like to know what else can go within this JSON configuration file, take a look [**_here_**](https://github.com/SeleniumHQ/selenium/blob/master/java/server/src/org/openqa/grid/common/defaults/DefaultHub.json)

### How do I disable some of the default servlets associated with the Hub ? <a name='disableservlets'></a>

The Hub by default exposes the below end-points via servlets (For the sake of e.g., lets assume that the hub is running on `localhost` and listening on `4444` port) : 

* **Registration URL** : **_http://localhost:4444/grid/register/_** - This is the end-point to which a selenium node will talk to, when you spawn a node, as part of the node wiring in itself to the hub. 

* **Client facing URL** : **_http://localhost:444/wd/hub_** - This is the end-point to which your test cases will talk to, as part of utilising the Grid as the execution environment. 

  _Hint :_ Remember using these two lines in your Java tests? 

```java
URL url = new URL("http://localhost:444/wd/hub");
RemoteWebDriver driver = new RemoteWebDriver(url, DesiredCapabilities.firefox());
```
 
* **Proxy Querying URL** : **_http://localhost:4444/grid/api/proxy/_** - This is the end point which can be used to get details about a particular node. 
* **Hub querying URL** : **_http://localhost:4444/grid/api/hub/_** - This is the end point which can be used to get details about the Hub in a JSON format.
* **Test Session querying URL** : **_http://localhost:4444/grid/api/testsession/_** - This is the end point which can be queried to get details about a particular session.
* **Display Help URL** : **_http://localhost: 4444/_** - This end point gets displayed whenever an invalid end-point on the Hub is loaded. 

  **Class Name** : `org.openqa.grid.web.servlet.DisplayHelpServlet` [ This is an optional servlet and can be disabled ]

* **Grid console URL** : **_http://localhost:4444/grid/console/_** - This is the end point which end users can load up on their browser to see details about all the nodes that are registered with the hub.

  **Class Name** : `org.openqa.grid.web.servlet.beta.ConsoleServlet` [ This is an optional servlet and can be disabled ]

* **Resources URL** : **_http://localhost:4444/grid/resources/_** - This end point is internally used by the Grid to retrieve the browser icons which get displayed in the **Grid Console URL**.

  **Class Name** : `org.openqa.grid.web.servlet.ResourceServlet` [ This is an optional servlet and can be disabled ]

* **Hub lifecycle manager URL** : **_http://localhost:4444/lifecycle-manager?action=shutdown_** - This end point facilitates shutting down the Hub.

  **Class Name** : `org.openqa.grid.web.servlet.LifecycleServlet` [ This is an optional servlet and can be disabled ]

* **Heartbeat URL** : **_http://localhost:4444/heartbeat_** - This is more of a Grid v1 URL. Its not used any more but exists for compatibility purposes. This can be invoked to check if a particular node is registered with the hub.

  **Class Name** : `org.openqa.grid.web.servlet.Grid1HeartbeatServlet` [ This is an optional servlet and can be disabled ]

You can disable some of the servlets _(the ones that are marked as optional)_. For e.g., if you wanted to disable the console servlet and the life cycle servlet, it can be done using the below command:

```
java -jar selenium-server-standalone-3.4.0.jar -role hub \
-withoutServlet org.openqa.grid.web.servlet.LifecycleServlet,\
org.openqa.grid.web.servlet.beta.ConsoleServlet
```
[**_Talk2Grid_**](https://github.com/RationaleEmotions/talk2grid) *is one library that internally consumes some of these URLs and exposes these information in a user friendly manner*.

### Capturing the Hub logs in a file. <a name='logfile'></a>

The Selenium hub logs can be redirected to a file.

For e.g., to enable debug level logs and redirect them to the file _/logs/grid.log_, use:

```
java -jar selenium-server-standalone-3.4.0.jar -role hub \
-log /logs/grid.log -debug true
```

[**<<Back Home**](./README.md)
