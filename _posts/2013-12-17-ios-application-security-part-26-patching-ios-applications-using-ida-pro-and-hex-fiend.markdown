---
layout: post
title: "iOS Application Security Part 26 – Patching iOS Applications using IDA Pro and Hex Fiend"
date: 2013-12-17 13:08
comments: true
categories: security
---

In the [previous](http://highaltitudehacks.com/security) applications we have looked at how we can hijack method implementations during runtime using Cycript, and even change the logic of the code rather than changing the complete implementation using GDB. All of these things have been done to serve a purpose, which is to make the application do what we want. However, using Cycript or GDB is a bit of a pain as one has to do repeat the same process everytime after you restart the application. This is where patching the application is useful. Once a change has been made in the application's binary, its permanent. So you don't have to repeat the same process over and over again. Once the binary is patched, you can then run it on a jailbroken device with the changed logic.

<!-- more -->

In this article, we will be using the same application [GDB-Demo](https://github.com/prateek147/gdb-demo) that we had used in [Part 22](http://resources.infosecinstitute.com/ios-application-security-part-22-runtime-analysis-manipulation-using-gdb/) of this series. If you remember, we had found a way to change the logic of the method that gets called when Login was tapped and hence bypassed the login authentication check. In this article, we are going to permanently patch this check so we are always authenticated.

The first thing you need to do is install the demo version of IDA Pro from their [website](https://hex-rays.com/products/ida/index.shtml). IDA Pro is a pretty awesome multi-processor disassembler and debugger. Once it is downloaded, open it up and choose the option _Go_ which just opens up IDA without any preselected binary. Please note that when you run an application on the simulator using Xcode, the code is compiled for the i386 architecture, whereas when you run the application on a device using Xcode, it is compiled for the ARM architecture. The demo version of IDA Pro supports both these architectures, however in this tutorial we are going to compile the application on i386 architecture (i.e on a simulator) to save the effort of copying the application binary from the device to our computer.

![]({{site.baseurl}}/images/posts/ios26/1.png)

Now open Xcode and run the [GDB-Demo](https://github.com/prateek147/gdb-demo) application that you had just downloaded using simulator. ake sure the application builds successfully and that it installs propery on the simulator. This will generate an application directory inside the folder _/Users/$username/Library/Application Support/iPhone Simulator/$ios version of simulator/Applications/_. In my case, the location is _/Users/Prateek/Library/Application Support/iPhone Simulator 6.1/Applications/_. Once you are in this directory, you have to find your application folder. Using the command _ls -al_ will give you the last modified date of these folders. The latest one would be our application.

![]({{site.baseurl}}/images/posts/ios26/2.png)

Use the command _open DirectoryName_ and this will open the directory in Finder.

![]({{site.baseurl}}/images/posts/ios26/3.png)

Go inside the folder GDB-Demo.app (this is the application bundle) by right clicking on it and choosing the option _Show Package contents_. Inside this folder, you will find the application binary with the name GDB-Demo. This is the binary that we will provide to IDA Pro.

![]({{site.baseurl}}/images/posts/ios26/4.png)

Now drag and drop the application binary on the IDA pro icon. Click on Ok and proceed.

You will see the disassembled code like this.

![]({{site.baseurl}}/images/posts/ios26/6.png)

Coming back to the application, we know that the application has a login page like the one shown below. We had already modified the logic of this application using GDB in [Part 22](http://resources.infosecinstitute.com/ios-application-security-part-22-runtime-analysis-manipulation-using-gdb/). We also know that the method whose logic was changed was _-(IBAction)loginButtonTapped:(id)sender_

![]({{site.baseurl}}/images/posts/ios26/7.png)

In IDA Pro, you can see a functions window on the left. Choose any function from the list of functions and press _Ctrl+F_. Now search for the function loginButtonTapped. Once you have found it, double click on it. You will be shown the dissassembly for this function on the right side.

![]({{site.baseurl}}/images/posts/ios26/8.png)

To view this in sort of a graphical format, double click on the function name in the functions window and press _Space_. The view will change to something like this. This is a better way of examining the disassembly as it also helps us in understanding the flow of the function. If you want to switch to the previous view, you can press _Space_ again.

![]({{site.baseurl}}/images/posts/ios26/9.png)

Scroll down in the function disassembly on the right side. You can see that the flow of the function can lead to different blocks of code depending on specific conditions. It is obvious that somewhere within this function there is a check for whether the username and password are correct or not and authenticate or disallow the user. After scrolling down for a bit, we arrive at this block. This looks very interesting. On the right side, i can see _UIAlertView_ in the code section whereas the left section shows a string named _adminPage_.

![]({{site.baseurl}}/images/posts/ios26/10.png)

I would really like the flow to go to the left hand side, where it says _adminPage_. The instruction that decides which block of code the execution will jump to is just one instruction before.

![]({{site.baseurl}}/images/posts/ios26/11.png)

It says _jnz loc_2CBC_, where _loc_2CBC_ is a label and jnz stands for _Jump if not zero_ instruction. We can see that the code block on the left contains this label.

![]({{site.baseurl}}/images/posts/ios26/12.png)

This means execution will jump to left if the zero flag is not set. If i can modify the instruction _jnz_ to _jz_, then my purpose would be solved as the logic would be reversed and i will be authenticated. So what do i need to convert this from jnz to jz.

MORE MONEY !

Well, that was a bit of humour. A licensed version of IDA Pro will get the job done for you and you can simply modify this instruction. However, we are going to do this the free way, even though it requires a bit of extra effort and calculation but its worth it and we will also learn a few new things on the way. In the next article, we will also discuss an alternative named Hopper that is not as expensive as IDA but very good in terms of functionality. For that first we need to find the address of the _jnz_ instruction. To do that, double click on this instruction so that it gets yellow...

![]({{site.baseurl}}/images/posts/ios26/13.png)

Now press Space..

![]({{site.baseurl}}/images/posts/ios26/14.png)

As we can clearly see, the address of this instruction is _00002CB1_. However, we cannot just go ahead and change the address at this instruction, This is because this address is the absolute address of this instruction and it will be different every time the application is launched. What we need to find out is the offset of this instruction relative to the Mach-O binary. This instruction has to be modified in the code section of the assembly. Hence the offset of this instruction relative to the binary can be calculated as ..

**(Offset of code section relative to binary) + (Absolute address of the instruction to be changed - Starting address of the code section)**

Right now we just know the Absolute address of the instruction to be changed. We can find the other two things using otool. Browse to the application directory _/Users/Prateek/Library/Application Support/iPhone Simulator/6.1/Applications/1804F89F-AD44-4782-BB29-47F5C521D10D/GDB-Demo.app_ and use the following command as shown in the image below.

![]({{site.baseurl}}/images/posts/ios26/15.png)

Look for the text section.

![]({{site.baseurl}}/images/posts/ios26/16.png)

As you can see, the starting address is 0x000026f0(Hex) and the offset is 5872(Decimal). Please note that these things may be different in your case.

Hence, using these values and the above equation we can find the offset as ..

**5872(Decimal) + (0x00002CB1(Hex)- 0x000026f0(Hex)) = 0x1cb1**

Now, as we discussed earlier, we need to replace the jnz instruction with a jz instruction. You can see from [this](http://www.unixwiz.net/techtips/x86-jumps.html) link that the opcode for the JNZ instruction is _OF 85_ whereas the opcode for the JZ instruction is _OF 84_.

Now download the application _Hex Fiend_ and open it up. Drag and drop the application binary to it.

![]({{site.baseurl}}/images/posts/ios26/17.png)

Now click on Edit --> Jump to Offset and type the offset as 0x1cb1\. This will take you to the line with the jnz instruction.

![]({{site.baseurl}}/images/posts/ios26/18.png)

Now look for the opcode OF 85\. Change it to 0F 84 as shown below.

![]({{site.baseurl}}/images/posts/ios26/19.png)

Now save your changes and exit Hexfiend. As you remember, we had installed the app previously in the simulator. So fire up the iOS simulator, quit the GDB-Demo app if it is running and open it again. Now just tap on Login without entering anything in the username and password. It will direct you to the admin page.

![]({{site.baseurl}}/images/posts/ios26/20.png)

Perfect, we just patched a binary using old school techniques. In the next article, we will look at a tool named Hopper and learn how to patch iOS applications using it.