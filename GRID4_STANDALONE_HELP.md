# Grid-"o"-padesham
<p align="center"> 
<img src='./images/banner.jpg'>
</p>
*Image Courtesy*: [Boloji.com](http://www.boloji.com/index.cfm?md=Content&sd=Articles&ArticleID=1452)

[**<<Back Home**](./GRID4_STANDALONE.md)

### The standalone help. 

The standalone has documentation baked into it which can be invoked as below:

```bash
$ java -jar selenium-server-4.0.0-alpha-5.jar standalone -h
Usage: standalone [options]
  Options:
    --allow-cors
      Whether the Selenium server should allow web browser connections from
      any host
      Default: false
    --bind-bus
      Whether the connection string should be bound or connected
      Default: false
    --detect-drivers
      Autodetect which drivers are available on the current system, and add
      them to the node.
      Default: true
    --docker, -D
      Docker configs which map image name to stereotype capabilities (example
      `-D selenium/standalone-firefox:latest '{"browserName": "firefox"}')
    --docker-url
      URL for connecting to the docker daemon
    --events-implementation
      Full classname of non-default event bus implementation
    --host
      IP or hostname : usually determined automatically.
    --https-certificate
      Server certificate for https
    --https-private-key
      Private key for https
    --max-threads
      Maximum number of listener threads.
      Default: 24
    --plain-logs
      Use plain log lines
      Default: true
    -p, --port
      Port to listen on.
      Default: 4444
    --publish-events
      Connection string for publishing events to the event bus
    --registration-secret
      Node registration secret
    --structured-logs
      Use structured logs
      Default: false
    --subscribe-events
      Connection string for subscribing to events from the event bus
    --version
      Displays the version and exits.
      Default: false
```
