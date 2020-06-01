---
layout: post
title: "iOS Application Security Part 48 - Frida APIs"
date: 2018-07-24 05:18
comments: true
categories: security
---

In the previous article, we had a basic introduction to Frida. In this article, we will look at some of the APIs that Frida provides to automate a lot of this stuff.

Frida provides APIs in Javascript, Swift and C to interact with apps. This can be used to perform injection, runtime manipulation, reading the memory etc. It also has an API in python but it is very high level and restricted at the moment. It still has been used to create many useful scripts that are invaluable for iOS app security assessments. The most powerful API at the moment is the javascript API. So let's have a look at it. We will be performing the analysis on Damn Vulnerable iOS App.

<!--more-->

Let's hook into the applicaiton using the frida CLI. The Frida CLI tries to imitate a lot of the same functionalities of Cycript. Since these APIs are javascript, all these can be used to create a javascript script which can be then loaded into any application with just manipulating the application specific parameters.

![1]({{site.baseurl}}/images/posts/ios48/1.png)

The full Javascript API can be found [here](https://www.frida.re/docs/javascript-api/). Let's have a look at some APIs. One of the most important APIs is the ObjC.available. This tells Frida whether the Objective-C runtime is loaded into the application. If it is available, you can use the ObjC.classes command to list the classes.

![2]({{site.baseurl}}/images/posts/ios48/2.png)

This technique will also work for Swift apps, because Swift apps also run inside the Objective-C runtime. This output is for the Mail app written in Swift running on iOS 11.3.1.

![3]({{site.baseurl}}/images/posts/ios48/3.png)

You can hence iterate through all the classes using the following script, where the hasOwnProperty method makes sure that the object is actually a class.

<pre>	function listClasses()
	{
	if (ObjC.available)
	                 {
	                     for (var className in ObjC.classes)
	                     {
	                         if (ObjC.classes.hasOwnProperty(className))
	                         {
	                             console.log(className);
	                         }
	                     }
	                 }
	                 else
	                 {
	                     console.log("Objective-C Runtime is not available!");
	                 }
	}
</pre>

Or you can use this one-liner Object.keys(ObjC.classes).forEach(function (className) { ... });

![5]({{site.baseurl}}/images/posts/ios48/5.png)

The Process command can also be used to fetch a lot of valuable information about the running application. The last option, code signing policy can be used to tell Frida whether or not to run unsigned code. In case where Frida is used as a shared library on non-jailbroken devices (when compiling from source), this setting will be set to required.

![4]({{site.baseurl}}/images/posts/ios48/4.png)

You can also print out the methods for a specific class with the following command.

![6]({{site.baseurl}}/images/posts/ios48/6.png)

This can be then converted into a function and used whenever required.

![7]({{site.baseurl}}/images/posts/ios48/7.png)

One of the other most important APIs is Interceptor. This allows you to intercept calls to a particular method, log the arguments, modify the implementation etc. It is even possible to modify the arguments being sent to the methods.

The API looks like Interceptor.attach(target, callbacks), where the target is the hook to the method and the callbacks determine the event within the function which could be when the method is called (onEnter: function (args)), in which case you can modify the arguments or log them or onLeave (onLeave: function (retval)), in which case you can modify the return value.

Let's try and print out the method arguments for a particular method when it is called. We know that the method +[RNDecryptor decryptData:withPassword:error:] is being called from Hopper.

![8]({{site.baseurl}}/images/posts/ios48/8.png)

Let's hook into this method +[RNDecryptor decryptData:withPassword:error:] in DVIA and print out the encryption key when it is called.

<pre>	var classname = "RNDecryptor";
	var methodname= "+ decryptData:withPassword:error:";
	var hook = ObjC.classes[classname][methodname];
	Interceptor.attach(hook.implementation, {
	  onEnter: function(args) {
	    // args[0] is self
	    // args[1] is selector 
	    // args[2] is the Data
	    // args[3] is the Password, or in this case the encryption key
	    // args[4] is the error
	    var enckey = ObjC.Object(args[3]);
	    console.log("\nEncryptionKey:"
	        + enckey.toString() + "\"]");
	  }
	});
</pre>

![9]({{site.baseurl}}/images/posts/ios48/9.png)

The following code will hook into the method "isJailbroken:" in DVIA and print out the return value when the function has finished executing.

<pre>	var classname = "JailbreakDetectionVC";
	var methodname= "- isJailbroken";
	var hook = ObjC.classes[classname][methodname];
	Interceptor.attach(hook.implementation, {
		onLeave: function(retvalue) {
	   	 console.log("ClassName: " + classname);
	     console.log("MethodName: " + methodname);
	     console.log("\tReturn Type: " + typeof retvalue);
	     console.log("\tReturn Value: " + retvalue);
	     } 
	});

</pre>

![10]({{site.baseurl}}/images/posts/ios48/10.png)

In the next article, we will look at how we can manipulate the return values of these functions.

**References**

1.  http://www.mopsled.com/2015/log-ios-method-arguments-with-frida/
2.  https://github.com/interference-security/frida-scripts/
3.  https://github.com/dweinstein/awesome-frida