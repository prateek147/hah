---
layout: post
title: "iOS Application Security Part 52 - Inspecting iOS apps with Objection"
date: 2018-07-28 10:48
comments: true
categories: security 
---
In the previous few articles, we have looked at how we can use Frida to perform dynamic instrumentation of applications. In this article, we will look at a tool based using Frida's capabilities, known as <a hred="https://github.com/sensepost/objection">objection</a>, which can be very useful in testing iOS applications on non-jailbroken devices. The only thing that is required is an unencrypted IPA (insert Frida Gadget using [insert_dylb](https://github.com/Tyilo/insert_dylib)) or the source code. Since in the previous article we already looked at how we can add a Frida dylib into the source code and do instrumentation, we will carry forward from there in this article. We will be using [Damn Vulnerable iOS App](http://damnvulnerableiosapp.com) for this article.

<!--more-->

Here are the various features of Objection as mentioned on their Github [page](https://github.com/sensepost/objection).

![1]({{site.baseurl}}/images/posts/ios52/1.png) 

The first thing is to install Objection on the computer which can be installed very easily with _pip3 install objection_. In some cases, you might be better off setting up a virtual environemnt for python3\. It is recommended to go through the installation instructions mentioned on their Github page.

![2]({{site.baseurl}}/images/posts/ios52/2.png)

Once done, run the command _objection_ to see if it was successfully installed.

![3]({{site.baseurl}}/images/posts/ios52/3.png)

Make sure you have an application that has FridaGadget.dylib injected into it. Start the application on the device and it will pause as its waiting for a frida client to attach to it. Now from your computer, run the _objecion device_type_ command to do a quick test.

![4]({{site.baseurl}}/images/posts/ios52/4.png)

Now run the command _objection explore -q_ to attach to the application. Keep in mind that this is not early instrumentation since you are attaching to the application after is is being launched. For early instrumentation you can just use Frida with the spawn command.

![5]({{site.baseurl}}/images/posts/ios52/5.png)

You can just press TAB on your computer to see all the list of available commands. One of the most useful features of objection is the autocompletion feature so we don't have to remember all these commands.

![14]({{site.baseurl}}/images/posts/ios52/14.png)

For any extra tasks not performed by objection, you can just load the corresponding fridascript with the import command.

![15]({{site.baseurl}}/images/posts/ios52/15.png)

Ok, now we can use objection to do various tasks. It is important to note that whatever is happening here is happening within the context of the application with all the sandbox restrictions still being employed in place. Also, all the ios specific commands start with _ios_. Also, any command you want to run on your computer from within the objection interpreter must have ! prepended to it.

Use _pwd print_ to print out the current working directory.

![6]({{site.baseurl}}/images/posts/ios52/6.png)

A simple _ls_ command will dump the contents from the current working directory within the application context.

![7]({{site.baseurl}}/images/posts/ios52/7.png)

Let's run _env_ and this will give us all the folders related to the application. We are mostly interested in the application data here which is mostly present in the Documents folder.

![8]({{site.baseurl}}/images/posts/ios52/8.png)

Now let's head over to the Documents directory and run the ls command there.

![9]({{site.baseurl}}/images/posts/ios52/9.png)

The data stored in the info.plist file can be dumped with _ios plist cat filename_ command.

![10]({{site.baseurl}}/images/posts/ios52/10.png)

Optionally, another way of achieving the same would be to download the file to your computer with the _file download filename_ command and then use the OS command cat (prepended with an !) to list the contents of the file.

![11]({{site.baseurl}}/images/posts/ios52/11.png)

You can see all the data stored using the NSUserDefaults or UserDefaults (in new SDKs) using the _ios nsuserdefaults get_ command.

![12]({{site.baseurl}}/images/posts/ios52/12.png)

And you can use the _ios keychain dump_ to dump the keychain.

![13]({{site.baseurl}}/images/posts/ios52/13.png)

In the next article, we will continue looking at some of the other useful functionalities of Objection.

**References**

1.  https://github.com/sensepost/objection