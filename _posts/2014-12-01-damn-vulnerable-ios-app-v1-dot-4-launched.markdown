---
layout: post
title: "Damn Vulnerable iOS App v1.4 launched"
date: 2014-12-01 18:07
comments: true
categories: security
---

I am so excited to release the latest version of [Damn Vulnerable iOS app for iOS 8.](http://damnvulnerableiosapp.com) Up till now, DVIA has been downloaded more than 75000 times and i can't wait for the count to reach 6 digits :-) Following vulnerabilities and challenges have been added in the latest version.

1.  Sensitive information in memory
2.  Webkit Caching (Insecure data storage)
3.  Certificate pinning bypass

<!-- more -->

You can download the latest version from [here](http://damnvulnerableiosapp.com/#downloads). The source code is available on the project's github page [here](https://github.com/prateek147/DVIA).

### Manual Installation

The easiest way is to install the application from Cydia. Add the source repo.kylelevin.com and search for DamnVulnerableiOSApp. ![3]({{site.baseurl}}/images/posts/dvia/3.png)  You can directly download the deb file also on your device and use dpkg -i DamnVulnerableiOSApp.deb to install the application followed by the command _uicache_ ![4]({{site.baseurl}}/images/posts/dvia/4.png) Or you can download the .ipa file from the [downloads](http://damnvulnerableiosapp.com/#downloads) page, change its name from DamnVulnerableiOSApp.ipa to DamnVulnerableIOSApp.zip and unzip this file. This will unzip to a folder named Payload. Inside it, there will be a file named DamnVulnerableIOSApp.app. Then copy the .app file to the /Applications directory on the device using Scp. You can also use sftp or the utility iExplorer to upload this application. ![1]({{site.baseurl}}/images/posts/dvia/1.png) Now login as the mobile user, use the command su to get root privileges and give the DVIA binary executable permissions. Then use the exit command to go back as the mobile user, and use the command uicache to install the application. If this doesn’t work, you can reboot the device or try this method again. ![2]({{site.baseurl}}/images/posts/dvia/2.png) To compile the application, you should follow the instructions mentioned [here](http://damnvulnerableiosapp.com/2013/12/get-started/). Any commits to the source code on Github or suggestions to improve the app are welcome. Special thanks to [@crylico](http://twitter.com/crylico) to help test the application before release and hosting the application on his repo. Happy hacking ! -Prateek