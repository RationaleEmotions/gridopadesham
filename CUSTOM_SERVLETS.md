<p align="center"> 
<img src='./images/banner.jpg'>
</p>
*Image Courtesy*: [Boloji.com](http://www.boloji.com/index.cfm?md=Content&sd=Articles&ArticleID=1452)

[**<<Back Home**](./README.md)

# Add your own servlets to the Hub and the Node.

Sometimes we may want to enhance the Selenium Hub or the Selenium Node to customize it and add extra functionalities. Some of them can be :

* A mechanism to execute a script such as an AutoIT script (or) on the node on which the test is running to support automation.
* Lets say you have multiple Selenium Grid setups, and you would like to check if a Selenium Grid is already overloaded before routing a test to it. If the Hub is overloaded, then you might want to dynamically move on to the next Hub. 
* A mechanism to remotely shutdown a Hub or a Node.
* A mechanism to start and stop video recording in a node.

As you can see, the use cases are endless. So lets see how we can build this customization into the Hub (or) the node.

Selenium Grid lets you plug-in customizations into the Selenium Grid either at:

* **Selenium Hub**
* **Selenium Node**

The customization can be as a servlet by the following two ways :

1. By extending `javax.servlet.http.HttpServlet` : One can extend the `HttpServlet`, build a servlet and inject this servlet at either the _Selenium Hub_ / _Selenium Node_  (or)
2. By extending `org.openqa.grid.web.servlet.RegistryBasedServlet` : One can extend the `RegistryBasedServlet`, build a servlet and inject this servlet at the _Selenium Hub_.

So why does Selenium provide both these two mechanisms ? When you extend `RegistryBasedServlet`, you also get access to the internals of the Hub called as `org.openqa.grid.internal.Registry`. The `Registry` contains the vitals of the Hub and can be accessed via the method `getRegistry()`. So when you extend `RegistryBasedServlet`, your custom servlet gets access to this data structure as well using which you can get access to :

1. List of proxies attached to the Selenium Hub **(or)**
2. Set of active sessions running on the Selenium Hub and much more.

In order to start building our customization, we first need to consume the selenium server as a dependency (dependency is a lingo in the Maven world)

So add the below to your pom file :

```xml
<dependency>
	<groupId>org.seleniumhq.selenium</groupId>
	<artifactId>selenium-server</artifactId>
	<version>3.6.0</version>
</dependency>
```

Now lets look at how our servlet would look like (this servlet is going to retrieve all the proxies that are wired into the hub as a JSON payload).

```java
package rationale.emotions.servlets;

import com.google.gson.JsonArray;
import com.google.gson.JsonObject;
import org.openqa.grid.internal.ProxySet;
import org.openqa.grid.internal.Registry;
import org.openqa.grid.internal.RemoteProxy;
import org.openqa.grid.web.servlet.RegistryBasedServlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Iterator;

public class ListProxiesServlet extends RegistryBasedServlet {

    public ListProxiesServlet() {
        this(null);
    }

    public ListProxiesServlet(Registry registry) {
        super(registry);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse response) throws ServletException, IOException {
        ProxySet proxySet = getRegistry().getAllProxies();
        Iterator<RemoteProxy> iterator = proxySet.iterator();
        JsonArray array = new JsonArray();
        while (iterator.hasNext()) {
            JsonObject jsonObject = new JsonObject();
            RemoteProxy proxy = iterator.next();
            jsonObject.addProperty("IP_Address", proxy.getRemoteHost().toString());
            array.add(jsonObject);
        }
        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");
        response.setStatus(200);
        response.getWriter().append(array.toString());
    }
}
```

Now lets build our jar using : `mvn clean package`.

Now lets start the Hub and inject the servlet using the command :

```
java -cp simpleproxy-1.0-SNAPSHOT.jar:selenium-server-standalone-3.6.0.jar \ 
org.openqa.grid.selenium.GridLauncherV3 -role hub \
-servlets rationale.emotions.servlets.ListProxiesServlet
```

Once the hub is started, you should see an output as below :

```
09:39:36.529 INFO - Selenium build info: version: '3.6.0', revision: '6fbf3ec767'
09:39:36.530 INFO - Launching Selenium Grid hub
09:39:37.168 INFO - binding rationale.emotions.servlets.ListProxiesServlet to /grid/admin/ListProxiesServlet/*
2017-09-28 09:39:37.191:INFO::main: Logging initialized @925ms to org.seleniumhq.jetty9.util.log.StdErrLog
```

Now start a node and wire it into the hub.

Now run a `curl` command to invoke the newly injected servlet at the hub's side.

```
09:52 $ curl -i -H "Accept: application/json" "http://localhost:4444/grid/admin/ListProxiesServlet"
HTTP/1.1 200 OK
Date: Thu, 28 Sep 2017 04:22:09 GMT
Content-Type: application/json;charset=utf-8
Content-Length: 42
Server: Jetty(9.4.5.v20170502)

[{"IP_Address":"http://192.168.1.4:5555"}]
```

**Note:** 

* When adding a servlet at the hub the URL always begins with `http://<Hub_IP_Address>:<Hub_Port>/grid/admin/<Servlet_Name_Goes_Here>`

So for our servlet the URL was `http://localhost:4444/grid/admin/ListProxiesServlet`

Now lets look at a servlet that can be injected into the Selenium Node. Here's how the servlet would look like :

```java
package rationale.emotions.servlets;

import com.google.gson.JsonObject;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class HelloWorldServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");
        response.setStatus(200);
        JsonObject jsonObject = new JsonObject();
        jsonObject.addProperty("Greeting", req.getRemoteAddr());
        response.getWriter().append(jsonObject.toString());
    }
}
```
Now lets build our jar using : `mvn clean package`.

Now lets start the Node and inject the servlet using the command :
```
java -cp simpleproxy-1.0-SNAPSHOT.jar:selenium-server-standalone-3.6.0.jar \
org.openqa.grid.selenium.GridLauncherV3  -role node \
-hub http://localhost:4444/grid/register \
-servlets rationale.emotions.servlets.HelloWorldServlet

```

* When adding a servlet at the node the URL always begins with `http://<Node_IP_Address>:<Node_Port>/extra/<Servlet_Name_Goes_Here>`

Once the node is started, you should see an output as below :

```
10:05:32.109 INFO - Selenium build info: version: '3.6.0', revision: '6fbf3ec767'
10:05:32.110 INFO - Launching a Selenium Grid node
10:05:32.530 INFO - binding rationale.emotions.servlets.HelloWorldServlet to /extra/HelloWorldServlet/*
2017-09-28 10:05:32.550:INFO::main: Logging initialized @713ms to org.seleniumhq.jetty9.util.log.StdErrLog
```

Now run a `curl` command to invoke the newly injected servlet at the node's side.

```
09:52 $ curl -i -H "Accept: application/json" "http://localhost:5555/extra/HelloWorldServlet"
HTTP/1.1 200 OK
Date: Thu, 28 Sep 2017 04:37:44 GMT
Content-Type: application/json;charset=utf-8
Content-Length: 30
Server: Jetty(9.4.5.v20170502)

{"Greeting":"0:0:0:0:0:0:0:1"}
```

**Remember:** 

* When injecting a servlet at the node's side, you would always need to have your servlet extend `javax.servlet.http.HttpServlet`

* When injecting a servlet at the hub's side, and if you are extending `org.openqa.grid.web.servlet.RegistryBasedServlet`, make sure you add a default constructor to your servlet, else the hub servlet cannot be invoked at all.


[**<<Back Home**](./README.md)
