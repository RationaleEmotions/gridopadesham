<p align="center"> 
<img src='./images/banner.jpg'>
</p>
*Image Courtesy*: [Boloji.com](http://www.boloji.com/index.cfm?md=Content&sd=Articles&ArticleID=1452)

[**<<Back Home**](./README.md)

# Configuration parameters for the Node

To see the documentation related to the Selenium Node, run :

```
java -jar selenium-server-standalone-3.4.0.jar -role node -help
```
You should see something like below :

```
Usage: <main class> [options]
Options:
--version, -version
   Displays the version and exits.
   Default: false
-browserTimeout
   <Integer> in seconds : number of seconds a browser session is allowed to
   hang while a WebDriver command is running (example: driver.get(url)). If the
   timeout is reached while a WebDriver command is still processing, the session
   will quit. Minimum value is 60. An unspecified, zero, or negative value means
   wait indefinitely.
   Default: 0
-capabilities, -browser
   <String> : comma separated Capability values. 
   Example: -capabilities browserName=firefox,platform=linux -capabilities browserName=chrome,platform=linux
   Default: [Capabilities [{seleniumProtocol=WebDriver, browserName=chrome, maxInstances=5}], 
   Capabilities [{seleniumProtocol=WebDriver, browserName=firefox, maxInstances=5}], 
   Capabilities [{seleniumProtocol=WebDriver, browserName=internet explorer, maxInstances=1}]]
-cleanUpCycle
   <Integer> in ms : specifies how often the hub will poll running proxies
   for timed-out (i.e. hung) threads. Must also specify "timeout" option
-custom
   <String> : comma separated key=value pairs for custom grid extensions.
   NOT RECOMMENDED -- may be deprecated in a future revision. Example: -custom
   myParamA=Value1,myParamB=Value2
   Default: {}
-debug
   <Boolean> : enables LogLevel.FINE.
   Default: false
-downPollingLimit
   <Integer> : node is marked as "down" if the node hasn't responded after
   the number of checks specified in [downPollingLimit].
   Default: 2
-host
   <String> IP or hostname : usually determined automatically. Most commonly
   useful in exotic network configurations (e.g. network with VPN)
-hub
   <String> : the url that will be used to post the registration request.
   This option takes precedence over -hubHost and -hubPort options.
   Default: http://localhost:4444
-hubHost
   <String> IP or hostname : the host address of the hub we're attempting to
   register with. If -hub is specified the -hubHost is determined from it.
-hubPort
   <Integer> : the port of the hub we're attempting to register with. If
   -hub is specified the -hubPort is determined from it.
-id
   <String> : optional unique identifier for the node. Defaults to the url
   of the remoteHost, when not specified.
-jettyThreads, -jettyMaxThreads
   <Integer> : max number of threads for Jetty. An unspecified, zero, or
   negative value means the Jetty default value (200) will be used.
-log
   <String> filename : the filename to use for logging. If omitted, will log
   to STDOUT
-maxSession
   <Integer> max number of tests that can run at the same time on the node,
   irrespective of the browser used
   Default: 5
-nodeConfig
   <String> filename : JSON configuration file for the node. Overrides
   default values
-nodePolling
   <Integer> in ms : specifies how often the hub will poll to see if the
   node is still responding.
   Default: 5000
-nodeStatusCheckTimeout
   <Integer> in ms : connection/socket timeout, used for node "nodePolling"
   check.
   Default: 5000
-port
   <Integer> : the port number the server will use.
   Default: 5555
-proxy
   <String> : the class used to represent the node proxy. Default is
   [org.openqa.grid.selenium.proxy.DefaultRemoteProxy].
   Default: org.openqa.grid.selenium.proxy.DefaultRemoteProxy
-register
   if specified, node will attempt to re-register itself automatically with
   its known grid hub if the hub becomes unavailable.
   Default: true
-registerCycle
   <Integer> in ms : specifies how often the node will try to register
   itself again. Allows administrator to restart the hub without restarting (or
   risk orphaning) registered nodes. Must be specified with the "-register"
   option.
   Default: 5000
-role
   <String> options are [hub], [node], or [standalone].
   Default: node
-servlet, -servlets
   <String> : list of extra servlets the grid (hub or node) will make
   available. Specify multiple on the command line: -servlet tld.company.ServletA
   -servlet tld.company.ServletB. The servlet must exist in the path:
   /grid/admin/ServletA /grid/admin/ServletB
   Default: []
-timeout, -sessionTimeout
   <Integer> in seconds : Specifies the timeout before the server
   automatically kills a session that hasn't had any activity in the last X seconds. The
   test slot will then be released for another test to use. This is typically
   used to take care of client crashes. For grid hub/node roles, cleanUpCycle
   must also be set.
   Default: 1800
-unregisterIfStillDownAfter
   <Integer> in ms : if the node remains down for more than
   [unregisterIfStillDownAfter] ms, it will stop attempting to re-register from the hub.
   Default: 60000
-withoutServlet, -withoutServlets
   <String> : list of default (hub or node) servlets to disable. Advanced
   use cases only. Not all default servlets can be disabled. Specify multiple on
   the command line: -withoutServlet tld.company.ServletA -withoutServlet
   tld.company.ServletB
   Default: []
```

[**<<Back Home**](./README.md)
