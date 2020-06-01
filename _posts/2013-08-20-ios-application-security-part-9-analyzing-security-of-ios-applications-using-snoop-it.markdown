---
layout: post
title: "iOS Application Security Part 9 – Analyzing Security of iOS Applications using Snoop-it"
date: 2013-08-20 07:29
comments: true
categories: security
---

In some of the previous articles, we have looked at how we can dump class information of iOS apps using class-dump-z, hook into the runtime using Cycript and perform runtime manipulation and method swizzling, analyze the flow of the app using gdb etc. However, there could be a much better way of doing these things. We shouldn't be using seperate tools for all these tasks. It would be great if a tool could perform all these tasks and at the same time display the information in a much more presentable way.

<!-- more -->

Snoop-it is a tool that solves these problems. It allows for runtime analysis and blackbox security assessment of iOS apps by retrofitting existing apps with debugging and runtime tracing capabilities. It also provides a very neat web interface. At the time of writing of this article, Snoop-it is not released yet but is a couple of weeks away from launch. I mailed the authors and they were nice enough to provide me with a beta version for testing. You can check out its official page [here](https://code.google.com/p/snoop-it/) or you can follow the author on [Twitter](http://twitter.com/aykay/)

.

A quick list of all the features provided by Snoop-it could be seen in the screenshot below taken from its official [page](https://code.google.com/p/snoop-it/). ![1]({{site.baseurl}}/images/posts/ios9/1.png)

## Installation

To install Snoop-it on your device, you will have to download the deb package file and then upload it on your device using sftp. Once this is done, use the command _dpkg -i [packageName]_ to install Snoop-it on your device. Once this is done, respring or reboot your device.

![2]({{site.baseurl}}/images/posts/ios9/2.png)

Once this is done, you will see the Snoop-it app icon on your device. Open it up and you will see this user interface.

![IMG 0109]({{site.baseurl}}/images/posts/ios9/IMG_0109.PNG)

Go to Settings and configure the app according to your need. In this case, i have chosen the port number to be 12345 and i have also disabled the authentication. It might be a good idea however to have the authentication enabled if you are testing on a network with lots of users, or a network with a few naughty users.

![IMG 0110]({{site.baseurl}}/images/posts/ios9/IMG_0110.PNG)

Now, just open the Snoop-it web interface by browsing to the address provided on the Snoop-it application. In my case, the address is http://10.0.1.79:12345

![3]({{site.baseurl}}/images/posts/ios9/3.png)

You will see this web interface. If you read it up, its asking you to _select an application that needs to be analyzed from the Snoop-it application, open it up on the device, and then refresh this web interface_. So let's go back to the Snoop-it application and select the applications that we need to analyze. In my case, i am going to select the _MethodSwizzlingDemo_ app, the same app that we used in the [previous](http://resources.infosecinstitute.com/ios-application-security-part-8-method-swizzling-using-cycript/) article.

![IMG 0111]({{site.baseurl}}/images/posts/ios9/IMG_0111.PNG)

Now, make sure that the app is opened on your device and in foreground and now refresh the Snoop-it web interface.

![4]({{site.baseurl}}/images/posts/ios9/4.png)

And now as you can see, you have a beautiful interface now that you can use to perform a full fledged security assessment of the application.

## Analysis

On the left hand side, under _Analysis_, go to _Objective-C classes_. On the right hand side, you will see all the classes and info like properties and method names.

![5]({{site.baseurl}}/images/posts/ios9/5.png)

The ones in orange represent the classes that have instances. For e.g if you hover your mouse over the class _View Controller_, you will see that it has an instance which is live presently.

![6]({{site.baseurl}}/images/posts/ios9/6.png)

Similarly, you can see the methods and properties for AppDelegate.

![7]({{site.baseurl}}/images/posts/ios9/7.png)

Coming back to view controller, it is possible for us to invoke a method using Snoop-it. Just check any particular method, and click on _Setup and Invoke_ on the top right. As we saw in the [previous](http://resources.infosecinstitute.com/ios-application-security-part-8-method-swizzling-using-cycript/) article, with this technique we were able to bypass the authentication check for this application.

![8]({{site.baseurl}}/images/posts/ios9/8.png)

Select the instace (there is only one now, but there could be multiple instances if the view controller is being reused across the application), and click on _Invoke Method_.

![X]({{site.baseurl}}/images/posts/ios9/x.png)

This will invoke the method and will bypass the authentication.

![IMG 0112]({{site.baseurl}}/images/posts/ios9/IMG_0112.PNG)

Another awesome feature of Snoop-it is that we can switch to any View controller. For e.g, on the extreme left hand side, under _Analysis_, select _View Controller_, select the _View Controller_ class on the right hand side and click on _Display Controller_. You will be switched to that view controller. You can also click on _Close/Hide View Controller_ depending on whether the view controller is over another view controller or not.

![9]({{site.baseurl}}/images/posts/ios9/9.png)

You can then tap on _Reset display_ to come back. As you can understand, this feature will really help us relate the view controller to its view in the app. So if i have a view controller in the Classes section, i can use this feature to see its visual representation. I just love this feature of Snoop-it.

## Runtime Manipulation

Snoop-it also allows for many ways of runtime manipulation, including changing your hardware identifier attributes like Mac address, UDID, device model number etc.

![10]({{site.baseurl}}/images/posts/ios9/10.png)

You can also spoof your location. This could be particular useful for apps that use GeoEncryption techniques to protect their data.

![11]({{site.baseurl}}/images/posts/ios9/11.png)

And, you can trace methods and system calls on the flow. Please note that you will have to click on _Refresh_ on the top to see the method calls being made after every few seconds. Also, FYI since we are testing on a beta release it is possible that the authors may change it so that we don't have to click on _Refresh_ after every few seconds. This information might be a bit too much for some users, but if you have been developing iOS applications for a couple of years like me, then this information should be pretty much straightforward.

![12]({{site.baseurl}}/images/posts/ios9/12.png)

## Monitoring

Snoop-it also allows you to look at the various files and directories that are being accessed by the application. To do that, on the navigation menu on the left side, click on _Filesystem_ under _Monitoring_.This feature can be particular useful when a particular application is writing to a database file and this interface helps you in figuring out that filename. You can also download these files just by double clicking on them and then analyze it on your machine.

![13]({{site.baseurl}}/images/posts/ios9/13.png)

You can also see all the access made by the application using sensitive API. This could include looking for info on the Address book, accessing the camera, or just finding the UDID for the device. Here is the sensitive API accessed by the _App Store_ application which comes preinstalled on all iOS devices.

![Y]({{site.baseurl}}/images/posts/ios9/y.png)

We can also see all the information stored in the keychain by this application. Also, it is possible to see a list of all HTTP requests sent using NSURLConnection. Both of these features can be accessed under the navigation menu under _Monitoring_. I leave it up to you to try these features out. Also, we will be discussing how to dump information from keychain in a seperate article.

You will be happy to know that Snoop-it has a public API that we can make use of in order to automate tests or just to build our own graphical user interface. A documentation of the XML-RPC web service API can be found [here](http://code.google.com/p/snoop-it/wiki/ )

## Conclusion

In this article, we looked at how we can we can use Snoop-it to perform runtime analysis and black box security assessment of iOS Applications and how easy it makes our task. Snoop-it is still a few weeks away from release at the time of writing this article, though you can always mail the author for a beta version like i did. One of the new features that i would like added in Snoop-it is the ability to perform Method Swizzling.I am pretty sure it will be an awesome tool for anyone interested in performing security analysis of iOS Applications and it's only going to get better :)