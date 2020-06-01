---
layout: post
title: "iOS Application Security Part 45 - Enhancements in Damn Vulnerable iOS app version 1.5"
date: 2015-05-31 00:18
comments: true
categories: security
---

In this article, i would like to give a quick walkthrough of the new vulnerabilities and challenges that we have added in version 1.5 of [Damn Vulnerable iOS app](http://damnvulnerableiosapp.com).

In the Insecure Data storage section, we have added challenges for the following databases.

*   Realm Database
*   Couchbase Lite
*   YapDatabase

<!--more-->

![1]({{site.baseurl}}/images/posts/ios45/1.png) ![2]({{site.baseurl}}/images/posts/ios45/2.png) ![3]({{site.baseurl}}/images/posts/ios45/3.png)

We have also added a new section on _Extension vulnerabilities_, which covers vulnerabilities in different application extensions, a feature that was introduced with iOS 8.

![4]({{site.baseurl}}/images/posts/ios45/4.png) ![5]({{site.baseurl}}/images/posts/ios45/5.png)

In the _Runtime Manipulation section_, we have added a challenge where you can write a cycript script to brute force a login screen.

![6]({{site.baseurl}}/images/posts/ios45/6.png)

Another new section is _Attacks on third party libraries_, which demonstrates the security gaps that can occur in your application when you use third party libraries in your project.

![7]({{site.baseurl}}/images/posts/ios45/7.png) ![8]({{site.baseurl}}/images/posts/ios45/8.png) ![9]({{site.baseurl}}/images/posts/ios45/9.png) ![10]({{site.baseurl}}/images/posts/ios45/10.png) ![11]({{site.baseurl}}/images/posts/ios45/11.png) ![12]({{site.baseurl}}/images/posts/ios45/12.png)

In the section on _Side Channel Data leakage_, we have added another vulnerability demonstrating insecure storage of cookies.

![13]({{site.baseurl}}/images/posts/ios45/13.png)

The current downloadable IPA file from the website is a fat binary that will work on both 32 bit and 64 bit devices. This app will work on all iOS versions starting from iOS 7.0.

Some important links

1.  [Official Website](http://damnvulnerableiosapp.com)
2.  [Github Page](http://github.com/prateek147/DVIA)
3.  [Downloads Page](http://damnvulnerableiosapp.com#downloads)

We are working on getting the new solutions out as soon as possible so please be patient. For previous vulnerabilities, you can download the solutions for free from [here](http://damnvulnerableiosapp.com#solutions).

For any bugs, suggestions etc, please don't hesitate to contact me. Also, a very special thanks to [Egor](http://twitter.com/igrekde) for his contributions to the project.