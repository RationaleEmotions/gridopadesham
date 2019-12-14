<p align="center"> 
<img src='./images/banner.jpg'>
</p>
*Image Courtesy*: [Boloji.com](http://www.boloji.com/index.cfm?md=Content&sd=Articles&ArticleID=1452)

[**<<Back Home**](./README.md)

# Adding Grid support for your new WebDriver implementation <a name='serviceloader'></a>

Lets say you have worked on building support for a new browser variant such as [**_Vivaldi_**](https://en.wikipedia.org/wiki/Vivaldi_(web_browser)) and 
Lets say using Java you have successfully built :

* The client bindings [ The one that extends `RemoteWebDriver` and which will be used by users in their tests ]
* The server binary [ which will be used to interact with the actual **Vivaldi** browser ]

Now you would like to extend this support so that your users can run their **Vivaldi** based selenium tests in a Grid environment. Wondering what to do ?

Its pretty simple. 
All that Selenium expects you to do is the following :

1. Create a file with its name as  `org.openqa.selenium.remote.server.DriverProvider` and place it within a folder named `META-INF/services`. Inside this file, refer to the class that basically implements `org.openqa.selenium.remote.server.DriverProvider` (or) which extends `org.openqa.selenium.remote.server.DefaultDriverProvider`. And yes, you guessed it right again, this is the [**_Service Loader_**](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html) approach that Selenium is using here. 
    
    See [**_here_**](https://github.com/MachinePublishers/jBrowserDriver/blob/master/src/META-INF/services/org.openqa.selenium.remote.server.DriverProvider) for one such sample file and see [**_here_**](https://github.com/MachinePublishers/jBrowserDriver/blob/master/src/com/machinepublishers/jbrowserdriver/SeleniumProvider.java) for an actual implementation of `DriverProvider` interface.

2. Now spin off your Selenium node using a command such as below. Here we are assuming that our jar which contains the `Vivaldi` browser's client bindings is `vivaldi-browser.jar`.

```
java -cp vivaldi-browser.jar:selenium-server-standalone-3.141.59.jar \
org.openqa.grid.selenium.GridLauncherV3 -role node
```

If you are on Windows environment, then the command would be :

```
java -cp simple-proxy.jar;selenium-server-standalone-3.141.59.jar \
org.openqa.grid.selenium.GridLauncherV3 -role node
```

***Tip***: 

* Notice how we are using `;` as a separator between the two jar names. On Windows `;` is a valid path separator and on non windows, its `:`. 
* Since we are using `java -cp` we have to specify the class name that contains the `main()` method. And yes you guessed it right. `org.openqa.grid.selenium.GridLauncherV3` contains the `main()` method. 

And that's it, you now have enabled support for your new web browser in selenium grid.

[**<<Back Home**](./README.md)
