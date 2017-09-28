<p align="center"> 
<img src='./images/banner.jpg'>
</p>
*Image Courtesy*: [Boloji.com](http://www.boloji.com/index.cfm?md=Content&sd=Articles&ArticleID=1452)

[**<<Back Home**](./README.md)

# Build and use your own capability matcher

## What is a capability matcher ?
The capability matcher is what decides on which test should be routed to which node in the grid. If there aren’t any nodes that match what the user is asking for via their desired capabilities it’s upto the capability matcher to say so.

## How does the capability matcher work ?
The default capability matcher of `Grid2` considers the following attributes of a capability for matching :

1. Platform
2. Browser name
3. version (Browser version)
4. “applicationName” [ This is currently not documented much anywhere]

## Steps to build your own capability matcher

A custom capability matcher can be built by:

#### 1. Extending [**_org.openqa.grid.internal.utils.DefaultCapabilityMatcher_**](https://github.com/SeleniumHQ/selenium/blob/master/java/server/src/org/openqa/grid/internal/utils/DefaultCapabilityMatcher.java) **(or)**
#### 2. Implementing [**_org.openqa.grid.internal.utils.CapabilityMatcher_**](https://github.com/SeleniumHQ/selenium/blob/master/java/server/src/org/openqa/grid/internal/utils/CapabilityMatcher.java)

It's usually easy to build one's own capability matcher by extending `org.openqa.grid.internal.utils.DefaultCapabilityMatcher`.

Lets quickly see how a typical capability matcher can look like:

```java
public class CrazyCapabilityMatcher extends DefaultCapabilityMatcher {
    private final String crazyNodeName = "crazyNodeName";
    @Override
    public boolean matches(Map<String, Object> nodeCapability, 
                           Map<String, Object> requestedCapability) {
        boolean basicChecks = super.matches(nodeCapability, requestedCapability);
        if (! requestedCapability.containsKey(crazyNodeName)){
            // If the user didnt set the custom capability 
            // lets just return what the DefaultCapabilityMatcher
            // would return. That way we are backward compatibility and 
            // arent breaking the default behavior of the grid
            return basicChecks;
        }
        return (basicChecks && 
        nodeCapability.containsKey(crazyNodeName) && 
        nodeCapability.get(crazyNodeName).equals(requestedCapability.get(crazyNodeName)));
    }
}
```

Here we have defined our custom capability matcher to look for a capability named `crazyNodeName` and if its present then capability matching is to be done taking into consideration the value of `crazyNodeName`. 

If this capability is not in the user's desired capabilities, then we are to fall back to the logic of Selenium's default capability matcher.

## Wiring in your custom capability matcher

Once we have built a jar out of our custom capability matcher, it can be wired in using two ways (*Here* `caps-matcher.jar` *is the jar we built containing our capability matcher*):

#### 1. Via a configuration parameter to the hub.
```
java -cp caps-matcher.jar:selenium-server-standalone-3.4.0.jar \
-capabilityMatcher com.rationaleemotions.matcher.CrazyCapabilityMatcher \
org.openqa.grid.selenium.GridLauncherV3 -role hub
```

If you are on Windows environment, then the command would be :

```
java -cp caps-matcher.jar;selenium-server-standalone-3.4.0.jar \
-capabilityMatcher com.rationaleemotions.matcher.CrazyCapabilityMatcher \
org.openqa.grid.selenium.GridLauncherV3 -role hub
```

#### 2. Via a Hub configuration file.

Lets create a hub configuration JSON file (called `hub.json`) such as below :

```json
{
    "capabilityMatcher": "com.rationaleemotions.matcher.CrazyCapabilityMatcher"
}
```
Now we wire it in using the below command:

```
java -cp caps-matcher.jar:selenium-server-standalone-3.4.0.jar \
-hubConfig hub.json \
org.openqa.grid.selenium.GridLauncherV3 -role hub
```

If you are on Windows environment, then the command would be :

```
java -cp caps-matcher.jar;selenium-server-standalone-3.4.0.jar \
-hubConfig hub.json \
org.openqa.grid.selenium.GridLauncherV3 -role hub
```
Since we are talking about capability matching, there's one more additional Hub configuration parameter, related to capability matching. Its called `-throwOnCapabilityNotPresent`. This is a boolean parameter and is set to `true` by default. It can be wired in, in the same two ways as how we wired in a `capabilityMatcher`.

* `true` (*default value*) - This causes the Grid to throw one of the below exceptions:
    * `CapabilityNotPresentOnTheGridException` - When the capabilities as requested by a test, is not present with any node in the Grid farm.
    * `GridException` - When there are no nodes hooked onto the Hub.
* `false` - Disables all these exceptions, and a new session request is quietly returned back to the Grid's new session queue, hoping that in the *near future* a node may wire itself in, which can satisfy this new sessions' needs.

For more details on the Hub's configuration parameters, refer [**_here_**](./HUB_CONFIG.md)

***Tip***: 

* Notice how we are using `;` as a separator between the two jar names. On Windows `;` is a valid path separator and on non windows, its `:`. 
* Since we are using `java -cp` we have to specify the class name that contains the `main()` method. And yes you guessed it right. `org.openqa.grid.selenium.GridLauncherV3` contains the `main()` method. 

## Capability matcher in action

Now that we have a hub up and running with our custom capability matcher, lets put it to action.

### Spawn a node using the custom capability.

Lets create a node configuration JSON file (called `node.json`) such as below:

```json
{
  "capabilities":
  [
    {
      "browserName": "firefox",
      "maxInstances": 5,
      "crazyNodeName": "Rambo",
      "seleniumProtocol": "WebDriver"
    }
  ]
}
```
Using the above node configuration file, lets start a node:

```
java -jar selenium-server-standalone-3.4.0.jar -role node \
-nodeConfig node.json
```

We now have a Java test which looks like below :

```java
import java.net.MalformedURLException;
import java.net.URL;
 
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.testng.annotations.AfterClass;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
 
public class CrazyNodeTest {
    RemoteWebDriver wd = null;
 
    @BeforeClass
    public void beforeClass() throws MalformedURLException {
        DesiredCapabilities dc = new DesiredCapabilities();
        dc.setBrowserName(DesiredCapabilities.firefox().getBrowserName());
        dc.setCapability("crazyNodeName", "Rambo");
        wd = new RemoteWebDriver(new URL("http://localhost:4444/wd/hub"),dc);
    }

    @Test
    public void f() {
        wd.get("http://www.facebook.com");
    }
  
    @AfterClass
    public void afterClass() {
        wd.quit();
    }
}
```

So now our Hub will try and do the following in addition to performing the default capability matching:

#### Use case #1: `crazyNodeName` capability is not set with any node.

Our matcher `com.rationaleemotions.matcher.CrazyCapabilityMatcher` finds the capability `crazyNodeName`, so it will try to match it with what is available with the node i.e., it will check if the value of `Spiderman` in the requested capabilities (from the test) is equal to the value of in the available capabilities (from the node). But here since there are no nodes with this capability, the matching fails.

It then checks what is the value for the configuration parameter `-throwOnCapabilityNotPresent`. If the hub was started with its value as:

* `true` (default value), then a `CapabilityNotPresentOnTheGridException` exception is thrown. 
* `false` then the new session request is pushed back to the Hub's session queue in the hope that some other node will wire itself in with this capability in the near future, and eventually the match will happen.

#### Use case #2: `crazyNodeName` capability is available with one node with value as `Spiderman`.

Our matcher `com.rationaleemotions.matcher.CrazyCapabilityMatcher` finds the capability `crazyNodeName`, so it will try to match it with what is available with the node i.e., it will check if the value of `Spiderman` in the requested capabilities (from the test) is equal to the value of `Rambo` in the available capabilities (from the node). Now these two don't match. It then checks what is the value for the configuration parameter `-throwOnCapabilityNotPresent`. If the hub was started with its value as:

* `true` (default value), then a `CapabilityNotPresentOnTheGridException` exception is thrown. 
* `false` then the new session request is pushed back to the Hub's session queue in the hope that some other node will wire itself in with this capability in the near future, and eventually the match will happen.

#### Use case #3: `crazyNodeName` capability is available with one node with value as `Rambo`.

The test gets routed to the node.

Now lets consider a Java class such as this one:

```java
import java.net.MalformedURLException;
import java.net.URL;
 
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.testng.annotations.AfterClass;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
 
public class CrazyNodeTest {
    RemoteWebDriver wd = null;
 
    @BeforeClass
    public void beforeClass() throws MalformedURLException {
        DesiredCapabilities dc = new DesiredCapabilities();
        dc.setBrowserName(DesiredCapabilities.firefox().getBrowserName());
        wd = new RemoteWebDriver(new URL("http://localhost:4444/wd/hub"),dc);
    }

    @Test
    public void f() {
        wd.get("http://www.facebook.com");
    }
  
    @AfterClass
    public void afterClass() {
        wd.quit();
    }
}
```

For this test, in all the above listed 3 use cases, there will not be any errors thrown, because our matcher `com.rationaleemotions.matcher.CrazyCapabilityMatcher` just defaults back to the default capability matching provided by Selenium, since it didn't see the custom capability `crazyNodeName` in the desired capabilities from the test.

[**<<Back Home**](./README.md)
