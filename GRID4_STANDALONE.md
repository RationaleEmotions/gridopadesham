# Grid-"o"-padesham
<p align="center"> 
<img src='./images/banner.jpg'>
</p>
*Image Courtesy*: [Boloji.com](http://www.boloji.com/index.cfm?md=Content&sd=Articles&ArticleID=1452)

## Standalone Mode

Grid-4 standalone mode basically bakes in the following aspects of Grid-4

* **Node** - The component that actually runs your tests in chrome/firefox/IE/safari.
* **Distributor** - Imagine this to be central place which has knowledge of what all nodes are currently registered along with the capabilities of each of the nodes. This is the component that helps create new sessions by matching capabilities that are associated with every node.
* **sessions** - Imagine this to be a datastore that contains references to all the sessions that are created.

#### Setup:

The setup is pretty straight forward.

You start off by downloading the standalone uber jar from the Selenium downloads site [here](https://www.selenium.dev/downloads/)

After you downloaded the uber jar, you can fire up the JVM by running the below command:

```bash
java -jar selenium-server-4.0.0-alpha-5.jar standalone
```

For detailed information on the command line options refer [here](./GRID4_STANDALONE_HELP.md)

This will spin off a Standalone JVM that listens by default on port `4444`.

The console should look something like below:

```bash
22:11 $ java -jar selenium-server-4.0.0-alpha-5.jar standalone
22:11:36.706 INFO [EventBusOptions.createBus] - Creating event bus: org.openqa.selenium.events.local.GuavaEventBus
22:11:36.994 INFO [NodeOptions.lambda$addSystemDrivers$1] - Adding Capabilities {browserName: chrome} 9 times
22:11:36.996 INFO [NodeOptions.lambda$addSystemDrivers$1] - Adding Capabilities {browserName: firefox} 9 times
22:11:36.997 INFO [NodeOptions.lambda$addSystemDrivers$1] - Adding Capabilities {browserName: safari} 1 times
22:11:37.063 INFO [LocalDistributor.add] - Added node a3cee105-6cd8-446d-9fd7-70b2b16096c1.
22:11:37.063 INFO [Host.lambda$new$0] - Changing status of node a3cee105-6cd8-446d-9fd7-70b2b16096c1 from DOWN to UP. Reason: http://192.168.0.101:4444 is ok
22:11:37.305 INFO [Standalone.lambda$configure$1] - Started Selenium standalone 4.0.0-alpha-5 (revision b3a0d621cc)
```

#### Standalone backed by a Docker container

This is where things get interesting. The Selenium standalone now lets you back your node via Docker containers. Previously in Grid3 for you to be able to do something like this, you had to build something like [**Just-Ask**](https://github.com/rationaleEmotions/just-ask).

But now its pretty straight forward. 

As of alpha-5 selenium standalone doesn't work with `unix://` as the URL and so you would need to enable the http rest api support enabled in the docker daemon.

For e.g., on **OSX** the following can be done [ Refer [here](https://docs.docker.com/docker-for-mac/troubleshoot/#/known-issues) for details ]

```bash
docker run -d -v /var/run/docker.sock:/var/run/docker.sock \
-p 127.0.0.1:1234:1234 bobrik/socat \
TCP-LISTEN:1234,fork UNIX-CONNECT:/var/run/docker.sock
```

Once you do this, a `HTTP` `GET` on `http://localhost:1234/_ping` should return `OK`

```bash
19:43 $ curl http://localhost:1234/_ping
OK
```

Now lets start the standalone by having it pointed to the docker host

```bash
java -jar selenium-server-4.0.0-alpha-5.jar standalone \
-D selenium/standalone-chrome:latest '{"browserName": "chrome"}' \
--docker-url http://localhost:1234/ --detect-drivers false
```

The most important parts to remember here is that you need to disable the automatic driver detection capability of the standalone by using `--detect-drivers false`. If it's not done, then the new sessions will not be forwarded to the docker containers.

You should wait for the console to show you a message which says that the server is up and running. Something like below ( This is true especially the first time when docker daemon downloads images to your local machine ):

```bash
19:31 $ java -jar selenium-server-4.0.0-alpha-5.jar standalone -D selenium/standalone-chrome:latest '{"browserName": "chrome"}' --docker-url http://localhost:1234/ --detect-drivers false
19:31:53.737 INFO [EventBusOptions.createBus] - Creating event bus: org.openqa.selenium.events.local.GuavaEventBus
19:31:54.517 INFO [Docker.getImage] - Obtaining image: selenium/standalone-chrome:latest
19:31:54.545 INFO [V140Docker.getImage] - Listing local images: Reference{domain='docker.io', repository='selenium', name='standalone-chrome', tag='latest', digest='null'}
19:31:54.590 INFO [Docker.getImage] - Obtaining image: selenium/standalone-chrome:latest
19:31:54.590 INFO [V140Docker.getImage] - Listing local images: Reference{domain='docker.io', repository='selenium', name='standalone-chrome', tag='latest', digest='null'}
19:31:54.608 INFO [DockerOptions.lambda$configure$1] - Mapping Capabilities {browserName: chrome} to docker image selenium/standalone-chrome:latest 8 times
19:31:54.639 INFO [LocalDistributor.add] - Added node 0bcdf2d3-2809-40c9-9bd5-0403f1b73b44.
19:31:54.640 INFO [Host.lambda$new$0] - Changing status of node 0bcdf2d3-2809-40c9-9bd5-0403f1b73b44 from DOWN to UP. Reason: http://192.168.1.5:4444 is ok
19:31:54.680 INFO [Standalone.lambda$configure$1] - Started Selenium standalone 4.0.0-alpha-5 (revision b3a0d621cc)
```

Now what if you wanted to start a standalone that supports both firefox and chrome backed by docker containers ? It's simple. You just run a command that looks like below:

```bash
java -jar selenium-server-4.0.0-alpha-5.jar standalone \
-D selenium/standalone-chrome:latest '{"browserName": "chrome"}' \
-D selenium/standalone-firefox:latest '{"browserName": "firefox"}' \
--docker-url http://localhost:1234/ --detect-drivers false
```


#### Server status check

To check if the standalone is up and running you can either load the URL `` in the browser (or) run the below curl command.

To check if the node is ready, you can load the URL `http://localhost:4444/status` or use the following curl command.

**Note:** *There doesnt seem to exist a console similar to Grid3 yet in Grid4. Maybe its on its way. Maybe its buried somewhere and yet to be uncovered.*

```bash
22:18 $ curl http://localhost:4444/status
{
  "value": {
    "ready": true,
    "message": "Selenium Grid ready.",
    "nodes": [
      {
        "id": "a3cee105-6cd8-446d-9fd7-70b2b16096c1",
        "uri": "http:\u002f\u002f192.168.0.101:4444",
        "maxSessions": 19,
        "stereotypes": [
          {
            "capabilities": {
              "browserName": "safari"
            },
            "count": 1
          },
          {
            "capabilities": {
              "browserName": "chrome"
            },
            "count": 9
          },
          {
            "capabilities": {
              "browserName": "firefox"
            },
            "count": 9
          }
        ],
        "sessions": [
        ]
      }
    ]
  }
}
```

Now you can point against this standalone and run a simple test like below:

#### Sample test code

```java
import java.net.MalformedURLException;
import java.net.URL;
import org.openqa.selenium.By;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.remote.RemoteWebDriver;

public class StandaloneBrowserSample {

  public static void main(String[] args) throws MalformedURLException {
    RemoteWebDriver driver = null;
    try {
      driver = new RemoteWebDriver(new URL("http://localhost:4444"), new ChromeOptions());
      driver.get("https://selenium.dev/");
      String text = driver.findElement(By.tagName("body")).getText();
      System.err.println("Text [" + text + "]");
    } finally {
      if (driver != null) {
        driver.quit();
      }
    }
  }
}
```

**Did you notice ??**

The remote url has now changed to **http://IP:port** 

In the JSON response above did you notice that there's now a key called **sessions** ? Well, try setting up a breakpoint in the above java code and then invoking the `http://localhost:4444/status` again. You should see something like this in the **sessions** array.

So as you can see, there's a lot of interesting information about a session that got created in the `/status` response.

```json
"sessions": [
          {
            "sessionId": "29126a2498c4fe6708370340a0a3aa49",
            "stereotype": {
              "browserName": "chrome"
            },
            "currentCapabilities": {
              "acceptInsecureCerts": false,
              "browserName": "chrome",
              "browserVersion": "81.0.4044.92",
              "chrome": {
                "chromedriverVersion": "81.0.4044.69 (6813546031a4bc83f717a2ef7cd4ac6ec1199132-refs\u002fbranch-heads\u002f4044@{#776})",
                "userDataDir": "\u002fvar\u002ffolders\u002fmj\u002f81r6v7nn5lqgqgtfl18spfpw0000gn\u002fT\u002f.com.google.Chrome.OSZTEU"
              },
              "goog:chromeOptions": {
                "debuggerAddress": "localhost:60844"
              },
              "networkConnectionEnabled": false,
              "pageLoadStrategy": "normal",
              "platformName": "mac os x",
              "proxy": {
              },
              "setWindowRect": true,
              "strictFileInteractability": false,
              "timeouts": {
                "implicit": 0,
                "pageLoad": 300000,
                "script": 30000
              },
              "unhandledPromptBehavior": "dismiss and notify",
              "webauthn:virtualAuthenticators": true
            }
          }
        ]
```

Here's how it looks like, when you are running a safari based test against th standalone.

```json
"sessions": [
          {
            "sessionId": "903420D6-CD11-4420-8BF0-067121C4180E",
            "stereotype": {
              "browserName": "safari"
            },
            "currentCapabilities": {
              "acceptInsecureCerts": false,
              "browserName": "Safari",
              "browserVersion": "13.1",
              "platformName": "macOS",
              "safari:automaticInspection": false,
              "safari:automaticProfiling": false,
              "safari:diagnose": false,
              "safari:platformBuildVersion": "19E287",
              "safari:platformVersion": "10.15.4",
              "safari:useSimulator": false,
              "setWindowRect": true,
              "strictFileInteractability": false,
              "webkit:WebRTC": {
                "DisableICECandidateFiltering": false,
                "DisableInsecureMediaCapture": false
              }
            }
          }
        ]
```
