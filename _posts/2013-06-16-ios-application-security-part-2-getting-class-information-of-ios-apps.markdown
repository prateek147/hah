---
layout: post
title: "iOS Application security Part 2 - Getting class information of iOS apps"
date: 2013-06-16 02:56
comments: true
categories: security
---

Have you ever checked out an iOS app and thought it was cool, and wondered if you could find some information about the source code of the app, the third-party libraries it uses, or how the code is designed internally ? Have you ever wondered if it was possible to dump all the images, plist files used in any app either preinstalled on your device or downloaded from the App store? If the answer is Yes, then you have come to the right place.

<!-- more -->

In this article, we will look at how we can analyze any preinstalled app on your device or any other app downloaded from App store and discover things about the source code of the app like the classes that it uses, the names of the view controllers it uses, the internal libraries, and even intricate details like the variables and methods names used in any particular class or view controller. We will then look at how we can decrypt the applications downloaded from the App store and dump all the images, plist files that the app uses.

## Dumping class information for Preinstalled apps on the device

Now we are at a stage that we can analyze apps for class information. So let's dump the class information for the Apple _Maps_ app. The first step would be to locate the Apple _Maps_ app executable. All iOS apps that come preinstalled with the device are stored in the directory _/Applications_. So let's navigate to that directory.

![20]({{site.baseurl}}/images/posts/ios2/20.png)

Here you will see all the apps that come preinstalled with the device. Now let's navigate inside the _Maps_ app directory and list the directories.

![21]({{site.baseurl}}/images/posts/ios2/21.png)

As you can see, we can see all the images, plist files etc used by this app. We will discuss later how it is possible to fetch all the images and other files from a particular iOS app. Anyways, hidden in all this mess is an executable for the app with the name _Maps_ as can be seen on the left side in the image below. Note that the name of the executable will be the same as the name of the app. Note that we can see some pdf's in the app bundle as well. I really don't see the need of including a pdf file in the bundle.

![22]({{site.baseurl}}/images/posts/ios2/22.png)

To dump the class information for this app, just use the command _class-dump-z Maps_

![23]({{site.baseurl}}/images/posts/ios2/23.png)

As you can see there is just too much output in the terminal right now, hence its better to save the output to a file, in this case with the filename _class-dump-Maps_.

![24]({{site.baseurl}}/images/posts/ios2/24.png)

You can now use _sftp_ to ftp into the device and download the file. You can fetch any file with the command _get_ followed by the path of the file as shown below.

![25]({{site.baseurl}}/images/posts/ios2/25.png)

Since the file is now downloaded locally on the system, let's open it up in _TextMate_ (you can use textedit or any other app as well)

![26]({{site.baseurl}}/images/posts/ios2/26.png)

We can learn a lot about the way the code is designed just by looking at the interface files. For e.g over here you can see a View controller named _InfoCardController_. As you might have already guessed, this is the VC to display more info about a particular location when we tap on the right arrow button as shown in the image below.

![27]({{site.baseurl}}/images/posts/ios2/27.PNG)

Now lets have a look at this view in the app. This page is actually displayed by _InfoCardViewController_ which we found from _class-dump-z_ information.

![28]({{site.baseurl}}/images/posts/ios2/28.PNG)

If you look at this image and the class information above, you can easily see what are the methods names that get called when you tap on these buttons. For e.g if i tap on _Direction to here_, the method that will get called is

_-(void)_directionsTo:(id)to person:(void*)person property:(int)property identifier:(int)identifier;_

Similarly, if i tap on _Add to Bookmarks_, the method that will get called is

_-(void)_addToBookmarks:(id)bookmarks person:(void*)person property:(int)property identifier:(int)identifier;_

You can find a lot of other information from the app as well, for e.g here is a class named _UserLocationSearchResults_ which inherits from SearchResult.

![28x]({{site.baseurl}}/images/posts/ios2/28x.png)

You can download the class information for the Apple _Maps_ app from here.

How much you can explore here is only up to your curiosity :).

## Dumping class information for apps downloaded from the App store

Their are two important things to know if you want perform analysis of the apps that you download from the App store.

