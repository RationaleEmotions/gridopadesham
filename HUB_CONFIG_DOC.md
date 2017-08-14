<p align="center"> 
<img src='./images/banner.jpg'>
</p>
*Image Courtesy*: [Boloji.com](http://www.boloji.com/index.cfm?md=Content&sd=Articles&ArticleID=1452)

# Configuration parameters for the Hub

To see the documentation related to the Selenium Hub, run :

```
java -jar selenium-server-standalone-3.4.0.jar -role hub -help
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
-matcher, -capabilityMatcher
   <String> class name : a class implementing the CapabilityMatcher
   interface. Specifies the logic the hub will follow to define whether a request can
   be assigned to a node. For example, if you want to have the matching process
   use regular expressions instead of exact match when specifying browser
   version. ALL nodes of a grid ecosystem would then use the same capabilityMatcher,
   as defined here.
   Default: org.openqa.grid.internal.utils.DefaultCapabilityMatcher@64a294a6
-cleanUpCycle
   <Integer> in ms : specifies how often the hub will poll running proxies
   for timed-out (i.e. hung) threads. Must also specify "timeout" option
   Default: 5000
-custom
   <String> : comma separated key=value pairs for custom grid extensions.
   NOT RECOMMENDED -- may be deprecated in a future revision. Example: -custom
   myParamA=Value1,myParamB=Value2
   Default: {}
-debug
   <Boolean> : enables LogLevel.FINE.
   Default: false
-host
   <String> IP or hostname : usually determined automatically. Most commonly
   useful in exotic network configurations (e.g. network with VPN)
-hubConfig
   <String> filename: a JSON file (following grid2 format), which defines
   the hub properties
-jettyThreads, -jettyMaxThreads
   <Integer> : max number of threads for Jetty. An unspecified, zero, or
   negative value means the Jetty default value (200) will be used.
-log
   <String> filename : the filename to use for logging. If omitted, will log
   to STDOUT
-maxSession
   <Integer> max number of tests that can run at the same time on the node,
   irrespective of the browser used
-newSessionWaitTimeout
   <Integer> in ms : The time after which a new test waiting for a node to
   become available will time out. When that happens, the test will throw an
   exception before attempting to start a browser. An unspecified, zero, or negative
   value means wait indefinitely.
   Default: -1
-port
   <Integer> : the port number the server will use.
   Default: 4444
-prioritizer
   <String> class name : a class implementing the Prioritizer interface.
   Specify a custom Prioritizer if you want to sort the order in which new session
   requests are processed when there is a queue. 
   Default to null ( no priority = FIFO)
-role
   <String> options are [hub], [node], or [standalone].
   Default: hub
-servlet, -servlets
   <String> : list of extra servlets the grid (hub or node) will make
   available. Specify multiple on the command line: 
   -servlet tld.company.ServletA
   -servlet tld.company.ServletB. The servlet must exist in the path:
   /grid/admin/ServletA /grid/admin/ServletB
   Default: []
-timeout, -sessionTimeout
   <Integer> in seconds : Specifies the timeout before the server
   automatically kills a session that hasn't had any activity in the last X seconds. 
   The test slot will then be released for another test to use. This is typically
   used to take care of client crashes. 
   For grid hub/node roles, cleanUpCycle must also be set.
   Default: 1800
-throwOnCapabilityNotPresent
   <Boolean> true or false : If true, the hub will reject all test requests
   if no compatible proxy is currently registered. If set to false, the request
   will queue until a node supporting the capability is registered with the grid.
   Default: true
-withoutServlet, -withoutServlets
   <String> : list of default (hub or node) servlets to disable. Advanced
   use cases only. Not all default servlets can be disabled. Specify multiple on
   the command line: -withoutServlet tld.company.ServletA -withoutServlet
   tld.company.ServletB
   Default: []
```
