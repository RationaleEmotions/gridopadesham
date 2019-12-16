<p align="center"> 
<img src='./images/banner.jpg'>
</p>
*Image Courtesy*: [Boloji.com](http://www.boloji.com/index.cfm?md=Content&sd=Articles&ArticleID=1452)

[**<<Back Home**](./README.md)

# Programmatically start/stop a Local Grid.

The below code snippet shows how to start and stop a local grid using Selenium `v3.141.59`

Lets start off by defining a simple interface that looks like below:

```java
package com.rationaleemotions.grid;

public interface Bootable {

  void start();

  void stop();
}
```

Now lets implement this interface and build logic to start and stop hub and node.

```java
package com.rationaleemotions.grid;

import org.openqa.grid.internal.utils.configuration.GridHubConfiguration;
import org.openqa.grid.web.Hub;

public class LocalHub implements Bootable {

  private Hub hub;

  public LocalHub(GridHubConfiguration cfg) {
    hub = new Hub(cfg);
  }

  @Override
  public void start() {
    hub.start();
  }

  public boolean isNodeRegistered() {
    return !hub.getRegistry().getAllProxies().isEmpty();
  }

  @Override
  public void stop() {
    hub.stop();
  }
}
```

```java
package com.rationaleemotions.grid;

import java.util.concurrent.TimeUnit;
import org.openqa.grid.internal.utils.SelfRegisteringRemote;
import org.openqa.grid.internal.utils.configuration.GridNodeConfiguration;
import org.openqa.selenium.remote.server.SeleniumServer;

public class LocalNode implements Bootable {

  private SelfRegisteringRemote node;
  private LocalHub hub;

  public LocalNode(LocalHub hub, GridNodeConfiguration cfg) {
    this.hub = hub;
    node = new SelfRegisteringRemote(cfg);
    SeleniumServer server = new SeleniumServer(node.getConfiguration());
    node.setRemoteServer(server);
  }

  @Override
  public void start() {
    if (node.startRemoteServer()) {
      node.sendRegistrationRequest();
    }
    int attempt = 1;
    boolean registered = false;
    while (attempt++ <= 10) {
      if (hub.isNodeRegistered()) {
        registered = true;
        break;
      }
      try {
        TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
      }
    }

    if (!registered) {
      throw new IllegalStateException("Node registration failed");
    }
  }

  @Override
  public void stop() {
    node.stopRemoteServer();
  }
}
```

And here's a sample that uses the above mentioned implementation.

```java
package com.rationaleemotions;

import com.rationaleemotions.grid.LocalHub;
import com.rationaleemotions.grid.LocalNode;
import java.util.concurrent.TimeUnit;
import org.openqa.grid.internal.utils.configuration.GridHubConfiguration;
import org.openqa.grid.internal.utils.configuration.GridNodeConfiguration;

public class LocalGridBooter {

  public static void main(String[] args) throws InterruptedException {
    GridHubConfiguration hubCfg = new GridHubConfiguration();
    hubCfg.port = 4444;
    LocalHub hub = new LocalHub(hubCfg);
    hub.start();

    GridNodeConfiguration nodeCfg = new GridNodeConfiguration();
    nodeCfg.port = 5555;
    LocalNode node = new LocalNode(hub, nodeCfg);
    node.start();

    TimeUnit.SECONDS.sleep(10); //Simulating the grid usage.

    node.stop();
    hub.stop();
  }
}
```

[**<<Back Home**](./README.md)
