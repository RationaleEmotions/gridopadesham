<p align="center"> 
<img src='./images/banner.jpg'>
</p>

# Understanding the Node parameters

## Configuration parameters that can be set at the Node.

The **Selenium Grid** has many a configuration parameters that would help you to tweak its behaviour to suite your needs.

The good part of all this is that you don't need to look for documentation else where. Its embedded as part of the selenium standalone uber jar.

To see the documentation related to the Selenium Node, run :

```
java -jar selenium-server-standalone-3.4.0.jar -role node -help
```

See [**_here_**](./NODE_CONFIG_DOC.md) for the documentation of the Node.

Lets see some of the most commonly used configuration parameters applicable for the Node.

1. [Wiring in your node to a Hub running on a different host](#wirein)
2. [Having the node listen on a different port](#differentport)
3. [Explicitly specifying the host for the node.](#specifyhost)
4. [Controlling the number of concurrent sessions on a node.](#maxsession)
5. [Specifying the Node configuration via a JSON file](#jsonconfig)
6. [Capturing the Node logs in a file](#logfile)
7. [Disabling default servlets](#disableservlets)

### Wiring in your node to a Hub running on a different host. <a name='wirein'></a>

You may want to spawn your node and wire it in with a Hub that is running on a different host. You can now do that using the below command :

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-hub http://192.168.1.2:7777/grid/register
```

You can alternately do this using the below command as well :

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-hubHost 192.168.1.2 -hubPort 7777
```

The `-hub` (or) the `-hubHost`/`-hubPort` combination parameters is usually not required if your node is running on the same machine as your hub.

Also remember that these two sets of configuration parameters are meant to be mutually exclusive, i.e., you should use only one of them to point your node to a hub. If you end up providing both, then the values provided via `-hubHost`/`-hubPort` is **ignored** and their values are determined from the URL specified via the `-hub` parameter.

### Having the node listen on a different port.<a name='differentport'></a>
The Selenium node by default always listens on port `5555`.In order to have the node use a different port (say for e.g., `8080`) spawn the node as shown below:

```
java -jar selenium-server-standalone-3.4.0.jar -role node -port 8080
```

### Explicitly specifying the host for the node.<a name='specifyhost'></a>

The selenium uber jar by default always uses the first **Non Loopback ip4** that it finds.

Quoting the definition of _Loopback address_ from [**_Webopedia_**](http://www.webopedia.com/TERM/L/loopback_address.html)

> Loopback address is a special IP number (127.0.0.1) that is designated for the software loopback interface of a machine. The loopback interface has no hardware associated with it, and it is not physically connected to a network. The loopback interface allows IT professionals to test IP software without worrying about broken or corrupted drivers or hardware.

Sometimes you may want to override this and provide the hostname to be used. You can do that as below :

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-host 192.168.1.2 -port 8080
```

### Controlling the number of concurrent sessions on a node.<a name='maxsession'></a>

By default when you spin off a node, it supports running 5 concurrent tests. This is a cumulative value across all the different browser flavors that are supported by the node.

Now to change this value, a node could be spun off as below :

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-maxSession 20
```

This causes the node to run at the max 20 conccurrent tests across browser flavors. You can have different values for different nodes. So this is essentially a per node value. 

So lets say you have spun off two nodes as below :

* **Node #1**

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-maxSession 2
```
* **Node #2**

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-maxSession 3
```

Now to quickly test this, we can run a shell script such as the one below :

```sh
#!/bin/bash
counter=1
for number in {1..6}
do
    curl -i \
    -H "Accept: application/json" \
    -X POST -d '{"desiredCapabilities":{"browserName":"chrome"}}' \
    http://localhost:4444/wd/hub/session
    echo "Created session " $counter
    let counter++
done 
```

When you run the above shell script, it will stall and not exit, at the **6th** attempt of creating a new session, because the **6th** session is going to go into the Hub's waiting queue. You can confirm that there are 5 chrome sessions consumed by opening up the Grid console : http://localhost:4444/grid/console

### Specifying the Node configuration via a JSON file. <a name='jsonconfig'></a>
Just as the hub, the node can also be tweaked via a JSON configuration file.


Lets say we were to restrict the maximum session count to `2` then a node configuration file would look like below :

```json
{
    "maxSession": 2
}
``` 

Here's the command : 

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-nodeConfig node.json
```

To learn more about what all goes into a typical node configuration file, read [**_here_**](./NODE_CONFIG_JSON.md).

Its easier to tweak a node via its node configuration JSON file.

### Capturing the Node logs in a file. <a name='logfile'></a>

The Selenium node logs can be redirected to a file.

For e.g., to enable debug level logs and redirect them to the file _/logs/node.log_, use:

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-log /logs/node.log -debug true
```

### How do I disable some of the default servlets associated with the Node ? <a name='disableservlets'></a>

The Node by default exposes the below end-points via servlets (For the sake of e.g., lets assume that the node is running on `localhost` and listening on `5555` port) : 

* **Client facing URL** : **_http://localhost:5555/wd/hub_** - This is the end-point to which the hub will route all your selenium test automation requests (*your W3C/JSONWireProtocol commands*). 
* **Resources URL** : **_http://localhost:5555/resources/_** - This end point is internally used by the Node to retrieve resources.

  **Class Name** : `org.openqa.grid.web.servlet.ResourceServlet` [ This is an optional servlet and can be disabled ]

* **Display Help URL** : **_http://localhost:5555/_** - This end point gets displayed whenever an invalid end-point on the node side is loaded. 

  **Class Name** : `org.openqa.grid.web.servlet.DisplayHelpServlet` [ This is an optional servlet and can be disabled ]

You can disable some of the servlets _(the ones that are marked as optional)_. For e.g., if you wanted to disable the help servlet and the resources servlet, it can be done using the below command:

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-withoutServlet org.openqa.grid.web.servlet.DisplayHelpServlet,\
org.openqa.grid.web.servlet.ResourceServlet
```
