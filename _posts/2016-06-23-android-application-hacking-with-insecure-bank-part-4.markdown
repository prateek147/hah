---
layout: post
title: "Android Application hacking with Insecure Bank Part 4"
date: 2015-03-29 00:17
comments: true
categories: security
---

In this article, we will look at a very handy framework for analysis of android applications named Drozer. Drozer is a very useful tool as it eliminates the need for having seperate tools for performing different security checks in an android application. It has a list of modules that you can use to interact with the application using Android's Inter-Process communication. Additionally, you can also install exploits and use it to exploit an android device.

<!--more-->

The main purpose of this article is to make sure you are familiar with drozer so we can use it in the future articles.

The first thing to do is to install the drozer community edition from [this](https://www.mwrinfosecurity.com/products/drozer/) link. You need to install both the drozer installer and the Agent.apk file which is the application that needs to be deployed on the device/emulator and acts as a communicator between the system and the application to be audited.

Once drozer has been installed on your system, install the agent.apk on your device/emulator.

![1]({{site.baseurl}}/images/posts/ib4/1.png)

You will first need to set up port forwarding so that your system can connect to a TCP socket opened by the Agent inside the emulator, or on the device. By default, drozer uses port 31415:

![2]({{site.baseurl}}/images/posts/ib4/2.png)

Also make sure to start the agent application and start the server.

![3]({{site.baseurl}}/images/posts/ib4/3.png)

Now you can connect to the agent using the following command.

![4]({{site.baseurl}}/images/posts/ib4/4.png)

You can list all the different modules by using the list command.

![5]({{site.baseurl}}/images/posts/ib4/5.png)

Every module requires different options. If you want to see the different options for a particular module, use run followed by the module name followed by -h.

![6]({{site.baseurl}}/images/posts/ib4/6.png)

For e.g, to see a list of all the packages installed, you can use the module app.package.list.

![7]({{site.baseurl}}/images/posts/ib4/7.png)

Now, to find info about a particular packages, use the module app.package.info. It will give out a lot of info about the application, for e.g the path where the application files are stored, the permissions that the application uses etc.

![8]({{site.baseurl}}/images/posts/ib4/8.png)

Another useful module is app.package.attacksurface. It tells you about the exported components as well as whether the application is debuggable or not. We will look at exploiting debuggable applications in later articles.

![9]({{site.baseurl}}/images/posts/ib4/9.png)

Now, let's do the same thing we did in the last article, call an exported activity in the insecure bank application. For that, we will use the module app.activity.start.

![10]({{site.baseurl}}/images/posts/ib4/10.png)

And you will see the same result.

![11]({{site.baseurl}}/images/posts/ib4/11.png)

In some cases, the activity might have an intent filter. For e.g, below is a sample intent filter.

<activity android:name="ShareActivity"><intent-filter><action android:name="android.intent.action.SEND"><category android:name="android.intent.category.DEFAULT"><data android:mimetype="text/plain"></data></category></action></intent-filter></activity>

Drozer supports calling activities by specifying actions and extra paramters also.

![12]({{site.baseurl}}/images/posts/ib4/12.png)

Here is an example of calling an activity with extra parameter

_run app.activity.start --component com.mwr.example.intenttest com.mwr.example.intenttest.IntentActivity --flags ACTIVITY_NEW_TASK --extra string URL "Some Text"_

In this article, we got comfortable with using Drozer. Drozer can do much more, and we will be discussing all those features as we discuss more vulnerabilities in InsecureBank in the next article.