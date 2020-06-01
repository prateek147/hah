---
layout: post
title: "iOS Application Security Part 15 – Static Analysis of iOS Applications using iNalyzer"
date: 2013-09-17 13:11
comments: true
categories: security
---

In the previous article, we looked at how we can use Sogeti Data protection tools to boot an iDevice using a custom ramdisk with the help of a bootrom exploit. In this article, we will look at a tool named iNalyzer than we can use for black box assessment of iOS applications. iNalyzer allows us to view the class information, perform runtime analysis and many other things. Basically it automates the efforts of decrypting the application, dumping class information and presents it in a much more presentable way. We can also hook into a running process just like Cycript and invoke methods during runtime. iNalyzer is developed and maintained by [AppSec Labs](https://appsec-labs.com) and its offical page can be found [here](https://appsec-labs.com/iNalyzer). iNalyzer is also made available open source and its github page can be found [here](https://github.com/appsec-labs/iNalyzer).

<!-- more -->

iNalyzer require some dependencies to be installed before use. Please make sure to install [Graphviz](http://www.graphviz.org/download..php) and [Doxygen](http://www.stack.nl/~dimitri/doxygen/download.html#srcbin) as iNalyzer won't function without these. Also, please note that while performing my tests on a Mac OS X Mountain Lion 10.8.4, i had problems with the latest version of Graphviz and it used to hang every time. Hence i downloaded an older version of Graphviz (v 2.30.1) and it worked fine for me. You can find older versions of Graphviz for Mac OS [here](http://www.graphviz.org/pub/graphviz/stable/macos/).

The first step is to install iNalyzer on your jailbroken iDevice. To do that, go to Cydia --> Manage and make sure the source http://appsec-labs.com/cydia/ is added as shown in the image below.

![1]({{site.baseurl}}/images/posts/ios15/1.PNG)

Then go to Search and search for _iNalyzer_. Depending on the iOS version that you are running, you should download the corresponding version of iNalyzer.

![2]({{site.baseurl}}/images/posts/ios15/2.PNG)

As you can see, i have already installed iNalyzer.

![3]({{site.baseurl}}/images/posts/ios15/3.PNG)

Now ssh into the device and navigate inside the iNalyzer application directory. iNalyzer is installed in the _/Applications_ directory because it needs to run as a root user. If you don't understand this concept, please make sure to read the previous articles in this series.

![4]({{site.baseurl}}/images/posts/ios15/4.png)

Run ./iNalyzer to start iNalyzer.

![5]({{site.baseurl}}/images/posts/ios15/5.png)

Now if you go to your homescreen and look at the iNalyzer app icon, you will see a badge icon number on top of it. This indicates that the app can be remotely accessed via a web interface and the port number is the badge icon number represented here. If you run ./iNalyzer again, it will stop iNalyzer. Hence, make sure to remember that running ./iNalyzer starts and closes the application alternatively.

![6]({{site.baseurl}}/images/posts/ios15/6.png)

Now find the ip address of your iDevice and open the url ip:port from your browser. In my case the port number is 5544 and the ip address of the device is 10.0.1.23 . Hence the url is http://10.0.1.23:5544/ . Once you go there, you will be presented with an interface as shown in the figure below. You can then select an application and iNalyzer will prepare a zip file and download it on your system for analysis.

![7]({{site.baseurl}}/images/posts/ios15/7.png)

However, i had some problems while performing this. Hence, we will also be looking at an alternative solution to do the same step. To do that, first of all make sure iNalyzer is running. Then navigate inside the iNalyzer directory and run _iNalyzer5_ without any arguments.

![8]({{site.baseurl}}/images/posts/ios15/8.png)

You will see all the list of apps available for analysis. In this case, let's select the Defcon App for analysis.

![9]({{site.baseurl}}/images/posts/ios15/9.png)

You will see that iNalyzer begins its work. It decrypts the app, finds out the class information and other things. As you can see from the figure below, once iNalyzer has finished its job, it will create an ipa file and store it at the location as highlighted in the image below.

![10]({{site.baseurl}}/images/posts/ios15/10.png)

So now we need to get this ipa file and download it on our system. We can do that via sftp.

![11]({{site.baseurl}}/images/posts/ios15/11.png)

Once we have the ipa file, change its extension to zip. Then unzip the file.

![12]({{site.baseurl}}/images/posts/ios15/12.png)

Now with a terminal, navigate inside the folder Payload-->Doxygen.

![13]({{site.baseurl}}/images/posts/ios15/13.png)

You will see a shell script named doxMe.sh. If you look inside it, you will notice that it automates the task of running Doxygen for us. Doxygen also runs Graphviz for generating graphs and the results are stored inside a folder with the name _html_. Basically, iNalyzer has already stored all the class information for us inside a folder named _Reversing Files_ and it uses Doxygen and Graphviz to display the information in a much more presentable format.This shell script also opens up the _index.html_ file inside the created _html_ folder.

![14]({{site.baseurl}}/images/posts/ios15/14.png)

So lets run this shell script and let iNalyzer do all the things for us.

![15]({{site.baseurl}}/images/posts/ios15/15.png) ![16]({{site.baseurl}}/images/posts/ios15/16.png)

Once this is done, iNalyzer will automatically open up the index.html file stored inside the html folder that was created. Here is what it looks like. In this case, i am using chrome. However, the developer of this tool personally recommended me to use firefox browser for runtime analysis as the other browsers may be buggy. As you can see from the image below, the first page gives a strings analysis of the entire app. It divides the strings into SQL and URL strings.

![17]({{site.baseurl}}/images/posts/ios15/17.png)

You can also have a look at all the view controller classes used in the app.

![18]({{site.baseurl}}/images/posts/ios15/18.png)

Tapping on any of the view controllers will show you its methods and properties.

![19]({{site.baseurl}}/images/posts/ios15/19.png)

You can also look at the contents of the Info.plist file.

![20]({{site.baseurl}}/images/posts/ios15/20.png)

If you go under the Classes Tab and under _Class Index_ you will see a list of all the classes being used in the app. Some of them are Apple's own classes while some are created by the developer of this app.

![21]({{site.baseurl}}/images/posts/ios15/21.png)

If you go under the _Class Hierarchy_ tab, you will see the class information and relationships being represented in a graphical format. This gives us a fair amount of knowledge on how this application works. These graphs are generated by the Graphviz tool.

![22]({{site.baseurl}}/images/posts/ios15/22.png)

If you go to the files tab, you can have a look at all the interface files that iNalyzer generated.

![23]({{site.baseurl}}/images/posts/ios15/23.png) **Conclusion**

In this article, we looked at static analysis of iOS applications using iNalyzer and how easy it makes our job. In the next article, we will look at how we can use iNalyzer further for runtime analysis of iOS applications.

**References**

*   iNalyzer  
    [https://appsec-labs.com/iNalyzer](https://appsec-labs.com/iNalyzer)