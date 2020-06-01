---
layout: post
title: "iOS Application Security Part 47 - Inspecting Apps with Frida"
date: 2018-07-23 15:48
comments: true
categories: security
---

In this article, we will talk about Frida. Frida is a dynamic instumentation toolkit which can prove to be extremely useful in iOS application assessments. It can be used to assess apps on jailbroken and non-jailbroken devices (provided you have the source code) .We will look at all these examples in this and the coming few articles.

Let's start first with assessment over jailbroken devices. Frida basically works on a client-server model. The client is running on your computer and the server on the iOS device. To install frida on your computer, simple issue the following command.

<!--more-->

![1]({{site.baseurl}}/images/posts/ios47/1.png) ![2]({{site.baseurl}}/images/posts/ios47/2.png)

On your jailbroken device, add the source https://build.frida.re. Then go to search and search for Frida.

![3]({{site.baseurl}}/images/posts/ios47/3.PNG) ![4]({{site.baseurl}}/images/posts/ios47/4.PNG)

One of the important things is to make sure both the Frida versions on the iOS device and the computer are same. Otherwise, frida won't work. Now ssh into your jailbroken device and you will see a process with the name frida-server which is running.

![5]({{site.baseurl}}/images/posts/ios47/5.png)

From your computer, simply issue the command _frida-ps -U_. Make sure the jailbroken device is connected to your computer over USB. If you get similar output, it means Frida is all set up and running.

![6]({{site.baseurl}}/images/posts/ios47/6.png)

Actually, frida comes with a bunch of command line tools as can be seen here.

![7]({{site.baseurl}}/images/posts/ios47/7.png)

Issuing the command frida-ps will just show you a list of the processes running on your computer. But we are interested in i-devices. Running the command frida-ls-devices will show all the devices connected to the computer. We can interface with this device with frida-ps -U command. If there are more than one devices, you will need to specify the UDID.

![8]({{site.baseurl}}/images/posts/ios47/8.png)

Ok, so we are all set. Let's now look at what we can do with Frida. In this case, we will be using the application Damn Vulnerable iOS app which you can download from [damnvulnerableiosapp.com](http://damnvulnerableiosapp.com). You might face issues with installing the older Objective C version of the app on iOS 10 or later versions because the original app was signed with SHA-1 but since iOS 10 Apple is using SHA-256 hashing algorithm. So you might need to unzip the IPA file, strip out the 64 bit architecture using jtool (or lipo) and then sign it. You can then repackage the app and install it again to the device using Cydia Impactor.

![9]({{site.baseurl}}/images/posts/ios47/9.png)

To see a list of all the applications installed on the device along with their unique identifier, run the _frida-ps -Uai_ command.

![10]({{site.baseurl}}/images/posts/ios47/10.png)

To see all the running apps, use the _frida-ps -Ua_ command.

![12]({{site.baseurl}}/images/posts/ios47/12.png)

To attach to a specific process, run the _frida -U processname_ command. Make sure that the application is running in foreground on the device. The frida CLI can be used to emulate a lot of the features of Cycript.

![11]({{site.baseurl}}/images/posts/ios47/11.png)

However, in this article, we will look at the frida-trace CLI. This CLI can help you trace various method calls during the application runtime. This can be extremely useful in understanding the inner workings of the application. Its always a good idea to look at the help output as there are many options here.

![20]({{site.baseurl}}/images/posts/ios47/20.png)

To trace a particular function, you can specify it with the -i option. You can also provide a regex in the method section as shown in the figure below. For e.g, in the example below, you can trace the Crypto calls.

![16]({{site.baseurl}}/images/posts/ios47/16.png) ![19]({{site.baseurl}}/images/posts/ios47/19.png)

If you use the -I option, you can determine which module from Frida do you want to include, which is essentially the dylib from where all functions will be retrieved.

![17]({{site.baseurl}}/images/posts/ios47/17.png) ![18]({{site.baseurl}}/images/posts/ios47/18.png)

In case of DVIA (Objective-C version), you can go to the Broken Cryptography section and start the challenge. To trace a particular Objective C method, you can specify it with the -m option. If the method calls match the provided regex, you will see the output as shown in the image below.

![14]({{site.baseurl}}/images/posts/ios47/14.png) ![15]({{site.baseurl}}/images/posts/ios47/15.png)

This is extremely useful in understanding the inner working of the application and the same info can be used to patch or bypass certain methods. In the next article, we will look at more examples of using Frida.