*   The apps are stored in a different location, _/var/mobile/Applications/_
*   Unlike the apps that come preinstalled with the device, the apps are _encrypted_, hence you will have to _decrypt_ them first.

To decrypt the apps, we will be using a command line tool called _Clutch_. Please note that _Clutch_ was being offered by _Hackulous_ which has been shut down a few months back. But the binary for _Clutch_ is still available on the internet.

Now you need to upload the binary onto your device. To do that, we are going to use _sftp_. To upload a file onto the device, just use the _put_ command.

![29]({{site.baseurl}}/images/posts/ios2/29.png)

Now, ssh into your device and type _clutch_. This will give you a list of all the apps that could be _cracked_.

![30]({{site.baseurl}}/images/posts/ios2/30.png)

To crack a particular app, just type _clutch app-name_ For e.g if we want to crack the Facebook app, we will type _clutch Facebook_

![31]({{site.baseurl}}/images/posts/ios2/31.png)

Once it is done cracking, it will tell you the location where it has saved the _ipa_ file. Now an _ipa_ file is just a compressed version of the whole app bundle. To unzip it, just use the _unzip_ command and save it to a directory by using the _-d_ command as shown in the figure below. Note that you can also copy this _ipa_ file on your system using _sftp_ and then unzip it over there. You will then have access to all the images of the app as well as any other files that may be present in the unzipped folder.

![32]({{site.baseurl}}/images/posts/ios2/32.png)

Now that we have the decrypted file, we can use _class-dump-z_ to dump the class information for it and save it in a file which in this case is named _class-info-Facebook_.

![33]({{site.baseurl}}/images/posts/ios2/33.png)

Once this is done, you can exit the ssh session, log in via _sftp_ and then download the _class-info-Facebook_ file.

![33x]({{site.baseurl}}/images/posts/ios2/33x.png)

You can now check out this file using any text viewer. For e.g here is a protocol named _FBFacebookRequestSender_ which has methods for sending asynchronous requests as well as a method to check if the Facebook Session is valid or not.

![34]({{site.baseurl}}/images/posts/ios2/34.png)

## Fetching images and other files from a particular app.

As discussed previously in the article, one of the methods would be to use _sftp_ to fetch all the files that you want from that app's directory. However, there are much easier ways to do this, one of which is to use [iExplorer](http://www.macroplant.com/iexplorer/download-ie3-mac.php). Download it from the official website. Once this is done, just open it up and make sure your device is connected to the system via USB.

![35]({{site.baseurl}}/images/posts/ios2/35.png)

To view the filesystem, just click on _files_.

![36]({{site.baseurl}}/images/posts/ios2/36.png)

To check out files for a particular app, click on _Apps_

![37]({{site.baseurl}}/images/posts/ios2/37.png)

As you can see, it is very easy to browse the filesystem and upload/download files. In this case, lets download all the image and files present in the _Facebook_ app. On the left side, look for _Facebook_ and click on it. This will take you to the directory containing Facebook app files. All the images and files are containing inside the _Facebook.app_ directory.

![38]({{site.baseurl}}/images/posts/ios2/38.png)

To download all the files, just press _Cmd + A_, and right click and select _Export to Folder_. Then choose the location where you want to save all the files.

## Conclusion

In the first two parts of this article, we have learnt how to setup a mobile auditing environment on a jailbroken device. We then learnt how to dump the class information for any particular app and use it to understand the design of the code and its internal workings. We also learnt how to decrypt an app downloaded from the App store and audit it for information. We then learnt how to un-munge images from apps using both sftp and iExplorer.

Well, the good thing is that it is possible to know all the methods that get called by using the class information that we get from class-dump-z. _But is it possible to perform some runtime modification in the app ?_ For e.g if a method like -(BOOL)isFacebookSessionValid returns false in a particular case, is it possible for us to manipulate the app in such a way that it returns YES and hence let the application do unexpected things ? _Further, is it possible to create our own custom method and execute it instead of this method whenever this method gets called ?_ Is it possible to modify the values of instance variables during runtime, or after any specific instruction ?The answer is [YES](http://cycript.org), and we will learn about it in the next article :).

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).