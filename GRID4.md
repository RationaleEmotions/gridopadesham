# Grid-"o"-padesham
<p align="center"> 
<img src='./images/banner.jpg'>
</p>
*Image Courtesy*: [Boloji.com](http://www.boloji.com/index.cfm?md=Content&sd=Articles&ArticleID=1452)


**Selenium Grid-4** is the latest and greatest that's happening in Selenium Grid.

The Grid has been completely re-architected so that it's more scaleable and supports elasticity of the Grid (ability for the Grid to scale up and scale down with ease.

Like **Grid-3**, the **Grid-4** standalone also comes baked in with help.

```bash
$ java -jar selenium-server-4.0.0-alpha-5.jar
Selenium Server commands

A list of all the commands available. To use one, run `java -jar selenium.jar
commandName`.

  distributor  Adds this server as the distributor in a selenium grid.
  hub          A grid hub, composed of sessions, distributor, and router.
  info         Prints information for commands and topics.
  node         Adds this server as a node in the selenium grid.
  router       Creates a router to front the selenium grid.
  sessions     Adds this server as the session map in a selenium grid.
  standalone   The selenium server, running everything in-process.

For each command, run with `--help` for command-specific help

Use the `--ext` flag before the command name to specify an additional classpath
to use with the server (for example, to provide additional commands, or to
provide additional driver implementations). For example:

  java -jar selenium.jar --ext example.jar:dir standalone --port 1234
```


The Selenium Grid standalone jar basically supports running in different modes which are referred to as **commands** that can be passed to the standalone uber jar.

### Run as a standalone

This is probably where most of us will start when it comes to getting introduced with Selenium Grid 4. Read more about it [here](./GRID4_STANDALONE.md)

