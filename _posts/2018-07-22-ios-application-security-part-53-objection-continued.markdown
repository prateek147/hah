---
layout: post
title: "iOS Application Security Part 53 - Objection continued"
date: 2018-07-29 17:48
comments: true
categories: security 

---

In this article, we will continue looking into Objection and some of the use cases it provides.

One of the most useful features objection provides is the ability to bypass jailbreak detection. This might not be always effective since it's only looking for certain checks that an application will do to detect a jailbroken device and hooks them to return a false value. But any application can deploy a check not looked into by objection and the jailbreak detection bypass will fail. Neverthless, this feature might be useful in many cases where the apps are doing basic checks only.

<!--more-->
![1a]({{site.baseurl}}/images/posts/ios53/1a.png)

It is also important to note that an application can just use some native C code to detect all the injected dylibs into the application and simply exit the app without any warning if it finds a dylib with the name FridaGadget. This has been observed in some apps that i have tested. A simple bypass would be to just change the name of the Frida dylib file to something else.

You can also simulate a jailbroken environment to understand how an application behaves in a jailbroken environment.

![2a]({{site.baseurl}}/images/posts/ios53/2a.png)

Objection uses jobs to list all the tasks that it is performing in the background. You can use the command _jobs list_ to list the tasks and _jobs kill UDID_ to kill them.

![3a]({{site.baseurl}}/images/posts/ios53/ 3a.png)

The hooking module is one of the most useful modules as it allows you to list the classes, methods, trace all the function calls and even dump the args or modify the return value.

Use the command _ios hooking list classes_ to list all the classes.

![4a]({{site.baseurl}}/images/posts/ios53/4a.png)

Use the command _ios hooking list class_methods classname_ to list all the methods for a particular class.

![5a]({{site.baseurl}}/images/posts/ios53/5a.png)

Use the command _ios hooking watch classname_ to trace all the methods for a particular class.

![6a]({{site.baseurl}}/images/posts/ios53/6a.png)

We can now look for certain methods and dump the arguments (--dump-args) and return value (--dump-return). And the --dump-backtrace command will give you a list of the previous methods being called.

![12a]({{site.baseurl}}/images/posts/ios53/12a.png)

Let's try and solve the Jailbreak Detection challenge in Damn Vulnerable iOS App.

![8a]({{site.baseurl}}/images/posts/ios53/8a.png)

And you can also set return values of methods. In this case, it is used to bypass Jailbreak Detection in Damn Vulnerable iOS App.

![7a]({{site.baseurl}}/images/posts/ios53/7a.png)

You can also enable Touch ID Bypass which as discussed in previous articles can be bypassed by hooking into the method -[LAContext evaluatePolicy:localizedReason:reply:]. This hooking technique will only work in some cases as discussed in a previous article on Touch ID bypass in this series. On devices with FaceID do some incorrect attmepts and then click on Enter passcode which will trigger the bypass.

![11a]({{site.baseurl}}/images/posts/ios53/11a.png)

Use the command _ios pasteboard monitor_ to monitor the contents of the Pasteboard.

![9a]({{site.baseurl}}/images/posts/ios53/9a.png)

Use the command _ios sslpinning disable_ to disable sslpinning. Ofcourse this method is not foolproof and tries to hook into some low level methods that are called while doing SSL pinning validation.

![10a]({{site.baseurl}}/images/posts/ios53/10a.png)

And finally, you can use the _ios ui_ module to take a screenshot or just dump the view hierarchy.

![13a]({{site.baseurl}}/images/posts/ios53/13a.png)

In this article, we had a good look at some of the advanced functionalities that objection provides. In the next article, we will look at another essential framework named Needle to help with iOS security assessments.

**References**

1.  https://github.com/sensepost/objection