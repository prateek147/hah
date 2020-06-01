---
layout: post 
title: "Wifite Walkthrough part 2: Cracking WPA access points"
date: 2014-10-27 01:43
comments: true
categories: security
---

In this article, we will look at cracking access points using WPA-PSK or WPA2-PSK using Wifite.

If you have used tools like airodump-ng, aircrack-ng etc to crack WPA access points before, you would know that the required thing to successfully crack a WPA-PSK network is a captured WPA four-way handshake. More details about the WPA four-way handshake can be found on [this](http://en.wikipedia.org/wiki/IEEE_802.11i-2004) wikipedia page.

<!-- more -->

As mentioned in the previous article, there is a bug in Wifite that may or may not be there in your particular version of Wifite. The bug basically doesn't aireplay-ng to function properly and displays an error like _aireplay-ng exited unexpectedly_ . In order to fix this, you will have to make slight modifications in the code of wifite. You can install gedit (apt-get install gedit) which is a text editor and then edit the wifite python script (found in /usr/bin/wifite) using the steps mentioned [here](https://code.google.com/p/wifite/issues/detail?id=127). To open wifite, use the command _gedit /usr/bin/wifite_. This will open up the source code of wifite. Then replace every occurence of _cmd = ['aireplay-ng',_ with _cmd = ['aireplay-ng','--ignore-negative-one',_

To start wifite for cracking a WPA access point, give it the option _-wpa_ to only target WPA networks. Also, give it a dictionary file as an input for cracking the WPA passphrase with the _-dict_ option. In kali linux, the wordlists are stored at the location _/usr/share/wordlists_. Wifite will now start scanning for WPA access points.

![1]({{site.baseurl}}/images/posts/wifite2/1.png)

Press Ctrl+C to give a target number. In my case, the target number is 2 which is an access point i have configured for testing purposes. The access point uses WPA2-PSK encryption with the key as "password".

![2]({{site.baseurl}}/images/posts/wifite2/2.png)

Wifite will now start listening for the handshake. Once it has found it, it will automatically start cracking the passphrase using the dictionary file that we supplied.

![4]({{site.baseurl}}/images/posts/wifite2/4.png)

And as you can see, Wifite has successfully found the passphrase for the access point.

Sometimes, things may not work as smoothly. In order to capture a WPA handshake between the client and the access point, the client has to connect to the wireless network during that period when we are monitoring the network. If the client is already connected, there will be no handshake that is captured. Wifite does this by automatically sending deauthentication packets to a particular client or a broadcast deauthentication packet if it is required. You can specify the time between deauthentication packets using the -wpadt flag. Hence, when the client tries to reconnect to the access point, the handshake is captured.

You can also specify which tool you want to use to crack the passphrase once the four-way handshake has been successfully captured. By default, aircrack-ng is selected. You can also use cowpatty, pyrit or tshark to crack the passphrase.

![5]({{site.baseurl}}/images/posts/wifite2/5.png)

Another cool option in Wifite is to anonymize your MAC address using the _-mac_ option. Even though it is quite trivial using simple commands or _macchanger_ utility to change the MAC address for a specific interface, it is good to have this feature in the tool itself. However, in order to make this work, you first have to take that specific interface for which you want to change the MAC address down to managed mode if it is in monitor mode previously. You can use the command _iwconfig_ to check all the interfaces that are in monitor mode and then take them down using the command _airmon-ng stop interface-name_ command. As we can see from the image below, the mon0 interface is in monitor mode.

![6]({{site.baseurl}}/images/posts/wifite2/6.png)

Hence, lets take it down using the command _airmon-ng stop mon0_

. ![7]({{site.baseurl}}/images/posts/wifite2/7.png)

Now we can add the _-mac_ option to anonymize the MAC address. As you can see, Wifite is intelligent enough to change the MAC address to something that is similar the existing MAC address of the interface and not to something ridiculous (for e.g AA:BB:CC:DD:EE:FF) which is a giveaway.

![8]({{site.baseurl}}/images/posts/wifite2/8.png)

And when you stop the capture, Wifite is nice enough to change the MAC address back to the original one.

In this article, we looked at how we can use Wifite to crack networks using WPA-PSK or WPA2-PSK. Wifite is great at its job and automates almost everything, however it is important to understand how it uses the tools like airodump-ng, aircrack-ng etc under the hood to perform its task. I would recommend that you go through the source code of Wifite and figure out how this is done.