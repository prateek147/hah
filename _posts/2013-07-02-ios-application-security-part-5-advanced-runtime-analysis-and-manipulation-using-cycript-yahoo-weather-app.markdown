---
layout: post
title: "iOS Application security Part 5 – Advanced Runtime analysis and manipulation using Cycript (Yahoo Weather App)"
date: 2013-07-02 17:26
comments: true
categories: [ios, security]
---

In the previous article, we learnt how to setup Cycript on your idevice, hook into a running process and obtain information about its properties in runtime. In this article, we will look at some advanced runtime analysis techniques. We will look at how we can obtain information about a particular class (methods, instance variables) and modify them at runtime.

<!-- more -->

## Finding methods for a particular class

Let's say we are analyzing the flow of an app during its runtime. It would be really good to know what are the methods being called in a particular view controller or in a particular class. Since Cycript is a blend of Objective-C and Javascript, we can write a function that has both Objective-C and Javscript syntax. We can define functions in the interpreter and use them anytime we want to find out some particular information. A good source for finding such code snippets is available [here](http://iphonedevwiki.net/index.php/Cycript_Tricks) and we will be using most of the code snippets from here for this article.

First of all, lets make sure we are hooked into the running process.

![1]({{site.baseurl}}/images/posts/ios5/1.png) 

Let's define a method that prints out the methods for a particular class. You can find the code snippet on the Cycript tricks page [here](http://iphonedevwiki.net/index.php/Cycript_Tricks).

![2]({{site.baseurl}}/images/posts/ios5/2.png)

Now that we have the method defined, we can input any class here and get the corresponding methods for it. From the previous article, we found out that the delegate class for this app was YWAppDelegate. Hence, let's try and see all the methods contained in this class.

![3]({{site.baseurl}}/images/posts/ios5/3.png)

This gives us all the methods defined in the class YWAppDelegate. Everything after the @selector is the name of the method. Note that this will also give us information about the private methods. Also, this will also include the getters and setters for the properties defined in the class.

Similarly, we can also print out the methods of YahooSlidingViewController.

![4]({{site.baseurl}}/images/posts/ios5/4.png)

We know that YahooSlidingViewController manages the sliding meny and works as a facade over the other view controllers. In order to find out the view controller that is actually responsible for displaying weather in the app, we can use the following command.

![5]({{site.baseurl}}/images/posts/ios5/5.png)

Hence, the YWMainViewController is the view controller responsible for displaying the weather in the app. So the view shown in the screenshot below is actually the one coming from YWMainViewController.

![IMG 0094]({{site.baseurl}}/images/posts/ios5/IMG_0094.PNG)

Let's print out the methods for YWMainViewController.

![6]({{site.baseurl}}/images/posts/ios5/6.png)

As you can see, there is a method named userDidRequestUpdate.

![9y]({{site.baseurl}}/images/posts/ios5/9y.png)

From the method name, its obvious that this method gets called whenever the user pulls down on the app to refresh. With Cycript, we can call this method anytime we want. We will have to reference this view controller and then call this method on it. Here is how it's done.

![9]({{site.baseurl}}/images/posts/ios5/9.png)

And if you see in the app, the update method gets called even though we didn't actually pull down.

![IMG 0097]({{site.baseurl}}/images/posts/ios5/IMG_0097.PNG)

As told before in this article, these methods also contain the getters and setters of the properties.

From a security point of view, such power to manipulate the runtime of an application gives us a lot of advantages.We can call any method whenever we want in the app. Imagine a flow in the app where the user first logs in the app by entering the username/password and then once he is logged in, a method named _didLogin_ gets called. In our case, we can just call this method ourselves without having to enter any username/password combination.

It would be a bit helpful if we could print out all the variables used in a particular view controller. So lets define a function that prints out all the instance variables. You can find the code snippets [here](http://iphonedevwiki.net/index.php/Cycript_Tricks)

![7]({{site.baseurl}}/images/posts/ios5/7.png)

Now, lets print out the instance variables for YWMainViewController.

![8]({{site.baseurl}}/images/posts/ios5/8.png)

As you can see, there is an instance variable named location view controllers. In the Yahoo weather app, you can swipe left and right to see the weather for different locations. From its name, it looks like the variable locationViewControllers is an array of view controllers which is responsible for holding a list of location view controllers. Using Cycript, we can also print out the value of this instance variable.

![9x]({{site.baseurl}}/images/posts/ios5/9x.png)

Now let me swipe right to another location New york.

![IMG 0098]({{site.baseurl}}/images/posts/ios5/IMG_0098.PNG)

Let's print the value of this variable now.

![10]({{site.baseurl}}/images/posts/ios5/10.png)

As you can see, this array always has 3 location view controllers inside it and the others are null. It doesn't contain all the reference of location view controllers so as to manage memory better. So at a particular time, we can have the view controller that we are looking at, and the left and right view controllers being instantiated. When we move to a different location, it automatically adjusts to make the visible view controller instance the one in the center and instantiates the left and right view controllers so the user can swipe left and right and won't face any delay. This is one example of writing code that doesn't take up much memory.

### Conclusion

In the previous two articles, we have performed the runtime analysis of the Yahoo weather app. In the next article, we will be looking at some more Cycript tricks and will focus specially on a technique known as method swizzling.

**References:**

*   Cycript  
    [http://www.cycript.org/](http://www.cycript.org/)

*   Cycript tricks  
    [http://iphonedevwiki.net/index.php/Cycript_Tricks](http://iphonedevwiki.net/index.php/Cycript_Tricks)

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).