---
layout: post
title: "iOS Application Security Part 51 - Dumping decrypted IPA and Dynamic Instrumentation on a non-jailbroken device"
date: 2018-07-27 19:48
comments: true
categories: security
---
In this article, we will look at how to dump decrypted IPA file for an application using frida and then look at how to set up Frida for dynamic instrumentation on a non-jailbroken device.

To dump an IPA, we will use an open source tool known as frida-ios-dump which can be found on https://github.com/AloneMonkey/frida-ios-dump.

The first thing is to set up port forwarding. This can be done by using iproxy. By default frida-ios-dump will connect from local port 2222 to remote port 22\. So this is what we will set up with iproxy as well.

<!--more-->

![1]({{site.baseurl}}/images/posts/ios51/1.png) 

Next, clone the repo from github.

![2]({{site.baseurl}}/images/posts/ios51/2.png)

Now navigate under the tool directory, open the file dump.py and and change the user/pass to that of your device. This will allow frida-ios-dump to connect to your device over the tunnel you just created. All of this is assuming your device is connected to the computer over USB. If its over Wifi (SSH), then the public key for the device must be added to the target device's ~/.ssh/authorized_keys file.

![3]({{site.baseurl}}/images/posts/ios51/3.png)

Now you can use the command _python dump.py AppName_ to dump the IPA file from the device.

![4]({{site.baseurl}}/images/posts/ios51/4.png)

The next thing we need to learn is to do dynamic instrumentation on a non jailbroken device. This requires us to have the source code of the application or a decrypted IPA. In this article, we will only discuss the scenario where source code is required. The way it works is that we include a dylib in the application source code. This dylib needs to be obviously signed before being deployed into the device. Since the device is not jailbroken in this case, you need to sign it with your official Apple developer certificate (and not a self signed certificate). In this case, we will be inserting the dylib into Damn Vulnerable iOS application. You can clone DVIA from [here](https://github.com/prateek147/DVIA-v2.git).

You can grab the latest release of the Frida gadget from the Frida releases page. Look for the iOS gadget. At the time of writing this article, the latest Frida version is 12.0.4 and could be downloaded from [here](https://github.com/frida/frida/releases/download/12.0.4/frida-gadget-12.0.4-ios-universal.dylib.xz). Once it is downloaded, run the following commands.

![5]({{site.baseurl}}/images/posts/ios51/5.png)

Rename this dylib file to FridaGadget.dylib. Create a directory named Frameworks and put this dylib inside there. Now open the Xcode project for the application for which you want to perform the instrumentation. Drag and drop the folder on the very top level of the directory structure (similar to App Delegate) and make sure the following options are selected.

![6]({{site.baseurl}}/images/posts/ios51/6.png) ![7]({{site.baseurl}}/images/posts/ios51/7.png)

Under Project navigator, go to the _Build Phases_ section and under the _Link Binary with Libraries_, drag and drop the FridaGadget.dylib file from the Frameworks folder. Also, make sure the _Copy Bundle Resources_ section contains the Frameworks folder.

![8]({{site.baseurl}}/images/posts/ios51/8.png)

In my case, i also had to disable the setting ENABLE Bitcode by going to Build Settings and disabling it. Since we are running the app locally this shoudn't really matter.

![9]({{site.baseurl}}/images/posts/ios51/9.png)

Now we are all set, run the app and you should a log like this in the console.

![10]({{site.baseurl}}/images/posts/ios51/10.png)

Running the command _frida -Uai_ will now show this app in the output. You can now trace the application via the normal frida commands.

![11]({{site.baseurl}}/images/posts/ios51/11.png)

Since we attached to the application in the image above, the application finished launching. This can be done for late instrumentation. In order to attach to the application during launch for early instrumentation, you need to spawn it. This can be done with the -f option in frida which is used for spawning.

![12]({{site.baseurl}}/images/posts/ios51/12.png) ![13]({{site.baseurl}}/images/posts/ios51/13.png) ![14]({{site.baseurl}}/images/posts/ios51/14.png)

One last thing, you can also attach to the application in the iOS simulator by using the -R command.

![15]({{site.baseurl}}/images/posts/ios51/15.png) **References**

1.  https://frida.re/docs/ios/#without-jailbreak