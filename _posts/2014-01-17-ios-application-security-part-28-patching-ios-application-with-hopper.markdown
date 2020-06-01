---
layout: post
title: "iOS Application Security Part 28 - Patching iOS Application with Hopper"
date: 2014-01-17 21:41
comments: true
categories: security
---

In [Part 26](http://highaltitudehacks.com/2013/12/17/ios-application-security-part-26-patching-ios-applications-using-ida-pro-and-hex-fiend) of this series, we looked at how we can use IDA Pro and Hex Fiend to patch an iOS application and modify its implementation. Patching an application has the specific advantage that once a change has been made, it is permanent. However, if you look back at the article on [IDA Pro](http://highaltitudehacks.com/2013/12/17/ios-application-security-part-26-patching-ios-applications-using-ida-pro-and-hex-fiend), you will realize that the process of patching the application was a bit tedious, mainly because we didn't have a licensed version of IDA Pro which costs a lot. In this article, we will look at a utility named Hopper which we can use as an alternative to IDA Pro. It is less costly than IDA Pro and also provides a sleek interface to work with.

<!-- more -->

According to Hopperapp.com ..

_Hopper is a reverse engineering tool for OS X, Linux and Windows, that lets you disassemble, decompile and debug (OS X only) your 32/64bits Intel Mac, Windows and iOS (ARM) executables! Take a look at the feature list below!_

And...

_Even if Hopper can disassemble any kind of Intel executable, it does not forget its main platform. Hopper is specialized in retrieving Objective-C information in the files you analyze, like selectors, strings and messages sent._

In this article, i am using a paid version of Hopper which cost about $60\. I think it is an incredible price given the things we can do with this application. I would recommend you check out the demo version which lets you perform some tasks to get a feel of Hopper. Anyways, once you download the Hopper app, this is the interface we are looking at.

![1]({{site.baseurl}}/images/posts/ios28/1.png)

In this article also, we will use the same demo application that we used in [Part 26](http://highaltitudehacks.com/2013/12/17/ios-application-security-part-26-patching-ios-applications-using-ida-pro-and-hex-fiend), the [GDB-Demo](https://github.com/prateek147/gdb-demo) application that you can download from my github profile. I highly recommend that you read Part 26 before you proceeed with this article. Just to quickly recap, the GDB-Demo had a login form like this.

![2]({{site.baseurl}}/images/posts/ios28/2.png)

It accepts a certain username/password combination in order to allow us to login. Our task is to patch this application in such a way that the application allows us to login even if the username/password combination is not correct. Please note that in this article, we will be debugging and patching the application which is x86 architecture on a laptop , however you can do the same patching with ARM executable as well by copying the binary from the device.

Once you have downloaded the [GDB-Demo](https://github.com/prateek147/gdb-demo) application, run it using Xcode. This will install the application in the iOS simulator. Now our task is to find the location of the application binary on our system. If you run an application in Xcode, it will generate an application directory inside the folder _/Users/$username/Library/Application Support/iPhone Simulator/$ios version of simulator/Applications/_. In my case, the location is _/Users/Prateek/Library/Application Support/iPhone Simulator 6.1/Applications/_. Once you are in this directory, you have to find your application folder. Using the command _ls -al_ will give you the created date of these folders. The latest one would be our application.

![4]({{site.baseurl}}/images/posts/ios28/4.png)

Use the command _open DirectoryName_ and this will open the directory in Finder.

![5]({{site.baseurl}}/images/posts/ios28/5.png)

Go inside the folder GDB-Demo.app (this is the application bundle) by right clicking on it and choosing the option _Show Package contents_. Inside this folder, you will find the application binary with the name GDB-Demo. This is the binary that we will provide to Hopper.

![6]({{site.baseurl}}/images/posts/ios28/6.png)

Now open Hopper app and go to File->Read Executable To Disassemble. Give the location of the GDB-Demo binary. Also make sure to quit Xcode but keep the simulator open.

![2]({{site.baseurl}}/images/posts/ios28/2.png)

Hopper will now start dissasembling the application.

![7]({{site.baseurl}}/images/posts/ios28/7.png)

On the left hand side, if you select the _Strings_ section, you will see all the constant strings that Hopper was able to dump from the binary. If you have read part 26 on this series, you will note that the password is also present in this list ;-).

![8]({{site.baseurl}}/images/posts/ios28/8.png)

If you select the labels section on the left, it will give you all the labels it was able to dump from the application. This includes labels to method implementations, constant strings, classes etc..

![9]({{site.baseurl}}/images/posts/ios28/9.png)

We know that the method that is important is loginButtonTapped. So lets search for it in the labels section. Once we find the method, tap on it and it will take you to its disassembly.

![10]({{site.baseurl}}/images/posts/ios28/10.png)

One kickass feature of Hopper is that it can provide Pseudo code for a function. To check out the Pseudo code for this function, click on Pseudo Code on the top right.

![11]({{site.baseurl}}/images/posts/ios28/11.png)

As you can see, Hopper provides you with a Pseudo code for this function.

![12]({{site.baseurl}}/images/posts/ios28/12.png)

This is such an amazing feature. It helps us so much in figuring out what this method is supposed to do. In this case, it just gives away the password. Another awesome feature of this application is Show CFG which helps you find the flow of the application. Just click on _Show CFG_ next to the Pseudo code button.

![13]({{site.baseurl}}/images/posts/ios28/13.png)

If we scroll down a bit in this function, we get to the point shown in the image below. We can see that the left flow takes us to a point where we see the text _Incorrect Username or Password_, whereas the right flow has some text _admin page_. Obviously, i would like the flow to go to the right side.

![14]({{site.baseurl}}/images/posts/ios28/14.png)

Now lets check the condition that decides which way the flow goes. As we can see from the image below, the assembly instruction is _jne 0xcbc_

![15]({{site.baseurl}}/images/posts/ios28/15.png)

0x2cbc is a label that corresponds to the right hand side. So if we can modify the instruction in such a way that the flow is always taken towards the right hand side, then our task will be accomplished.

![16]({{site.baseurl}}/images/posts/ios28/16.png)

Well, its pretty simple to do this. Just replace _jne 0xcbc_ by _jmp 0xcbc_. To do this, click on this specific instruction in the dissassembly and click on Modify->Assemble instruction

![17]({{site.baseurl}}/images/posts/ios28/17.png)

Then we write down the instruction that we want and click on Assemble and Go Next. That's it, this is the only change we want to do.

![18]({{site.baseurl}}/images/posts/ios28/18.png)

Now lets save this executable and overwrite the previous one. Go to File -> Produce New Executable and overwrite the original executable.

![19]({{site.baseurl}}/images/posts/ios28/19.png)

Now go the simulator application, quit any running instance of GDB-Demo application and restart the application. Tap on Login and you will see that the login has been bypassed.

![20]({{site.baseurl}}/images/posts/ios28/20.png)

Congratulations, we just patched an application using Hopper. This was just a small feature of Hopper. Hopper lets us do many more things. I would recommend you check them out and no, i am not associated with Hopper nor do i know the author personally. I just think its a cool app and for $60, its an extremely good deal !

In the next article, we will learn about Insecure or Broken Cryptography.