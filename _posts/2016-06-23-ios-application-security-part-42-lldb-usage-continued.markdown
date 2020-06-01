---
layout: post
title: "iOS Application Security Part 42 - LLDB Usage continued"
date: 2015-05-12 00:17
comments: true
categories: security
---

In this article, we will look at some of the most important commands in LLDB to debug applications.

If you have been following this blog series, you would have noticed that we have been using GDB until now for debugging applications, but the support for GDB has been disabled by Apple. Apple has compiled a very useful list of GDB to LLDB commands to get you up to date with debugging via LLDB that can be found [here](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/lldb-command-examples.html).

<!--more-->

We will look at some of the most important commands after hooking into an application. In this case, lets start debugging the Twitter app. So make sure that the Twitter app is running in the foreground on the device and start a listener for the Twitter app.

![1]({{site.baseurl}}/images/posts/ios42/1.png) 

On your system, do the usual process of connecting to the debugserver application on the device to perform remote debugging. You can also use usbmuxd if you feel that debugging over Wifi is slow.

![2]({{site.baseurl}}/images/posts/ios42/2.png)

Once the connection has been established, you can now run debbugger commands to analyze the application. Let's print out the AppDelegate object.

![3]({{site.baseurl}}/images/posts/ios42/3.png)

We can read registers using the _register read_ command. To read all the registers, use the _register read --all_ command.

![4]({{site.baseurl}}/images/posts/ios42/4.png)

Note the difference in the registers ? It's because of the new arm64 architecture for this application. Another important command is _image list_ which will let you identify the location of the main executable and all the shared libraries.

![5]({{site.baseurl}}/images/posts/ios42/5.png)

The _image dump sections_ command will dump all the sections of the main executable and the shared libraries. You can later use this to dump information from the memory.

![6]({{site.baseurl}}/images/posts/ios42/6.png)

Setting a breakpoint is very similar to GDB. First, see if there are any breakpoints set using the _br l_. Then set a breakpoint for the objc_msgSend function using the _b objc_msgSend_ command. Then, resume the application using the _process continue_ command.

![7]({{site.baseurl}}/images/posts/ios42/7.png) ![8]({{site.baseurl}}/images/posts/ios42/8.png)

Once a breakpoint is hit, you can use the command _di -f_ to see the disassembled code.

![9]({{site.baseurl}}/images/posts/ios42/9.png)

We can also configure LLDB to execute a command once every breakpoint is hit. This could be very handy in tracing method calls in the application. To do that, use the command _target stop-hook add_ and enter the commands that you want to enter once the breakpoint is hit. In this case, i have asked LLDB to print out all the registers and continue the program execution.

![10]({{site.baseurl}}/images/posts/ios42/10.png)

If you don't understand the purpose of these registers right now, don't worry. I will cover arm64 architecture in a later article.

It is usually a good idea to strip the debug symbols from the application binary before submitting to the App store. You can do this by going to Build Settings and set the option _Strip Debug Symbols During Copy_ to Yes.

![11]({{site.baseurl}}/images/posts/ios42/11.png)

Hope you enjoyed this article. We have just scratched the surface of LLDB right now. In the next article, we will look at importing symbols from binaries and settings breakpoints on application specific methods.