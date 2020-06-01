---
layout: post
title: "iOS Application Security Part 49 - Runtime Patching with Frida"
date: 2018-07-24 17:42
comments: true
categories: security
---

In the previous article, we looked at Frida APIs and some examples of how to hook into methods, log the arguments, find the return value etc. In this article, we will look at how we can use Frida to do runtime patching of the application. Specifically, we will solve the following 2 challenges in DVIA-v2\. You can download the Swift version of the app from damnvulnerableiosapp.com. An app can still have some sections in Objective-C which can be swizzled and this is what we will be taking advantage of in this article.

1.  Jailbreak Detection Bypass (Jailbreak Detection -> Jailbreak Test 2)
2.  Login Bypass (Runtime Manipulation -> Login Method 1)

<!--more-->

Let’s attach to the application first.

![1]({{site.baseurl}}/images/posts/ios49/1.png) 

A quick reverse and subsequent trace of the application shows the method that is of interest. Since this method is Objective-C it can be swizzled.

![2]({{site.baseurl}}/images/posts/ios49/2.png)

Let’s use the following code to log the return value.

<pre>	var classname = "JailbreakDetection";
	var methodname= "isJailbroken";
	var hook = ObjC.classes[classname][methodname];
	Interceptor.attach(hook.implementation, {
	  onLeave: function(retvalue) {
	    // args[0] is self
	    // args[1] is selector 
	    // args[2] is the return value
	    console.log("\nReturnValue:"
	        + retvalue + "\"]");
	  }
	});
</pre>

![3]({{site.baseurl}}/images/posts/ios49/3.png)

As we can see, the return value is 1, which means the device is detected as jailbroken. Our task is to bypass this check. So we need to make the return value as 1.

![4]({{site.baseurl}}/images/posts/ios49/4.PNG)

We can do so with the following script. And let's also throw an if condition to check if the Objective C runtime is available.

<pre>		if (ObjC.available){
		var classname = "JailbreakDetection";
		var methodname= "isJailbroken";
		var hook = ObjC.classes[classname][methodname];
		Interceptor.attach(hook.implementation, {
		  onLeave: function(retvalue) {
		    // args[0] is self
		    // args[1] is selector 
		    // args[2] is the return value
			newretvalue = ptr("0x0");
			retvalue.replace(newretvalue);
		    console.log("\nNewReturnValue:"
		        + retvalue + "\"]");
		  }
		});
		}
</pre>

And as we can see, now the device is detected as Not Jailbroken.

![6]({{site.baseurl}}/images/posts/ios49/6.PNG) ![7]({{site.baseurl}}/images/posts/ios49/7.png)

Let's use the exact same technique to bypass the challenge for Login Bypass (Runtime Manipulation -> Login Method 1)

<pre>	
	if (ObjC.available){
	var classname = "LoginValidate";
	var methodname= "isLoginValidated";
	var hook = ObjC.classes[classname][methodname];
	Interceptor.attach(hook.implementation, {
	  onLeave: function(retvalue) {
	    // args[0] is self
	    // args[1] is selector 
	    // args[2] is the return value
		newretvalue = ptr("0x1");
		retvalue.replace(newretvalue);
	    console.log("\nNewReturnValue:"
	        + retvalue + "\"]");
	  }
	});
	}

</pre>

![8]({{site.baseurl}}/images/posts/ios49/8.PNG)  
![9]({{site.baseurl}}/images/posts/ios49/9.png)

In the next article, we will look at some more examples of Frida.