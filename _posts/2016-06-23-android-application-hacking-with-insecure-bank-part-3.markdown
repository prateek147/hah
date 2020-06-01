---
layout: post
title: "Android Application hacking with Insecure Bank Part 3"
date: 2015-03-28 00:17
comments: true
categories: security
---

In this article, we will look at attacking components in Android applications, starting with activities. But first, it is essential to understand what Android application components are. Android application components are the essential building blocks of an Android application. The behaviour and interaction between these components is defined in the manifest.xml file in the application. Mainly there are 4 types of components and it is important to understand their purpose and function.

<!-- more -->

The description of the 4 components given below has been taken from <a href="http://www.tutorialspoint.com/android/android_application_components.htm">this</a> link.

<table>

<tbody>

<tr>

<th style="width:25%">Components</th>

<th>Description</th>

</tr>

<tr>

<td>Activities</td>

<td>They dictate the UI and handle the user interaction to the smartphone screen</td>

</tr>

<tr>

<td>Services</td>

<td>They handle background processing associated with an application.</td>

</tr>

<tr>

<td>Broadcast Receivers</td>

<td>They handle communication between Android OS and applications.</td>

</tr>

<tr>

<td>Content Providers</td>

<td>They handle data and database management issues.</td>

</tr>

</tbody>

</table>

#### Activities

An activity represents a single screen with a user interface. For example, an email application might have one activity that shows a list of new emails, another activity to compose an email, and another activity for reading emails. If an application has more than one activity, then one of them should be marked as the activity that is presented when the application is launched. An activity is implemented as a subclass of Activity class as follows:

<pre>public class MainActivity extends Activity {

}</pre>

#### Services

A service is a component that runs in the background to perform long-running operations. For example, a service might play music in the background while the user is in a different application, or it might fetch data over the network without blocking user interaction with an activity. A service is implemented as a subclass of Service class as follows:

<pre>public class MyService extends Service {

}</pre>

#### Broadcast Receivers

Broadcast Receivers simply respond to broadcast messages from other applications or from the system. For example, applications can also initiate broadcasts to let other applications know that some data has been downloaded to the device and is available for them to use, so this is broadcast receiver who will intercept this communication and will initiate appropriate action. A broadcast receiver is implemented as a subclass of BroadcastReceiver class and each message is broadcasted as an Intent object.

<pre>public class MyReceiver  extends  BroadcastReceiver {

}</pre>

#### Content Providers

A content provider component supplies data from one application to others on request. Such requests are handled by the methods of the ContentResolver class. The data may be stored in the file system, the database or somewhere else entirely. A content provider is implemented as a subclass of ContentProvider class and must implement a standard set of APIs that enable other applications to perform transactions.

<pre>public class MyContentProvider extends  ContentProvider {

}</pre>

#### Additional Components

There are additional components which will be used in the construction of above mentioned entities, their logic, and wiring between them. These components are:

<table class="src">

<tbody>

<tr>

<th style="width:25%">Components</th>

<th>Description</th>

</tr>

<tr>

<td>Fragments</td>

<td>Represents a behavior or a portion of user interface in an Activity.</td>

</tr>

<tr>

<td>Views</td>

<td>UI elements that are drawn onscreen including buttons, lists forms etc.</td>

</tr>

<tr>

<td>Layouts</td>

<td>View hierarchies that control screen format and appearance of the views.</td>

</tr>

<tr>

<td>Intents</td>

<td>Messages wiring components together.</td>

</tr>

<tr>

<td>Resources</td>

<td>External elements, such as strings, constants and drawables pictures.</td>

</tr>

<tr>

<td>Manifest</td>

<td>Configuration file for the application.</td>

</tr>

</tbody>

</table>

#### One of the most important attributes of components is the exported property. Here is the documentation from android about it.

#### android:exported

Whether or not the activity can be launched by components of other applications — "true" if it can be, and "false" if not. If "false", the activity can be launched only by components of the same application or applications with the same user ID.The default value depends on whether the activity contains intent filters. The absence of any filters means that the activity can be invoked only by specifying its exact class name. This implies that the activity is intended only for application-internal use (since others would not know the class name). So in this case, the default value is "false". On the other hand, the presence of at least one filter implies that the activity is intended for external use, so the default value is "true".This attribute is not the only way to limit an activity's exposure to other applications. You can also use a permission to limit the external entities that can invoke the activity (see the permission attribute).

Hence, if an activity is exported, it can be called by external applications. In order to test the vulnerable activity exercise in InsecureBank application, let's first start the application on Genymotion emulator and start the backend server as well.

![1]({{site.baseurl}}/images/posts/ib3/1.png)

Once we start the app, we are presented with this login page. If the activity after a successful login is exported, then we can call that activity directly.

![2]({{site.baseurl}}/images/posts/ib3/2.png)

Let's have a look at the manifest file of the application to see if we can find the relevant activity. To look at the manifest file, first decompress the application using apktool as shown in the image below. This will create an application folder and the manifest file will be located inside it.

![4]({{site.baseurl}}/images/posts/ib3/4.png)

Here is how the manifest file looks like. As you can see, there is an activity named _.PostLogin_ which is set as exported.

![3]({{site.baseurl}}/images/posts/ib3/3.png)

We can call this activity directly using the activity manager tool in the emulator. Let's have a look at the usage first.

![5]({{site.baseurl}}/images/posts/ib3/5.png)

To start an activity with the am tool, here is the command.

<pre>adb shell
am start -n com.package.name/com.package.name.ActivityName
</pre>

In this case, the package name is com.android.insecurebankv2 as can be seen from the manifest file.

So let's call the PostLogin activity by using the command shown below.

![6]({{site.baseurl}}/images/posts/ib3/6.png)

In the application, you can see that you have successfully bypassed the login page.

![7]({{site.baseurl}}/images/posts/ib3/7.png)

You can also call activity or other components using drozer. We will look at drozer in the next article.

There are a couple of ways to prevent from these kinds of vulnerabilities. First of all, the android:exported property should always be set to FALSE unless really necessary. Secondly, if the application needs to be called from some specific external applications, you can add custom permissions to the activity and only allow applications that requests that permission to call the activity.

Before we move on to the next article, make sure you have a proper understanding of the [Android manifest](http://developer.android.com/guide/topics/manifest/manifest-intro.html) file and the different components in an andorid application.